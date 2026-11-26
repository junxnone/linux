-----

| Title     | Tools Networks Proxy V2Ray Nginx                     |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:22:21Z`                               |
| Updated @ | `2026-11-26T10:22:21Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/172) |

-----

# Nginx

把 V2Ray/Xray 放在 **Nginx 之后**是一种常见部署方式：Nginx 对外监听 443，负责 TLS
终结、托管一个正常的网站，并把特定路径的 WebSocket 或 gRPC
请求转发给本机的代理入站；除此之外的请求返回网站内容。这样服务器对外表现得就是一个普通的 Web
站点，同一个域名和端口上还能同时提供网站服务。工具介绍见
[V2Ray](0066_Tools_Networks_Proxy_V2Ray)，传输细节见 WebSocket、gRPC 文章。

## 工作方式

下图展示 Nginx 如何按路径把流量分给网站和代理：

``` mermaid
flowchart LR
    C["客户端 / 浏览器"] -->|"TLS 443"| N["Nginx (TLS 终结)"]
    N -->|"path = /ws-xxxx"| V1["V2Ray WebSocket 入站<br/>127.0.0.1:10000"]
    N -->|"path = /GunService/Tun (gRPC)"| V2["V2Ray gRPC 入站<br/>127.0.0.1:10001"]
    N -->|"其他路径"| W["正常网站内容"]
```

图中：

  - **TLS 在 Nginx 终结**：证书配在 Nginx 上，Nginx 与后端之间是本机的明文通信，所以后端入站里**不用再开
    TLS**；
  - **按路径分流**：约定好的随机路径转给 V2Ray 入站，其他请求给网站；
  - 关键结论：对访问者而言只有一个 HTTPS 站点；只有知道路径的客户端会走到代理。

## WebSocket 分流

``` nginx
server {
    listen 443 ssl http2;
    server_name example.com;
    ssl_certificate     /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    location /ws-your-random-path {
        proxy_pass http://127.0.0.1:10000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 300s;
    }

    location / {
        root /var/www/html;
        index index.html;
    }
}
```

要点：

  - `proxy_http_version 1.1` 以及 `Upgrade`、`Connection` 两个头是 WebSocket
    升级**必需**的；
  - `proxy_read_timeout` 调大，避免长时间没有数据的连接被 Nginx 切断（默认 60 秒）；
  - V2Ray 入站监听 `127.0.0.1:10000`，传输为 `ws`，`path` 与 `location` 一致；
  - 只监听本机地址，不要对公网开放后端端口。

## gRPC 分流

``` nginx
location /GunService/Tun {
    grpc_pass grpc://127.0.0.1:10001;
    grpc_read_timeout 300s;
    grpc_send_timeout 300s;
    client_body_timeout 300s;
}
```

其中路径必须是 `/${serviceName}/Tun`，`serviceName` 与 V2Ray gRPC
配置里的一致；`grpc_pass grpc://...` 表示用**明文 gRPC** 转发给后端（TLS 已在
Nginx 终结）；几个超时调大避免长连接被切断。

## 共享 443：按 SNI 分流（stream 模块）

如果同一台服务器上既有 Nginx 网站，又有需要自己终结 TLS 的服务（如 REALITY 或直接用 V2Ray 的 TLS），可以让
Nginx 的 **stream 模块**在 TCP 层按 SNI 分流，不解密：

``` nginx
stream {
    map $ssl_preread_server_name $backend {
        www.example.com   127.0.0.1:8443;   # 交给本机 Nginx https 站点
        default           127.0.0.1:9443;   # 其他 SNI 交给代理
    }
    server {
        listen 443;
        ssl_preread on;
        proxy_pass $backend;
    }
}
```

其中：

  - `ssl_preread on` 让 Nginx 只读取 TLS 握手里的 SNI，**不解密**；
  - `map` 按 SNI 选后端；
  - 缺点是后端看不到客户端真实 IP，可以用 `proxy_protocol on;` 配合后端的
    `acceptProxyProtocol`（见 TCP、WebSocket 文章）。

## 回落（fallback）与 Nginx

VLESS/Trojan 也可以反过来用：由代理自己监听 443 并终结 TLS，**认证失败的连接回落给 Nginx**（如
`127.0.0.1:80`）。两种架构的区别是：

| 方案                | TLS 由谁终结 | 特点                                        |
| ----------------- | -------- | ----------------------------------------- |
| Nginx 在前，代理在后     | Nginx    | 证书管理在 Nginx，适合同一域名上有多项服务；用路径分流            |
| 代理在前，Nginx 在后（回落） | 代理       | 可使用 VLESS Vision/REALITY 等高级特性，网站服务作为回落目标 |

表中第一列是方案，第二列是 TLS 终结位置，第三列是特点。使用 XTLS Vision 或 REALITY
时通常选"代理在前"，因为它们要接管 TLS 握手。

## 注意事项

  - Nginx 的 WebSocket、gRPC 配置的**路径、超时**必须与客户端一致；
  - 通过 CDN 访问时，CDN 的超时和协议支持（是否支持 WebSocket、gRPC）会影响可用性；
  - 证书自动续期后要 `nginx -s reload`（可用 certbot 的 `--deploy-hook`）；
  - 修改后用 `nginx -t` 检查语法，再 `systemctl reload nginx`；
  - 后端入站只监听 `127.0.0.1`，防火墙只开 443。

## 优缺点

| 优点                            | 缺点                                     |
| ----------------------------- | -------------------------------------- |
| 同一域名和端口同时提供网站和代理              | 多一层转发，增加少量延迟                           |
| 证书和 TLS 配置集中在 Nginx，可复用已有站点配置 | Nginx 终结 TLS，不能直接用 XTLS Vision/REALITY |
| 可以用 CDN、WAF 等成熟的 Web 基础设施     | 超时、路径等细节不一致会导致连接失败，排错需要同时看两边日志         |
| 便于做路径级访问控制和限速                 | 需要维护 Nginx 配置                          |

表中左边是优点，右边是缺点。

## Reference

  - [Nginx - WebSocket
    proxying](https://nginx.org/en/docs/http/websocket.html)
  - [Nginx -
    ngx\_http\_grpc\_module](https://nginx.org/en/docs/http/ngx_http_grpc_module.html)
  - [Nginx -
    ngx\_stream\_ssl\_preread\_module](https://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html)
