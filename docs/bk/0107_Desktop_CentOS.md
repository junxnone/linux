-----

| Title     | Desktop CentOS                                       |
| --------- | ---------------------------------------------------- |
| Created @ | `2018-11-29T04:59:44Z`                               |
| Updated @ | `2024-07-13T03:42:20Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/107) |

-----

# CentOS 安装桌面

  - Desktop
      - [Gnome](##Gnome)
      - [KDE](##KDE)

## Gnome

### Install

``` 
 yum -y groups install "GNOME Desktop" 
```

### Start

    startx

## KDE

### Install

``` 
 yum -y groups install "KDE Plasma Workspaces" 
```

### Start

    echo "exec startkde" >> ~/.xinitrc
    startx

## Reference

  - [How to install Desktop Environments on
    CentOS 7?](https://unix.stackexchange.com/questions/181503/how-to-install-desktop-environments-on-centos-7)
