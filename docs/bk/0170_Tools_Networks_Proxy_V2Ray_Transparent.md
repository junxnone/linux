-----

| Title     | Tools Networks Proxy V2Ray Transparent               |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:22:17Z`                               |
| Updated @ | `2026-11-26T10:22:17Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/170) |

-----

# Transparent

透明代理（Transparent Proxy）指**应用程序不需要任何代理设置**，由操作系统或网关把它们的流量自动重定向给代理软件。对
Linux 来说，常见做法是用 **iptables/nftables** 把出站流量重定向到 V2Ray/Xray 的
`dokodemo-door` 入站，再由路由决定走代理还是直连。它适合给**不支持代理的程序**，或者整个局域网（把 Linux
机器当网关/旁路由）提供代理。基础概念见
[Proxy](0080_Tools_Networks_Proxy)，入站处理器见 DokodemoDoor 文章。

## 工作方式

下图展示透明代理里的流量路径，以及"为什么必须放行代理自己的流量"：

``` mermaid
flowchart LR
    A["应用 / 局域网设备<br/>(不知道有代理)"] --> N["netfilter: iptables / nftables<br/>REDIRECT 或 TPROXY"]
    N --> D["dokodemo-door :12345<br/>followRedirect, 还原原始目标"]
    D --> R{"routing"}
    R -->|"直连"| F["freedom"]
    R -->|"代理"| V["vmess / vless 出站"]
    V -->|"标记 mark=255 (跳过重定向)"| Net["网络"]
    F -->|"标记 mark=255"| Net
```

图中：

  - **netfilter 规则**把应用发出的 TCP/UDP 流量重定向给 dokodemo-door 监听的端口；
  - **dokodemo-door** 开启 `followRedirect`，取回流量**原本要去的地址**，交给路由；
  - 路由决定直连（`freedom`）还是走代理出站；
  - 出站自己发出的连接必须**带上特殊标记（`sockopt.mark`，如 255）**，并在 iptables
    规则里放行这个标记，否则它们又被重定向回来，形成**死循环**；
  - 关键结论：透明代理的难点不是转发，而是"哪些流量要重定向、哪些必须放行"。

## 两种重定向方式

| 方式               | 适用        | 说明                                                        |
| ---------------- | --------- | --------------------------------------------------------- |
| REDIRECT（nat 表）  | 仅 TCP     | 把目标端口改写到本地端口，dokodemo-door 用 `SO_ORIGINAL_DST` 取回原目标；配置简单 |
| TPROXY（mangle 表） | TCP 和 UDP | 不改写数据包，用策略路由把包交给本地套接字，保留原目标地址；要处理 UDP（如 DNS、游戏）必须用它       |

表中第一列是方式，第二列是适用协议，第三列是说明。官方示例的思路是：TCP 用 REDIRECT，UDP 用 TPROXY；需要
`CAP_NET_ADMIN` 权限，入站 `streamSettings.sockopt.tproxy` 要设为对应的值（`redirect`
或 `tproxy`）。

## 配置示例

入站（接收被重定向的流量）：

``` json
{
  "inbounds": [{
    "port": 12345,
    "protocol": "dokodemo-door",
    "settings": { "network": "tcp,udp", "followRedirect": true },
    "sniffing": { "enabled": true, "destOverride": ["http", "tls"] },
    "streamSettings": { "sockopt": { "tproxy": "tproxy" } }
  }],
  "outbounds": [
    { "protocol": "vmess", "settings": { "vnext": [] }, "streamSettings": { "sockopt": { "mark": 255 } } },
    { "protocol": "freedom", "tag": "direct", "streamSettings": { "sockopt": { "mark": 255 } } }
  ]
}
```

其中：

  - `followRedirect: true`：识别被重定向的数据并还原目标；
  - `sniffing`：从流量里嗅探域名，才能按域名分流（见 Sniffing 文章）；
  - 每个出站的 `sockopt.mark: 255`：给 V2Ray 自己发出的包打上标记，用来在 iptables 里放行。

TCP 重定向示例（本机出站 + 网关转发，思路取自官方文档）：

``` bash
iptables -t nat -N V2RAY
# 1. 放行代理服务器自己的地址，非常重要，避免回环
iptables -t nat -A V2RAY -d 123.123.123.123 -j RETURN
# 2. 放行内网和保留地址
iptables -t nat -A V2RAY -d 10.0.0.0/8 -j RETURN
iptables -t nat -A V2RAY -d 127.0.0.0/8 -j RETURN
iptables -t nat -A V2RAY -d 169.254.0.0/16 -j RETURN
iptables -t nat -A V2RAY -d 172.16.0.0/12 -j RETURN
iptables -t nat -A V2RAY -d 192.168.0.0/16 -j RETURN
# 3. 放行 V2Ray 自己发出的、带 mark 255 的流量
iptables -t nat -A V2RAY -m mark --mark 255 -j RETURN
# 4. 其余 TCP 重定向到 dokodemo-door
iptables -t nat -A V2RAY -p tcp -j REDIRECT --to-ports 12345
# 本机出站生效 (OUTPUT) 和作为网关转发的流量 (PREROUTING)
iptables -t nat -A OUTPUT -p tcp -j V2RAY
iptables -t nat -A PREROUTING -p tcp -j V2RAY
```

UDP（TPROXY）还需要策略路由，让被标记的包送到本机：

``` bash
ip rule add fwmark 1 lookup 100
ip route add local 0.0.0.0/0 dev lo table 100
iptables -t mangle -N V2RAY_UDP
iptables -t mangle -A V2RAY_UDP -d 192.168.0.0/16 -j RETURN
iptables -t mangle -A V2RAY_UDP -p udp -j TPROXY --on-port 12345 --tproxy-mark 1
iptables -t mangle -A PREROUTING -p udp -j V2RAY_UDP
```

要点：

  - `ip rule` 把带 `fwmark 1` 的包交给路由表 100，表 100 里的 `local 0.0.0.0/0 dev
    lo` 让所有目标都被当作"本机地址"接收；
  - 作为网关使用时需要开启 IP 转发：`sysctl -w net.ipv4.ip_forward=1`；
  - 本机自己的 UDP 流量（OUTPUT）要先打标记再走
    PREROUTING，更复杂，常见脚本里会单独处理，**这里只是思路示意，完整可用的规则请以官方文档和所用发行版为准**；
  - 新系统用 **nftables**，规则写法不同但思路一样（放行 → 重定向）。

## 注意事项

  - **放行代理服务器地址和带 mark 的流量**，否则死循环、断网；
  - 把 DNS 也纳入考虑：被污染的本地 DNS 会让按域名的分流失效，可结合 FakeDNS、嗅探（见 DNS、Sniffing 文章）；
  - 先在小范围测试：改完规则后，保留一个本地终端或定时任务用来恢复 iptables，避免把自己断在 SSH 之外；
  - 容器和旁路由场景要注意网络命名空间和转发。

## 优缺点

| 优点                      | 缺点                                      |
| ----------------------- | --------------------------------------- |
| 不支持代理的程序、整个局域网都能用       | 配置复杂，规则写错容易断网或回环                        |
| 对应用无感，不需要逐个设置代理         | 需要 root/`CAP_NET_ADMIN`，只适合 Linux 网关/本机 |
| 可同时处理 TCP 和 UDP（TPROXY） | 调试困难，需要熟悉 netfilter 和策略路由               |
| 可按域名、IP 分流              | 只还原目标地址，要按域名分流必须结合嗅探                    |

表中左边是优点，右边是缺点。

## Reference

  - [Dokodemo-door - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/en_US/config/protocols/dokodemo.md)
  - [Linux Kernel - TPROXY
    文档](https://www.kernel.org/doc/Documentation/networking/tproxy.txt)
  - [iptables TPROXY 说明 - man
    iptables-extensions](https://man7.org/linux/man-pages/man8/iptables-extensions.8.html)
