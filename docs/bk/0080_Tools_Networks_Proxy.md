-----

| Title     | Tools Networks Proxy                                |
| --------- | --------------------------------------------------- |
| Created @ | `2021-11-15T19:15:17Z`                              |
| Updated @ | `2026-11-26T10:27:57Z`                              |
| Labels    | \`\`                                                |
| Edit @    | [here](https://github.com/junxnone/linux/issues/80) |

-----

# Proxy

代理（Proxy）是客户端和目标服务器之间的"中间人"：客户端不直接连目标，而是把请求交给代理，由代理转发并把响应带回来。在 Linux
上使用代理的典型原因有两类：

  - **访问受限网络**：企业内网只能通过公司代理出网，或需要访问被屏蔽、速度很慢的外部资源（GitHub、软件源、镜像仓库等）。
  - **访问控制与审计**：由网关统一做认证、缓存、过滤和日志。

本文先讲代理的基本类型，再讲 Linux 下各类工具如何走代理，最后介绍几款常见的代理软件和排错方法。网络相关的其他内容见 [Tools
Networks](0060_Tools_Networks)。

## 代理的类型与协议

### 按位置分

| Name              | Description                              |
| ----------------- | ---------------------------------------- |
| 正向代理（Forward）     | 客户端主动配置，代理替客户端访问外部；本文讨论的主要是这种            |
| 反向代理（Reverse）     | 代理站在服务器一侧，替服务器接收请求（如 Nginx），客户端不感知       |
| 透明代理（Transparent） | 客户端无需配置，由网关用 iptables/nftables 把流量重定向到代理 |

表中第一列是代理类型，第二列说明谁配置、谁受益；正向代理需要应用自己支持代理设置，透明代理则对应用完全无感，适合不支持代理的程序。

### 按协议分

| Name        | Description                                                        |
| ----------- | ------------------------------------------------------------------ |
| HTTP 代理     | 对明文 HTTP 直接转发请求；对 HTTPS 用 `CONNECT` 方法让代理与目标建立 TCP 隧道，代理看不到 TLS 内容 |
| SOCKS5      | 工作在更底层，转发任意 TCP（以及 UDP），支持用户名密码认证，不关心上层协议                          |
| SOCKS4 / 4a | 老版本，只支持 TCP，无认证；4a 支持由代理解析域名                                       |

表中前两行是最常用的两种：HTTP 代理适合浏览器和各类命令行工具；SOCKS5 更通用。注意 `socks5://` 与
`socks5h://` 的区别：前者由**本地**解析域名，后者把域名交给**代理端**解析，能避免 DNS
泄漏或本地解析不出的问题，curl 等工具里常用 `socks5h`。

## 代理工作方式

下图展示本地代理软件的典型用法：应用把请求交给本机监听的代理端口，由代理软件加密后转发给远端服务器：

``` mermaid
flowchart LR
    A["支持代理的应用<br/>curl / git / apt"] -->|"读取 http_proxy 等配置"| L["本地代理<br/>127.0.0.1:1080 (SOCKS5)<br/>127.0.0.1:8080 (HTTP)"]
    B["不支持代理的应用"] -->|"proxychains 或透明代理"| L
    L -->|"加密隧道"| S["远端代理服务器<br/>shadowsocks / v2ray"]
    S --> T["目标网站"]
```

图中：

  - 左侧的两类应用：支持代理的应用通过环境变量或自身配置主动连接本地代理端口；不支持的应用要借助 proxychains 或
    iptables 透明代理"强制"改道。
  - 中间的"本地代理"是在本机监听端口的软件，常见端口 1080（SOCKS5）、8080（HTTP），具体以配置为准。
  - 到远端服务器的"加密隧道"：shadowsocks、v2ray 等软件在本地与远端之间用自己的协议加密传输。
  - 在企业内网里，这个"本地代理 + 远端服务器"通常直接是公司提供的 HTTP 代理，应用直连它，不需要自己再搭隧道。
  - 关键结论：先确认"本地代理在监听哪个端口、什么协议"，再让各个工具指向它。

## Linux 下配置代理

### 环境变量

大多数命令行工具（curl、wget、pip、apt 部分场景等）都读取这几个环境变量：

``` bash
export http_proxy=http://proxy.example.com:8080
export https_proxy=http://proxy.example.com:8080
export all_proxy=socks5h://127.0.0.1:1080
export no_proxy=localhost,127.0.0.1,.example.com
```

各变量的含义：

  - `http_proxy` / `https_proxy`：分别用于 HTTP、HTTPS 目标；值里的 `http://`
    指的是**代理本身的协议**，不是目标的协议，所以 HTTPS 目标通常也填 `http://代理地址`。
  - `all_proxy`：所有协议的兜底代理，常用来配 SOCKS。
  - `no_proxy`：逗号分隔的"不走代理"的主机/域名/网段，内网地址和 `localhost` 一定要加；不同工具对 CIDR（如
    `10.0.0.0/8`）和域名通配的支持不一致。
  - 大小写：有的程序读小写，有的读大写，两种都设最保险。
  - 带用户名密码时写作 `http://user:pass@host:port`；密码里的特殊字符需要 URL
    编码，且不要把含密码的配置提交到仓库。
  - `sudo` 默认会清掉环境变量，需要 `sudo -E`，或在 sudoers 里配置 `env_keep`。

取消：`unset http_proxy https_proxy all_proxy no_proxy`。

### 常用工具

下表汇总各工具的配置方式：

| 工具           | 配置方式                                                                                                                                                                                                      |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| curl         | 临时：`curl -x socks5h://127.0.0.1:1080 https://example.com`；永久：`~/.curlrc` 里写 `proxy = ...`                                                                                                                 |
| wget         | 读取环境变量，或 `~/.wgetrc` 里写 `http_proxy = ...`、`use_proxy = on`                                                                                                                                               |
| apt          | 新建 `/etc/apt/apt.conf.d/95proxy`，写入 `Acquire::http::Proxy "http://proxy.example.com:8080/";` 与 `Acquire::https::Proxy ...`；apt 以 root 运行，所以通常需要这种文件而不是用户的环境变量，更换软件源见 [APT Source](0119_Ubuntu_APT_Source) |
| git          | `git config --global http.proxy http://proxy.example.com:8080`；SOCKS 写 `socks5h://127.0.0.1:1080`；取消用 `git config --global --unset http.proxy`                                                            |
| git over ssh | 在 `~/.ssh/config` 里为目标主机设置 `ProxyCommand nc -X 5 -x 127.0.0.1:1080 %h %p`（`-X 5` 表示 SOCKS5；需要 OpenBSD 版 netcat）                                                                                           |
| pip          | `pip config set global.proxy http://proxy.example.com:8080`，或每次加 `--proxy`                                                                                                                                |
| npm          | `npm config set proxy ...`、`npm config set https-proxy ...`                                                                                                                                               |
| docker 拉取镜像  | 守护进程读取的是 systemd 单元的环境，见下面的单独说明                                                                                                                                                                           |

表中第一列是工具，第二列是配置位置和写法；其中 apt 和 docker 是以 root/守护进程运行，**不会继承你 shell
里的环境变量**，是最容易漏配的两个。

### Docker

Docker 有两处需要分别配置：

1.  **守护进程**（`docker pull` 走的是它）：新建
    `/etc/systemd/system/docker.service.d/http-proxy.conf`：

<!-- end list -->

``` ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080"
Environment="HTTPS_PROXY=http://proxy.example.com:8080"
Environment="NO_PROXY=localhost,127.0.0.1"
```

然后执行 `sudo systemctl daemon-reload && sudo systemctl restart docker`。

2.  **容器内部**（让容器里的进程也走代理）：编辑 `~/.docker/config.json`：

<!-- end list -->

``` json
{
  "proxies": {
    "default": {
      "httpProxy": "http://proxy.example.com:8080",
      "httpsProxy": "http://proxy.example.com:8080",
      "noProxy": "localhost,127.0.0.1"
    }
  }
}
```

注意容器里的 `127.0.0.1` 指的是容器自己，不是宿主机；要访问宿主机上的本地代理，需要用宿主机的真实 IP 或 docker
网桥地址，并让代理监听在该地址上。

### 让不支持代理的程序走代理

  - **proxychains / proxychains-ng**：用 `LD_PRELOAD` 劫持程序的 `connect()`
    调用，配置文件里写上代理（`/etc/proxychains.conf`），用法 `proxychains4 curl
    https://example.com`。对静态链接的程序和 setuid 程序无效。
  - **透明代理**：在网关或本机用 iptables / nftables 把出站流量重定向到 redsocks、tun2socks
    等工具，对应用完全无感；配置较复杂，要小心代理自身流量造成回环。
  - **协议转换**：Privoxy、Polipo 等可以把 SOCKS 代理转成 HTTP 代理，供只认 HTTP
    代理的程序使用，也常用来给局域网内其他机器共享出口。

## 代理软件

| Name                                                                                                        | Description                                                                                  |
| ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| [shadowsocks](0067_Tools_Networks_Proxy_shadowsocks)                                                        | 基于 SOCKS5 的加密代理，轻量，有 Python 版和 libev 版；本地端是 SOCKS5，服务端是加密中继                                  |
| [V2Ray](0066_Tools_Networks_Proxy_V2Ray)                                                                    | 支持多种协议（VMess、Shadowsocks、SOCKS、HTTP 等）和灵活的路由规则，可按域名或 IP 分流；社区维护版本为 v2fly，另有 Xray 等分支         |
| [dev-sidecar](0138_Tools_Networks_Proxy_dev_sidecar) \[[github](https://github.com/docmirror/dev-sidecar)\] | 面向开发者的本地加速工具，通过本地代理和 DNS/hosts 优化改善 GitHub 等开发资源的访问；HTTPS 加速依赖安装它生成的根证书，使用前应了解其对 HTTPS 流量的影响 |
| [Squid](0141_Tools_Networks_Proxy_Squid)                                                                    | 经典的 HTTP 缓存/正向代理服务，企业内网里常见，可做认证、缓存和访问控制                                                      |
| Nginx                                                                                                       | 常用作反向代理和负载均衡；配合模块也可做简单的正向代理                                                                  |
| [proxychains-ng](0140_Tools_Networks_Proxy_proxychains)                                                     | 命令行强制应用走代理的工具，见上文                                                                            |

表中第一列是软件名，第二列是它的定位和特点。选择建议：只想让命令行工具通过公司代理出网，不需要装任何代理软件，配置环境变量、apt、git、docker
即可；需要自己搭中继并按规则分流时再选 shadowsocks 或 V2Ray。

## 排错

  - **确认配置是否生效**：`env | grep -i proxy` 查看当前 shell；`git config --get
    http.proxy`、`apt-config dump | grep -i proxy` 查看对应工具。
  - **确认代理端口在监听**：`ss -lntp | grep 1080`。
  - **看一次完整的请求过程**：`curl -v -x http://proxy.example.com:8080
    https://example.com`，从输出里看是否连上了代理、是否收到 `407`（需要代理认证）、`CONNECT` 是否成功。
  - **某些工具仍然直连**：多半是它不读环境变量（如 apt、docker 守护进程）或程序是以
    root/服务方式运行；按上面的表单独配置。
  - **内网地址也走了代理**：检查 `no_proxy` 是否包含内网域名和网段。
  - **域名解析失败**：SOCKS 场景改用 `socks5h://`，让代理端解析。

## 使用须知

  - 只在符合所在网络的管理规定和当地法律法规的前提下使用代理工具；公司网络里不要私自搭建绕过管控的出口。
  - 代理的认证信息和订阅/节点配置属于敏感信息，不要写进脚本或提交到公开仓库。
  - 经过第三方代理的明文流量对代理是可见的，敏感数据应使用 HTTPS 等加密协议。

## 系列文章

**协议与组成部分**

  - 总览：[Protocol](0182_Tools_Networks_Proxy_Protocol)、[Transport](0181_Tools_Networks_Proxy_Transport)、[Comparison](0180_Tools_Networks_Proxy_Comparison)
  - 代理协议：[VMess](0142_Tools_Networks_Proxy_Protocol_VMess)、[VLESS](0143_Tools_Networks_Proxy_Protocol_VLESS)、[Shadowsocks](0144_Tools_Networks_Proxy_Protocol_Shadowsocks)、[Trojan](0145_Tools_Networks_Proxy_Protocol_Trojan)、[Socks](0146_Tools_Networks_Proxy_Protocol_Socks)、[HTTP](0147_Tools_Networks_Proxy_Protocol_HTTP)、[MTProto](0148_Tools_Networks_Proxy_Protocol_MTProto)、[WireGuard](0152_Tools_Networks_Proxy_Protocol_WireGuard)
  - 内置处理器：[Freedom](0149_Tools_Networks_Proxy_Protocol_Freedom)、[Blackhole](0150_Tools_Networks_Proxy_Protocol_Blackhole)、[DokodemoDoor](0151_Tools_Networks_Proxy_Protocol_DokodemoDoor)、[Loopback](0153_Tools_Networks_Proxy_Protocol_Loopback)
  - 传输：[TCP](0154_Tools_Networks_Proxy_Transport_TCP)、[WebSocket](0155_Tools_Networks_Proxy_Transport_WebSocket)、[HTTP2](0156_Tools_Networks_Proxy_Transport_HTTP2)、[gRPC](0157_Tools_Networks_Proxy_Transport_gRPC)、[mKCP](0158_Tools_Networks_Proxy_Transport_mKCP)、[QUIC](0159_Tools_Networks_Proxy_Transport_QUIC)、[DomainSocket](0160_Tools_Networks_Proxy_Transport_DomainSocket)
  - 安全层：[TLS](0161_Tools_Networks_Proxy_Security_TLS)、[XTLS](0162_Tools_Networks_Proxy_Security_XTLS)、[Reality](0163_Tools_Networks_Proxy_Security_Reality)

**V2Ray 功能模块与部署**

  - 功能模块：[Routing](0164_Tools_Networks_Proxy_V2Ray_Routing)、[DNS](0165_Tools_Networks_Proxy_V2Ray_DNS)、[Sniffing](0166_Tools_Networks_Proxy_V2Ray_Sniffing)、[Mux](0167_Tools_Networks_Proxy_V2Ray_Mux)、[Policy](0168_Tools_Networks_Proxy_V2Ray_Policy)、[API](0169_Tools_Networks_Proxy_V2Ray_API)
  - 部署与使用：[Transparent](0170_Tools_Networks_Proxy_V2Ray_Transparent)、[Systemd](0171_Tools_Networks_Proxy_V2Ray_Systemd)、[Nginx](0172_Tools_Networks_Proxy_V2Ray_Nginx)、[Client](0173_Tools_Networks_Proxy_V2Ray_Client)、[Troubleshooting](0174_Tools_Networks_Proxy_V2Ray_Troubleshooting)

**工具**

  - [shadowsocks](0067_Tools_Networks_Proxy_shadowsocks)、[V2Ray](0066_Tools_Networks_Proxy_V2Ray)、[Xray](0175_Tools_Networks_Proxy_Xray)、[sing-box](0176_Tools_Networks_Proxy_sing_box)、[Clash](0177_Tools_Networks_Proxy_Clash)、[Hysteria](0178_Tools_Networks_Proxy_Hysteria)、[TUIC](0179_Tools_Networks_Proxy_TUIC)
  - [Privoxy](0139_Tools_Networks_Proxy_Privoxy)、[proxychains](0140_Tools_Networks_Proxy_proxychains)、[Squid](0141_Tools_Networks_Proxy_Squid)、[dev-sidecar](0138_Tools_Networks_Proxy_dev_sidecar)

## Reference

  - [dev-sidecar - gitee](https://gitee.com/docmirror/dev-sidecar)
  - [dev-sidecar - github](https://github.com/docmirror/dev-sidecar)
  - [shadowsocks - Github](https://github.com/shadowsocks/shadowsocks)
  - [V2Fly](https://www.v2fly.org/)
  - [Docker - Configure the daemon to use a
    proxy](https://docs.docker.com/engine/daemon/proxy/)
  - [Docker - Use a proxy server with the Docker
    CLI](https://docs.docker.com/engine/cli/proxy/)
  - [curl -
    Proxies](https://everything.curl.dev/usingcurl/proxies/index.html)
  - [Git -
    http.proxy](https://git-scm.com/docs/git-config#Documentation/git-config.txt-httpproxy)
  - [proxychains-ng - Github](https://github.com/rofl0r/proxychains-ng)
  - [Squid](http://www.squid-cache.org/)
