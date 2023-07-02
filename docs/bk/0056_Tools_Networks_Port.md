-----

| Title         | Tools Networks Port                                 |
| ------------- | --------------------------------------------------- |
| Created @     | `2021-03-04T03:10:46Z`                              |
| Last Modify @ | `2022-12-22T02:09:30Z`                              |
| Labels        | \`\`                                                |
| Edit @        | [here](https://github.com/junxnone/linux/issues/56) |

-----

# 端口查看工具

## Reference

  - [Linux
    查看端口占用情况](https://www.runoob.com/w3cnote/linux-check-port-usage.html)

## Tools

  - lsof
  - netstat
  - [nmap](./nmap)

## lsof

| UseCase | Command      |
| ------- | ------------ |
| 查看指定端口  | lsof -i:1200 |
| 查看使用的端口 | lsof -i      |

## netstat

| UseCase | Command                        |
| ------- | ------------------------------ |
| 查看指定端口  | `sudo netstat -anp\|grep 6015` |
