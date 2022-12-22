---
Title | Tools OS xset
-- | --
Created @ | `2021-10-21T08:02:21Z`
Last Modify @| `2022-12-22T01:26:16Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/8)

---
## Brief
-  用于设置显示选项
- reboot 后重置

## UseCase

Usecase | cmd
-- | --
显示配置信息 | `xset q`
关闭屏保 | `xset s 0`
关闭 standby | `xset -dpms`<br>`xset dpms 0 0 0`
强制进入休眠 | `xset -display :0.0 dpms force off`
强制退出休眠 | `xset -display :0.0 dpms force on`
