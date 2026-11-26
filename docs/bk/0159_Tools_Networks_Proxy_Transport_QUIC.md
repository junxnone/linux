-----

| Title     | Tools Networks Proxy Transport QUIC                  |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:16:33Z`                               |
| Updated @ | `2026-11-26T10:16:33Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/159) |

-----

# QUIC

QUIC（Quick UDP Internet Connections）是 Google 提出的、**基于 UDP
的多路并发传输协议**，后来被 IETF 标准化（RFC 9000），也是 HTTP/3
的传输基础。V2Ray 4.7 起支持把 QUIC 作为传输方式。QUIC 把传输层和 TLS 1.3
合在一起，**强制加密**，并且解决了 TCP 上的几个老问题。整体介绍见
Transport 总览。

## QUIC 的主要优势

1.  **更低的握手延迟**：连接建立只需 1-RTT，复用旧会话时可以 0-RTT（TCP + TLS 通常要 2-3 个 RTT）。
2.  **多路复用且没有队头阻塞**：一条连接里的多个流相互独立，一个流丢包不会阻塞其他流（HTTP/2 over TCP 做不到）。
3.  **连接迁移**：用 **Connection ID** 标识连接，客户端从 WiFi 切换到 4G 等导致 IP
    变化时，连接不会断开。

## 工作方式

下图对比 TCP + TLS 与 QUIC 的握手往返：

``` mermaid
sequenceDiagram
    participant C as 客户端
    participant S as 服务端
    Note over C,S: TCP + TLS 1.3 (约 2 RTT)
    C->>S: TCP SYN
    S-->>C: SYN-ACK
    C->>S: ACK + TLS ClientHello
    S-->>C: ServerHello + 证书 ...
    Note over C,S: QUIC (1 RTT)
    C->>S: Initial 包 (含 TLS ClientHello)
    S-->>C: Initial + Handshake (含 ServerHello 与证书)
    C->>S: 完成握手 + 应用数据
```

图中：

  - 上半部分是 TCP + TLS 1.3：先 TCP 三次握手，再做 TLS，首个应用数据要等 2 个往返；
  - 下半部分是 QUIC：握手数据直接放在 UDP 数据包里，并且传输握手与 TLS 握手合并，1 个往返就能开始传输应用数据；
  - 关键结论：QUIC 把"建立连接"和"协商加密"合并，所以更快，并且是内置加密的。

## 配置

``` json
{
  "streamSettings": {
    "network": "quic",
    "quicSettings": {
      "security": "none",
      "key": "",
      "header": { "type": "none" }
    }
  }
}
```

字段含义：

| 字段            | 含义                                                                             |
| ------------- | ------------------------------------------------------------------------------ |
| `security`    | 对 QUIC 数据包的额外加密：`none`（默认）、`aes-128-gcm`、`chacha20-poly1305`；加密后数据包无法被识别为 QUIC |
| `key`         | 加密用的密钥，可以是任意字符串，`security` 不为 `none` 时有效                                       |
| `header.type` | 数据包头部伪装：`none`、`srtp`、`utp`、`wechat-video`、`dtls`、`wireguard`                  |

表中第一列是字段，第二列是含义。几点注意：

  - **对接的两端配置必须完全一致**，否则连接失败；
  - QUIC **强制要求 TLS**；传输配置里没有启用 TLS 时，V2Ray 会自己签发一个证书进行通信；
  - 默认使用 12 字节的 Connection ID，**30 秒没有数据**自动断开连接，可能影响长时间空闲的长连接；
  - 加密和伪装都不启用时，数据包就是原始的 QUIC 包，可以与其他 QUIC 工具对接，为避免被识别，建议至少开启其中一项。

## 优缺点

| 优点                            | 缺点                                                    |
| ----------------------------- | ----------------------------------------------------- |
| 握手快（1-RTT / 0-RTT），内置 TLS 1.3 | 基于 UDP，部分网络会对 UDP 限速或丢弃                               |
| 多路复用无队头阻塞，对丢包更友好              | 不能被 Nginx、CDN 等传统 HTTP 网关转发                           |
| 连接迁移，网络切换不断线                  | 用户态实现 QUIC 的 CPU 占用比 TCP 高，缺少内核/硬件卸载                  |
| 没有 TCP 的"TLS in TCP"问题        | 新版本的 V2Fly/Xray 对 QUIC 传输的维护和推荐程度各不相同，使用前请确认所用版本是否仍支持 |

表中左边是优点，右边是缺点。需要基于 QUIC 的高速代理时，也可以考虑 Hysteria、TUIC 等专门的协议（见相应文章）。

## Reference

  - [RFC 9000 - QUIC: A UDP-Based Multiplexed and Secure
    Transport](https://www.rfc-editor.org/rfc/rfc9000)
  - [RFC 9001 - Using TLS to Secure
    QUIC](https://www.rfc-editor.org/rfc/rfc9001)
  - [QUIC - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/config/transport/quic.md)
