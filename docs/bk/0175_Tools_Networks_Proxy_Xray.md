-----

| Title     | Tools Networks Proxy Xray                            |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:25:43Z`                               |
| Updated @ | `2026-11-26T10:25:43Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/175) |

-----

# Xray

Xray（核心程序 **Xray-core**，属于 **Project X**）是从 V2Ray 分出来的一个网络工具平台，保持了与
V2Ray 相近的"入站 — 路由 — 出站"结构和配置思路，但在协议和传输上独立演进，加入了 **XTLS**（Vision
流控）、**REALITY**、**XHTTP** 等原创特性，并支持 VLESS 加密、后量子算法等新功能。它是目前使用
VLESS + REALITY 方案时最常见的核心。工具家族的整体关系见
[V2Ray](0066_Tools_Networks_Proxy_V2Ray)，总览见
[Proxy](0080_Tools_Networks_Proxy)。

## 与 V2Ray 的关系

下图展示这几个项目的关系：

``` mermaid
flowchart LR
    V["V2Ray (v2ray-core)<br/>原项目"] --> F["V2Fly (v2fly/v2ray-core)<br/>社区维护的 V2Ray"]
    V --> X["Xray (XTLS/Xray-core)<br/>Project X, 独立演进"]
    X --> X1["XTLS Vision / REALITY / XHTTP / VLESS Encryption"]
    F --> F1["v5 配置格式 / Trojan / Hysteria2 等"]
```

图中：

  - **V2Ray** 是原始项目；
  - **V2Fly** 是社区维护的 V2Ray 延续版本（文档站 v2fly.org）；
  - **Xray** 是另一条分支，由 Project X 团队演进，保持与 V2Ray v4 配置的大体兼容，但新增了大量独有特性；
  - 两条分支的配置字段**大体相似，但并不完全兼容**，新特性各自独有；
  - 关键结论：使用前要确认客户端和服务端用的是同一种核心，文档也要看对应项目的。

## 主要特性

| 特性                           | 说明                                                       |
| ---------------------------- | -------------------------------------------------------- |
| XTLS Vision                  | 对"TLS in TLS"做优化，见 XTLS 文章                               |
| REALITY                      | 借用真实站点的 TLS 握手外观，无需自己的域名证书，见 Reality 文章                  |
| VLESS                        | 轻量协议，见 VLESS 文章；支持可选的 VLESS Encryption（含后量子的 ML-KEM-768） |
| XHTTP                        | 基于 HTTP 的新传输，用 HTTP 请求/响应流承载数据，可经 CDN                    |
| Tunnel                       | 旧称 dokodemo-door，监听端口并转发                                 |
| Hysteria / WireGuard / TUN 等 | 较新版本新增的入站/出站和传输（以所用版本文档为准）                               |
| 路由、DNS、策略、API                | 沿用并扩展 V2Ray 的模块                                          |

表中第一列是特性，第二列是说明。Xray 的文档站在 xtls.github.io。

## 安装与使用

官方 README 给出的方式：

  - **Linux 脚本**：官方安装脚本 `XTLS/Xray-install`；
  - **Docker**：`ghcr.io/xtls/xray-core`（官方镜像）；
  - **面板**：社区有 Remnawave、3X-UI 等管理面板。

核心命令行（示意，以所用版本为准）：

``` bash
xray run -c /usr/local/etc/xray/config.json      # 运行
xray run -test -c /usr/local/etc/xray/config.json  # 检查配置
xray uuid                                         # 生成 UUID
xray x25519                                       # 生成 REALITY 密钥对
xray tls ping example.com                         # 检测 REALITY target
```

与 systemd 配合的写法见 V2Ray Systemd 文章，把 `ExecStart` 换成 `xray run -c ...` 即可。

## 与 V2Fly 的选择

| 对比                    | V2Fly (v2ray-core)  | Xray (Xray-core)               |
| --------------------- | ------------------- | ------------------------------ |
| XTLS Vision / REALITY | 没有                  | 有                              |
| VLESS                 | 支持                  | 支持，并有扩展（Encryption、Vision 流控等） |
| Trojan、Hysteria2      | v5 配置支持             | Trojan、Hysteria 等新版本支持         |
| 配置格式                  | v4 与 v5 两种格式，v5 更重构 | 延续 v4 风格，持续扩展                  |
| 文档                    | v2fly.org           | xtls.github.io                 |
| 社区                    | V2Fly 社区            | Project X 社区                   |

表中第一列是对比项，后两列是两个核心各自的情况。选择建议：需要 REALITY/XTLS Vision 选 Xray；偏好 V2Ray
原有生态和配置的选 V2Fly；追求更多协议（Hysteria2、TUIC）和统一平台可考虑 sing-box。

## 优缺点

| 优点                                     | 缺点                      |
| -------------------------------------- | ----------------------- |
| 独有的 XTLS Vision、REALITY、XHTTP，性能与外观优化多 | 与 V2Fly 不兼容，生态和文档需要分别对应 |
| 更新快，新特性多                               | 更新快也意味着字段、行为变化快，旧教程容易过期 |
| 保持 V2Ray 风格的配置，迁移成本较低                  | 配置项繁杂，新特性文档分散           |
| 社区面板和客户端支持广                            | 部分特性依赖客户端同样用 Xray 核心    |

表中左边是优点，右边是缺点。

## Reference

  - [XTLS/Xray-core](https://github.com/XTLS/Xray-core)
  - [Xray 文档站](https://xtls.github.io/)
  - [XTLS/Xray-install](https://github.com/XTLS/Xray-install)
  - [XTLS/REALITY](https://github.com/XTLS/REALITY)
