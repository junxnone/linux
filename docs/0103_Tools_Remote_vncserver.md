---
Title | Tools Remote vncserver
-- | --
Created @ | `2019-01-14T05:35:17Z`
Updated @| `2023-06-08T09:12:48Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/103)

---
# VNC Tools

- VNC - `Virtual Network Computing` Linux 远程访问 Desktop
- VNC 由AT&T 的剑桥研究实验室开发，可实现远程图像显示和控制。
- Tools
  - TigerVNC
  - TightVNC
  - TurboVNC
  - RemoteVNC
  - RealVNC
  - vino/vinagre
  - x11vnc
- RFB protocol - `remote framebuffer protocol`


## Ubuntu Install tightvncserver

```
sudo apt install  tightvncserver
```

### xfce4 Desktop Setup

```
sudo apt install xfce4 xfce4-goodies
```

- `~/.vnc/xstartup`

```
#!/bin/bash
xrdb $HOME/.Xresources
startxfce4 &
```

### 设置密码
```
vncpasswd
```



### 打开一个server 1920 x 1080
```
vncserver -geometry 1920x1080
```

### 查看启动的 vncserver

```
ps -ef |grep vnc
```

### Kill VNCServer

```
vncserver -kill :1
```
> 1 是 编号/端口


## Connect

- 使用 `vncviewer` 连接 IP:Port  `xxx.xxx.xx.xxx:n`

> n 从 5901 开始

## Issues
### 显示复选框
![image](https://user-images.githubusercontent.com/2216970/150967111-67938d74-01b2-4277-a7fd-27065891ede2.png)

- `vi ~/.vnc/xstartup`
- 添加 `gnome-session &`
- reboot

## Reference
- [vncviewer](https://www.realvnc.com/en/connect/download/viewer/)
- [TigerVNC](https://tigervnc.org/)
- [vnc简介](https://github.com/levinit/itnotes/blob/main/vnc.md)
- [Virtual Network Computing - wikipedia](https://en.wikipedia.org/wiki/Virtual_Network_Computing)
- [RFB protocol](https://en.wikipedia.org/wiki/RFB_protocol)
- [How to Install and Configure VNC on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-vnc-on-ubuntu-22-04)
- [VNC Server setup on Ubuntu 22.04 LTS](https://gist.github.com/indyfromoz/739cd53d47b91ba1d3b540ab53b1f46c)
