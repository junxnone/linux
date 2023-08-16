---
Title | Tools Text tr
-- | --
Created @ | `2019-08-22T09:40:46Z`
Updated @| `2023-08-16T14:35:14Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/26)

---
# tr 
- tr 指令从标准输入设备读取数据，经过字符串转译后，将结果输出到标准输出设备。
- Linux tr 命令用于转换或删除文件中的字符。


## UseCase

Usecase | cmd
-- | --
替换大小写 | `cat testfile \| tr a-z A-Z`
替换空格为逗号 | `cat log.txt \| tr ' ' ','`
删除 Linux 换行符 | `cat testfile \| tr -d '\n'`
删除 Windows 换行符 | `cat testfile \| tr -d '\r\n'`

## Reference
- [Linux tr命令](https://www.runoob.com/linux/linux-comm-tr.html)

