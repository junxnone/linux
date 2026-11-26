-----

| Title     | Tools Networks Proxy V2Ray                          |
| --------- | --------------------------------------------------- |
| Created @ | `2019-07-03T15:39:39Z`                              |
| Updated @ | `2026-11-26T09:10:18Z`                              |
| Labels    | \`\`                                                |
| Edit @    | [here](https://github.com/junxnone/linux/issues/66) |

-----

# V2Ray

V2Ray（核心程序 v2ray-core，项目组 Project
V）是一个**模块化的代理/网络工具平台**：它不是单一协议，而是把"入站（inbound）—
路由（routing）— 出站（outbound）"拆成可组合的模块。你可以让它在本机同时监听 SOCKS 和 HTTP 端口，按域名/IP
规则决定每个请求是直连、走远端代理还是直接屏蔽，传输层还可以选 TCP、mKCP、WebSocket、HTTP/2 等。与
[Shadowsocks](0067_Tools_Networks_Proxy_shadowsocks)
相比，它功能更丰富、配置也更复杂。上层概览见
[Proxy](0080_Tools_Networks_Proxy)。

目前社区维护的版本是 **v2fly**（`v2fly/v2ray-core`，文档站 v2fly.org），原 v2ray.com
的文档和脚本已不再更新；另有 Xray 等衍生项目，配置思路相近但并不完全兼容，使用前要确认自己用的是哪一个。

## 基本概念

下图展示 V2Ray 处理一次请求的模块化流程，也是理解配置文件结构的关键：

``` mermaid
flowchart LR
    I1["inbound: socks<br/>127.0.0.1:1080"] --> R{"routing<br/>按规则选出口"}
    I2["inbound: http<br/>127.0.0.1:8080"] --> R
    R -->|"国内域名 / 私有 IP"| O1["outbound: freedom<br/>(直连, tag: direct)"]
    R -->|"广告域名"| O2["outbound: blackhole<br/>(丢弃, tag: adblock)"]
    R -->|"其他 (默认)"| O3["outbound: vmess<br/>(经远端服务器)"]
    O3 --> S["远端 V2Ray 服务端<br/>inbound: vmess"]
    S --> F["outbound: freedom"]
```

图中：

  - **inbound（入站）**：本机对外暴露的接口，例如监听本机的 SOCKS5 或 HTTP 端口，让应用把流量交过来；服务端的
    inbound 则是接收来自客户端的 VMess 连接。
  - **routing（路由）**：按域名、IP 等条件把每个连接分到不同的出口。没有命中规则时用**第一个** outbound
    作为默认出口。
  - **outbound（出站）**：真正把流量送出去的方式：`freedom` 直连（不经代理）、`blackhole`
    丢弃（用来屏蔽）、`vmess`/`shadowsocks`/`socks` 等经远端转发。
  - **服务端**：接收 VMess 的 inbound，再用 `freedom` 出站去访问目标。
  - 关键结论：客户端靠"路由 + 多出口"实现分流（国内直连、国外走代理、广告屏蔽），这是 V2Ray 相对 Shadowsocks
    最实用的差异。

## 支持的协议

  - Blackhole
  - Dokodemo-door
  - Freedom
  - HTTP
  - MTProto
  - Shadowsocks
  - Socks
  - VMess

下表说明常见协议的作用：

| 协议            | 作用                                                           |
| ------------- | ------------------------------------------------------------ |
| VMess         | V2Ray 自研的加密传输协议，客户端到服务端的主要协议；要求客户端与服务端系统时间相差不超过约 90 秒，否则连接失败 |
| Shadowsocks   | 兼容 Shadowsocks 协议，可与现有 SS 服务端互通                              |
| Socks / HTTP  | 作为 inbound 供本机应用连接，也可作为 outbound 经上游代理转发                     |
| Freedom       | 直连，不经代理，用于国内流量或服务端出口                                         |
| Blackhole     | 丢弃流量，用于广告屏蔽                                                  |
| Dokodemo-door | "任意门"，接收被重定向的任意 TCP/UDP 流量，用于透明代理                            |
| MTProto       | 为 Telegram 设计的代理协议                                           |

表中第一列是协议名，第二列是它在配置里承担的角色。

## Install

### Windows 和 Mac OS 安装方式

通过上述方式下载的压缩包，解压之后可看到 v2ray 或 v2ray.exe。直接运行即可。

### Linux

**自动安装脚本**

    bash <(curl -L -s https://install.direct/go.sh)

> root下执行

> 注意：`install.direct` 是旧版 v2ray.com 的脚本地址，已经不再维护；现在推荐使用 v2fly
> 社区提供的安装脚本（fhs-install-v2ray），在其 GitHub
> 仓库的说明里可以找到最新的命令，安装后默认配置文件位于
> `/usr/local/etc/v2ray/config.json`，服务名为 `v2ray`，用 `systemctl enable
> --now v2ray` 启动。

**手动安装**：从 release 页面下载对应架构的压缩包，解压得到 `v2ray` 可执行文件以及
`geoip.dat`、`geosite.dat`（下文路由里的 `geoip:cn`、`geosite:cn`
就来自这两个文件，需要与可执行文件放在同一目录，或设置环境变量指定资源目录）。

## Docker Install & Setup

    docker pull v2fly/v2fly-core

> v2ray/official moved to v2fly/v2fly-core

**Server\&Client**

    $ sudo docker run -d --name v2ray -e TZ=Asia/Shanghai -v /etc/v2ray:/etc/v2ray -p 8888:8888 --restart always v2fly/v2fly-core run -c /etc/v2ray/config.json

参数含义：`-v /etc/v2ray:/etc/v2ray` 把宿主机的配置目录挂载进容器；`-p 8888:8888`
映射端口，**容器端口要与配置里 inbound 的 `port` 一致**；`--restart
always` 让容器随 Docker 自动重启；`run -c` 指定配置文件。

**polipo 局域网代理** 同shadowsocks，端口更改为 v2ray 映射的端口即可

> Polipo 已停止维护，更推荐用 Privoxy 把本地 SOCKS5 转成 HTTP 代理，供局域网内其他机器使用，例如在
> `/etc/privoxy/config` 里设置：
> 
>     listen-address  0.0.0.0:8118
>     forward-socks5t / 127.0.0.1:1080 .

> `forward-socks5t` 表示通过 SOCKS5 转发并把域名交给代理端解析；`0.0.0.0`
> 表示对局域网开放，要注意防火墙与访问控制，避免暴露给公网。

### Setup Config

> 更换 xxxxx 为你的 server ip/port/id

配置文件是 JSON，顶层字段有 `log`、`inbounds`、`outbounds`、`routing` 等：

| 字段         | 含义                                                           |
| ---------- | ------------------------------------------------------------ |
| log        | 日志级别和日志文件路径                                                  |
| inbounds   | 入站列表：监听端口、协议、认证                                              |
| outbounds  | 出站列表：出口协议和目标服务器；**第一个**为默认出口，`tag` 用于被路由规则引用                 |
| routing    | 路由规则，决定每个请求走哪个出口                                             |
| `id`（UUID） | VMess 的用户标识，相当于账号，客户端与服务端必须一致，可用 `uuidgen` 或 `v2ray uuid` 生成 |

表中第一列是配置字段，第二列是含义。

  - **Server config**

<!-- end list -->

``` json
{
  "log": {
    "loglevel": "warning",
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log"
  },
  "inbounds": [
    {
      "port": xxxxx,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "xxxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "alterId": 64
          }
        ]
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    }
  ]
}
```

  - **Client Config**

<!-- end list -->

``` json
{
  "log": {
    "loglevel": "warning",
    "access": "/etc/v2ray/access.log",
    "error": "/etc/v2ray/error.log"
  },
  "inbounds": [
    {
      "port": 1080,
      "protocol": "http",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "auth": "noauth"
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "xxx.xxx.xxx.xxx",
            "port": xxxxx,
            "users": [
              {
                "id": "xxxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
                "alterId": 64
              }
            ]
          }
        ]
      }
    },
    {
      "protocol": "freedom",
      "settings": {},
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "adblock"
    }
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "domain": [
          "tanx.com",
          "googleadsserving.cn"
        ],
        "type": "field",
        "outboundTag": "adblock"
      },
      {
        "domain": [
          "amazon.com",
          "microsoft.com",
          "jd.com",
          "youku.com",
          "baidu.com"
        ],
        "type": "field",
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "outboundTag": "direct",
        "domain": ["geosite:cn"]
      },
      {
        "type": "field",
        "outboundTag": "direct",
        "ip": [
          "geoip:cn",
          "geoip:private"
        ]
      }
    ]
  }
}
```

对原笔记里的配置做了几处修正，使用时请留意：

  - 原来的客户端配置缺少最外层的 `{`，已补上。
  - 广告屏蔽列表里原本同时包含 `baidu.com`，而它又在直连列表里；路由规则是**按顺序匹配、先匹配先生效**，所以
    `baidu.com` 会被屏蔽，这显然不是本意，已从屏蔽列表里去掉。`googeadsserving.cn` 是笔误，应为
    `googleadsserving.cn`。
  - `alterId`：旧版 VMess 用它抵御探测；新版本（v4.28 以后）已使用 AEAD 认证，推荐设为 `0`，保留 64
    只是兼容旧配置。服务端与客户端需保持一致的设置。

路由字段含义：

| 字段                    | 含义                                                                                                              |
| --------------------- | --------------------------------------------------------------------------------------------------------------- |
| `domainStrategy`      | 域名解析策略：`AsIs` 不解析，只按域名匹配；`IPIfNonMatch` 域名规则都没命中时再解析成 IP 匹配 IP 规则；`IPOnDemand` 遇到 IP 规则就立刻解析域名，匹配更准但会多一次 DNS 查询 |
| `rules[].type`        | 固定为 `field`                                                                                                     |
| `rules[].domain`      | 域名匹配条件，可以写具体域名，或 `geosite:cn` 这样的预定义列表                                                                          |
| `rules[].ip`          | IP 匹配条件，`geoip:cn` 表示中国大陆 IP 段，`geoip:private` 表示私有地址（10.x、192.168.x 等）                                         |
| `rules[].outboundTag` | 命中后走哪个出站（对应 outbound 的 `tag`）                                                                                   |

表中第一列是路由字段，第二列是它的含义。这套规则的整体效果是：广告域名屏蔽，国内域名和私有地址直连，其余流量默认走第一个出站（vmess）。

## 路由

> 通过多个出口 `outbounds` 设定

  - freedom 无代理
  - blackhole 屏蔽

路由的粒度很灵活：可以按域名、IP、端口、来源、协议嗅探结果（`sniffing`）分流，也可以按入站 `tag`
区分不同的本地端口。需要的是在客户端里把 geo
数据文件（`geoip.dat`、`geosite.dat`）保持更新，否则分流结果会逐渐不准。

## 传输层与进阶功能

V2Ray 的 outbound/inbound 都可以通过 `streamSettings` 选择传输方式：

| 传输          | 特点                                        |
| ----------- | ----------------------------------------- |
| TCP         | 默认，最简单，可配合 TLS                            |
| mKCP        | 基于 UDP 的可靠传输，丢包环境下延迟更低，但流量开销更大            |
| WebSocket   | 把流量封装在 WebSocket 里，常配合 TLS 和 Nginx 反向代理使用 |
| HTTP/2、QUIC | 使用对应协议承载，利于与普通 Web 流量混在一起                 |

表中第一列是传输方式，第二列是特点。此外还有：

  - **Mux 多路复用**：把多个连接复用到同一条底层连接上，减少握手次数，但并不总能提升速度，高并发下建议实测。
  - **动态端口**：服务端定期更换端口。
  - **多重代理**：一个出站可以通过另一个出站再转发（`proxySettings`），形成链式代理。

## History

  - Shadowsocks 是 clowwindy
    开发的自用软件，开发的初衷是为了让自己能够简单高效地访问受限的网络资源，自己使用了很长一段时间后觉得不错才共享出来的；后来原作者停止维护，由社区接手出现了
    shadowsocks-libev、shadowsocks-rust 等实现。
  - V2Ray 是 Project V 团队独立开发的项目，从一开始就设计为一个可扩展的平台，自研了 VMess 协议。

## Diff with Shadowsocks

| 对比项   | Shadowsocks       | V2Ray                            |
| ----- | ----------------- | -------------------------------- |
| 定位    | 单一的加密代理协议         | 模块化平台，协议、路由、传输可自由组合              |
| 主协议   | Shadowsocks（AEAD） | VMess（也支持 Shadowsocks、Socks 等）   |
| 路由分流  | 基本没有，需要靠外部工具      | 内置，按域名/IP 分流                     |
| 传输方式  | TCP/UDP，插件扩展      | TCP、mKCP、WebSocket、HTTP/2、QUIC 等 |
| 配置复杂度 | 简单，几个字段           | 复杂，JSON 结构多                      |
| 资源占用  | 低                 | 相对较高                             |

表中第一列是对比项，后两列分别是两者的情况。原笔记里提到的 V2Ray 优点——自研的 VMess 协议改进了 Shadowsocks
早期版本的一些缺点、网络性能更好、功能更丰富（mKCP、动态端口、路由功能、传出代理多重代理、数据包伪装、WebSocket、Mux
多路复用）——主要是相对**早期 Shadowsocks 流加密方式**而言。现在 Shadowsocks 已普遍使用 AEAD 并有 2022
版协议，两者在安全性上的差距已经缩小，选哪个主要看是否需要内置路由和多种传输：只需要一个简单的加密中继选
Shadowsocks，需要分流和更多传输方式选 V2Ray。

## 使用须知

  - 只在符合所在网络管理规定和当地法律法规的前提下使用。
  - 配置里的 UUID、服务器地址属于敏感信息，不要提交到公开仓库。
  - 服务端和客户端的系统时间要同步（开启 NTP），VMess 对时间偏差敏感。

## Reference

  - [v2ray](https://www.v2ray.com/)
  - [v2ray - release](https://github.com/v2ray/v2ray-core/releases)
  - [V2Ray 跟 Shadowsocks
    有什么区别](http://blog.whiterabbitxyj.com/2018/08/31/V2Ray/)
  - [V2Ray,新一代科学上网神器](https://percysu.com/2018/08/10/V2Ray/)
  - [V2Ray安装使用教程 -
    月光博客](https://www.williamlong.info/archives/5724.html)
  - [Docker 部署
    V2Ray](https://toutyrater.github.io/app/docker-deploy-v2ray.html)
  - [配置生成器](https://intmainreturn0.com/v2ray-config-gen/)
  - [Config -
    rules](https://github.com/PaPerseller/chn-iplist/blob/master/v2ray-config_rule.json)
  - [V2Fly 文档](https://www.v2fly.org/)
  - [v2fly/v2ray-core - Github](https://github.com/v2fly/v2ray-core)
  - [v2fly/fhs-install-v2ray -
    Github](https://github.com/v2fly/fhs-install-v2ray)
