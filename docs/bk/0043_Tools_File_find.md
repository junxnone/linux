-----

| Title         | Tools File find                                     |
| ------------- | --------------------------------------------------- |
| Created @     | `2019-07-02T02:39:44Z`                              |
| Last Modify @ | `2022-12-21T15:13:48Z`                              |
| Labels        | \`\`                                                |
| Edit @        | [here](https://github.com/junxnone/linux/issues/43) |

-----

## Reference

  - [find -size
    查出指定文件大小的命令](https://www.cnblogs.com/rusking/p/7403160.html)

## Brief

  - 查找文件

<!-- end list -->

    find /your/path/ -name "find_name"

## UseCase

| Usecase        | cmd                                   |
| -------------- | ------------------------------------- |
| 查找 0 字节文件      | `find ./ -size 0`<br>`find ./ -empty` |
| 查找小于 `10c` 的文件 | `find ./ -size -10c`                  |
| 查找大于 `10c` 的文件 | `find ./ -size +10c`                  |

> c/k/M/G，默认为块(1块(b)=512c)
