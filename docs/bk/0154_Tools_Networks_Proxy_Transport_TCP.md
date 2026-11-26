-----

| Title     | Tools Networks Proxy Transport TCP                   |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:16:22Z`                               |
| Updated @ | `2026-11-26T10:16:22Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/154) |

-----

# TCP

TCP 是 V2Ray/Xray 的**默认传输方式**：直接把代理协议的数据放在 TCP
连接上发送，不做额外的封装。它最简单、最稳定、开销最小，常与
TLS 配合使用（`security: "tls"`）。TCP 传输还有一个可选的 **HTTP 头部伪装**功能，用来让数据看起来像 HTTP
请求。传输方式的整体介绍见 Transport 总览，协议见各 Protocol 文章。

## 工作方式

下图对比"TCP 直接承载"与"TCP + TLS"两种常见组合：

``` mermaid
flowchart LR
    subgraph plain["TCP (security: none)"]
        A1["VMess / VLESS 数据"] --> T1["TCP 连接"]
    end
    subgraph tls["TCP + TLS"]
        A2["VMess / VLESS 数据"] --> S2["TLS 握手与加密"] --> T2["TCP 连接"]
    end
```

图中：

  - **左边**：协议数据直接放进 TCP。VMess 自带加密，所以仍然是加密的；VLESS 自己不加密，这样用等于明文，官方文档要求
    VLESS 必须配合安全层使用。
  - **右边**：先做 TLS 握手，数据经 TLS 加密后再放进 TCP，流量外观是标准 TLS，这是最推荐的组合。
  - 关键结论：TCP 传输本身不改变数据，安全性取决于上层协议和是否启用 TLS。

## 配置

``` json
{
  "streamSettings": {
    "network": "tcp",
    "security": "tls",
    "tcpSettings": {
      "acceptProxyProtocol": false,
      "header": { "type": "none" }
    }
  }
}
```

字段含义：

| 字段                    | 含义                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------- |
| `acceptProxyProtocol` | 仅用于入站：是否接收 PROXY protocol。填 `true` 时，TCP 连接建立后请求方必须先发送 PROXY protocol v1 或 v2，否则连接被关闭 |
| `header.type`         | 数据包头部伪装：`none`（默认，不伪装）或 `http`（伪装成 HTTP 请求/响应）                                        |

表中第一列是字段，第二列是含义。**PROXY protocol** 专用于传递请求的真实来源 IP
和端口，常见的反代软件（HAProxy、Nginx）可以配置发送它，如果你不了解它，请忽略该项。

## HTTP 头部伪装

`header.type` 设为 `http` 时，数据会被包上一层 HTTP 请求（客户端）和响应（服务端）的头部，伪装的内容由
`request`、`response` 字段配置：

  - `request`：`version`（默认 `1.1`）、`method`（默认
    `GET`）、`path`（数组，多值时每次随机选一个）、`headers`（键值，值为数组，每次随机选一个，如
    `Host`、`User-Agent`）；
  - `response`：对应的响应状态和头部；
  - **必须在对应的入站和出站两端同时配置，且内容必须一致**；
  - 这种伪装**无法被 Nginx 等 HTTP 服务器分流**，但可以被 VLESS 的 fallbacks `path` 分流；
  - 它只是在数据外面加了 HTTP 头，**不是真正的 HTTP
    流量**，也无法抵御深入的分析，所以不要把它当成安全措施；更可靠的做法是用真正的
    TLS 或 WebSocket。

## 优缺点

| 优点                              | 缺点                                   |
| ------------------------------- | ------------------------------------ |
| 最简单、开销最小，性能最好                   | 不加 TLS 时，流量特征明显（如 VMess 的随机数据流）      |
| 稳定，兼容性最好，所有实现都支持                | 单个 TCP 连接，丢包环境下受 TCP 队头阻塞影响          |
| 可与 TLS、REALITY、XTLS Vision 灵活组合 | HTTP 头伪装不是真正的 HTTP，无法被 Nginx 转发，效果有限 |
| 可使用 PROXY protocol 传递真实来源 IP    | 没有内置的多路复用（需要 Mux）                    |

表中左边是优点，右边是缺点。

## Reference

  - [TCP - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/config/transport/tcp.md)
  - [PROXY protocol 规范 -
    HAProxy](https://www.haproxy.org/download/2.2/doc/proxy-protocol.txt)
