---
Title | Tools Remote scp
-- | --
Created @ | `2021-10-13T15:16:48Z`
Updated @| `2024-04-06T04:34:29Z`
Labels | ``
Edit @| [here](https://github.com/junxnone/linux/issues/10)

---

# scp
- scp - `secure copy`

```
scp [参数] [源路径] [目标路径]

-r 递归复制整个目录。 
-1 强制 scp 命令使用协议 ssh1 
-2 强制 scp 命令使用协议 ssh2 
-4 强制 scp 命令只使用 IPv4 寻址 
-6 强制 scp 命令只使用 IPv6 寻址 
-B 使用批处理模式（传输过程中不询问传输口令或短语） 
-C 允许压缩。（将-C 标志传递给 ssh，从而打开压缩功能） 
-p 保留原文件的修改时间，访问时间和访问权限。 
-q 不显示传输进度条。 
-v 详细方式显示输出。scp 和 ssh(1) 会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。  
-c cipher 以 cipher 将数据传输进行加密，这个选项将直接传递给 ssh。  
-F ssh_config 指定一个替代的 ssh 配置文件，此参数直接传递给 ssh。 
-i identity_file 从指定文件中读取传输时使用的密钥文件，此参数直接传递给 ssh。  
-l limit 限定用户所能使用的带宽，以 Kbit/s 为单位。   
-o ssh_option 如果习惯于使用 ssh_config(5) 中的参数传递方式，  
-P port 注意是大写的 P, port 是指定数据传输用到的端口号  
-S program 指定加密传输时所使用的程序。此程序必须能够理解 ssh(1) 的选项。
```

## UseCase

UseCase | cmd
-- | --
copy 远程文件夹到本地 | `scp -r username@IP:/test_directory ./`
通过代理服务器连接的远程主机copy | `scp -ro ProxyCommand="ncat --proxy-type socks4 --proxy proxy_ip_url:proxy_port %h %p" -P tgt_ssh_port tgt_username@tgt_IP:~/your_files/ .`
自动输入密码 | `sshpass -p 'password' scp [-r] src_file_path  tgt_file_path`


## Reference
- [scp通过代理proxy代理传输文件](https://www.appblog.cn/2020/04/26/scp%E9%80%9A%E8%BF%87%E4%BB%A3%E7%90%86proxy%E4%BB%A3%E7%90%86%E4%BC%A0%E8%BE%93%E6%96%87%E4%BB%B6/)
- [在shell脚本中scp自动输入密码](https://blog.csdn.net/qq_35975447/article/details/108064795)


