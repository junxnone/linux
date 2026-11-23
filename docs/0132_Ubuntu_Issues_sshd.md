---
Title | Ubuntu Issues sshd
-- | --
Created @ | `2024-08-27T08:07:19Z`
Updated @| `2024-08-27T08:07:19Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/132)

---
# Linux 系统过载 无法通过 ssh 连接
## 原因
 
- 目前观测到的一般为内存不足导致的
  - 多线程编译 `make -jn`
  - 运行占用内存较大的应用时


## 解决方案
- 限制 内存使用
  - `ulimit`
- 增加 sshd 进程优先级
  - `nice`
  - `renice` 
