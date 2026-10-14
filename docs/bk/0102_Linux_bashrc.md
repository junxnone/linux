-----

| Title     | Linux bashrc                                         |
| --------- | ---------------------------------------------------- |
| Created @ | `2019-02-19T09:53:26Z`                               |
| Updated @ | `2026-10-14T01:33:51Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/102) |

-----

# Linux bashrc

## 加载机制

  - 区分 login shell 与 interactive non-login shell
      - login shell（如 ssh 登录、`su -`、tty 登录）：按顺序查找 `~/.bash_profile` →
        `~/.bash_login` → `~/.profile`，只读取找到的第一个
          - Ubuntu（基于 Debian）默认并不创建 `~/.bash_profile`，只提供
            `~/.profile`，所以实际生效的是 `~/.profile`
      - interactive non-login shell（如打开一个新终端窗口）：读取 `~/.bashrc`
      - non-interactive shell（如执行脚本）：读取 `$BASH_ENV` 指向的文件（如果设置了）
  - Ubuntu 默认的 `~/.profile` 模板里会判断当前是 bash 且 `~/.bashrc` 存在时自动 source
    它，从而让登录 shell 也能拿到 bashrc 里的别名/函数：
    ``` bash
    if [ -n "$BASH_VERSION" ]; then
        if [ -f "$HOME/.bashrc" ]; then
            . "$HOME/.bashrc"
        fi
    fi
    ```
  - 全局级别：`/etc/bash.bashrc` 对所有用户生效，每个用户的 `~/.bashrc` 通常会 source 它。

## 常见自定义内容

  - 别名 alias
    ``` bash
    alias ll='ls -alF'
    alias grep='grep --color=auto'
    ```
  - PATH / 环境变量
    ``` bash
    export PATH="$HOME/bin:$PATH"
    export EDITOR=vim
    ```
  - 命令行提示符 PS1
    ``` bash
    PS1='\u@\h:\w\$ '
    ```
  - history 相关
    ``` bash
    export HISTSIZE=10000
    export HISTCONTROL=ignoredups:erasedups
    shopt -s histappend
    ```
  - 自定义函数
    ``` bash
    mkcd() { mkdir -p "$1" && cd "$1"; }
    ```
  - 非交互式场景提前退出（Ubuntu 默认 `~/.bashrc` 模板开头常见写法，避免脚本 source 时产生意外输出）
    ``` bash
    case $- in
        *i*) ;;
          *) return;;
    esac
    ```

## Issues

### 新创建的用户没有 .bashrc

**原因**：`useradd` 默认不会把 `/etc/skel/` 下的模板文件拷贝到新用户家目录，除非显式加上 `-m`（create
home）参数，Ubuntu 的默认配置也未开启自动拷贝。`/etc/skel/`
目录下有哪些文件，新建用户的家目录理论上就应生成哪些文件（通常是
`.bashrc`、`.profile`、`.bash_logout`，不含 `.bash_profile`）。

**Solution**

1.  补建 home 目录并拷贝骨架文件：
    ``` bash
    useradd -m newuser   # -m 会从 /etc/skel 拷贝模板文件（含 .bashrc/.profile/.bash_logout 等）
    ```
    用户已建好但缺文件时手动补拷：
    ``` bash
    cp /etc/skel/.bashrc /etc/skel/.profile /etc/skel/.bash_logout ~newuser/
    chown newuser:newuser ~newuser/.bashrc ~newuser/.profile ~newuser/.bash_logout
    ```
2.  同时添加 `.bashrc` 与 `.profile` 两个文件即可（不需要额外建 `.bash_profile`——Ubuntu
    默认也不依赖它）。`.bashrc` 要能在登录时被自动加载，靠的是 `.profile` 里显式/隐式 source
    `.bashrc`，原理见上面"加载机制"一节。

## Reference

  - [为Ubuntu新创建用户创建默认.bashrc并自动加载](https://www.cnblogs.com/sddai/p/10644870.html)
  - [GNU Bash Manual - Bash Startup
    Files](https://www.gnu.org/software/bash/manual/bash.html#Bash-Startup-Files)
  - [man useradd
    (/etc/skel)](https://man7.org/linux/man-pages/man8/useradd.8.html)
