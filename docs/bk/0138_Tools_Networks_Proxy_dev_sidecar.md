-----

| Title     | Tools Networks Proxy dev-sidecar                     |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T09:26:07Z`                               |
| Updated @ | `2026-11-26T09:26:07Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/138) |

-----

# dev-sidecar

dev-sidecar（简称 ds，"开发者边车"）是一个面向开发者的**本地加速工具**。名字取自 service mesh 里的
sidecar（边车）：它不替代你的网络，而是在本机起一个代理，把开发者常用的 HTTPS 站点（GitHub、Stack
Overflow、npm、Hugging Face、Docker Hub 等）的请求改走更快的路径。和
[shadowsocks](0067_Tools_Networks_Proxy_shadowsocks)、[V2Ray](0066_Tools_Networks_Proxy_V2Ray)
这类需要自己准备远端服务器的工具不同，ds 是开箱即用的桌面应用，不需要服务端。代理的整体概念见
[Proxy](0080_Tools_Networks_Proxy)。

> 官方仓库只有 GitHub 上的
> [docmirror/dev-sidecar](https://github.com/docmirror/dev-sidecar)。README
> 中说明 Gitee 上的同步仓库已被封禁，请只认 GitHub 官方地址，不要从来源不明的镜像下载安装包。

## 工作方式

下图展示 ds 如何插入到"浏览器/命令行 → 目标网站"之间：

``` mermaid
flowchart LR
    A["浏览器 / git / npm"] -->|"系统代理"| P["ds 本地代理<br/>127.0.0.1:31181"]
    P --> D["DNS 优选<br/>挑选访问更快的 IP"]
    P --> I["请求拦截<br/>(默认模式: 需要根证书)"]
    D --> T["目标站点"]
    I -->|"改走镜像 / 加速通道"| T
```

图中：

  - **系统代理**：ds 启动后会自动修改操作系统的代理设置，指向本机的
    `127.0.0.1:31181`，所以走系统代理的浏览器等程序会自动经过它；命令行工具要读环境变量或自身配置（见
    [Proxy](0080_Tools_Networks_Proxy)）。
  - **DNS 优选**：对配置的域名，从指定的 DNS 获取多个 IP，测速后选最快的，类似手动改 hosts 但自动化。
  - **请求拦截**：默认模式下 ds 会解密 HTTPS
    流量，按规则把请求重定向、代理到镜像站点，或修改握手参数；因为要解密，必须在系统和浏览器里信任
    ds 自己生成的根证书。
  - 关键结论：ds 能做的越多（拦截、改写），对流量的介入就越深，也就越依赖对根证书的信任。

## 特性

| 特性                | 说明                                                                |
| ----------------- | ----------------------------------------------------------------- |
| DNS 优选            | 根据网络状况选择最佳的域名解析 IP，缓解部分站点慢或打不开；对 IP 本身被封的站点无效                     |
| 请求拦截              | 把打不开的站点代理到加速镜像，可配多个备份站点，访问失败或超时会自动切换                              |
| GitHub 加速         | 加速页面、release/source/zip 下载、clone、头像、raw/blame、gist 等；部分功能借助社区油猴脚本 |
| Stack Overflow 加速 | 把依赖的公共 CDN 请求代理到加速 CDN                                            |
| npm 加速            | 可开启 npm 代理，并在官方与国内 registry 间一键切换                                 |

表中第一列是特性名，第二列是做了什么。注意它的 README 明确说明：**只访问 HTTPS 时默认生效，如要加速仅 HTTP
的站点，需要在"系统代理"里手动打开"代理 HTTP 请求"**。

## 两种模式

| 模式   | 做了什么                          | 需要根证书 | 适用                                           |
| ---- | ----------------------------- | ----- | -------------------------------------------- |
| 安全模式 | 关闭拦截和增强，不使用远程配置，只保留 DNS 优选和测速 | 否     | 最安全；效果相当于自动帮你找最快的 IP，类似改 hosts，能否访问取决于 IP 质量 |
| 默认模式 | 开启拦截，使用远程配置，开启 DNS 优选和测速      | 是     | 功能完整，需要信任根证书                                 |

表中第一列是模式，第二列是行为，第三列说明是否需要安装根证书，最后一列是适用场景。对安全敏感的环境建议只用安全模式。

## Linux 安装

官方只保证 **Ubuntu x86\_64 GNOME 桌面版**，其他发行版未测试。

1.  到 [GitHub
    Release](https://github.com/docmirror/dev-sidecar/releases) 下载对应架构的
    deb 包（`DevSidecar-x.x.x-linux-<架构>.deb`），安装失败时换 `universal` 版本。
2.  使用 root 安装：`sudo dpkg -i DevSidecar-x.x.x.deb`，然后在应用列表里打开
    dev-sidecar。
3.  非 Debian 系的 glibc 发行版可用 AppImage：`chmod +x
    DevSidecar-x.x.x.AppImage` 后运行（官方标注未充分测试）。
4.  Alpine 等特殊发行版建议在 Debian 容器（如 distrobox）里运行，并把 gsettings 透传到宿主机，细节见官方
    `doc/linux.md`。

自动安装证书功能需要免密 sudo（官方文档提到"需要开启 sudo 免密支持，否则请自行安装证书"）。

## 根证书

默认模式要求系统信任 ds 生成的根证书：

  - 第一次打开时按提示点击"安装根证书"。证书是**本地随机生成**的，文件位于 `~/.dev-sidecar` 目录（如
    `dev-sidecar.ca.crt`）。
  - **Linux 上 Firefox 和 Chrome 不走系统证书**，需要在浏览器里单独导入：Firefox 在"设置 → 隐私与安全
    → 证书 → 管理证书 → 证书颁发机构 → 导入"，选择上面的证书文件并勾选信任网站标识。
  - 如果浏览器提示证书不受信任，一般是证书安装位置不对，重新安装后重启浏览器。

信任这张根证书意味着持有它私钥的程序能解密你的 HTTPS
流量。证书是本机生成的，私钥不外传，但仍应只在自己信任的机器上使用，不要导入来源不明的根证书，也不要使用来源不明的远程配置地址（README
对此有安全警告）。

## 配置

在"加速服务 → 拦截设置"里用 JSON 配置拦截规则，没有配置域名的不会被拦截：

``` json
{
  "github.com": {
    "/.*/.*/releases/download/": {
      "redirect": "download.fastgit.org"
    },
    ".*": {
      "proxy": "github.com"
    }
  },
  "clients*.google.com": {
    ".*": { "abort": true }
  }
}
```

这段配置的含义：

  - 第一层键是要拦截的**域名**；第二层键是匹配 URL 的**正则**。
  - `redirect`：临时重定向到另一个地址（URL 会变），适合下载资源。
  - `proxy`：代理到指定地址（URL 不变，没有跨域问题）。
  - `abort`：直接取消请求，适合确定访问不了又没有替代的资源，快速失败节省时间。
  - 还有 `cacheDays`（缓存天数）、`backup`（备用站点）、`sni`（修改握手 SNI）等，完整说明见官方
    wiki。上面例子里的 `download.fastgit.org` 来自旧
    README，对应的镜像服务可能已失效，使用前请换成当前可用的。

DNS 优选则在 `dns.mapping` 里为域名指定使用哪个 DNS，例如 `"api.github.com":
"cloudflare"`，目前只支持 IPv4 解析。

## 常见问题

  - **前两个开关打不开 / 没有加速效果**：先看是否同时开了别的**端口型代理软件**（ds 不能与它们同时使用；hosts 模式或
    TUN 网卡模式的软件通常可以共存），再检查浏览器是否在使用系统代理、有没有冲突的浏览器插件。
  - **打开 GitHub 提示"代理请求超时"**：看测速界面里 github.com 是否有可用 IP；没有 IP
    时，可能是网络提供商封锁了 DNS 服务商的 IP，可以 ping 1.1.1.1 /
    9.9.9.9 验证。
  - **重启后无法上网**：旧版本的 ds 在开着时重启电脑会遗留系统代理设置，手动打开 ds 再关闭即可恢复；官方称 1.8.9
    版本已解决。
  - **想只在命令行里加速 git**：让 git 走 ds 的本地代理，例如 `git config --global
    http.proxy http://127.0.0.1:31181`，不用时用 `--unset` 取消；具体写法见
    [Proxy](0080_Tools_Networks_Proxy)。

## 使用须知

  - 本应用启动会自动修改系统代理，与其他代理软件会冲突；已经有稳定代理出口的话，通常没有必要再用 ds。
  - 只在符合所在网络管理规定和当地法律法规的前提下使用；公司网络里使用前应确认是否允许安装并信任第三方根证书。
  - 只从官方仓库获取安装包。

## Reference

  - [dev-sidecar - github](https://github.com/docmirror/dev-sidecar)
  - [dev-sidecar Linux
    支持文档](https://github.com/docmirror/dev-sidecar/blob/master/doc/linux.md)
  - [dev-sidecar Wiki -
    加速服务使用说明](https://github.com/docmirror/dev-sidecar/wiki/%E5%8A%A0%E9%80%9F%E6%9C%8D%E5%8A%A1%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)
  - [dev-sidecar
    Release](https://github.com/docmirror/dev-sidecar/releases)
