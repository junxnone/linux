-----

| Title     | Ubuntu IncreaseSwap                                 |
| --------- | --------------------------------------------------- |
| Created @ | `2022-08-17T08:56:29Z`                              |
| Updated @ | `2023-06-28T06:45:07Z`                              |
| Labels    | \`\`                                                |
| Edit @    | [here](https://github.com/junxnone/linux/issues/83) |

-----

# 增加 Swap 分区

  - 内存比较小的机器在运行大的应用时，容易内存不足卡死或者 OOM
  - 最好的办法当然是物理解决方案 - **加内存**，容易的workaround 方式就是增加 `swap`
  - 增加 swap 会导致整体性能下降，毕竟数据存储在硬盘上读写速度比在内存中不是慢一点半点

## Steps

| Steps & Description  | Command                          |
| -------------------- | -------------------------------- |
| 查看 swap              | `free -h`                        |
| 关闭 swap              | `sudo swapoff`                   |
| 删除旧 swap             | `sudo rm /swapfile`              |
| 查看硬盘空间，决定合适的 swap 大小 | `df -h`                          |
| 创建分区                 | `sudo fallocate -l 8G /swapfile` |
| 查看创建的分区              | `ls -lh /swapfile`               |
| 更改权限                 | `sudo chmod 600 /swapfile`       |
| 创建 swap 分区           | `sudo mkswap /swapfile`          |
| 打开 swap 功能           | `sudo swapon /swapfile`          |
| 查看 swap 状态           | `sudo swapon --show`             |

## Reference

  - [Ubuntu 20.04增加SWAP分区](https://blog.csdn.net/weixin_37532614/article/details/119239715)
