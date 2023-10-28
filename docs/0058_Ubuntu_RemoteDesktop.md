---
Title | Ubuntu RemoteDesktop
-- | --
Created @ | `2021-01-06T03:05:58Z`
Updated @| `2023-10-28T06:39:14Z`
Labels | `enhancement`
Edit @| [here](https://github.com/junxnone/linux/issues/58)

---
# Ubuntu Remote Desktop
- [vncserver](./0103_Tools_Remote_vncserver)
- xrdp 方式 windows 访问 Linux

## xrdp

- 安装 xrdp

```
wget http://www.c-nergy.be/downloads/Std-Xrdp-Install-0.5.3.zip
unzip Std-Xrdp-Install-0.5.3.zip
sudo chmod +x Std-Xrdp-Install-0.5.3.sh
./Std-Xrdp-Install-0.5.3.sh
```

> 0.5.3 only working with Ubuntu 18.04, Download the [New Version](https://c-nergy.be/repository.html)

- 卸载 xrdp

```
sudo apt purge xrdp
```

- host 端登录后不能识别 USB 鼠标键盘
`sudo -E apt-get install xserver-xorg-input-all`
> reboot after install the package, then the keyboard & mouse are back

## VNC

```
sudo apt-get update
sudo apt install lightdm
sudo apt install xserver-xorg-video-dummy xorg-video-abi-23 xserver-xorg-core
sudo reboot
sudo apt-get install x11vnc net-tools
sudo x11vnc -storepasswd /etc/x11vnc.pass
sudo chmod 755 /etc/x11vnc.pass
sudo x11vnc -auth guess -rfbauth /etc/x11vnc.pass -rfbport 5900 -forever -display :0
```
- **配置 systemd 开机启动**

```
sudo vim /etc/systemd/system/x11vnc.service
```
```
[Unit]
Description=x11vnc (Remote access)
After=network-online.target
 
[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -auth guess -rfbauth /etc/x11vnc.pass -rfbport 5900 -forever -display :0
ExecStop=/bin/kill -TERM $MAINPID
ExecReload=/bin/kill -HUP $MAINPID
KillMode=control-group
Restart=on-failure
 
[Install]
WantedBy=graphical.target
```
```
sudo systemctl daemon-reload
sudo systemctl enable x11vnc
sudo systemctl start x11vnc
```

## Reference
- [Issues with xRDP and Ubuntu 18.04.2 – How to fix it](http://c-nergy.be/blog/?p=13390)
- [配置 VNC](https://www.mobibrw.com/2019/19379)
- [xRDP Installation Script](https://c-nergy.be/repository.html)
- [realvnc](https://www.realvnc.com/en/connect/download/viewer/)
- [TightVNC](https://www.tightvnc.com/download-old.php)
- [[Ubuntu Server 22.04] 　Ubuntu Desktop RDP grdctl](https://qiita.com/QiitaYkuyo/items/c8e700da451e894e5d53)
- [grdctl manpages](https://manpages.ubuntu.com/manpages/lunar/en/man1/grdctl.1.html)
- [How To Install XRDP (Remote Desktop) on Ubuntu 20.04](https://tecadmin.net/how-to-install-xrdp-on-ubuntu-20-04/)
