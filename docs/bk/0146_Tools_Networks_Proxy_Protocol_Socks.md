-----

| Title     | Tools Networks Proxy Protocol Socks                  |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:13:33Z`                               |
| Updated @ | `2026-11-26T10:13:33Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/146) |

-----

# Socks

SOCKS 是一个**通用的代理协议**：它工作在比 HTTP
更低的层面，不关心上层是什么协议，只负责"帮我连到某个地址和端口，并把数据转发过去"，因此可以代理任意
TCP（SOCKS5 还能代理 UDP）。常用版本是 SOCKS4、SOCKS4a 和 SOCKS5，其中 SOCKS5 由 RFC 1928
定义，用户名/密码认证由 RFC 1929 定义。V2Ray 的 socks 协议兼容 Socks 4、4a 和
5，既能做入站（让本机应用连进来），也能做出站（经上游 SOCKS 代理转发）。总览见
[Proxy](0080_Tools_Networks_Proxy)，客户端用法见 `socks5://` 与 `socks5h://`
的区别（[Proxy](0080_Tools_Networks_Proxy)）。

## 版本对比

| 版本      | 特点                                                              |
| ------- | --------------------------------------------------------------- |
| SOCKS4  | 只支持 TCP CONNECT，目标必须是 IPv4 地址，只有简单的 USERID 字段，没有真正的认证           |
| SOCKS4a | SOCKS4 的扩展，允许把**域名**交给代理解析（把 IP 写成 `0.0.0.x` 并在后面附上域名）          |
| SOCKS5  | 支持 TCP CONNECT、BIND 和 UDP ASSOCIATE，地址可以是 IPv4、域名、IPv6，支持多种认证方式 |

表中第一列是版本，第二列是特点。现在基本都使用 SOCKS5，SOCKS4 只在老旧场景下出现。

## SOCKS5 的工作过程

下图展示 SOCKS5 建立一次 TCP 代理连接的三步：

``` mermaid
sequenceDiagram
    participant C as 客户端
    participant S as SOCKS5 代理
    participant T as 目标服务器
    C->>S: 1. 协商: VER=5, 支持的认证方法列表
    S-->>C: 选定的认证方法 (00 无认证 / 02 用户名密码)
    C->>S: 2. 认证 (若需要): 用户名 + 密码
    S-->>C: 认证结果
    C->>S: 3. 请求: CMD=CONNECT, 目标地址类型 + 地址 + 端口
    S->>T: 代理端建立到目标的连接
    S-->>C: 应答: REP=00 成功 + 绑定地址
    C->>T: 之后所有数据由代理双向转发
```

图中：

  - **第 1 步：协商**。客户端发送版本号（5）和自己支持的认证方法列表；服务器从中选一个返回。常见方法有
    `0x00`（无需认证）、`0x01`（GSSAPI）、`0x02`（用户名/密码），`0xFF`
    表示没有可接受的方法。
  - **第 2 步：认证**。若选了用户名/密码，按 RFC 1929
    发送版本（1）、用户名长度和用户名、密码长度和密码，**密码是明文传输**。
  - **第 3 步：请求**。客户端发送命令（`0x01` CONNECT、`0x02` BIND、`0x03` UDP
    ASSOCIATE）、地址类型（`0x01` IPv4、`0x03` 域名、`0x04` IPv6）、目标地址和端口。
  - **应答**包含状态码 `REP`：`0x00` 成功，`0x01` 一般失败，`0x02` 规则不允许，`0x03`
    网络不可达，`0x04` 主机不可达，`0x05` 连接被拒，`0x06` TTL 超时，`0x07`
    不支持的命令，`0x08` 不支持的地址类型。
  - 成功之后，代理在客户端与目标之间**透明地转发字节流**。
  - 关键结论：握手有两到三个往返，之后就是纯转发；协议本身没有任何加密，也没有对数据内容的认证。

## UDP ASSOCIATE

SOCKS5 通过 UDP ASSOCIATE 代理 UDP：客户端先用 TCP 连接发 `UDP ASSOCIATE` 请求，代理返回一个
UDP 中继地址；之后客户端把每个 UDP 数据报加上 SOCKS5 UDP 头后发给该地址，头里包含 `RSV(2 字节) | FRAG(1
字节) | ATYP | DST.ADDR | DST.PORT | DATA`。该 UDP 关联的生命周期绑定在那条 TCP
控制连接上，TCP 断开关联就结束。V2Ray 的文档提示：**SOCKS5 在使用 UDP
时，认证可能被绕过**，如果把 SOCKS5 代理共享给不可信网络，应配合防火墙使用。

## socks5 与 socks5h

在客户端工具（如 curl）里写代理地址时：

| 写法           | 域名由谁解析                             |
| ------------ | ---------------------------------- |
| `socks5://`  | 本地解析成 IP 后再让代理连接该 IP               |
| `socks5h://` | 把域名交给代理解析（SOCKS5 的域名地址类型），本地不做 DNS |

表中第一列是 URL 方案，第二列说明域名解析发生的位置。`socks5h` 可以避免 DNS 泄漏和本地解析不出的问题，是更常用的写法；`h`
表示 hostname。

## V2Ray 配置示例

入站（本机提供 SOCKS5，供应用连接）：

``` json
{
  "inbounds": [{
    "port": 1080,
    "listen": "127.0.0.1",
    "protocol": "socks",
    "settings": { "auth": "noauth", "udp": true }
  }]
}
```

出站（通过上游 SOCKS 代理转发）：

``` json
{
  "outbounds": [{
    "protocol": "socks",
    "settings": {
      "servers": [{ "address": "127.0.0.1", "port": 1234, "users": [{ "user": "test user", "pass": "test pass" }] }]
    }
  }]
}
```

其中：

  - `auth`：`noauth` 无认证，`password` 用户名密码认证（需要配 `accounts`）；
  - `udp`：是否允许 UDP；
  - 出站里 `servers` 是上游 SOCKS 服务器列表；官方文档提示 SOCKS
    出站自身**没有加密**，不适合直接经公网传输，更有意义的用法是在只能通过
    SOCKS 代理访问内网时，作为其他协议连接代理服务器的**前置代理**。

## 优缺点

| 优点                        | 缺点                 |
| ------------------------- | ------------------ |
| 通用，不关心上层协议，TCP/UDP 都能代理   | 没有加密，也不保证完整性       |
| 握手简单、实现多，几乎所有工具都支持        | 用户名密码认证是明文         |
| `socks5h` 可以把 DNS 解析交给代理端 | UDP 关联的认证存在可被绕过的风险 |
| 适合作为本地"接入口"，与加密协议组合       | 特征明显，不适合直接暴露到公网    |

表中左边是优点，右边是缺点。典型用法是：SOCKS5 只在本机回环地址上作为入口，真正跨网络的一段由
VMess、VLESS、Shadowsocks 等加密协议承担。

## Reference

  - [RFC 1928 - SOCKS Protocol
    Version 5](https://datatracker.ietf.org/doc/html/rfc1928)
  - [RFC 1929 - Username/Password Authentication for SOCKS
    V5](https://datatracker.ietf.org/doc/html/rfc1929)
  - [SOCKS4 协议 -
    socks4.protocol](http://ftp.icm.edu.pl/packages/socks/socks4/SOCKS4.protocol)
  - [Socks - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/config/protocols/socks.md)
