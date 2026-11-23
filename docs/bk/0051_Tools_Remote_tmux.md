-----

| Title     | Tools Remote tmux                                   |
| --------- | --------------------------------------------------- |
| Created @ | `2018-11-29T05:32:54Z`                              |
| Updated @ | `2024-02-27T03:33:20Z`                              |
| Labels    | \`\`                                                |
| Edit @    | [here](https://github.com/junxnone/linux/issues/51) |

-----

# tmux

  - TMUX（Terminal
    Multiplexer）是一个终端复用器，它允许用户在一个终端中创建多个会话（session），每个会话可以包含多个窗口（window），每个窗口可以运行不同的命令或程序。通过
    Tmux，用户可以在不同的会话和窗口之间快速切换，提高工作效率。
  - [tmux arch](./tmux_arch)
  - [tmux plugins](./tmux_plugins)

## Install

### Install on Ubuntu

    sudo apt install tmux

### 从源码编译安装

    sudo apt install libevent-dev libncurses-dev
    wget https://github.com/tmux/tmux/releases/download/3.0/tmux-3.0-rc3.tar.gz
    tar zvxf tmux-3.0-rc3.tar.gz
    cd tmux-3.0-rc3
    ./configure && make
    sudo make install

## Config

  - 配置文件: `~/.tmux.conf`

### 设置鼠标

  - 设置后可以鼠标选择窗口，可以滚动信息来查看历史信息

<!-- end list -->

``` 
set -g mouse on 
```

> 设置了此选项后选择复制时需要同时按住**Shift**按键

## UseCase

### Command Line

| Target             | Commands                                                                                                                                |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
| Set base directory | inside tmux: <kbd>Ctrl - b</kbd> + <kbd>:</kbd> + `attach -c the_directory` <br>outside tmux: `tmux attach -t session_name -c work_dir` |
| New session        | `tmux new -s xxx`                                                                                                                       |
| List the Session   | `tmux ls`                                                                                                                               |
| Open the session   | `tmux at -t xxx`                                                                                                                        |
| Close the session  | `tmux kill-session -t xxxx`                                                                                                             |
| rename the window  | `tmux rename-window new_name`                                                                                                           |

### HotKeys

| HotKey                                     | Description                 |
| ------------------------------------------ | --------------------------- |
| <kbd>Ctrl - b</kbd> + <kbd>%</kbd>         | 垂直分割                        |
| <kbd>Ctrl - b</kbd> + <kbd>"</kbd>         | 水平分割                        |
| <kbd>Ctrl - b</kbd> + <kbd>o</kbd>         | 交换窗格                        |
| <kbd>Ctrl - b</kbd> + <kbd>x</kbd>         | 关闭窗格                        |
| <kbd>Ctrl - b</kbd> + <kbd>⍽</kbd>         | 空格键 - 切换布局                  |
| <kbd>Ctrl - b</kbd> + <kbd>q</kbd>         | 显示窗格编号，按相应数字切换窗格            |
| <kbd>Ctrl - b</kbd> + <kbd>{</kbd>         | 与上一个窗格交换位置                  |
| <kbd>Ctrl - b</kbd> + <kbd>}</kbd>         | 与下一个窗格交换位置                  |
| <kbd>Ctrl - b</kbd> + <kbd>z</kbd>         | 切换窗格最大化/最小化                 |
| <kbd>Ctrl - b</kbd> + <kbd>s</kbd>         | list the session            |
| <kbd>Ctrl - b</kbd> + <kbd>w</kbd>         | list the windows            |
| <kbd>Ctrl - b</kbd> + <kbd>←↑↓→</kbd>      | 切换 Pane                     |
| <kbd>Ctrl - b</kbd> + <kbd>c</kbd>         | 当前 `Session` 创建 新 `Windows` |
| <kbd>Ctrl - b</kbd> + <kbd>Shift - d</kbd> | Attach 其他 client            |
| <kbd>Ctrl - b</kbd> + <kbd>\!</kbd>        | move pane to new window     |
| <kbd>Ctrl - b</kbd> + <kbd>,</kbd>         | rename the window           |

## Issues

### 窗口显示不全

  - 当其他 `Client` (屏幕小于当前屏幕)连接进入，`Window` 会自适应最小屏幕变小
  - 其他 `Client` 没有在使用后，但是未退出，可以使用如下方法关掉对方连接

使用如下快捷键:

<kbd>Ctrl</kbd> + <kbd>b</kbd> +<kbd>Shift</kbd> + <kbd>d</kbd>

或者：

    tmux attach -d

## Reference

  - [Tmux 快捷键 &
    速查表](https://blog.csdn.net/xiaxuesong666/article/details/80579945)
  - [no support for other mouse
    config](https://superuser.com/questions/210125/scroll-shell-output-with-mouse-in-tmux)
  - [tmux：打造精致与实用并存的终端](https://segmentfault.com/a/1190000008188987)
  - [man -tmux](http://man.openbsd.org/OpenBSD-current/man1/tmux.1)
  - [The Tao of
    tmux](https://leanpub.com/the-tao-of-tmux/read#leanpub-auto-creating-windows)
  - [参考配置文件](https://github.com/einverne/dotfiles/blob/master/tmux/.tmux.conf)
  - [Tmux Resurrect & Continuum: 持久保存 Tmux
    会话](https://linuxtoy.org/archives/tmux-resurrect-and-continuum.html)
  - [Tmux Plugin Manager](https://github.com/tmux-plugins/tpm)
  - [tmux conf - gpakosz](https://github.com/gpakosz/.tmux)
