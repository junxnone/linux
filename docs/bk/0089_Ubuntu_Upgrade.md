-----

| Title     | Ubuntu Upgrade                                      |
| --------- | --------------------------------------------------- |
| Created @ | `2019-09-04T05:07:55Z`                              |
| Updated @ | `2023-06-06T07:15:15Z`                              |
| Labels    | \`\`                                                |
| Edit @    | [here](https://github.com/junxnone/linux/issues/89) |

-----

# Ubuntu OS Version Upgrade

  - lts 升级 (`16.xx` --\> `18.xx` --\> `20.xx` --\> `22.xx`)
      - 升级 `Ubuntu 16.04` 到 `18.04`
      - 升级 `Ubuntu 18.04` 到 `20.04`
  - 小版本号升级 (`20.04` --\> `20.06`)

## Upgrade Major Version

### 1 先更新当前的系统

    sudo apt update && sudo apt dist-upgrade && sudo apt autoremove

### 2 Reboot OS

### 3 升级至 LTS OS

    sudo do-release-upgrade

### Issues

#### 重装 快于 升级

  - 升级时间比较长 ，如果系统可以重装还是重装快

#### 控制升级版本

  - `/etc/update-manager/release-upgrades` --\> `Prompt=lts` 控制升级版本

#### python 相关报错

  - 如果 python 链接为 python3,则会出现如下报错，重新链接回 python2.7 即可

<!-- end list -->

    Your python3 install is corrupted. Please fix the '/usr/bin/python3' symlink.

#### Ubuntu 20.04 upgrade failed

``` 
Done The following packages have been kept back:   
gdb gir1.2-peas-1.0 libpeas-1.0-0 libsmbclient libwbclient0 samba-libs 
```

1.  Remove the libpython3.8-stdlib, and save the packages info in the
    Prompts

<!-- end list -->

    sudo apt-get autoremove libpython3.8-stdlib

<details>
<summary>Packages</summary>

``` 
  apg apparmor apport apport-gtk apport-symptoms aptdaemon aptdaemon-data apturl apturl-common avahi-utils blt blueman catfish chrome-gnome-shell
  command-not-found cups-pk-helper dc dctrl-tools deja-dup distro-info distro-info-data dkms docker-compose duplicity firefox foomatic-db-compressed-ppds
  fprintd gdm3 gedit gedit-common gir1.2-accountsservice-1.0 gir1.2-appindicator3-0.1 gir1.2-atspi-2.0 gir1.2-cheese-3.0 gir1.2-clutter-1.0
  gir1.2-clutter-gst-3.0 gir1.2-cogl-1.0 gir1.2-coglpango-1.0 gir1.2-dbusmenu-glib-0.4 gir1.2-dee-1.0 gir1.2-gck-1 gir1.2-gcr-3 gir1.2-gdm-1.0
  gir1.2-geoclue-2.0 gir1.2-gnomebluetooth-1.0 gir1.2-graphene-1.0 gir1.2-gst-plugins-base-1.0 gir1.2-gstreamer-1.0 gir1.2-gtkclutter-1.0
  gir1.2-gtksource-3.0 gir1.2-gtksource-4 gir1.2-gudev-1.0 gir1.2-gweather-3.0 gir1.2-handy-0.0 gir1.2-ibus-1.0 gir1.2-javascriptcoregtk-4.0
  gir1.2-json-1.0 gir1.2-mutter-6 gir1.2-nm-1.0 gir1.2-nma-1.0 gir1.2-notify-0.7 gir1.2-packagekitglib-1.0 gir1.2-polkit-1.0 gir1.2-rb-3.0 gir1.2-rsvg-2.0
  gir1.2-secret-1 gir1.2-snapd-1 gir1.2-soup-2.4 gir1.2-totem-1.0 gir1.2-totemplparser-1.0 gir1.2-udisks-2.0 gir1.2-unity-5.0 gir1.2-upowerglib-1.0
  gir1.2-vte-2.91 gir1.2-webkit2-4.0 gir1.2-wnck-3.0 gkbd-capplet gnome-control-center gnome-control-center-faces gnome-getting-started-docs gnome-menus
  gnome-online-accounts gnome-session-bin gnome-session-common gnome-shell gnome-shell-common gnome-shell-extension-appindicator
  gnome-shell-extension-desktop-icons gnome-shell-extension-prefs gnome-shell-extension-ubuntu-dock gnome-startup-applications gnome-terminal
  gnome-terminal-data gnome-tweak-tool gnome-tweaks gnome-user-docs ibus ibus-data ibus-table indicator-applet indicator-appmenu indicator-bluetooth
  indicator-printers ippusbxd ipython3 jayatana language-selector-common language-selector-gnome libamtk-5-0 libamtk-5-common libbamf3-2
  libboost-mpi-python1.71-dev libboost-mpi-python1.71.0 libcolord-gtk1 libdmapsharing-3.0-2 libffi6 libfprint-2-2 libgnome-panel0 libgnomekbd8
  libgpod-common libgpod4 libgsound0 libgupnp-av-1.0-2 libgupnp-dlna-2.0-3 libldb1 liblirc-client0 libmpdec2 libmtp-common libmtp-runtime libmtp9
  libnetplan0 libnvidia-cfg1-470 libnvidia-common-470 libnvidia-compute-470:i386 libnvidia-decode-470 libnvidia-encode-470 libnvidia-extra-470
  libnvidia-fbc1-470 libnvidia-gl-470 libnvidia-ifr1-470 libpam-fprintd libpfm4 libpurple-bin libpython3-stdlib libpython3.8-stdlib librsync2
  librygel-core-2.6-2 librygel-db-2.6-2 librygel-renderer-2.6-2 librygel-server-2.6-2 libsgutils2-2 libsmbclient libtalloc2 libtepl-4-0 libtevent0
  libunity-gtk2-parser0 libunity-gtk3-parser0 libwbclient0 libwhoopsie-preferences0 libyelp0 libz3-4 libz3-dev lightdm-gtk-greeter-settings llvm-10
  llvm-10-dev llvm-10-runtime llvm-10-tools lsb-release meld menulibre mscompress mugshot nautilus-extension-gnome-terminal nautilus-share netplan.io
  networkd-dispatcher nvidia-compute-utils-470 nvidia-dkms-470 nvidia-driver-450 nvidia-driver-460 nvidia-driver-470 nvidia-kernel-common-470
  nvidia-kernel-source-470 nvidia-prime nvidia-utils-470 onboard onboard-common onboard-data openprinting-ppds orca pastebinit plymouth-theme-spinner
  plymouth-theme-ubuntu-text plymouth-theme-xubuntu-text printer-driver-foo2zjs printer-driver-foo2zjs-common printer-driver-m2300w printer-driver-ptouch
  printer-driver-pxljr printer-driver-sag-gdi python-apt-common python-pip-whl python-talloc python3 python3-appdirs python3-apport python3-apt
  python3-aptdaemon python3-aptdaemon.gtk3widgets python3-attr python3-backcall python3-bcrypt python3-blinker python3-brlapi python3-cached-property
  python3-cairo python3-certifi python3-cffi-backend python3-chardet python3-click python3-colorama python3-commandnotfound python3-crypto
  python3-cryptography python3-cups python3-cupshelpers python3-dateutil python3-dbus python3-debconf python3-debian python3-decorator python3-defer
  python3-distlib python3-distro python3-distro-info python3-distupgrade python3-distutils python3-docker python3-dockerpty python3-docopt
  python3-entrypoints python3-fasteners python3-filelock python3-future python3-gdbm python3-gi python3-gi-cairo python3-httplib2 python3-ibus-1.0
  python3-idna python3-importlib-metadata python3-ipython python3-ipython-genutils python3-jedi python3-jsonschema python3-jwt python3-keyring
  python3-keyrings.alt python3-launchpadlib python3-lazr.restfulclient python3-lazr.uri python3-lib2to3 python3-lockfile python3-louis
  python3-macaroonbakery python3-mako python3-markupsafe python3-monotonic python3-more-itertools python3-nacl python3-netifaces python3-oauthlib
  python3-openssl python3-paramiko python3-parso python3-pexpect python3-pickleshare python3-pip python3-pkg-resources python3-problem-report
  python3-prompt-toolkit python3-protobuf python3-psutil python3-ptyprocess python3-pyatspi python3-pygments python3-pymacaroons python3-pyrsistent
  python3-requests python3-requests-unixsocket python3-rfc3339 python3-secretstorage python3-setuptools python3-simplejson python3-six
  python3-software-properties python3-speechd python3-systemd python3-texttable python3-tk python3-traitlets python3-tz python3-update-manager
  python3-urllib3 python3-venv python3-virtualenv python3-wadllib python3-wcwidth python3-websocket python3-wheel python3-xcffib python3-xdg python3-xkit
  python3-yaml python3-zipp python3.8 python3.8-venv rhythmbox-plugin-alternative-toolbar rhythmbox-plugins rygel samba-libs sgt-launcher sgt-puzzles
  shimmer-themes snapd software-properties-common software-properties-gtk ssh-import-id switcheroo-control syslinux syslinux-common syslinux-legacy
  system-config-printer system-config-printer-common system-config-printer-udev tk8.6-blt2.5 totem-plugins ubuntu-advantage-desktop-daemon
  ubuntu-advantage-tools ubuntu-desktop ubuntu-desktop-minimal ubuntu-docs ubuntu-drivers-common ubuntu-minimal ubuntu-release-upgrader-core
  ubuntu-release-upgrader-gtk ubuntu-session ubuntu-standard ufw unattended-upgrades unity-gtk-module-common unity-gtk2-module unity-gtk3-module
  update-manager update-manager-core update-notifier update-notifier-common usb-creator-common usb-creator-gtk virtualenv whoopsie-preferences
  xfce4-panel-profiles xfpanel-switch xinput xorg xserver-xorg xserver-xorg-legacy xserver-xorg-video-nvidia-470 xubuntu-artwork xubuntu-core
  xubuntu-default-settings xubuntu-desktop xubuntu-wallpapers xwayland yelp yelp-xsl
```

</details>

2.  Install focal version libpython3.8-stdlib

<!-- end list -->

    sudo apt-get install libpython3.8-stdlib

3.  Reinstall the pacakges you saved

<!-- end list -->

    cat packages.txt |xargs -n 1 |xargs -i sh -c "sudo apt install {}"

4.  Update Grub

<!-- end list -->

    sudo update-grub2

5.  Reboot OS

## Upgrade Mnior Version

  - Example : `20.04.01` --\> `20.04.03`

<!-- end list -->

    sudo apt update
    sudo apt upgrade

## Reference

  - [How to fix update problems
    (Ubuntu 20.04)](https://askubuntu.com/questions/1231849/how-to-fix-update-problems-ubuntu-20-04)
