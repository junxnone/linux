---
Title | Ubuntu UserAdd
-- | --
Created @ | `2018-12-01T13:58:08Z`
Updated @| `2024-10-13T12:01:20Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/104)

---
# 添加用户


- [adduser vs useradd](./0069_Tools_OS_adduser)

## UseCase

Usecase | cmd
-- | --
Create new user with home folder | `sudo useradd  -r -m  -s /bin/bash gpu` <br>`-m 创建用户目录在 /home/gpu`
指定目录为用户名主目录 | `usermod -d /home/xx/gpu gpu`
添加用户到某个group | `sudo usermod -aG guestgroup username`


## 为用户添加 `root` 权限
### 方法一 修改 `/etc/sudoers` 文件

```
chmod +w /etc/sudoers 
vi /etc/sudoers 
```
- 添加如下内容：
```
gpu ALL=(ALL:ALL) ALL
```

### 方法二 把用户添加到 `sudo` Group

```
sudo usermod -a -G sudo newuser
```

- 为用户 `newuser` 移除 `sudo` 权限
```
sudo deluser newuser sudo
```

## Issues 

- 无 `.bashrc`
  - 从其他用户 copy `.bashrc`/`.profile` 到 当前用户，重新登录即可

## Reference
- [Ubuntu 16.04下添加新用户](https://www.linuxidc.com/Linux/2017-04/142690.htm)


