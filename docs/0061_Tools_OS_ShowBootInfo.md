---
Title | Tools OS ShowBootInfo
-- | --
Created @ | `2020-04-22T10:18:11Z`
Updated @| `2023-06-20T08:45:24Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/61)

---
# 开机相关信息

## 登录用户

- who

```
who -r
who -b
```

## 启动时间

- uptime 查看启动时间
```
uptime -s
```
```
2023-06-20 01:45:05
```

- uptime 查看启动到现在有多久
```
uptime -p
```
```
up 14 hours, 53 minutes
```


## reboot shutdown 历史记录

- last
```
last
last -x
last -x reboot
last -x shutdown
```



## Reference
- [Linux查看系统开机时间](https://www.cnblogs.com/kerrycode/p/3759395.html)


