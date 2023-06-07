---
Title | Tools Remote vncserver
-- | --
Created @ | `2019-01-14T05:35:17Z`
Updated @| `2023-06-07T09:55:27Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/103)

---
# VNCServer

- VNC - `Virtual Network Computing` Linux 远程访问 Desktop

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

## Connect
- 使用 `vncviewer` 连接 IP:Port  `xxx.xxx.xx.xxx:n`


## Reference
- [vncviewer](https://www.realvnc.com/en/connect/download/viewer/)

