---
Title | Tools Text grep
-- | --
Created @ | `2019-08-06T05:13:14Z`
Last Modify @| `2022-12-23T15:29:47Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/25)

---
# grep 文本过滤工具

## Brief
- 过滤出感兴趣的内容

```
grep -nr your_word yourfile
```

## UseCase

Usecase | cmd
-- | --
反向匹配 - 不包含 keyword | `grep -v keyword`
以 xxx 开头 | `grep '^xxx'`
以 xxx 结尾 | `grep 'xxx$'`
查询包含字母的行 | `grep -n '[a-z]'`
查询以三位数字开头的文件 | `ls \|grep '^[0-9][0-9][0-9]*'`
找出文件中包含123或者包含abc的行 | `grep -E '123\|abc' filename`<br>**其他实现**<br> - egrep `egrep '123\|abc' filename`<br> - awk `awk '/123\|abc/' filename`



## Examples 

- 替换以 `r` windows 换行 为 Linux 换行 `\r\n`
```
find . -not -type d -exec file "{}" ";" | grep CRLF|awk -F ':' '{print $1}'|xargs -i dos2unix {}
```
