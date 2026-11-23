---
Title | Tools OS KernelUpgrade
-- | --
Created @ | `2018-09-28T06:29:41Z`
Updated @| `2025-05-29T08:55:45Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/73)

---
# Kernel 升级管理
- mainline
- ukuu - `开始收费了??不再支持了?`


## mainline

在Ubuntu 24.10上安装Linux内核6.12，可以使用Mainline工具来进行安装。具体步骤如下：
1. 添加Mainline的软件源：在终端中输入命令`sudo add-apt-repository ppa:cappelikan/ppa -y`。
2. 更新软件源索引：执行`sudo apt update`命令。
3. 安装Mainline工具：使用命令`sudo apt install mainline -y`。
4. 安装内核6.12：工具安装完成后，运行`sudo mainline install 6.12`，工具会自动开始内核相关文件的下载和安装。
5. 重启系统：安装完成后，执行`sudo reboot`命令，系统会自动切换到新安装的内核版本。

安装完成后，可以使用`uname -r`命令查看当前内核版本，确认安装是否生效。

## ukuu
- Install

```
sudo add-apt-repository -y ppa:teejee2008/ppa
sudo apt update
sudo apt install ukuu
```
- GUI ukuu 
search the version you want.


## Reference
- [在Ubuntu 16.04，Ubuntu 16.10上轻松安装Linux Kernel 4.10](https://www.linuxidc.com/Linux/2017-07/145838.htm)
