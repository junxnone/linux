-----

| Title     | Tools Networks Proxy Protocol Blackhole              |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:13:42Z`                               |
| Updated @ | `2026-11-26T10:13:42Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/150) |

-----

# Blackhole

Blackhole（黑洞）是 V2Ray/Xray
的**出站协议**，作用是**阻止所有数据出站**：收到要转发的数据后直接丢弃并关闭连接。它和路由配合，用来"禁止访问某些站点"，最典型的用途是**屏蔽广告和跟踪域名**。和
Freedom 一样，它是内置的出站处理器，不是网络协议。总览见 [Proxy](0080_Tools_Networks_Proxy)。

## 工作方式

下图展示 Blackhole 如何配合路由屏蔽广告域名：

``` mermaid
flowchart LR
    I["inbound: socks / http"] --> R{"routing"}
    R -->|"广告域名 (tag: adblock)"| B["outbound: blackhole<br/>丢弃并关闭连接"]
    R -->|"国内域名 / 私有 IP"| F["outbound: freedom (direct)"]
    R -->|"其他"| V["outbound: vmess / vless"]
```

图中：

  - 路由根据规则把请求分流到三类出口；命中广告域名规则的连接被送进 Blackhole，数据被丢弃，连接关闭，应用会立刻得到连接失败的结果；
  - 因此浏览器加载页面时，这些广告资源会"秒失败"，不会拖慢页面，也不会真正访问广告服务器；
  - 关键结论：Blackhole 只负责"丢"，要不要丢由路由规则决定。

## 配置

``` json
{
  "protocol": "blackhole",
  "tag": "adblock",
  "settings": {
    "response": { "type": "none" }
  }
}
```

配置项含义：

| `response.type` | 行为                        |
| --------------- | ------------------------- |
| `none`（默认）      | 直接关闭连接，不返回任何数据            |
| `http`          | 先回一个简单的 HTTP 403 响应，再关闭连接 |

表中第一列是取值，第二列是行为。不配置 `response` 时与 `none` 相同。用 `http`
对浏览器更友好，会立刻显示错误而不是等待超时。

配合的路由规则示例：

``` json
{
  "type": "field",
  "domain": ["geosite:category-ads-all"],
  "outboundTag": "adblock"
}
```

其中 `outboundTag` 与上面出站的 `tag` 对应。`geosite:category-ads-all`
是预定义的广告域名列表（来自 geosite
数据文件，列表名以所用的数据文件为准）。路由规则**按顺序匹配**，所以屏蔽规则要放在直连、代理规则前面，否则会被先匹配的规则放行。

## 优缺点

| 优点                          | 缺点                                                                                |
| --------------------------- | --------------------------------------------------------------------------------- |
| 零成本屏蔽，不需要额外软件               | 只能按域名/IP/端口等维度屏蔽，无法按页面内容过滤（对比 [Privoxy](0139_Tools_Networks_Proxy_Privoxy) 的内容过滤） |
| 与路由规则配合灵活                   | 对 HTTPS 只能按域名屏蔽，无法区分同一域名下的不同路径                                                    |
| `response: http` 能给出明确的失败反馈 | 屏蔽列表需要维护，误屏蔽会导致站点异常（如同域名的正常资源）                                                    |

表中左边是优点，右边是缺点。

## Reference

  - [Blackhole - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/en_US/config/protocols/blackhole.md)
  - [Routing - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/en_US/config/routing.md)
