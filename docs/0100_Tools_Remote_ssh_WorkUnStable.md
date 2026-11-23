---
Title | Tools Remote ssh WorkUnStable
-- | --
Created @ | `2019-07-10T06:44:57Z`
Updated @| `2026-01-22T05:31:52Z`
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

### 场景1：临时修改MAC（重启网卡/系统后失效，快速测试）
1. 先**禁用目标网卡**（替换`ens33`为你的网卡名）：
   ```bash
   sudo ip link set ens33 down
   ```
2. 修改MAC地址（格式：`xx:xx:xx:xx:xx:xx`，注意前两位建议避开特殊值，如`00/02/06`）：
   ```bash
   # 示例：修改为00:11:22:33:44:55
   sudo ip link set ens33 address 00:11:22:33:44:55
   ```
3. 启用网卡：
   ```bash
   sudo ip link set ens33 up
   ```
4. 验证修改结果（查看`link/ether`后的值）：
   ```bash
   ip addr show ens33
   ```


### 场景2：永久修改MAC（Netplan配置，重启不失效，Ubuntu 18.04+推荐）
Netplan是Ubuntu新系统的默认网络配置工具，配置文件为`yaml`格式，步骤如下：
1. 进入Netplan配置目录，查看配置文件（文件名通常为`00-installer-config.yaml`/`01-netcfg.yaml`）：
   ```bash
   cd /etc/netplan
   ls
   ```
2. 编辑配置文件（用nano/vim，**YAML文件严格缩进，不能用Tab，只能用空格**）：
   ```bash
   sudo nano 00-installer-config.yaml
   ```
3. 在网卡配置中添加`macaddress`字段，示例配置（适配静态IP/DHCP，二选一）：
   #### 情况A：网卡为DHCP自动获取IP，仅改MAC
   ```yaml
   network:
     ethernets:
       ens33:  # 你的网卡名
         dhcp4: true  # DHCPv4开启
         macaddress: 00:11:22:33:44:55  # 要修改的MAC地址
     version: 2
     renderer: networkd  # 后端用networkd，桌面版可改为NetworkManager
   ```
   #### 情况B：网卡为静态IP，同时改IP+MAC（下文会详细讲静态IP配置）
   ```yaml
   network:
     ethernets:
       ens33:  # 你的网卡名
         dhcp4: false  # 关闭DHCP，手动设静态IP
         addresses: [192.168.1.100/24]  # 静态IP/子网掩码（/24=255.255.255.0）
         gateway4: 192.168.1.1  # 网关地址
         nameservers:
           addresses: [8.8.8.8, 114.114.114.114]  # DNS服务器
         macaddress: 00:11:22:33:44:55  # 要修改的MAC地址
     version: 2
     renderer: networkd
   ```
4. 保存退出（nano：`Ctrl+O`→回车→`Ctrl+X`），应用Netplan配置（生效并永久保留）：
   ```bash
   # 先测试配置文件语法是否正确（无报错再执行应用）
   sudo netplan try
   # 语法正确后，应用配置
   sudo netplan apply
   ```
5. 验证MAC修改结果：
   ```bash
   ip addr show ens33
   ```

> 补充：若为Ubuntu桌面版（带图形化网络管理），可将`renderer`改为`NetworkManager`，避免图形化配置与Netplan冲突。

