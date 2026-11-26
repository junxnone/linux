-----

| Title     | Tools Networks Proxy sing-box                        |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:25:45Z`                               |
| Updated @ | `2026-11-26T10:25:45Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/176) |

-----

# sing-box

sing-box 是一个**通用代理平台**（官方的说法是 "The universal proxy platform"），由
nekohasekai 开发（SagerNet 组织）。和 V2Ray/Xray 一样是"入站 — 路由 —
出站"的结构，但它从设计上就面向"**多协议、多平台、统一内核**"：一个 JSON
配置里同时支持 VMess、VLESS、Trojan、Shadowsocks、Hysteria2、TUIC、WireGuard
等多种协议，并内置 TUN 虚拟网卡、规则集、DNS
规则。许多客户端（特别是移动端）都把它作为核心。许可证为
GPL-3.0 并带有"衍生作品不得使用该名称"的附加条款。总览见 [Proxy](0080_Tools_Networks_Proxy)。

## 结构

下图展示 sing-box 配置里几大模块的关系：

``` mermaid
flowchart LR
    I["inbounds<br/>tun / mixed / socks / http / vmess / vless ..."] --> R{"route<br/>规则 / rule_set / 嗅探 action"}
    R --> O["outbounds<br/>direct / block / vmess / vless / hysteria2 / selector / urltest ..."]
    D["dns<br/>服务器 + 规则"] -.-> R
    E["endpoints<br/>wireguard / tailscale ..."] -.-> R
```

图中：

  - **inbounds**：入站，常用有 `tun`（接管整机流量）、`mixed`（同一端口同时支持 SOCKS 和
    HTTP）以及各种代理协议的服务端；
  - **route**：路由规则，可以按域名、IP、协议、进程、规则集（`rule_set`）等匹配，决定使用哪个出站；较新版本把"嗅探"也作为路由里的
    `sniff` 动作；
  - **outbounds**：出站，`direct`（直连）、`block`（屏蔽）、各种代理协议，以及分组出站：`selector`（手动选择）、`urltest`（按延迟自动选择）；
  - **dns**：独立的 DNS 模块，有服务器和规则，可以按域名选择上游，支持 DoH、DoT、DoQ 等；
  - **endpoints**：较新版本里 WireGuard 等既能入站又能出站的类型归在这里；
  - 关键结论：和 V2Ray 的概念一一对应，但把 TUN、规则集、分组出站、DNS 规则都内置成一等公民。

## 配置示例

``` json
{
  "log": { "level": "info" },
  "inbounds": [
    { "type": "mixed", "tag": "mixed-in", "listen": "127.0.0.1", "listen_port": 1080 }
  ],
  "outbounds": [
    {
      "type": "vless",
      "tag": "proxy",
      "server": "server.example.com",
      "server_port": 443,
      "uuid": "5783a3e7-e373-51cd-8642-c83782b807c5",
      "flow": "xtls-rprx-vision",
      "tls": { "enabled": true, "server_name": "example.com" }
    },
    { "type": "direct", "tag": "direct" }
  ],
  "route": {
    "rules": [ { "ip_is_private": true, "outbound": "direct" } ],
    "final": "proxy"
  }
}
```

字段含义（与 V2Ray/Xray 的差异点）：

| 字段                | 含义                                               |
| ----------------- | ------------------------------------------------ |
| `inbounds[].type` | 入站类型，如 `mixed`、`tun`、`vless`；V2Ray 里叫 `protocol` |
| `outbounds[].tag` | 出站标识，被路由引用                                       |
| `route.rules`     | 路由规则列表，每条规则写匹配条件和 `outbound`（或 `action`）         |
| `route.final`     | 没有规则命中时使用的出站，相当于"默认出站"，不再依赖"第一个出站"               |
| `tls`             | 每个出站内嵌的 TLS 配置，而不是单独的 `streamSettings`           |

表中第一列是字段，第二列是含义。注意：**配置字段和结构会随版本调整**（如嗅探的写法、DNS
配置格式有过较大变化），不同版本的示例不能混用，以所用版本的官方文档为准。上面的示例只是说明结构。

## 特点

  - **协议覆盖广**：除 V2Ray 系协议外，还支持 Hysteria2、TUIC、WireGuard、Naive 等；
  - **TUN 内置**：不依赖外部的 iptables 脚本，也能接管整机流量；
  - **规则集（rule\_set）**：用二进制 `.srs` 或 JSON 规则集代替 geosite/geoip
    数据文件，可远程更新；
  - **分组出站**：`selector`、`urltest` 实现节点手动选择、自动测速选择；
  - **Clash API 兼容**：可以接入 Clash 系的面板；
  - **平台广**：Linux、Windows、macOS、Android、iOS 都有以它为核心的客户端。

## 优缺点

| 优点                           | 缺点                                   |
| ---------------------------- | ------------------------------------ |
| 单一内核覆盖多种协议，包括 Hysteria2、TUIC | 配置结构与 V2Ray/Xray 不同，需要重新学习           |
| 内置 TUN、规则集、DNS 规则，开箱完整       | 版本变化快，旧教程和配置容易过期                     |
| 移动端和桌面端客户端多，生态广              | Xray 独有的部分特性（如 XHTTP）不一定支持，具体以所用版本为准 |
| GPL-3.0 许可，开源                | 许可证对衍生作品的命名有限制                       |

表中左边是优点，右边是缺点。

## Reference

  - [SagerNet/sing-box](https://github.com/SagerNet/sing-box)
  - [sing-box 文档](https://sing-box.sagernet.org/)
  - [sing-box - Hysteria2
    说明](https://github.com/SagerNet/sing-box/blob/testing/docs/manual/proxy-protocol/hysteria2.md)
