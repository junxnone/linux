---
Title | Tools OS Disk ncdu
-- | --
Created @ | `2026-10-13T07:45:26Z`
Updated @| `2026-10-13T07:45:26Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/137)

---
# ncdu
- `NCurses Disk Usage` - 基于 TUI 的磁盘空间分析工具
- `du` 的交互式替代品: 扫描一次后可用方向键逐层下钻, 快速定位占空间的大目录/大文件, 并支持就地删除

## Install

```
$ sudo apt install ncdu     # Debian/Ubuntu
$ sudo dnf install ncdu     # Fedora
$ sudo yum install ncdu     # CentOS/RHEL (EPEL)
$ brew install ncdu         # macOS
```

## Usage

```
$ ncdu --help
ncdu <options> <directory>

  -h,--help                  This help message
  -q                         Quiet mode, refresh interval 2 seconds
  -v,-V,--version            Print version
  -x                         Same filesystem
  -e                         Enable extended information
  -r                         Read only
  -o FILE                    Export scanned directory to FILE
  -f FILE                    Import scanned directory from FILE
  -0,-1,-2                   UI to use when scanning (0=none,2=full ncurses)
  --si                       Use base 10 (SI) prefixes instead of base 2
  --exclude PATTERN          Exclude files that match PATTERN
  -X, --exclude-from FILE    Exclude files that match any pattern in FILE
  -L, --follow-symlinks      Follow symbolic links (excluding directories)
  --exclude-caches           Exclude directories containing CACHEDIR.TAG
  --confirm-quit             Confirm quitting ncdu
  --color SCHEME             Set color scheme (off/dark)
```

- 界面示例

```
ncdu 1.15.1 ~ Use the arrow keys to navigate, press ? for help
--- /home/user ----------------------------------------------------------
   12.4 GiB [##########] /work
    4.1 GiB [###       ] /.cache
    2.8 GiB [##        ] /Downloads
  986.2 MiB [          ] /.local
  120.5 MiB [          ]  core.dump
    4.0 KiB [          ] /.ssh
@                        .bashrc -> dotfiles/bashrc
 Total disk usage:  20.4 GiB  Apparent size:  20.1 GiB  Items: 184320
```

## 快捷键

Key | Description
-- | --
`↑` `↓` / `k` `j` | 上下移动
`→` / `l` / `Enter` | 进入目录
`←` / `h` | 返回上层目录
`n` / `s` / `C` | 按 文件名 / 大小 / item 数 排序
`t` | 目录排在文件前面
`a` | 切换 apparent size / disk usage
`g` | 切换 百分比 / 图形条 显示
`i` | 查看选中项的详细信息
`d` | 删除选中的文件或目录
`r` | 重新扫描当前目录
`e` | 显示/隐藏 隐藏文件
`b` | 在当前目录打开 shell
`q` | 退出
`?` | 帮助

## UseCase

### 分析当前目录

```
$ ncdu
```

### 分析指定目录

```
$ ncdu /var
```

### 只扫描当前文件系统 `-x`
- 不跨挂载点, 避免扫到 `/proc` `/sys` `/mnt` 和网络盘

```
$ sudo ncdu -x /
```

### 排除指定目录 `--exclude`

```
$ ncdu --exclude .git --exclude node_modules ~/work
```

### 只读模式 `-r`
- 禁用删除操作, 防止误删
- `-r -r` 同时禁用 shell spawn, 适合 root 下排查

```
$ sudo ncdu -rx /
```

### 导出 / 导入扫描结果 `-o` / `-f`
- 服务器上扫描 -> 本地查看, 避免 ssh 交互卡顿, 结果也可留档对比

```
# 扫描并导出 (-o- 输出到 stdout)
$ sudo ncdu -1xo- / | gzip > /tmp/root.ncdu.gz

# 导入查看 (-f- 从 stdin 读取)
$ zcat /tmp/root.ncdu.gz | ncdu -f-
```

### 直接分析远程主机

```
$ ssh -C user@remote-host ncdu -o- / | ncdu -f-
```

### 清理 Docker / 构建缓存占用

```
$ sudo ncdu -x /var/lib/docker
```

## Reference
- [ncdu Homepage](https://dev.yorhel.nl/ncdu)
- [ncdu man page](https://dev.yorhel.nl/ncdu/man)

