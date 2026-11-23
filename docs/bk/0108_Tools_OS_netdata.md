-----

| Title     | Tools OS netdata                                     |
| --------- | ---------------------------------------------------- |
| Created @ | `2019-01-29T07:38:15Z`                               |
| Updated @ | `2024-09-24T01:59:25Z`                               |
| Labels    | \`\`                                                 |
| Edit @    | [here](https://github.com/junxnone/linux/issues/108) |

-----

# netdata web资源监控工具

  - 资源监控工具
      - CPU
      - RAM
      - IO
      - Network

-----

![](media/7c8f5428fe91ec9263bcca95610ab73b79a54157.gif)

## Setup with Docker

    docker run -d --name=netdata \
      -p 19999:19999 \
      -v /proc:/host/proc:ro \
      -v /sys:/host/sys:ro \
      -v /var/run/docker.sock:/var/run/docker.sock:ro \
      --env PGID=999 \
      --cap-add SYS_PTRACE \
      --security-opt apparmor=unconfined \
      netdata/netdata

> 访问 xxx.xxx.xxx.xxx:19999 即可

## Reference

  - [Github](https://github.com/netdata/netdata)
