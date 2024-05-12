---
Title | Tools shuf
-- | --
Created @ | `2019-09-18T08:15:08Z`
Updated @| `2024-05-12T12:14:40Z`
Labels | `enhancement`
Edit @| [here](https://github.com/junxnone/linux/issues/28)

---
# shuf 随机选取工具
- 随机排列输入行
- 可以用来随机排列文件/数据
- 可以用 `-n` 随机选取文件

```
-e, --echo                  将每个ARG视为输入行。
-i, --input-range=LO-HI     将数字范围LO（最低）到HI（最高）之间的作为输入行。
-n, --head-count=COUNT      只输出前COUNT行。
-o, --output=FILE           将结果写入到文件而不是标准输出。
    --random-source=FILE    将FILE中内容作为随机数据源。
-r, --repeat                输出行可以重复。
-z, --zero-terminated       行终止符为NUL（空字符）而不是默认的换行符。
--help                      显示帮助信息并退出。
--version                   显示版本信息并退出。
```


## UseCase

- **随机选取三张图片**

```
ls images/* |shuf -n 3
```
