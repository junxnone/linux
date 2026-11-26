-----

| Title     | Tools Networks Proxy Hysteria                        |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:25:50Z`                               |
| Updated @ | `2026-11-26T10:25:50Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/178) |

-----

# Hysteria

Hysteria（现在通常指 **Hysteria 2**，协议版本 2.0.0 起）是一个基于 **QUIC** 的 TCP/UDP
代理，设计目标是**速度快、安全、抗识别**。它有两个很有特点的设计：一是整个协议**伪装成标准的 HTTP/3
服务**——没有凭据的第三方（中间人或主动探测者）看到的就是一个普通的 HTTP/3
网站；二是允许在客户端**指定带宽**，并以此调整发送速率，在丢包、不稳定的网络上也能跑满带宽。总览见
[Proxy](0080_Tools_Networks_Proxy)，QUIC 传输的基础见 QUIC 文章。

## 工作方式

下图展示 Hysteria 2 从认证到转发的过程：

``` mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务端 (表现为 HTTP/3 站点)
    C->>S: QUIC 握手 (TLS 1.3)
    C->>S: HTTP/3 请求 POST /auth (Hysteria-Auth, Hysteria-CC-RX)
    S-->>C: 状态 233 HyOK (Hysteria-UDP, Hysteria-CC-RX)
    Note over C,S: 认证通过后，该 QUIC 连接成为代理连接
    C->>S: 每个 TCP 连接: 新建 QUIC 双向流 + TCPRequest(目标地址)
    S-->>C: TCPResponse(OK) 然后双向转发
    C->>S: UDP 包: 用 QUIC 不可靠数据报封装 UDPMessage
```

图中：

  - **QUIC 握手**：先建立标准的 QUIC 连接（内置 TLS 1.3），服务端必须实现一个标准 HTTP/3 服务器；
  - **认证请求**：真正的客户端发送一个特殊的 HTTP/3 请求 `POST /auth`，头里带凭据
    `Hysteria-Auth`、客户端最大接收速率 `Hysteria-CC-RX` 和随机填充
    `Hysteria-Padding`；
  - **认证成功**：服务端返回状态码 **233**（`HyOK`），并告知是否支持
    UDP、服务端的接收速率；其他状态码一律视为认证失败；认证失败或普通浏览器访问时，服务端表现得像普通
    Web 服务器；
  - **TCP 转发**：每个 TCP 连接用一个新的 QUIC **双向流**，先发 `TCPRequest`（含目标地址
    `host:port`，ID 为 `0x401`），服务端回复 `TCPResponse`（状态 `0x00` 成功），之后双向转发；
  - **UDP 转发**：UDP 包封装为 `UDPMessage`，通过 QUIC 的**不可靠数据报**（Unreliable
    Datagram Extension）发送，字段含 Session ID、Packet ID、分片
    ID/数量、地址和负载，超过数据报大小的包需要分片；
  - 关键结论：认证和代理都发生在一条标准的 QUIC/HTTP/3 连接里，没有凭据的访问者看到的只是 Web 服务。

## 拥塞控制与带宽设置

Hysteria 的一个独有特性是**在客户端设置发送/接收速率**：

  - 认证时客户端通过 `Hysteria-CC-RX`
    告诉服务端自己的最大接收速率（字节/秒），服务端据此决定向客户端发送的速度；服务端也通过响应里的同名头告知客户端；
  - 特殊值：客户端发 `0` 表示不知道自己的速率，服务端必须使用拥塞控制算法（如 BBR）自行调整；服务端回 `0`
    表示无限制，客户端可以随意发送；服务端回 `"auto"`
    表示不指定，客户端要用拥塞控制算法自行调整；
  - 直觉上：在丢包严重但带宽充足的链路上，传统拥塞控制会因为误判拥塞而大幅降速，直接按设定的带宽发送能更好地利用链路；代价是设定不准时可能对共享网络造成压力。

## Salamander 混淆

协议支持一个可选的混淆层，代号 **Salamander**：

$$ \\mathrm{hash} = \\mathrm{BLAKE2b\\text{-}256}(\\mathrm{key}
\\parallel \\mathrm{salt}),\\qquad \\mathrm{payload}\[i\] \\leftarrow
\\mathrm{payload}\[i\] \\oplus \\mathrm{hash}\[i \\bmod 32\] $$

其中：

  - $\\mathrm{key}$：用户预设的共享密钥；$\\mathrm{salt}$：每个 QUIC 包随机生成的 8
    字节盐，作为包的开头发送；
  - $\\parallel$：字节串拼接；$\\mathrm{BLAKE2b\\text{-}256}$：输出 32 字节的哈希函数；
  - $\\mathrm{payload}\[i\]$：QUIC 包的第 $i$ 个字节；$\\oplus$：按位异或；$i \\bmod
    32$：用 32 字节的哈希循环对包做异或；
  - 作用：把 QUIC 包变成看不出特征的随机字节（外观不再是 QUIC，但代价是失去了"像
    HTTP/3"这一伪装）；接收方用同样的算法还原，无效的包直接丢弃。

## 配置与使用

服务端和客户端使用 YAML 配置（示意，字段以官方文档为准）：

``` yaml
# 服务端 config.yaml
listen: :443
tls:
  cert: /path/to/fullchain.pem
  key: /path/to/privkey.pem
auth:
  type: password
  password: 请换成足够长的随机密码
masquerade:
  type: proxy
  proxy:
    url: https://example.com/
    rewriteHost: true
```

``` yaml
# 客户端 config.yaml
server: server.example.com:443
auth: 与服务端一致的密码
socks5:
  listen: 127.0.0.1:1080
http:
  listen: 127.0.0.1:8080
```

其中：

  - `tls`：服务端证书（也可以用 ACME 自动申请），Hysteria 内置 HTTP/3 服务；
  - `auth`：认证，常见为密码；
  - `masquerade`：**伪装**：未通过认证的请求，用这里配置的方式处理，例如反向代理到一个真实网站，使服务端像该网站；
  - 客户端模式很丰富：官方 README 提到 SOCKS5、HTTP 代理、TCP/UDP 转发、Linux TProxy、TUN 等。

## 优缺点

| 优点                            | 缺点                                                |
| ----------------------------- | ------------------------------------------------- |
| 基于 QUIC，在丢包/不稳定网络上速度表现好       | 基于 UDP，网络对 UDP 限速或封锁时不可用                          |
| 外观是标准的 HTTP/3 服务，主动探测只能看到普通网站 | 不能被 Nginx 等传统 HTTP 网关转发，通常自己监听 UDP 443            |
| 可设置带宽，在高丢包链路上利用率高             | 带宽设置不当会占满共享网络或造成拥塞                                |
| TCP/UDP 都支持，UDP 用数据报，延迟低      | 用户态 QUIC 的 CPU 开销较高；Salamander 混淆会失去"像 HTTP/3"的外观 |
| 协议简洁，规范公开（Hysteria 2 协议文档）    | 与旧的 Hysteria 1 不兼容，生态各客户端支持程度不同                   |

表中左边是优点，右边是缺点。

## Reference

  - [Hysteria 2
    协议规范](https://github.com/apernet/hysteria/blob/master/PROTOCOL.md)
  - [apernet/hysteria - Github](https://github.com/apernet/hysteria)
  - [Hysteria 官方文档](https://v2.hysteria.network/)
  - [RFC 9000 - QUIC](https://www.rfc-editor.org/rfc/rfc9000)
  - [RFC 9114 - HTTP/3](https://www.rfc-editor.org/rfc/rfc9114)
  - [RFC 9221 - QUIC Unreliable Datagram
    Extension](https://www.rfc-editor.org/rfc/rfc9221)
