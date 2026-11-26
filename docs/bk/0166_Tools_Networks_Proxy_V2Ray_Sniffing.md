-----

| Title     | Tools Networks Proxy V2Ray Sniffing                  |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:19:58Z`                               |
| Updated @ | `2026-11-26T10:19:58Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/166) |

-----

# Sniffing

Sniffing（流量嗅探）是 V2Ray/Xray
**入站**上的一个选项：它在代理刚接到连接时，**偷看连接最开头的几个字节**，从中识别出应用层协议和域名（HTTP
的 `Host` 头、TLS 握手里的 SNI 等），再把识别结果用作路由和转发的依据。它主要解决一个问题：很多入站（尤其是透明代理的
dokodemo-door）拿到的只有 **IP 地址**，看不到域名，也就无法按域名分流。工具介绍见
[V2Ray](0066_Tools_Networks_Proxy_V2Ray)。

## 工作方式

下图展示嗅探前后目标地址的变化：

``` mermaid
flowchart LR
    A["应用: 连接 93.184.216.34:443<br/>(只有 IP)"] --> I["入站 (如透明代理)"]
    I --> P["嗅探: 读 TLS ClientHello<br/>得到 SNI = example.com"]
    P --> R["路由: 按域名 example.com 匹配规则"]
    R --> O["出站: 把目标地址覆盖为 example.com:443<br/>(destOverride)"]
```

图中：

  - 应用原本只给出 IP 和端口，入站只能看到 `93.184.216.34:443`；
  - 嗅探模块读取连接开头的数据：TLS 的 ClientHello 里有 SNI，明文 HTTP 请求里有 `Host`；
  - 得到域名后，路由就能用**域名规则**（`domain`、`geosite`）匹配；
  - 按配置，目标地址还可以被**覆盖**（`destOverride`）为嗅探出的域名，再由出站去解析和连接——这样域名解析发生在代理服务器端，避免本地
    DNS 污染；
  - 关键结论：嗅探把"只有 IP 的流量"变成了"能按域名分流的流量"。

## 配置

``` json
{
  "inbounds": [{
    "port": 12345,
    "protocol": "dokodemo-door",
    "settings": { "network": "tcp,udp", "followRedirect": true },
    "sniffing": {
      "enabled": true,
      "destOverride": ["http", "tls"]
    }
  }]
}
```

字段含义：

| 字段                | 含义                                                                                         |
| ----------------- | ------------------------------------------------------------------------------------------ |
| `enabled`         | 是否启用嗅探                                                                                     |
| `destOverride`    | 嗅探成功后用识别出的域名覆盖目标地址，可选 `http`、`tls`，以及 `fakedns`（配合 FakeDNS 还原域名）；较新版本还有 `quic` 等，以所用版本文档为准 |
| `metadataOnly`    | 为 `true` 时只使用连接的元数据（如 FakeDNS）判断，不读取流量内容                                                   |
| `domainsExcluded` | 被排除的域名列表，这些域名即使嗅探到也不覆盖目标地址（避免某些应用被错误覆盖）                                                    |
| `routeOnly`（Xray） | 嗅探到的域名只用于路由，**不覆盖**目标地址，兼容性更好                                                              |

表中第一列是字段，第二列是含义。不同版本支持的字段不一致，使用前以所用版本的文档为准。

## 使用建议与风险

  - **只在需要时启用**：本机 SOCKS/HTTP 入站已经有域名时不需要嗅探；透明代理等"只有 IP"的入站需要。
  - **覆盖目标地址的副作用**：覆盖后出站按域名重新解析，可能解析出与应用原本连接的不同的 IP；对一些依赖固定 IP
    的应用（游戏、P2P）可能出问题，这时可用 `routeOnly` 或 `domainsExcluded`。
  - **只能嗅探到有明确特征的协议**：TLS、HTTP、QUIC 等；其他协议或加密后无法识别。
  - **BT 等协议**：嗅探还可以识别 `bittorrent`，在路由里用 `protocol` 条件把它屏蔽或单独处理。
  - **性能**：需要读取并缓冲连接开头的数据，有少量延迟。

## 优缺点

| 优点                         | 缺点                                   |
| -------------------------- | ------------------------------------ |
| 让只有 IP 的流量也能按域名分流          | 覆盖目标地址可能改变解析结果，影响部分应用                |
| 可识别协议（HTTP/TLS/BT），支持按协议路由 | 对无法识别的流量无效，加密协议（如不含 SNI 的 TLS）也看不到域名 |
| 解决透明代理里的 DNS 污染问题          | 增加少量延迟和复杂度                           |
| 配置简单，只在入站里加几行              | 不同版本字段不一致，需要对照文档                     |

表中左边是优点，右边是缺点。

## Reference

  - [V2Fly 文档站](https://www.v2fly.org/)
  - [Inbound - Xray
    文档（sniffing）](https://github.com/XTLS/Xray-docs-next/blob/main/docs/config/inbound.md)
