---
Title | Tools OS netdata
-- | --
Created @ | `2019-01-29T07:38:15Z`
Updated @| `2024-09-24T01:59:25Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/108)

---
# netdata web资源监控工具
- 资源监控工具
  - CPU
  - RAM
  - IO
  - Network

---

![](https://user-images.githubusercontent.com/1153921/113440964-449c2180-93a2-11eb-9664-663afa1257a8.gif)


## Setup with Docker

```
docker run -d --name=netdata \
  -p 19999:19999 \
  -v /proc:/host/proc:ro \
  -v /sys:/host/sys:ro \
  -v /var/run/docker.sock:/var/run/docker.sock:ro \
  --env PGID=999 \
  --cap-add SYS_PTRACE \
  --security-opt apparmor=unconfined \
  netdata/netdata
```
> 访问 xxx.xxx.xxx.xxx:19999 即可


## Reference
- [Github](https://github.com/netdata/netdata)
