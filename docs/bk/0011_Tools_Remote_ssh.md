-----

| Title         | Tools Remote ssh                                    |
| ------------- | --------------------------------------------------- |
| Created @     | `2021-10-11T03:46:20Z`                              |
| Last Modify @ | `2022-12-21T15:00:40Z`                              |
| Labels        | \`\`                                                |
| Edit @        | [here](https://github.com/junxnone/linux/issues/11) |

-----

# ssh 远程管理工具

  - 一些 Linux OS 安装完后默认没有安装 `openssh-server`, 此时是不能通过 ssh 远程连接到机器的.

## Install `openssh-server`

    sudo apt install openssh-server

## UseCase

| Usecase | Cmd                         |
| ------- | --------------------------- |
| 查看状态    | `sudo systemctl status ssh` |
| 设置开机自启动 | `sudo systemctl enable ssh` |
