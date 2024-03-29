-----

| Title     | Tools Networks Proxy V2Ray                          |
| --------- | --------------------------------------------------- |
| Created @ | `2019-07-03T15:39:39Z`                              |
| Updated @ | `2022-12-29T02:46:15Z`                              |
| Labels    | \`\`                                                |
| Edit @    | [here](https://github.com/junxnone/linux/issues/66) |

-----

# V2Ray

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

## Brief

![image](media/55db639821eb1bc7b3c7b85653e09dd599d1ad15.png)

## 支持的协议

  - Blackhole
  - Dokodemo-door
  - Freedom
  - HTTP
  - MTProto
  - Shadowsocks
  - Socks
  - VMess

## Install

### Windows 和 Mac OS 安装方式

通过上述方式下载的压缩包，解压之后可看到 v2ray 或 v2ray.exe。直接运行即可。

### Linux

**自动安装脚本**

    bash <(curl -L -s https://install.direct/go.sh)

> root下执行

## Docker Install & Setup

    docker pull v2fly/v2fly-core

> v2ray/official moved to v2fly/v2fly-core

**Server\&Client**

    $ sudo docker run -d --name v2ray -e TZ=Asia/Shanghai -v /etc/v2ray:/etc/v2ray -p 8888:8888 --restart always v2fly/v2fly-core run -c /etc/v2ray/config.json

**polipo 局域网代理** 同shadowsocks，端口更改为 v2ray 映射的端口即可

### Setup Config

  - **Server config**

> 更换xxxxx 为你的server ip/port/id

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

  - **Client Config**

<!-- end list -->

``` 

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
          "googeadsserving.cn",
          "baidu.com"
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
        "domain":["geosite:cn"]
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

## 路由

> 通过多个出口 `outbounds` 设定

  - freedom 无代理
  - blackhole 屏蔽

## History

  - Shadowsocks 是 clowwindy
    开发的自用的软件，开发的初衷只是为了让自己能够简单高效地科学上网，自己使用了很长一段时间后觉得不错才共享出来的。
  - V2Ray 是 clowwindy 被喝茶之后 V2Ray 项目组为表示抗议开发的，一开始就致力于让大家更好更快的科学上网。

## Diff with Shadowsocks

  - V2Ray 使用了新的自行研发的 VMess 协议，改正了 Shadowsocks 一些已有的缺点，更难被墙检测到
  - 网络性能更好
  - 更丰富的功能
      - mKCP
      - 动态端口
      - 路由功能
      - 传出代理 - 多重代理
      - 数据包伪装
      - WebSocket 协议
      - Mux:多路复用
