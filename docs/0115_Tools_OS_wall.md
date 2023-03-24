---
Title | Tools OS wall
-- | --
Created @ | `2023-03-24T02:19:01Z`
Updated @| `2023-03-24T02:19:01Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/115)

---
# wall
- 向所有终端广播信息

```
$ wall --help

Usage:
 wall [options] [<file> | <message>]

Write a message to all users.

Options:
 -g, --group <group>     only send message to group
 -n, --nobanner          do not print banner, works only for root
 -t, --timeout <timeout> write timeout in seconds

 -h, --help              display this help
 -V, --version           display version

For more details see wall(1).
```

## UseCase
- 需要重启时，提前广播信息


### 简单广播

- 用户发送如下命令

```
$ wall "The system will be restarted in 10 minutes."
```

- 其他终端会收到如下命令

```
Broadcast message from xxx@xxx (pts/0) (Fri Mar 24 10:22:32 2023

The system will be restarted in 10 minutes.
```

### 隐藏 第一行信息 `-n`
- 需要 root 用户 或 sudo 权限

```
$ sudo wall -n "The system will be restarted in 10 minutes."
```


### 广播到 Group

- 向 `devs` group 广播

```
$ wall -g devs "The system will be restarted in 10 minutes."
```


### 




## Reference

