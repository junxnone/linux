-----

| Title     | Tools Networks Proxy Comparison                      |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:25:54Z`                               |
| Updated @ | `2026-11-26T10:25:54Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/180) |

-----

# Comparison

代理协议和代理工具很多，名字也容易混淆：**协议**（VMess、VLESS、Shadowsocks、Trojan、Hysteria2
等）规定数据怎么在线上传输；**传输和安全层**（TCP、WebSocket、TLS、REALITY
等）规定承载方式；**工具/内核**（V2Ray、Xray、sing-box、mihomo
等）把它们实现成可运行的程序。本文把它们放在一起对比，并给出按场景选型的思路。各项细节见对应的专属文章，总览见
[Proxy](0080_Tools_Networks_Proxy)。

## 三个层次

下图把常见名词放进三个层次里，避免把不同层次的东西混着比：

``` mermaid
flowchart TB
    subgraph L1["工具 / 内核"]
        K1["V2Ray / V2Fly"] ~~~ K2["Xray"] ~~~ K3["sing-box"] ~~~ K4["mihomo (Clash)"] ~~~ K5["shadowsocks-libev / trojan-gfw / hysteria"]
    end
    subgraph L2["代理协议"]
        P1["VMess / VLESS"] ~~~ P2["Shadowsocks"] ~~~ P3["Trojan"] ~~~ P4["Hysteria2 / TUIC"] ~~~ P5["Socks / HTTP / WireGuard"]
    end
    subgraph L3["传输与安全"]
        T1["TCP / WebSocket / gRPC / HTTP2 / mKCP / QUIC"] ~~~ T2["TLS / REALITY / XTLS"]
    end
    L1 --> L2 --> L3
```

图中：

  - 最上层是**工具/内核**：实现了下面各种协议的程序，同一个内核通常支持多个协议；
  - 中间是**代理协议**：描述"客户端怎么告诉服务端要访问哪里、怎么认证"；
  - 最下面是**传输与安全**：数据具体用 TCP、WebSocket 还是 QUIC 发送，是否加 TLS；
  - 箭头表示依赖关系：内核实现协议，协议跑在传输和安全层之上；
  - 关键结论：比较时要在同一层次内比较——比如"VMess vs VLESS"是协议之间的比较，"V2Ray vs
    Xray"是内核之间的比较，而"WebSocket vs gRPC"是传输之间的比较。

## 代理协议对比

下表对常见协议做横向对比：

| 协议                      | 自带加密                         | 底层        | UDP               | 认证            | 抗主动探测                | 典型用途          |
| ----------------------- | ---------------------------- | --------- | ----------------- | ------------- | -------------------- | ------------- |
| Socks5                  | 无                            | TCP       | 支持（UDP ASSOCIATE） | 用户名/密码（明文）    | 无                    | 本地入口、前置代理     |
| HTTP                    | 无                            | TCP       | 不支持               | Basic（Base64） | 无                    | 本地入口、企业代理     |
| Shadowsocks (AEAD/2022) | 有                            | TCP/UDP   | 支持                | 预共享密钥         | 中（随机字节流，2022 版有重放保护） | 简单的加密中继       |
| VMess                   | 有（AEAD 头 + 数据加密）             | TCP（经传输层） | 支持                | UUID + 时间     | 中                    | V2Ray 的标准协议   |
| VLESS                   | 无（靠 TLS/REALITY/Encryption）  | TCP（经传输层） | 支持                | UUID          | 高（配合 TLS 回落、REALITY） | Xray 主流方案     |
| Trojan                  | 无（靠 TLS）                     | TCP/TLS   | 支持                | 密码哈希          | 高（回落到真实网站）           | 标准 TLS 外观的代理  |
| Hysteria2               | QUIC/TLS 1.3                 | UDP（QUIC） | 数据报               | 密码            | 高（HTTP/3 伪装）         | 丢包网络、高速传输     |
| TUIC                    | QUIC/TLS 1.3                 | UDP（QUIC） | 数据报 + 流，Full Cone | UUID + 密码     | 中                    | 游戏、语音等 UDP 场景 |
| WireGuard               | 有（Noise + ChaCha20-Poly1305） | UDP       | 整个 IP 包           | 公私钥           | 低（特征固定）              | 组网、点对点 VPN    |

表中每一行是一个协议，各列分别是：是否自带加密、底层承载、是否支持
UDP、认证方式、对主动探测的抵抗程度、典型用途。其中"抗主动探测"一列是粗略的定性评价，取决于具体配置和实现，不是绝对结论。读表时注意：**VLESS
和 Trojan 本身不加密**，必须配合 TLS 等安全层使用，否则明文。

## 传输方式对比

| 传输         | 底层  | 能经 Nginx/CDN     | 多路复用      | 特点             |
| ---------- | --- | ---------------- | --------- | -------------- |
| TCP (+TLS) | TCP | TLS 终结后可（需要协议配合） | 无（可用 Mux） | 最简单、性能最好       |
| WebSocket  | TCP | 可                | 无（可用 Mux） | 最通用的"藏在网站后"的方式 |
| HTTP/2     | TCP | 可                | 有         | 需要 TLS         |
| gRPC       | TCP | 可（需网关支持 gRPC）    | 有         | 基于 HTTP/2      |
| mKCP       | UDP | 不可               | 内部        | 低延迟，耗带宽        |
| QUIC       | UDP | 不可               | 有         | 1-RTT 握手，无队头阻塞 |

表中每一行是一个传输方式，列分别是底层、能否被 Nginx/CDN 转发、是否自带多路复用和特点。

## 工具对比

| 工具                     | 配置格式 | 支持的主要协议                                             | 特点                           |
| ---------------------- | ---- | --------------------------------------------------- | ---------------------------- |
| V2Ray / V2Fly          | JSON | VMess、VLESS、Shadowsocks、Trojan、Socks、HTTP 等         | 模块化，路由灵活；v4/v5 两种配置          |
| Xray                   | JSON | VLESS、VMess、Trojan、Shadowsocks、WireGuard、Hysteria 等 | XTLS Vision、REALITY、XHTTP 独有 |
| sing-box               | JSON | 以上大部分 + Hysteria2、TUIC 等                            | 通用平台，内置 TUN、规则集、DNS 规则       |
| mihomo (Clash)         | YAML | VMess、VLESS、SS、Trojan、TUIC、Hysteria 等               | 规则 + 分组，Web 面板               |
| shadowsocks-libev/rust | JSON | Shadowsocks                                         | 轻量，只做 SS                     |
| trojan-gfw             | JSON | Trojan                                              | 只做 Trojan                    |
| Hysteria               | YAML | Hysteria2                                           | 官方实现，客户端模式丰富                 |

表中每一行是一个工具，列分别是配置格式、支持的主要协议和特点。具体的协议覆盖随版本变化，以各项目文档为准。

## 选型思路

下图把常见需求映射到推荐组合：

``` mermaid
flowchart TD
    Q1{"需求是什么?"} -->|"企业内网出口 / 缓存"| A1["HTTP 代理 (Squid) + 环境变量"]
    Q1 -->|"本机转换协议"| A2["Privoxy / SOCKS5 本地入口"]
    Q1 -->|"需要自己搭中继"| Q2{"能用 UDP, 且网络丢包高?"}
    Q2 -->|"是"| B1["Hysteria2 / TUIC"]
    Q2 -->|"否"| Q3{"有域名并想保留网站?"}
    Q3 -->|"是"| C1["VLESS/Trojan + TLS + Nginx 回落 或 WebSocket/gRPC"]
    Q3 -->|"否 (无域名)"| C2["VLESS + REALITY + Vision (Xray)"]
    Q1 -->|"想简单加密中继"| D1["Shadowsocks AEAD / 2022"]
    Q1 -->|"统一管理多协议、移动端"| E1["sing-box / mihomo"]
```

图中：

  - 从"需求"出发，沿着判断走向推荐组合；每个终点是一种常见搭配；
  - **企业/内网场景**：不需要代理软件，用 HTTP 代理（如
    [Squid](0141_Tools_Networks_Proxy_Squid)）加环境变量即可，见
    [Proxy](0080_Tools_Networks_Proxy)；
  - **自建中继、UDP 可用且丢包高**：选 Hysteria2 或 TUIC；
  - **有域名且想保留一个网站**：VLESS 或 Trojan 配 TLS，用 Nginx 回落或 WebSocket/gRPC 分流；
  - **没有域名**：VLESS + REALITY + XTLS Vision（Xray）；
  - **只想要简单加密中继**：Shadowsocks AEAD 或 2022 版；
  - **想统一管理多协议或用于移动端**：sing-box、mihomo；
  - 关键结论：没有"最好的协议"，只有"最适合当前网络和约束的组合"。

## 通用建议

  - 优先选**仍在积极维护**的实现和协议版本，不使用 Shadowsocks 流加密、VMess MD5 认证等已弃用方式；
  - 能用 **AEAD/TLS 1.3** 的地方不要用明文或旧加密；
  - **客户端和服务端必须同种内核和兼容版本**，新特性（REALITY、Vision、Hysteria2）要确认两端都支持；
  - 凭据（UUID、密码、密钥）**不进仓库**；
  - 只在符合所在网络管理规定和当地法律法规的前提下使用。

## Reference

  - [VMess 协议 - V2Fly
    开发者文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/developer/protocols/vmess.md)
  - [VLESS Protocol - Xray
    文档](https://github.com/XTLS/Xray-docs-next/blob/main/docs/en/development/protocols/vless.md)
  - [SIP022 AEAD-2022
    Ciphers](https://github.com/shadowsocks/shadowsocks-org/blob/main/docs/doc/sip022.md)
  - [The Trojan
    Protocol](https://github.com/trojan-gfw/trojan/blob/master/docs/protocol.md)
  - [Hysteria 2
    协议规范](https://github.com/apernet/hysteria/blob/master/PROTOCOL.md)
  - [tuic-protocol/tuic](https://github.com/tuic-protocol/tuic)
  - [sing-box](https://github.com/SagerNet/sing-box)
  - [mihomo](https://github.com/MetaCubeX/mihomo)
