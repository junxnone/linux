---
Title | Tools Remote ssh WorkUnStable
-- | --
Created @ | `2019-07-10T06:44:57Z`
Updated @| `2024-04-06T04:36:40Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/100)

---
# SSH 连接不稳定

- ssh 不稳定，经常断开
- 有时候会连接不上
- mac地址 ： `88:88:88:88:87:88`

## Root Cause
- 网卡 firmware 的mac 地址不对
- 网络中有mac 地址相同的device，被分配为同一IP

## Solution

`System Setting -> Network -> Options -> Ethernet -> Cloned MAC address`

> 此项更改为一个新的mac地址（和本地网络中device无重复）
> 重启。


