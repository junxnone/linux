---
Title | Ubuntu Issues tabdelay
-- | --
Created @ | `2023-11-06T02:30:10Z`
Updated @| `2023-11-06T02:30:10Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/126)

---
# Terminal 按 tab 键卡死一段时间

- 查看 shell 是否是 `dash`

```
ls -lah /bin/sh
```
> Output:
```
lrwxrwxrwx 1 root root 4  3月 23  2022 /bin/sh -> dash
```
- 若是 `dash` 引起的，可以尝试重新配置 `dash`
```
sudo dpkg-reconfigure dash
```

> 选择 `NO`, 不要使用 `dash`

```
lrwxrwxrwx 1 root root 4 11月  6 10:17 /bin/sh -> bash
```
