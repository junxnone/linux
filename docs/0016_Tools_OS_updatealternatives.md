---
Title | Tools OS updatealternatives
-- | --
Created @ | `2020-06-03T06:38:33Z`
Updated @| `2023-11-02T05:56:32Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/16)

---
# updatealternatives 多版本软件管理工具
-  用于软件多版本切换
  - 显示安装应用的多个版本
  - 安装新的应用版本到管理器
  - 配置应用版本的优先级
  - 移除被管理的版本

```
$ update-alternatives --help
Usage: update-alternatives [<option> ...] <command>

Commands:
  --install <link> <name> <path> <priority>
    [--slave <link> <name> <path>] ...
                           add a group of alternatives to the system.
  --remove <name> <path>   remove <path> from the <name> group alternative.
  --remove-all <name>      remove <name> group from the alternatives system.
  --auto <name>            switch the master link <name> to automatic mode.
  --display <name>         display information about the <name> group.
  --query <name>           machine parseable version of --display <name>.
  --list <name>            display all targets of the <name> group.
  --get-selections         list master alternative names and their status.
  --set-selections         read alternative status from standard input.
  --config <name>          show alternatives for the <name> group and ask the
                           user to select which one to use.
  --set <name> <path>      set <path> as alternative for <name>.
  --all                    call --config on all alternatives.

<link> is the symlink pointing to /etc/alternatives/<name>.
  (e.g. /usr/bin/pager)
<name> is the master name for this link group.
  (e.g. pager)
<path> is the location of one of the alternative target files.
  (e.g. /usr/bin/less)
<priority> is an integer; options with higher numbers have higher priority in
  automatic mode.

Options:
  --altdir <directory>     change the alternatives directory.
  --admindir <directory>   change the administrative directory.
  --log <file>             change the log file.
  --force                  allow replacing files with alternative links.
  --skip-auto              skip prompt for alternatives correctly configured
                           in automatic mode (relevant for --config only)
  --quiet                  quiet operation, minimal output.
  --verbose                verbose operation, more output.
  --debug                  debug output, way more output.
  --help                   show this help message.
  --version                show the version.

```


## UseCase

- `--display`

```
$ update-alternatives --display python 
python - 手动模式
link best version is /usr/bin/python3.5
链接目前指向 /usr/bin/python2.7
link python is /usr/bin/python
/usr/bin/python2.7 - 优先级 1
/usr/bin/python3.5 - 优先级 2
```

- `--install`

```
# 添加 python link
$ update-alternatives --install /usr/bin/python python /usr/bin/python2.7 2

# 第一个参数: --install 表示向update-alternatives注册服务名。
# 第二个参数: 注册最终地址，成功后将会把命令在这个固定的目的地址做真实命令的软链，以后管理就是管理这个软链；
# 第三个参数: 服务名，以后管理时以它为关联依据。
# 第四个参数: 被管理的命令绝对路径。
# 第五个参数: 优先级，数字越大优先级越高。
```

- `--config`

```
$ update-alternatives --config python    
有 2 个候选项可用于替换 python (提供 /usr/bin/python)。
  选择       路径              优先级  状态
------------------------------------------------------------
  0            /usr/bin/python3.5   2         自动模式
* 1            /usr/bin/python2.7   1         手动模式
  2            /usr/bin/python3.5   2         手动模式

要维持当前值[*]请按<回车键>，或者键入选择的编号：
```

- `--remove`

```
$ update-alternatives –remove python /usr/bin/python2.7
```

## 应用

- Python
- GCC


## Reference
- [update-alternatives使用详解](https://www.jianshu.com/p/4d27fa2dce86)


