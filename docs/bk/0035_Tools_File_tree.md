-----

| Title         | Tools File tree                                     |
| ------------- | --------------------------------------------------- |
| Created @     | `2020-04-04T04:39:52Z`                              |
| Last Modify @ | `2022-12-22T09:42:57Z`                              |
| Labels        | \`\`                                                |
| Edit @        | [here](https://github.com/junxnone/linux/issues/35) |

-----

# tree 优雅地列出文件树

## Reference

  - [tree -
    linux-command](https://wangchujiang.com/linux-command/c/tree.html)
  - [tree - man.linuxde](https://man.linuxde.net/tree)

## Brief

  - 常用 `args`

| Args | Description          |
| ---- | -------------------- |
| \-d  | 只打印目录                |
| \-i  | 不打印缩进线               |
| \-f  | 全路径                  |
| \-s  | 文件大小                 |
| \-P  | pattern              |
| \-I  | 大写 `i` , 不打印 pattern |
| \-L  | 层级                   |
| \-p  | 文件权限                 |

## UseCase

| Usecase            | cmd                        |
| ------------------ | -------------------------- |
| 列出目录               | `tree -d your_path`        |
| 不打印缩进线             | `tree -i  your_path`       |
| 打印完整路径             | `tree -f your_path`        |
| 打印文件大小             | `tree -s your_path`        |
| 打印包含 `pattern` 文件  | `tree -P 'app*' your_path` |
| 不打印包含 `pattern` 文件 | `tree -I 'app*' your_path` |
| 打印 `n` 层文件         | `tree -L n your_path`      |
| 打印文件权限             | `tree -p your_path`        |
