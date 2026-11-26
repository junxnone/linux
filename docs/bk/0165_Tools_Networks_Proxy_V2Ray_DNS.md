-----

| Title     | Tools Networks Proxy V2Ray DNS                       |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:19:56Z`                               |
| Updated @ | `2026-11-26T10:19:56Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/165) |

-----

# DNS

V2Ray/Xray 内置了一个 **DNS 服务器模块**，它有两个主要作用：一是**根据域名解析出的 IP 去匹配路由规则**（例如
`geoip:cn`）；二是承担传统的 DNS 功能，**解析要连接的目标地址**。需要注意的是它只支持基础的 A、AAAA 记录查询；需要完整
DNS 能力时，官方建议配合 CoreDNS 之类的专用 DNS 服务。路由的整体介绍见 Routing 文章，工具介绍见
[V2Ray](0066_Tools_Networks_Proxy_V2Ray)。

## 工作方式

下图展示内置 DNS 如何选择上游服务器并过滤结果：

``` mermaid
flowchart TD
    Q["要解析的域名"] --> H{"hosts 静态表命中?"}
    H -->|"是"| R1["直接返回静态结果"]
    H -->|"否"| D{"某个 server 的 domains<br/>匹配该域名?"}
    D -->|"是"| S1["优先用该 server 查询"]
    D -->|"否"| S2["自上而下依次查询 servers"]
    S1 --> E{"结果符合 expectIPs?"}
    S2 --> E
    E -->|"是"| R2["返回 IP 列表"]
    E -->|"否"| N["换下一个 server"]
```

图中：

  - 解析请求先查 `hosts`（静态域名到 IP 的映射），命中就直接返回；
  - 否则，如果某个 `server` 的 `domains` 列表匹配当前域名，**优先使用该
    server**；没有匹配的则自上而下依次查询；
  - 如果 server 配了 `expectIPs`，只有返回的 IP 落在期望范围内（如 `geoip:cn`）才被接受，否则换下一个
    server；
  - 关键结论：通过"不同域名走不同 DNS + 结果校验"，可以实现**分流解析**，同时减少被污染的结果被采用的机会。

## 配置

``` json
{
  "dns": {
    "hosts": { "baidu.com": "127.0.0.1" },
    "servers": [
      {
        "address": "1.2.3.4",
        "port": 5353,
        "domains": ["domain:example.cn"],
        "expectIPs": ["geoip:cn"]
      },
      "8.8.8.8",
      "https://dns.google/dns-query",
      "localhost"
    ],
    "clientIp": "1.2.3.4",
    "tag": "dns_inbound"
  }
}
```

字段含义：

| 字段         | 含义                                                                      |
| ---------- | ----------------------------------------------------------------------- |
| `hosts`    | 静态域名到地址的映射；域名写法同路由的域名条件（纯字符串、`regexp:`、`domain:`、`keyword:`、`geosite:`） |
| `servers`  | DNS 服务器列表，可以是字符串，也可以是带 `domains`、`expectIPs` 的对象                        |
| `clientIp` | 当前系统的公网 IP，在 DNS 查询时告诉服务器客户端位置（EDNS Client Subnet），不能是私有地址              |
| `tag`      | 内置 DNS 发出的查询流量的标识，可以在路由里用 `inboundTag` 匹配，决定 DNS 查询走哪个出站                |

表中第一列是字段，第二列是含义。`servers` 里字符串的不同写法有不同的含义：

| 写法                               | 含义                                                   |
| -------------------------------- | ---------------------------------------------------- |
| `"8.8.8.8"`                      | 普通 UDP DNS，使用该地址的 53 端口                              |
| `"localhost"`                    | 使用本机预设的 DNS 配置；这类查询不受 V2Ray 控制，需要额外配置才能让它们经 V2Ray 转发 |
| `"https://host/dns-query"`       | **DoH**（DNS over HTTPS，RFC 8484），默认经过路由和出站发出         |
| `"https+local://host/dns-query"` | DoH 本地模式：请求不经过路由和出站，直接发出，减少一次转发，但不受代理控制              |

表中第一列是写法，第二列是含义。对象形式的 server 里 `address` 可以写同样的地址，`port` 默认 53（DoH
时无效，非标准端口写在 URL 里），`domains` 里的域名**优先**用该 server
查询，`expectIPs` 过滤结果。

## 与路由和 Freedom 的配合

  - 路由的 `domainStrategy` 为 `IPIfNonMatch` 或 `IPOnDemand` 时，会调用内置 DNS
    解析域名后再匹配 IP 规则（用 `geoip:cn` 之类）；
  - Freedom 出站的 `domainStrategy` **默认是 `AsIs`**，此时 DNS
    配置对它不生效；如果想让直连流量也用内置 DNS
    解析，要把它设为 `UseIP` 一类；
  - DNS 查询流量本身也要考虑走哪个出站：给 `tag` 设一个值，再在路由里用 `inboundTag` 把它引到合适的出站。

## FakeDNS

FakeDNS（"假"DNS，v4.35.0+）是一种机制：它对被查询的域名**返回一个虚构的 IP**（从自己的 `ipPool`
里分配），并记住"域名 ↔ 虚构 IP"的对应关系；当应用程序再按这个虚构 IP 发起连接时，入站的 `fakedns`
嗅探模块会把目标地址**还原成原来的域名**。这样在透明代理里，不需要真的解析 IP 就能按域名分流。

``` json
{
  "fakedns": { "ipPool": "240.0.0.0/8", "poolSize": 65535 }
}
```

其中 `ipPool` 是分配虚构 IP 的 CIDR 地址段，`poolSize` 是记住的"域名 ↔ IP"映射的个数。

## 优缺点

| 优点                        | 缺点                                                             |
| ------------------------- | -------------------------------------------------------------- |
| 内置，支持分流解析、静态 hosts、DoH    | 只支持 A/AAAA 基础查询，不是完整的 DNS 服务器                                  |
| `expectIPs` 可过滤被污染或不期望的结果 | 配置复杂，顺序和 `domains` 匹配规则需要仔细设计                                  |
| 可以通过 `tag` 控制 DNS 流量的路径   | 与路由的 `domainStrategy`、Freedom 的 `domainStrategy` 相互影响，容易出现配置遗漏 |
| FakeDNS 对透明代理有帮助          | FakeDNS 要求入站启用对应嗅探，且应用若缓存了虚构 IP 可能出问题                          |

表中左边是优点，右边是缺点。

## Reference

  - [DNS - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/en_US/config/dns.md)
  - [FakeDNS - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/en_US/config/fakedns.md)
  - [DNS - Xray
    文档](https://github.com/XTLS/Xray-docs-next/blob/main/docs/config/dns.md)
  - [RFC 8484 - DNS Queries over HTTPS
    (DoH)](https://www.rfc-editor.org/rfc/rfc8484)
