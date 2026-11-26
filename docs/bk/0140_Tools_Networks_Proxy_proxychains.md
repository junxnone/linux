-----

| Title     | Tools Networks Proxy proxychains                     |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T09:26:11Z`                               |
| Updated @ | `2026-11-26T09:26:11Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/140) |

-----

# proxychains

proxychains（现在通常用的是活跃维护的 **proxychains-ng**，命令是
`proxychains4`）是一个**强制命令行程序走代理**的工具。很多程序没有代理设置项，也不读
`http_proxy` 之类的环境变量；proxychains
不改程序本身，而是在程序启动时预加载一个共享库，**劫持它的网络连接调用**，让所有
TCP 连接都改走你指定的一条或多条代理链。支持的代理类型有 HTTP、SOCKS4、SOCKS5。代理的整体概念和各工具的配置方式见
[Proxy](0080_Tools_Networks_Proxy)。

## 工作原理

下图展示 proxychains 如何在不修改程序的情况下改变它的连接去向：

``` mermaid
flowchart LR
    A["proxychains4 curl ..."] -->|"设置 LD_PRELOAD"| B["libproxychains4.so<br/>被预加载进程序"]
    B -->|"劫持 connect() / getaddrinfo()"| C["代理链<br/>socks5 → http → ..."]
    C --> D["目标服务器"]
```

图中：

  - **`proxychains4 curl ...`**：用它启动目标程序，它先设置环境变量 `LD_PRELOAD` 再启动程序；
  - **`libproxychains4.so`**：被预加载到目标进程里，替换进程调用的
    `connect()`、`getaddrinfo()` 等函数；
  - **代理链**：被劫持的连接不再直连目标，而是按配置依次经过一个或多个代理；
  - 关键结论：它依赖"动态链接 + 预加载"，所以对**静态链接的程序、直接发系统调用的程序（如 Go 语言写的程序）、setuid
    程序**无效；只代理 TCP，UDP 和 ICMP（如 `ping`）不行。

## Install

``` bash
sudo apt install proxychains4       # Debian / Ubuntu
```

源码安装（需要 gcc、make）：

``` bash
git clone https://github.com/rofl0r/proxychains-ng.git
cd proxychains-ng
./configure --prefix=/usr --sysconfdir=/etc
make && sudo make install
sudo make install-config            # 安装默认配置 /etc/proxychains.conf
```

## 配置

配置文件按以下顺序查找：环境变量 `PROXYCHAINS_CONF_FILE` 指定的文件、当前目录的
`./proxychains.conf`、`~/.proxychains/proxychains.conf`、`/etc/proxychains.conf`（不同发行版的包可能放在
`/etc/proxychains4.conf`，以实际为准）。

最小的配置示例：

    strict_chain
    proxy_dns
    tcp_read_time_out 15000
    tcp_connect_time_out 8000
    
    [ProxyList]
    socks5  127.0.0.1 1080

各部分含义：

| 项                                                 | 含义                                             |
| ------------------------------------------------- | ---------------------------------------------- |
| `strict_chain` / `dynamic_chain` / `random_chain` | 代理链策略，见下表                                      |
| `proxy_dns`                                       | 让 DNS 查询也走代理（由代理端解析域名），避免 DNS 泄漏或本地解析失败，通常应该开启 |
| `tcp_read_time_out` / `tcp_connect_time_out`      | 读取和连接的超时时间，单位毫秒                                |
| `[ProxyList]`                                     | 代理列表，每行一个：`类型 地址 端口 [用户名 密码]`                  |

表中第一列是配置项，第二列是含义。代理类型可以是 `http`、`socks4`、`socks5`；需要认证时在后面加用户名和密码。

三种代理链策略的区别：

| 策略              | 行为                                 | 适用           |
| --------------- | ---------------------------------- | ------------ |
| `strict_chain`  | 严格按列表顺序依次经过**所有**代理，任何一个不可用都会失败    | 需要固定的多级代理路径  |
| `dynamic_chain` | 依次经过列表中**可用**的代理，不可用的自动跳过（至少有一个可用） | 代理列表里有不稳定的节点 |
| `random_chain`  | 从列表中随机选取代理组成链（可用 `chain_len` 指定长度） | 需要随机化出口      |

表中第一列是策略名，第二列是它怎么选择代理，第三列是适用场景。三者只能启用一个，其余注释掉。只有一个代理时，`strict_chain`
最简单。

## 使用

``` bash
proxychains4 curl https://example.com        # 让 curl 走代理链
proxychains4 -q git clone https://github.com/xxx/yyy.git   # -q 静默模式，不输出 proxychains 自己的日志
proxychains4 -f ./my.conf ssh user@host       # -f 指定配置文件
proxychains4 bash                             # 开一个子 shell，之后在里面运行的命令（动态链接的）都会走代理
```

正常情况下，输出里会看到类似
`|S-chain|-<>-127.0.0.1:1080-<><>-example.com:443-<><>-OK`
的日志，说明连接经过了该代理并成功；出现 `timeout` 或 `denied`
则说明代理不可达或拒绝。

## 常见问题

  - **没有生效，程序仍然直连**：多半是程序是静态链接或不走 libc（如 Go 程序）。用 `ldd $(which 程序)`
    检查是否是动态链接；Go 程序可以改用它自己的代理环境变量，或者使用透明代理（iptables + redsocks
    等）。
  - **`proxychains4: command not found`**：包名/命令名在不同发行版不同，Debian/Ubuntu
    的包叫 `proxychains4`，命令也是 `proxychains4`。
  - **`sudo proxychains4 ...` 不生效**：`sudo` 会清理 `LD_PRELOAD` 等环境，需要写成
    `sudo proxychains4 命令`（即把 proxychains4 放在 sudo 之后），或使用 `sudo -E`。
  - **DNS 解析失败**：确认配置了 `proxy_dns`，并且代理（如 SOCKS5）支持由代理端解析域名。
  - **ping 不通**：ping 用的是 ICMP，不走 TCP，proxychains 无法代理。
  - **速度很慢**：检查是不是使用了过长的 `strict_chain`，或超时设置过大。

## 与其他方式对比

| 方式                                    | 特点                                     |
| ------------------------------------- | -------------------------------------- |
| 环境变量（`http_proxy` 等）                  | 程序要自己支持，最简单，粒度是整个 shell                |
| proxychains                           | 对不支持代理的动态链接程序有效，按命令使用，不需要 root         |
| 透明代理（iptables + redsocks / tun2socks） | 对所有程序无感，包括静态链接程序，需要 root，配置复杂          |
| Privoxy                               | 把 SOCKS5 转成 HTTP 代理，给"只支持 HTTP 代理"的程序用 |

表中第一列是方式，第二列是特点；选择建议：程序支持代理就直接配环境变量，不支持时再用 proxychains。

## 使用须知

  - 只在符合所在网络管理规定和当地法律法规的前提下使用。
  - 代理链上的任一节点都能看到经过它的明文流量，敏感数据请使用 HTTPS/SSH 等加密协议。

## Reference

  - [proxychains-ng - Github](https://github.com/rofl0r/proxychains-ng)
  - [proxychains-ng
    README](https://github.com/rofl0r/proxychains-ng/blob/master/README)
  - [proxychains.conf
    示例](https://github.com/rofl0r/proxychains-ng/blob/master/src/proxychains.conf)
