-----

| Title     | Tools Networks Proxy V2Ray Policy                    |
| --------- | ---------------------------------------------------- |
| Created @ | `2026-11-26T10:20:13Z`                               |
| Updated @ | `2026-11-26T10:20:13Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/168) |

-----

# Policy

Policy（本地策略）是 V2Ray/Xray 里管理**连接的超时、缓冲区和流量统计**的配置。V2Ray
处理的每一个连接都对应一个**用户**，每个用户有一个**等级（level）**，策略按等级定义；此外还有针对整个系统的策略。入站和出站里可以设置用户等级，V2Ray
会按实际等级应用相应的策略。工具介绍见 [V2Ray](0066_Tools_Networks_Proxy_V2Ray)。

## 工作方式

下图展示一个连接如何找到它适用的策略：

``` mermaid
flowchart LR
    C["连接<br/>(入站/出站里的 level)"] --> L{"策略 levels 里<br/>找对应等级"}
    L -->|"找到 level 0"| P["套用该等级的超时、缓冲、统计"]
    L -->|"没有定义"| D["使用默认值"]
    Sys["系统级 system 策略"] --> S["入站/出站流量统计"]
```

图中：

  - 每个连接带有一个用户等级（入站和出站配置里的 `level` 或 `userLevel`，也可以在用户信息里指定）；
  - `policy.levels` 里按等级定义策略，连接据此取得超时、缓冲和统计设置；
  - `policy.system` 是整个系统的策略（目前主要是入站、出站的流量统计开关）；
  - 关键结论：等级是一种"策略分组"，不同来源的连接可以套用不同的超时和统计。

## 配置

``` json
{
  "policy": {
    "levels": {
      "0": {
        "handshake": 4,
        "connIdle": 300,
        "uplinkOnly": 2,
        "downlinkOnly": 5,
        "statsUserUplink": false,
        "statsUserDownlink": false,
        "bufferSize": 10240
      }
    },
    "system": {
      "statsInboundUplink": false,
      "statsInboundDownlink": false,
      "statsOutboundUplink": false,
      "statsOutboundDownlink": false
    }
  }
}
```

`levels` 里每个等级的字段含义：

| 字段                                      | 含义                                             | 默认值   |
| --------------------------------------- | ---------------------------------------------- | ----- |
| `handshake`                             | 连接握手的时限（秒）；入站在握手阶段（如 VMess 读取头部、确定目标地址）超过时限就断开 | 4     |
| `connIdle`                              | 连接空闲的时限（秒）；这段时间内上下行都没有数据就断开                    | 300   |
| `uplinkOnly`                            | 下行连接关闭后，等待多少秒再终止连接（秒）                          | 2     |
| `downlinkOnly`                          | 上行连接关闭后，等待多少秒再终止连接（秒）                          | 5     |
| `statsUserUplink` / `statsUserDownlink` | 是否启用该等级所有用户的上/下行流量统计                           | false |
| `bufferSize`                            | 每个连接的内部缓冲区大小，单位 kB；设为 0 表示禁用内部缓冲               | 因平台而异 |

表中第一列是字段，第二列是含义，第三列是文档里的默认值。`bufferSize` 的默认值（V2Ray
4.4+）：ARM、MIPS、MIPSLE 平台为 0；ARM64、MIPS64、MIPS64LE 为 4；其他平台为
512。`levels` 的键是**字符串形式的数字**（JSON 要求），引号不能省略。

文档提示：HTTP 浏览场景下，可以把 `uplinkOnly` 和 `downlinkOnly` 都设为 0，以提高连接关闭的效率。

`system` 里的四个开关分别控制入站/出站的上行和下行流量统计，为 `true` 时统计数据才会产生，供 API 的
StatsService 查询（见 API 文章）。

## 使用建议

  - 一般个人使用保持默认即可；需要给**不同用户或入口**设置不同的超时、统计时，才用多个等级。
  - 内存很小的设备（如路由器）可以调小 `bufferSize` 减少内存占用，代价是吞吐可能降低。
  - 想统计每个用户的流量，需要同时：给用户设置 `email`、把
    `statsUserUplink`/`statsUserDownlink` 打开，再配置 `stats` 与 `api`。
  - `connIdle` 太小会让长时间无数据的连接（如 SSH 空闲、长连接推送）被意外断开，太大则会占用资源。

## 优缺点

| 优点                  | 缺点                 |
| ------------------- | ------------------ |
| 可以按用户等级精细控制超时和缓冲    | 配置项多，默认值因平台而异，容易误改 |
| 流量统计需要通过它启用，便于计费和审计 | 统计只在启用后才产生，历史数据不能补 |
| 适合内存受限的设备调整缓冲       | 超时设置不当会导致长连接被误断开   |

表中左边是优点，右边是缺点。

## Reference

  - [Policy - V2Fly
    配置文档](https://github.com/v2fly/v2fly-github-io/blob/master/docs/en_US/config/policy.md)
  - [Policy - Xray
    文档](https://github.com/XTLS/Xray-docs-next/blob/main/docs/config/policy.md)
