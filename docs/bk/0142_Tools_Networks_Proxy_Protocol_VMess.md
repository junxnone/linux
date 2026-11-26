-----

| Title     | Tools Networks Proxy Protocol VMess                  |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:10:38Z`                               |
| Updated @ | `2026-11-26T10:10:38Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/142) |

-----

# VMess

VMess 是 V2Ray 原创的**加密传输协议**，通常作为 V2Ray 客户端和服务器之间的桥梁。它基于
TCP，**无状态**（客户端和服务器之间不需要握手，每次请求自带认证信息），用用户 UUID
做身份凭据，并且请求和响应使用不对称的格式。总览见
[Proxy](0080_Tools_Networks_Proxy)，工具本身见
[V2Ray](0066_Tools_Networks_Proxy_V2Ray)。

## 工作方式

下图展示一次 VMess 请求的结构以及服务端如何校验和转发：

``` mermaid
flowchart LR
    C["客户端"] -->|"EAuID + ALength + Nonce + AHeader + 数据"| S["VMess 服务端"]
    S --> V{"用 UUID 派生的密钥<br/>解密并校验认证 ID"}
    V -->|"失败"| X["断开连接"]
    V -->|"成功"| H["读出指令: 目标地址 / 端口 / 加密方式"]
    H --> T["连接目标并转发数据"]
    T -->|"响应头 + 数据 (密钥由请求密钥派生)"| C
```

图中：

  - **请求**由三部分组成：认证信息、指令部分、数据部分；AEAD 认证模式下依次是 `EAuID`（加密认证
    ID）、`ALength`（加密的指令长度）、`Nonce`（随机数）、`AHeader`（加密的指令）和数据。
  - **服务端校验**：用该用户 UUID 派生出的密钥（`CmdKey`）去解密认证 ID，认证 ID
    里含时间戳，用来判断请求是否合法、是否过期；验证失败则断开。
  - **指令部分**里包含：协议版本、本次请求使用的加密 IV 和 Key、响应认证值、数据加密方式、命令（TCP 或
    UDP）、目标地址（IPv4、域名、IPv6）和端口等。
  - **响应**的 Key 和 IV 由请求的 Key 和 IV 经 SHA-256 取前 16
    字节派生，响应头里带回请求中的"响应认证值"，客户端据此确认响应来自合法服务端。
  - 关键结论：VMess 没有单独的握手往返，请求一发出去就同时完成了认证和指令传递，所以延迟低，但也因此对**时间同步**敏感。

## 认证方式

VMess 协议头有两种认证方式：

| 方式      | 说明                                     | 状态  |
| ------- | -------------------------------------- | --- |
| AEAD 认证 | 用 AEAD 加密保护协议头，保证头部的完整性                | 推荐  |
| MD5 认证  | 用 MD5 + AES-128-CFB 加密协议头，不能保证头部自身的完整性 | 已弃用 |

表中第一列是认证方式，第二列是做法，第三列是当前状态。目前服务端可以根据协议头自动判断是 AEAD 还是 MD5；v4.28.1 起，客户端把
`alterId` 设为 `0` 就代表启用 VMessAEAD，服务端自动兼容两种客户端，v4.35.0 起还可以关闭对 MD5 认证的兼容。

## 数据加密

数据部分可选的加密方式（配置里的 `security`）：

| `security`          | 说明                                                            |
| ------------------- | ------------------------------------------------------------- |
| `aes-128-gcm`       | 推荐用于有 AES 硬件加速的 PC/服务器                                        |
| `chacha20-poly1305` | 推荐用于手机等没有 AES 指令的设备                                           |
| `auto`              | 默认；在 AMD64、ARM64、s390x 上选 aes-128-gcm，其他平台选 chacha20-poly1305 |
| `none`              | 不加密，但仍计算校验；在部分平台可能比有硬件加速的 aes-128-gcm 更慢                      |
| `zero`              | 不加密也不校验，理论上最快（v4.35.0+）                                       |

表中第一列是取值，第二列是含义和建议。数据被切成小块，每块最大 $2^{14}$ 字节，GCM 模式下每块带 16
字节认证标签。**不论选哪种方式，协议头始终受到加密和认证保护。** 不要在没有
TLS、没有强制校验证书的情况下使用 `none` 或 `zero`；经 CDN 等会解密 TLS 的中转时也不建议使用。

## 配置示例

客户端出站和服务端入站（字段含义见 [V2Ray](0066_Tools_Networks_Proxy_V2Ray)）：

``` json
{
  "outbounds": [{
    "protocol": "vmess",
    "settings": {
      "vnext": [{
        "address": "server.example.com",
        "port": 443,
        "users": [{ "id": "27848739-7e62-4138-9fd3-098a63964b6b", "alterId": 0, "security": "auto" }]
      }]
    }
  }]
}
```

``` json
{
  "inbounds": [{
    "port": 443,
    "protocol": "vmess",
    "settings": {
      "clients": [{ "id": "27848739-7e62-4138-9fd3-098a63964b6b" }]
    }
  }]
}
```

其中：

  - `id`：用户 UUID，必须是合法的 UUID，客户端与服务端必须一致，可用 `uuidgen` 生成；
  - `alterId`：设为 `0` 启用 AEAD；
  - 只要用 VMess，**系统时间必须同步**：客户端和服务端的 UTC 时间误差要在 90 秒以内（与时区无关），Linux 上可以装
    `ntp`/`chrony` 自动同步。

## 优缺点

| 优点                          | 缺点                                     |
| --------------------------- | -------------------------------------- |
| 无需握手，延迟低；协议头加密并有完整性保护（AEAD） | 对系统时间敏感，时间偏差大会连接失败                     |
| 自带加密，可以不依赖 TLS 单独使用         | 自带加密与 TLS 叠加会产生"双重加密"，浪费 CPU           |
| 支持多种数据加密方式和 UDP             | 协议与 V2Ray 实现耦合较紧，历史包袱多（MD5 认证、alterId） |
| 多种传输层可选（TCP、WebSocket 等）    | 后续演进主要在 VLESS 上进行                      |

表中左边是优点，右边是缺点。VLESS 就是针对其中"耦合紧、双重加密"提出的更轻量的替代，见 VLESS 文章。

## 使用须知

  - 只在符合所在网络管理规定和当地法律法规的前提下使用。
  - UUID 等同于账号密码，不要提交到公开仓库。

## Reference

  - [VMess 协议 - V2Fly
    开发者文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/developer/protocols/vmess.md)
  - [VMess - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/config/protocols/vmess.md)
  - [V2Fly](https://www.v2fly.org/)
