---
Title | Tools ldd
-- | --
Created @ | `2023-09-04T14:14:00Z`
Updated @| `2023-09-04T14:14:00Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/124)

---
# ldd
- 打印依赖库,
- 查看依赖库的位置
  - 如果有多个版本依赖库，查看调用哪个

```
$ ldd --help
Usage: ldd [OPTION]... FILE...
      --help              print this help and exit
      --version           print version information and exit
  -d, --data-relocs       process data relocations
  -r, --function-relocs   process data and function relocations
  -u, --unused            print unused direct dependencies
  -v, --verbose           print all information
```
