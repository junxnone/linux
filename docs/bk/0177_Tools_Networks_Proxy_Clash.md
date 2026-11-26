-----

| Title     | Tools Networks Proxy Clash                           |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:25:48Z`                               |
| Updated @ | `2026-11-26T10:25:48Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/177) |

-----

# Clash

Clash 是一个**基于规则的代理客户端内核**，用 Go 编写，以 **YAML** 配置。它的特点是"**规则驱动 +
节点分组**"：在配置里先列出一批代理节点（`proxies`），再把它们组织成可手动选择或自动测速的分组（`proxy-groups`），最后写一串规则（`rules`）决定每个请求走哪个分组。许多图形客户端都是以
Clash 系内核为基础的。原始的 Clash 项目已经停止维护（据我所知在 2023 年底），**社区以 mihomo（原
Clash.Meta）继续开发**，目前使用的主要是 mihomo 内核及其分支。总览见
[Proxy](0080_Tools_Networks_Proxy)。

## 结构

下图展示 Clash 配置里几个部分的关系：

``` mermaid
flowchart LR
    I["本地入站<br/>HTTP / SOCKS / mixed / TUN / redir"] --> R{"rules<br/>按顺序匹配"}
    R -->|"DOMAIN / GEOIP / IP-CIDR / PROCESS ..."| G["proxy-groups<br/>select / url-test / fallback / load-balance"]
    G --> P["proxies<br/>vmess / vless / ss / trojan / tuic / hysteria ..."]
    G --> D["DIRECT / REJECT"]
    D1["dns<br/>DoH / DoT / fake-ip"] -.-> R
```

图中：

  - **本地入站**：本地的 HTTP/SOCKS 代理端口、TUN 虚拟网卡、Linux 下的 netfilter 重定向；
  - **rules（规则）**：自上而下按顺序匹配，类型包括按域名、GEOIP、IP-CIDR、进程名等；命中后指定一个**分组**或直接
    `DIRECT`（直连）/`REJECT`（拒绝）；
  - **proxy-groups（分组）**：把节点组织起来：`select` 手动选择、`url-test`
    按延迟自动选最快、`fallback` 主节点失败自动切换、`load-balance` 负载均衡；
  - **proxies（节点）**：具体的代理节点；
  - **dns**：内置 DNS，支持 DoH/DoT 上游和 fake-ip 模式，尽量降低 DNS 污染的影响；
  - 关键结论：规则 → 分组 → 节点，三层结构使得"换节点"和"改分流"完全分开。

## 配置示例

``` yaml
mixed-port: 7890
mode: rule
proxies:
  - name: "node-a"
    type: vless
    server: server.example.com
    port: 443
    uuid: 5783a3e7-e373-51cd-8642-c83782b807c5
    tls: true
    servername: example.com
proxy-groups:
  - name: "PROXY"
    type: select
    proxies: ["node-a", "DIRECT"]
rules:
  - DOMAIN-SUFFIX,example.cn,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

字段含义：

| 字段             | 含义                                                               |
| -------------- | ---------------------------------------------------------------- |
| `mixed-port`   | 本地混合端口，同时提供 HTTP 和 SOCKS 代理                                      |
| `mode`         | 运行模式：`rule`（按规则）、`global`（全局）、`direct`（直连）                       |
| `proxies`      | 节点列表；每个节点有 `name`、`type`（协议）和各自的参数                               |
| `proxy-groups` | 节点分组；`type` 是分组策略（`select`、`url-test`、`fallback`、`load-balance`） |
| `rules`        | 规则列表，格式 `类型,匹配内容,目标`；`MATCH` 是兜底规则，必须放在最后                        |

表中第一列是字段，第二列是含义。字段名随内核和版本不同略有差异，以所用内核的文档为准。**节点和订阅里包含服务器地址、UUID
或密码，不要公开。**

## mihomo 内核

mihomo 的功能（来自其 README）包括：

  - 本地 HTTP/HTTPS/SOCKS 服务，支持认证；
  - 协议支持：VMess、VLESS、Shadowsocks、Trojan、Snell、TUIC、Hysteria 等；
  - 内置 DNS 服务器，支持 DoH/DoT 上游和 fake-ip，尽量减少 DNS 污染的影响；
  - 按域名、GEOIP、IP-CIDR 或进程匹配规则，把流量分到不同节点；
  - 远程分组（Remote groups）支持自动回退、负载均衡、按延迟自动选择；
  - 远程 Provider：可以从远程获取节点列表，不必写死在配置里；
  - Linux 上可用 netfilter TCP 重定向，作为网关部署；
  - 完整的 HTTP RESTful API 控制器，以及配套的 Web 面板 metacubexd。

## 优缺点

| 优点                         | 缺点                                   |
| -------------------------- | ------------------------------------ |
| 规则、分组、节点三层结构清晰，改分流和换节点互不影响 | 配置是 YAML，节点多时文件很长，通常靠订阅和 Provider 管理 |
| 内置 DNS、fake-ip、TUN，功能完整    | 原 Clash 项目已停止维护，生态分散在多个分支            |
| 有 REST API 和 Web 面板，易于管理   | 不同分支之间字段、功能不完全一致                     |
| 与大量图形客户端集成                 | 不同于 V2Ray/Xray，配置格式不能互相直接转换          |

表中左边是优点，右边是缺点。

## Reference

  - [MetaCubeX/mihomo](https://github.com/MetaCubeX/mihomo)
  - [mihomo 文档](https://wiki.metacubex.one/)
  - [metacubexd 面板](https://github.com/MetaCubeX/metacubexd)
