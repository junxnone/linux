-----

| Title     | Tools OS Port                                        |
| --------- | ---------------------------------------------------- |
| Created @ | `2024-09-23T14:51:25Z`                               |
| Updated @ | `2024-09-23T14:51:25Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/133) |

-----

# 端口占用

## lsof

    sudo lsof -i:5173

    COMMAND      PID USER   FD   TYPE   DEVICE SIZE/OFF NODE NAME
    docker-pr 778724 root    4u  IPv4 13546779      0t0  TCP *:5173 (LISTEN)
    docker-pr 778731 root    4u  IPv6 13573422      0t0  TCP *:5173 (LISTEN)

## netstat

    netstat -tunlp|grep 5173

    tcp        0      0 0.0.0.0:5173            0.0.0.0:*               LISTEN      -
    tcp6       0      0 :::5173                 :::*                    LISTEN      -
