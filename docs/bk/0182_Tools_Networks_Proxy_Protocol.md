-----

| Title     | Tools Networks Proxy Protocol                        |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:27:16Z`                               |
| Updated @ | `2026-11-26T10:27:16Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/182) |

-----

# Protocol

V2Ray 的配置里，`inbounds` 和 `outbounds` 都用 `protocol`
字段指定"协议"。这里的"协议"其实包含两类东西：一类是真正的**代理协议**（VMess、VLESS、Shadowsocks、Trojan
等），规定客户端和服务端之间的数据格式、认证和加密；另一类是**内置的入站/出站处理器**（Freedom、Blackhole、DokodemoDoor、Loopback
等），不是网络协议，只是核心里的功能模块。本文说明这些协议各是什么、怎么工作、有什么优缺点，并给出每个协议专属文章的入口。工具介绍见
[V2Ray](0066_Tools_Networks_Proxy_V2Ray)，代理的基本概念见
[Proxy](0080_Tools_Networks_Proxy)。

## 协议在配置中的位置

下图展示一个 V2Ray/Xray 配置里"协议"出现的位置，以及它与传输、安全层的关系：

``` mermaid
flowchart LR
    subgraph inbound["inbounds (入站)"]
        I1["protocol: socks / http / dokodemo-door<br/>(本机入口)"]
        I2["protocol: vmess / vless / trojan / shadowsocks<br/>(服务端入口)"]
    end
    inbound --> R{"routing"}
    R --> O
    subgraph O["outbounds (出站)"]
        O1["protocol: vmess / vless / trojan / shadowsocks / socks / http / wireguard<br/>(经远端或上游)"]
        O2["protocol: freedom / blackhole / loopback / dns<br/>(内置处理器)"]
    end
    O1 --> SS["streamSettings: 传输 + 安全层"]
```

图中：

  - 数据从**入站**进入，经过**路由**，选出一个**出站**；每个入站和出站都有一个 `protocol`；
  - **入站**常见两类：给本机应用用的入口（socks、http、dokodemo-door）和给远端客户端用的服务端入口（vmess、vless、trojan、shadowsocks）；
  - **出站**也是两类：经远端或上游转发的（vmess、vless、trojan、shadowsocks、socks、http、wireguard）和内置处理器（freedom
    直连、blackhole 屏蔽、loopback 回环、dns）；
  - 经远端转发的协议，再通过 `streamSettings` 决定用什么**传输**（TCP、WebSocket
    等）和**安全层**（TLS、REALITY）发出去；
  - 关键结论：协议只回答"请求是什么、怎么认证"，数据怎么发、是否再加密是传输和安全层的事。

## 协议一览

下表汇总所有协议，并给出各专属文章的链接：

| 协议           | 类别             | 入站/出站     | 一句话                                 | 专属文章                                                            |
| ------------ | -------------- | --------- | ----------------------------------- | --------------------------------------------------------------- |
| VMess        | 代理协议           | 入/出       | V2Ray 原创的加密协议，无状态，依赖时间，自带数据加密       | [VMess](0142_Tools_Networks_Proxy_Protocol_VMess)               |
| VLESS        | 代理协议           | 入/出       | VMess 的轻量继任者，不自带加密，必须配合 TLS/REALITY | [VLESS](0143_Tools_Networks_Proxy_Protocol_VLESS)               |
| Shadowsocks  | 代理协议           | 入/出       | 基于 SOCKS5 地址格式的加密代理，预共享密钥，无握手       | [Shadowsocks](0144_Tools_Networks_Proxy_Protocol_Shadowsocks)   |
| Trojan       | 代理协议           | 入/出       | 以真实 TLS 为外壳，认证失败回落到真实网站             | [Trojan](0145_Tools_Networks_Proxy_Protocol_Trojan)             |
| Socks        | 代理协议（通用）       | 入/出       | 通用的 TCP/UDP 代理协议，无加密                | [Socks](0146_Tools_Networks_Proxy_Protocol_Socks)               |
| HTTP         | 代理协议（通用）       | 入/出       | 最常见的代理形式，`CONNECT` 隧道转发 HTTPS       | [HTTP](0147_Tools_Networks_Proxy_Protocol_HTTP)                 |
| MTProto      | 代理协议（Telegram） | 入（旧版）     | Telegram 的代理协议，新版文档已不再列出            | [MTProto](0148_Tools_Networks_Proxy_Protocol_MTProto)           |
| WireGuard    | VPN 协议         | 入/出（Xray） | 网络层 VPN，Noise 握手，只用 UDP             | [WireGuard](0152_Tools_Networks_Proxy_Protocol_WireGuard)       |
| Freedom      | 内置处理器          | 出         | 直连，不经过代理                            | [Freedom](0149_Tools_Networks_Proxy_Protocol_Freedom)           |
| Blackhole    | 内置处理器          | 出         | 丢弃流量，用于屏蔽                           | [Blackhole](0150_Tools_Networks_Proxy_Protocol_Blackhole)       |
| DokodemoDoor | 内置处理器          | 入         | 任意门：端口映射和透明代理入口（Xray 新称 Tunnel）     | [DokodemoDoor](0151_Tools_Networks_Proxy_Protocol_DokodemoDoor) |
| Loopback     | 内置处理器          | 出         | 把流量送回路由再处理一次                        | [Loopback](0153_Tools_Networks_Proxy_Protocol_Loopback)         |

表中各列：**类别**把协议分成代理协议、VPN 协议和内置处理器；**入站/出站**说明它能出现在哪一侧（以 V2Ray/Xray
的文档为准，部分只在特定核心里存在）；**一句话**是最关键的特点；**专属文章**是详细介绍的入口。要对比各协议的加密、UDP
支持、抗探测能力等，见 [Comparison](0180_Tools_Networks_Proxy_Comparison)。

## 各协议如何工作（概览）

下面用一张图概括几种主要代理协议在**数据格式和认证**上的差异：

``` mermaid
flowchart TB
    subgraph vmess["VMess"]
        V1["认证 ID (UUID + 时间戳, AEAD) + 加密的指令头 + 加密数据块"]
    end
    subgraph vless["VLESS"]
        L1["版本 + UUID + Addons + 命令 + 目标地址 + 数据<br/>(自身不加密, 靠 TLS/REALITY)"]
    end
    subgraph ss["Shadowsocks"]
        S1["随机 salt + 加密 chunk 序列 (首个 chunk 含目标地址)"]
    end
    subgraph trojan["Trojan"]
        T1["TLS 通道内: hex(SHA224(password)) + CRLF + SOCKS5 式请求 + CRLF + 数据"]
    end
```

图中：

  - **VMess**：请求头里有加密的认证 ID（由 UUID 派生，含时间戳）和加密的指令部分，之后是加密的数据块；要求时间同步；
  - **VLESS**：请求头直接是 UUID 和目标地址，**不加密**，由外层 TLS/REALITY 保护；
  - **Shadowsocks**：以随机 salt 开头，之后是一串 AEAD 加密的 chunk，目标地址藏在第一个 chunk 里；
  - **Trojan**：在真实 TLS 里发送密码哈希和 SOCKS5 风格的请求；
  - 关键结论：VMess 和 Shadowsocks 把加密做在协议里，VLESS 和 Trojan 把加密交给
    TLS，这是设计思路上最大的分野。

## 阅读顺序建议

1.  **先看通用协议**：[Socks](0146_Tools_Networks_Proxy_Protocol_Socks)、[HTTP](0147_Tools_Networks_Proxy_Protocol_HTTP)，理解"代理协议是什么"；
2.  **再看加密代理协议**：[Shadowsocks](0144_Tools_Networks_Proxy_Protocol_Shadowsocks)、[VMess](0142_Tools_Networks_Proxy_Protocol_VMess)、[VLESS](0143_Tools_Networks_Proxy_Protocol_VLESS)、[Trojan](0145_Tools_Networks_Proxy_Protocol_Trojan)；
3.  **然后看内置处理器**：[Freedom](0149_Tools_Networks_Proxy_Protocol_Freedom)、[Blackhole](0150_Tools_Networks_Proxy_Protocol_Blackhole)、[DokodemoDoor](0151_Tools_Networks_Proxy_Protocol_DokodemoDoor)、[Loopback](0153_Tools_Networks_Proxy_Protocol_Loopback)；
4.  **再看传输和安全层**：[Transport
    总览](0181_Tools_Networks_Proxy_Transport)、[TLS](0161_Tools_Networks_Proxy_Security_TLS)、[XTLS](0162_Tools_Networks_Proxy_Security_XTLS)、[Reality](0163_Tools_Networks_Proxy_Security_Reality)；
5.  **最后看功能模块和部署**：[Routing](0164_Tools_Networks_Proxy_V2Ray_Routing)、[DNS](0165_Tools_Networks_Proxy_V2Ray_DNS)、[Sniffing](0166_Tools_Networks_Proxy_V2Ray_Sniffing)、[Mux](0167_Tools_Networks_Proxy_V2Ray_Mux)、[Policy](0168_Tools_Networks_Proxy_V2Ray_Policy)、[API](0169_Tools_Networks_Proxy_V2Ray_API)、[Transparent](0170_Tools_Networks_Proxy_V2Ray_Transparent)、[Systemd](0171_Tools_Networks_Proxy_V2Ray_Systemd)、[Nginx](0172_Tools_Networks_Proxy_V2Ray_Nginx)、[Client](0173_Tools_Networks_Proxy_V2Ray_Client)、[Troubleshooting](0174_Tools_Networks_Proxy_V2Ray_Troubleshooting)。

## 相关工具与协议

  - 同类内核和工具：[Xray](0175_Tools_Networks_Proxy_Xray)、[sing-box](0176_Tools_Networks_Proxy_sing_box)、[Clash](0177_Tools_Networks_Proxy_Clash)；
  - 基于 QUIC
    的新协议：[Hysteria](0178_Tools_Networks_Proxy_Hysteria)、[TUIC](0179_Tools_Networks_Proxy_TUIC)；
  - 非 V2Ray
    系的代理工具：[shadowsocks](0067_Tools_Networks_Proxy_shadowsocks)、[Privoxy](0139_Tools_Networks_Proxy_Privoxy)、[proxychains](0140_Tools_Networks_Proxy_proxychains)、[Squid](0141_Tools_Networks_Proxy_Squid)、[dev-sidecar](0138_Tools_Networks_Proxy_dev_sidecar)；
  - 总对比：[Comparison](0180_Tools_Networks_Proxy_Comparison)。

## 使用须知

  - 只在符合所在网络管理规定和当地法律法规的前提下使用。
  - 协议的细节和字段随核心版本（V2Fly 与 Xray）变化，以所用版本的官方文档为准。

## Reference

  - [V2Fly 配置文档 -
    协议](https://github.com/v2fly/v2fly-github-io/tree/master/docs/en_US/config/protocols)
  - [Xray 文档 -
    入站](https://github.com/XTLS/Xray-docs-next/tree/main/docs/config/inbounds)
  - [Xray 文档 -
    出站](https://github.com/XTLS/Xray-docs-next/tree/main/docs/config/outbounds)
  - [VMess 协议 - V2Fly
    开发者文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/developer/protocols/vmess.md)
