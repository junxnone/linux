-----

| Title     | Tools Networks Proxy Protocol Loopback               |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:13:49Z`                               |
| Updated @ | `2026-11-26T10:13:49Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/153) |

-----

# Loopback

Loopback（环回）是一个**出站协议**：它不把流量送到外部，而是**把流量重新送回路由模块再处理一次**，整个过程不离开核心程序。它是为了解决"某些位置只能选择出站、不能直接写路由规则或负载均衡器"这类限制而提供的变通手段。它不是网络协议，而是内置的出站处理器。Xray
的文档里有详细说明。总览见 [Proxy](0080_Tools_Networks_Proxy)。

## 工作方式

下图展示流量如何被 Loopback"绕回"路由：

``` mermaid
flowchart LR
    I["inbound (tag: in1)"] --> R1{"routing<br/>第一次分流"}
    R1 -->|"命中规则 A"| L["outbound: loopback<br/>inboundTag = 'again'"]
    L -->|"以 'again' 为入站 tag 重新进入"| R2{"routing<br/>第二次分流 (inboundTag: again)"}
    R2 -->|"TCP"| O1["outbound: proxy-tcp"]
    R2 -->|"UDP"| O2["outbound: proxy-udp"]
```

图中：

  - 流量先经第一次路由，命中规则 A 后被送到 `loopback` 出站；
  - Loopback 把这些数据当作一个**新的入站**，入站标识是它配置的 `inboundTag`（图中是
    `again`），重新进入路由；
  - 第二次路由里可以写 `inboundTag: ["again"]` 的规则，对这部分流量按更多条件继续细分（例如按 TCP/UDP
    走不同出站）；
  - 关键结论：Loopback 让"已经分流过一次的流量"可以再次进入路由，实现多级分流。

## 配置

``` json
{
  "protocol": "loopback",
  "tag": "to-again",
  "settings": {
    "inboundTag": "again",
    "sniffing": {}
  }
}
```

字段含义：

| 字段           | 含义                                             |
| ------------ | ---------------------------------------------- |
| `inboundTag` | 重新进入路由时使用的入站标识；路由里可以用 `inboundTag` 引用它，匹配这部分数据 |
| `sniffing`   | 流量探测，配置同入站的 `sniffing`，默认不开启                   |

表中第一列是字段，第二列是含义。

## 用途

1.  **间接使用负载均衡器**：有些地方（如链式代理里的 `dialerProxy`、负载均衡的
    `fallbackTag`）只能指定出站，不能直接写 `balancerTag`；借助
    Loopback 把流量送回路由，由路由规则再去选 balancer。
2.  **二次细分**：流量已经按某条规则分流后，还想按更多条件继续细分，比如同一组路由规则分出来的 TCP 和 UDP 流量要走不同的出站。

## 注意事项

  - **避免回环**：要让回环后的规则或 balancer 不要再选回原出站，否则会形成 `出站 → Loopback → 路由 →
    原出站` 的死循环。
  - 它只在核心内部转交流量，没有网络开销；但多一次路由匹配，规则复杂时会有少量 CPU 开销。

## 优缺点

| 优点                         | 缺点                             |
| -------------------------- | ------------------------------ |
| 不出核心，零网络开销，实现多级分流          | 配置易出错，容易写出死循环                  |
| 解决"只能选出站"的限制，间接用上 balancer | 逻辑不直观，排错需要对照路由规则和 tag          |
| 可以在回环时重新开启嗅探               | 新增的概念，各版本（v2fly、Xray）支持和字段略有差异 |

表中左边是优点，右边是缺点。普通的分流需求（直连、代理、屏蔽）用不到它，只有在多级分流、负载均衡这类进阶场景才考虑。

## Reference

  - [Loopback - Xray
    文档](https://github.com/XTLS/Xray-docs-next/blob/main/docs/config/outbounds/loopback.md)
