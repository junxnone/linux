-----

| Title     | Tools Networks Proxy Transport mKCP                  |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:16:31Z`                               |
| Updated @ | `2026-11-26T10:16:31Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/158) |

-----

# mKCP

mKCP 是 V2Ray 基于 **KCP 协议**实现的传输方式：它用 **UDP 来模拟 TCP
连接**，在应用层自己实现可靠传输、重传和拥塞处理。核心取舍是**牺牲带宽换取更低的延迟**——传输同样的内容，mKCP
一般比 TCP 消耗更多流量。它适合丢包率高、对延迟敏感的网络环境。使用时要确认主机防火墙放行对应的 UDP 端口。整体介绍见
Transport 总览。

## 为什么需要 mKCP

TCP 的可靠传输和拥塞控制是为"丢包 =
网络拥塞"的假设设计的：一旦丢包，发送窗口会大幅缩小，重传也有较长的超时。在**无线、跨境等丢包不一定是拥塞**的网络里，TCP
会把链路带宽用得很差。KCP 的做法是：

  - 更积极的重传：不用 TCP 那种指数退避，丢包后更快重发；
  - 更短的发送间隔：按固定的时间间隔（TTI）发送，而不是等待 ACK 触发；
  - 可选的拥塞控制：默认关闭，更"激进"地占用带宽。

代价就是额外的重复数据，所以流量开销变大。

## 工作方式

下图展示 mKCP 在整个传输栈中的位置：

``` mermaid
flowchart LR
    A["代理协议数据"] --> K["KCP: 分片 / 序号 / ACK / 重传"]
    K --> M["(可选) 头部伪装 + seed 混淆"]
    M --> U["UDP 数据报"]
    U --> N["网络"]
```

图中：

  - 代理协议的数据交给 KCP 处理，KCP 在 UDP 之上实现可靠、有序的字节流：给数据分片、编序号、确认、超时重传；
  - 之后可以加一层**头部伪装**（`header`）和用 `seed` 做的混淆；
  - 最后装进 UDP 数据报发送；
  - 关键结论：可靠性在 mKCP 内部完成，所以不依赖 TCP；传输的速度主要由上下行带宽参数决定。

## 配置

``` json
{
  "streamSettings": {
    "network": "kcp",
    "kcpSettings": {
      "mtu": 1350,
      "tti": 50,
      "uplinkCapacity": 5,
      "downlinkCapacity": 20,
      "congestion": false,
      "readBufferSize": 2,
      "writeBufferSize": 2,
      "header": { "type": "none" },
      "seed": "Password"
    }
  }
}
```

字段含义：

| 字段                                   | 含义                                            | 默认值   |
| ------------------------------------ | --------------------------------------------- | ----- |
| `mtu`                                | 最大传输单元，取值 576–1460                            | 1350  |
| `tti`                                | 传输时间间隔，单位 ms，取值 10–100，mKCP 按这个频率发数据          | 50    |
| `uplinkCapacity`                     | 上行链路容量，单位 **MB/s**（注意是 Byte 不是 bit），0 表示很小的带宽 | 5     |
| `downlinkCapacity`                   | 下行链路容量，单位 MB/s                                | 20    |
| `congestion`                         | 是否启用拥塞控制；开启后丢包严重时自动降速，网络畅通时适当提速               | false |
| `readBufferSize` / `writeBufferSize` | 单个连接的读/写缓冲区大小，单位 MB；高速传输时可适当调大                | 2     |
| `header.type`                        | 头部伪装类型，见下                                     | none  |
| `seed`                               | 可选的混淆密码，使用 AES-128-GCM 混淆流量，两端必须一致（v4.24.2+）  | 无     |

表中第一列是字段，第二列是含义，第三列是文档中的默认值。**上下行容量的含义**：客户端的 `uplinkCapacity`
决定客户端发送数据的速度，服务器的 `downlinkCapacity`
决定服务器发送数据的速度；要想让下行速度快，两端的
`downlinkCapacity` 都要设置得合适。

头部伪装 `header.type` 可选值：`none`（不伪装）、`srtp`（伪装成视频通话数据包）、`utp`（伪装成 uTP/BT
下载数据包）、`wechat-video`、`dtls`（伪装成 DTLS 1.2）、`wireguard`（只是外观，并不是真正的
WireGuard 协议）。这些只改变数据包头部的外观，两端必须一致，**不要把它们当作安全手段**。

## 优缺点

| 优点                      | 缺点                          |
| ----------------------- | --------------------------- |
| 延迟低，对丢包不敏感，在不稳定的网络上体验更好 | 流量开销大，同样内容比 TCP 消耗更多带宽      |
| 不依赖 TCP，没有 TCP 的队头阻塞    | 基于 UDP，部分网络会限制或丢弃 UDP       |
| 可调参数多，能针对网络调优           | 参数设置不当（带宽、TTI）会导致拥塞或浪费，需要实测 |
| 可选的 seed 混淆和头部伪装        | 不能被 Nginx、CDN 等 HTTP 网关转发   |

表中左边是优点，右边是缺点。**注意**：设置过大的带宽容量会造成大量重传，影响同一网络下的其他业务。

## Reference

  - [mKCP - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/config/transport/mkcp.md)
  - [mKCP 协议 - V2Fly
    开发者文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/developer/protocols/mkcp.md)
  - [KCP - A Fast and Reliable ARQ
    Protocol](https://github.com/skywind3000/kcp)
