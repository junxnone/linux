-----

| Title     | Tools Networks Proxy Transport gRPC                  |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:16:29Z`                               |
| Updated @ | `2026-11-26T10:16:29Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/157) |

-----

# gRPC

gRPC 是 Google 开源的 RPC 框架，它使用 **HTTP/2** 作为传输，调用双方之间有**双向流**。V2Ray/Xray 的
gRPC 传输（v4.36.0+）借用了 gRPC 的"流式调用"形式来承载代理数据：外观上是一个 gRPC
服务的调用，**自带多路复用**，并且因为本质是 HTTP/2，可以被
Nginx、Caddy 转发，也能经过支持 gRPC 的 CDN。实现参考了 Qv2ray/gun 项目。整体介绍见 Transport
总览。

## 工作方式

下图展示 gRPC 传输把代理数据装进 gRPC 流的结构：

``` mermaid
flowchart LR
    A["代理协议数据<br/>(VMess / VLESS ...)"] --> G["gRPC 流式调用<br/>路径 /serviceName/Tun"]
    G --> H["HTTP/2 + TLS (ALPN h2)"]
    H --> N["Nginx / Caddy / CDN (可选)"]
    N --> S["服务端 gRPC 入站"]
```

图中：

  - 代理数据被放进一个 **gRPC 双向流**，调用路径由 `serviceName` 决定，形如
    `/${serviceName}/Tun`；
  - gRPC 本身用 HTTP/2 传输，通常加 TLS；
  - 中间的 Nginx、Caddy 或 CDN 可以按这个路径转发（Nginx 用 `grpc_pass`）；
  - 关键结论：它与 HTTP/2 传输同源，但**多路复用是内置的**，并且更容易被 CDN 接受，因为 gRPC 是通用的服务调用方式。

## 配置

``` json
{
  "streamSettings": {
    "network": "grpc",
    "security": "tls",
    "grpcSettings": { "serviceName": "GunService" }
  }
}
```

字段含义：

| 字段            | 含义                                                  |
| ------------- | --------------------------------------------------- |
| `serviceName` | gRPC 服务的名称，作用类似 `path`，用来防止被探测是否部署了该传输；建议使用复杂的随机字符串 |

表中第一列是字段，第二列是含义。根据 gRPC 官方规范，该字段不建议使用英文字母、数字、下划线和点号之外的字符。如果用 Nginx、Caddy
等做分流，分流路径应为 `/${serviceName}/Tun`。

Nginx 分流示例思路：

``` nginx
location /GunService/Tun {
    grpc_pass grpc://127.0.0.1:10000;
}
```

其中 `grpc_pass` 把该路径的 gRPC 流量转发给本机的 V2Ray gRPC 入站（`grpc://` 表示明文 gRPC，TLS
已在 Nginx 终结）。

## 优缺点

| 优点                              | 缺点                                   |
| ------------------------------- | ------------------------------------ |
| 内置多路复用，连接复用度高                   | 比 WebSocket 复杂，部分 CDN/网关的 gRPC 支持有限制 |
| 基于 HTTP/2，可被 Nginx/Caddy/CDN 转发 | 仍是 TCP，丢包时有队头阻塞                      |
| `serviceName` 路径可抗简单探测          | 需要 TLS 和 HTTP/2 支持，部署要求高             |
| 外观是标准的 gRPC 调用                  | 长时间的双向流可能被中间设备按空闲时间中断                |

表中左边是优点，右边是缺点。

## Reference

  - [gRPC over HTTP/2
    协议](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)
  - [gRPC - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/config/transport/grpc.md)
  - [Qv2ray/gun](https://github.com/Qv2ray/gun)
