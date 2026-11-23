-----

| Title         | Tools script                                       |
| ------------- | -------------------------------------------------- |
| Created @     | `2021-11-04T08:32:12Z`                             |
| Last Modify @ | `2022-12-21T10:33:51Z`                             |
| Labels        | \`\`                                               |
| Edit @        | [here](https://github.com/junxnone/linux/issues/6) |

-----

# script 记录终端执行命令历史

## Reference

  - [【Linux学习
    】Linux使用Script命令来记录并回放终端会话](https://blog.csdn.net/ouyang_peng/article/details/78818492)

## Brief

  - `script` 记录终端执行命令的历史
  - `scriptreplay` 播放 `script` 记录的数据

<!-- end list -->

    Usage:
     script [options] [file]
    
    Make a typescript of a terminal session.
    
    Options:
     -a, --append                  append the output
     -c, --command <command>       run command rather than interactive shell
     -e, --return                  return exit code of the child process
     -f, --flush                   run flush after each write
         --force                   use output file even when it is a link
     -o, --output-limit <size>     terminate if output files exceed size
     -q, --quiet                   be quiet
     -t[<file>], --timing[=<file>] output timing data to stderr or to FILE
     -h, --help                    display this help
     -V, --version                 display version

    Usage:
     scriptreplay [-t] timingfile [typescript] [divisor]
    
    Play back terminal typescripts, using timing information.
    
    Options:
     -t, --timing <file>     script timing output file
     -s, --typescript <file> script terminal session output file
     -d, --divisor <num>     speed up or slow down execution with time divisor
     -m, --maxdelay <num>    wait at most this many seconds between updates
     -h, --help              display this help
     -V, --version           display version

## UseCase

| Usecase   | cmd                                     |
| --------- | --------------------------------------- |
| 开始记录      | `script`                                |
| 保存记录到某个文件 | `script your/log/file`                  |
| 停止记录      | `exit` <br><kbd>Ctrl</kbd>+<kbd>D</kbd> |
| 静默方式运行    | `script -q`                             |

  - 记录可以使用 `scriptreplay` 回放的格式

<!-- end list -->

    $ script -t 2>cmd.time -a cmd.his
    ...
    run some command
    ...
    $ exit

  - 播放 `script` 记录的文件

<!-- end list -->

    $ scriptreplay cmd.time cmd.his
