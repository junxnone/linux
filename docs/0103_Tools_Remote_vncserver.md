---
Title | Tools Remote vncserver
-- | --
Created @ | `2019-01-14T05:35:17Z`
Updated @| `2023-06-07T12:40:00Z`
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


## Ubuntu Install
```
sudo apt install vnc4server
sudo apt install gnome-panel gnome-settings-daemon metacity nautilus gnome-terminal
```

## Setup
### 设置密码
```
vncpasswd
```

## 配置vnc
```
# vim ~/.vnc/xstartup
```
```
export XKL_XMODMAP_DISABLE=1
unset SESSION_MANAGER
unset DBUS_SESSION_BUS_ADDRESS
gnome-panel &
gnmoe-settings-daemon &
metacity &
nautilus &
gnome-terminal &
```

### 打开一个server 1920 x 1080
```
vnc4server -geometry 1920x1080
```

### 查看启动的 vncserver

```
`ps -ef \|grep vnc`
```

### Kill VNCServer

```
vncserver -kill :1
```
> 1 是 编号


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

