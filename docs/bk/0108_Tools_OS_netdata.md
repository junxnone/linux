-----

| Title         | Tools OS netdata                                     |
| ------------- | ---------------------------------------------------- |
| Created @     | `2019-01-29T07:38:15Z`                               |
| Last Modify @ | `2022-12-22T08:04:37Z`                               |
| Labels        | \`\`                                                 |
| Edit @        | [here](https://github.com/junxnone/linux/issues/108) |

-----

## Reference

## Breif

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
