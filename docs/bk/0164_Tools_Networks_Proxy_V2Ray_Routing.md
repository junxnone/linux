-----

| Title     | Tools Networks Proxy V2Ray Routing                   |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:19:54Z`                               |
| Updated @ | `2026-11-26T10:19:54Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/164) |

-----

# Routing

路由（Routing）是 V2Ray/Xray
的内置模块：它根据**规则**把每一个入站连接分配给不同的出站，从而实现"选择性代理"。最常见的用法是**分流**：国内网站直连、国外网站走代理、广告域名屏蔽。工具整体介绍见
[V2Ray](0066_Tools_Networks_Proxy_V2Ray)，出口类型见 Freedom、Blackhole 文章。

## 工作方式

下图展示一条连接从进入到选定出站的判断过程：

``` mermaid
flowchart TD
    I["入站连接<br/>(目标 / 来源 / 协议 / 入站 tag)"] --> R1{"规则 1 命中?"}
    R1 -->|"是"| O1["outboundTag / balancerTag"]
    R1 -->|"否"| R2{"规则 2 命中?"}
    R2 -->|"是"| O2["outboundTag"]
    R2 -->|"否"| Rn{"... 规则 n"}
    Rn -->|"都未命中"| D["默认: 第一个 outbound"]
```

图中：

  - 路由**按规则顺序依次匹配**，命中第一条就确定出站，后面的规则不再看；
  - 一条规则里写了多个条件（如 `domain` 和 `port`）时，**必须全部满足**该规则才生效，因此域名条件和 IP
    条件通常分成不同的规则写；
  - 所有规则都没命中时，流量交给配置里**第一个 outbound**（默认出口）；
  - 关键结论：规则的**顺序**决定结果，特殊规则（如屏蔽广告）要放在宽泛规则（如 `geosite:cn` 直连）前面。

## 配置结构

``` json
{
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      { "type": "field", "domain": ["geosite:category-ads-all"], "outboundTag": "adblock" },
      { "type": "field", "domain": ["geosite:cn"], "outboundTag": "direct" },
      { "type": "field", "ip": ["geoip:cn", "geoip:private"], "outboundTag": "direct" }
    ]
  }
}
```

顶层字段含义：

| 字段               | 含义                                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------------- |
| `domainStrategy` | 域名解析策略：`AsIs` 只用域名匹配（默认）；`IPIfNonMatch` 域名规则都没命中时，把域名解析成 IP 再匹配 IP 规则；`IPOnDemand` 遇到任何 IP 规则就立即解析域名 |
| `domainMatcher`  | 域名匹配算法：`linear`（默认）或 `mph`（最小完美哈希，v4.36.1+，官方测试约 17 万条数据时匹配速度提升约 30%、内存减少约 15%）                      |
| `rules`          | 规则列表                                                                                                 |
| `balancers`      | 负载均衡器列表（v4.4+），规则可以指向某个均衡器而不是固定出站                                                                    |

表中第一列是字段，第二列是含义。注意 `IPIfNonMatch` 解析出的 IP 仅用于路由选择，转发时仍使用原域名；`IPOnDemand`
匹配更准但会多一次 DNS 查询。

## 规则字段

规则（RuleObject）常用字段：

| 字段                            | 含义                                           |
| ----------------------------- | -------------------------------------------- |
| `type`                        | 固定为 `field`                                  |
| `domain` / `domains`          | 域名条件（不同版本/文档写作 `domain` 或 `domains`，以所用版本为准） |
| `ip`                          | IP 条件                                        |
| `port` / `sourcePort`         | 目标端口/来源端口，如 `"53,443,1000-2000"`             |
| `network`                     | `tcp`、`udp` 或 `tcp,udp`                      |
| `source`                      | 来源 IP                                        |
| `user`                        | 用户邮箱                                         |
| `inboundTag`                  | 入站标识                                         |
| `protocol`                    | 嗅探出的协议：`http`、`tls`、`bittorrent` 等           |
| `outboundTag` / `balancerTag` | 命中后使用的出站标识或均衡器标识                             |

表中第一列是字段，第二列是含义。

**域名条件**支持的写法：

| 写法          | 含义与例子                                                                          |
| ----------- | ------------------------------------------------------------------------------ |
| 纯字符串        | 包含该字符串即匹配：`sina.com` 匹配 `sina.com`、`www.sina.com`、`sina.com.cn`，不匹配 `sina.cn`  |
| `regexp:`   | 正则：`regexp:\.goo.*\.com$` 匹配 `www.google.com`，不匹配 `google.com`                 |
| `domain:`   | **子域名（推荐）**：`domain:v2ray.com` 匹配 `v2ray.com`、`www.v2ray.com`，不匹配 `xv2ray.com` |
| `full:`     | 完整匹配：`full:v2ray.com` 只匹配 `v2ray.com`                                          |
| `geosite:`  | 预定义域名列表：`geosite:cn`、`geosite:google`                                          |
| `ext:文件:标签` | 从资源目录里的自定义 dat 文件加载                                                            |

**IP 条件**支持：单个 IP、CIDR（`10.0.0.0/8`）、`geoip:cn`（中国大陆
IP，用两位国家代码）、`geoip:!cn`（非中国大陆）、`geoip:private`（私有地址）、`ext:文件:标签`。

## geosite 与 geoip

`geosite.dat` 和 `geoip.dat` 是预编译的域名/IP 数据文件，放在 V2Ray 的资源目录下。社区有常用的数据集（如
v2fly/domain-list-community、Loyalsoldier 的
rules-dat），定期更新；**数据文件应当保持更新**，否则分流结果会逐渐不准。可以用
`geosite:category-ads-all` 做广告屏蔽，用 `geosite:cn` 做国内直连。

## 常见分流写法

典型的客户端分流策略：

1.  广告域名 → `blackhole`（屏蔽）；
2.  私有地址、国内域名、国内 IP → `freedom`（直连）；
3.  其余 → 默认第一个 outbound（代理）。

写在 outbounds 里时，把代理出站放在**第一个**，这样"没命中任何规则"的流量默认走代理。

## 优缺点

| 优点                               | 缺点                                                        |
| -------------------------------- | --------------------------------------------------------- |
| 内置，不依赖外部工具，可按域名/IP/端口/协议等多种条件分流  | 规则顺序敏感，写错顺序会导致意外结果                                        |
| `geosite`/`geoip` 预定义列表，省去手写大量域名 | 依赖数据文件，需要定期更新                                             |
| 可以配合负载均衡器实现多出口                   | 域名与 IP 规则的解析策略（`domainStrategy`）会影响 DNS 行为，需要和 DNS 配置一起考虑 |
| 可按嗅探出的协议和入站标识分流                  | 复杂规则难调试，需要借助日志                                            |

表中左边是优点，右边是缺点。

## Reference

  - [Routing - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/en_US/config/routing.md)
  - [Routing - Xray
    文档](https://github.com/XTLS/Xray-docs-next/blob/main/docs/config/routing.md)
  - [domain-list-community](https://github.com/v2fly/domain-list-community)
