---
Title | Tools history
-- | --
Created @ | `2019-01-15T14:20:09Z`
Updated @| `2023-06-07T12:49:05Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/49)

---
# history 命令行历史管理
- `history` 查看历史执行命令
- 重复执行历史命令

## UseCase 

Usecase | cmd
-- | --
查看历史执行命令 | `history`
执行历史命令 | `! number`
清除历史命令 | `history -c`
使用 Ctrl+R 搜索历史 | <kbd>Ctrl</kbd> + <kbd>R</kbd><br>`(reverse-i-search)`':`

## Config

- **设置显示格式：历史编号 - 时间 - 用户名 - 命令**

```
sudo vi /etc/profile
```
```
export HISTTIMEFORMAT="%Y-%m-%d:%H-%M-%S:`whoami`: "
export HISTSIZE=10000
```


- **忽略连续重复的条目**
```
export HISTCONTROL=ignoredups
```

## Reference
- [ubuntu history 命令详解](https://blog.csdn.net/sdfgh2046/article/details/5376086)
