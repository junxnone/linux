---
Title | Tools Networks Proxy Protocol MTProto
-- | --
Created @ | `2026-11-26T10:13:38Z`
Updated @| `2026-11-26T10:13:38Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/148)

---
# MTProto

MTProto 是 **Telegram 自己设计的通信协议**，Telegram 客户端和服务器之间用它传输消息；围绕它还有一个 **MTProxy（MTProto 代理）**，让客户端经由代理连接 Telegram 服务器。V2Ray 早期版本（v4 系列）里提供了一个名为 `mtproto` 的协议，用来实现这种代理，因此它出现在 V2Ray 的协议清单里。**需要注意：当前 V2Fly 和 Xray 的最新配置文档的协议列表里已经没有 `mtproto`，说明它在新版本里已被移除或不再维护**，使用前请先确认所用版本是否仍然支持。总览见 [Proxy](0080_Tools_Networks_Proxy)。

## 它是什么

- **MTProto（协议）**：Telegram 的加密协议，包括应用层的消息格式和传输层的封装，具体细节见官方文档。
- **MTProxy（代理）**：一个只为 Telegram 服务的代理，客户端（Telegram App）里可以直接填写代理地址、端口和 **secret（密钥）** 使用，不需要额外的代理软件。
- **V2Ray 的 mtproto 协议**：V2Ray v4 的入站/出站实现，需要配合路由规则，把来自 Telegram 的流量转到 `mtproto` 出站再送到 Telegram 服务器。

## 工作方式

下图展示 MTProxy 的基本链路：

```mermaid
flowchart LR
    A["Telegram 客户端<br/>填入代理地址 + secret"] -->|"混淆后的 MTProto 流"| P["MTProxy / V2Ray mtproto 入站"]
    P -->|"MTProto"| T["Telegram 服务器"]
```

图中：

- 客户端把代理地址、端口和 secret 填入 Telegram 的代理设置；
- 客户端与代理之间的流量是**混淆过的 MTProto 流**：传输层把数据用 AES-CTR 加密，密钥由握手载荷和 secret 派生，使流量看起来像随机字节；
- 代理再把流量转交给 Telegram 的服务器；
- 关键结论：这个代理只服务 Telegram 一个应用，secret 相当于访问代理的密码。

secret 的常见形态（MTProxy 的约定，具体以官方实现为准）：

secret 形式 | 含义
-- | --
16 字节的十六进制 | 基本的混淆传输
`dd` 前缀 | 安全模式，带随机填充，更抗长度分析
`ee` 前缀 + 域名 | "伪 TLS"模式，让握手看起来像连接到指定域名的 TLS

表中第一列是 secret 的格式，第二列是含义。这些是 MTProxy 的实现约定，不是 V2Ray 协议文档的一部分。

## V2Ray 里的用法（旧版本）

旧版 V2Ray 的做法是：增加一个 `mtproto` 入站，给 Telegram 客户端使用；再增加一个 `mtproto` 出站，并用路由规则把该入站的流量转到该出站。入站里用 `users` 列表配置每个用户的 secret。因为新版本的文档里已经没有这个协议，**本文不提供可直接使用的新版配置**，如果确实需要 Telegram 代理，建议使用官方 MTProxy 实现，或者让 Telegram 使用 SOCKS5 代理（Telegram 客户端原生支持 SOCKS5）并通过 V2Ray/Xray 的其他协议转发。

## 优缺点

优点 | 缺点
-- | --
Telegram 客户端原生支持，用户设置简单 | 只服务 Telegram，不能代理其他应用
secret 机制简单，无需额外账号体系 | V2Ray 新版本已不再支持，官方实现也要自行部署
传输层有混淆，伪 TLS 模式可以伪装握手 | 协议是 Telegram 私有的，文档和实现的演进由 Telegram 主导

表中左边是优点，右边是缺点。

## Reference

- [MTProxy - Telegram 官方实现](https://github.com/TelegramMessenger/MTProxy)
- [MTProto 协议 - Telegram 官方文档](https://core.telegram.org/mtproto)
- [V2Fly 配置文档 - 协议列表](https://github.com/v2fly/v2fly-github-io/tree/master/docs/en_US/config/protocols)

