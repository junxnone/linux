-----

| Title         | Security PAM                                        |
| ------------- | --------------------------------------------------- |
| Created @     | `2019-08-27T02:14:48Z`                              |
| Last Modify @ | `2022-12-21T10:19:02Z`                              |
| Labels        | \`\`                                                |
| Edit @        | [here](https://github.com/junxnone/linux/issues/65) |

-----

# PAM - Pluggable Authentication Module

## Reference

  - [CentOS -
    系统认证机制pam](https://baijiahao.baidu.com/s?id=1616480029377047639&wfr=spider&for=pc)
  - [PAM认证机制 -
    momenglin](https://www.cnblogs.com/momenglin/p/8486069.html)
  - [PAM认证机制 -
    zhangym199312](https://blog.csdn.net/zhangym199312/article/details/78021998)

## Brief

  - **PAM - Pluggable Authentication Module - 可插入式授权管理模块**

Sun公司1995年开发的一种与认证相关的通用框架机制，PAM只关注如何为服务验证用户的API，通过提供一些动态链接库和一套统一的API，将系统提供的服务和该服务的认证方式分开；PAM只是一个框架而已，自身不做认证。

**PAM应用在许多程序与服务上:**

  - 登录程序(login、su)的PAM身份验证（口令认证、限制登录）
  - passwd强制密码
  - 用户进程实时管理
  - 向用户分配系统资源

**API**

  - PAM-API 调用认证接口
  - PAM-SPI 提供认证接口

![image](media/fe2150f23fb85f8389d07290e56694c43da0cd7b.png)
![image](media/18461cd71e46b87c2e15dba66ddcda7c685daedc.png)
