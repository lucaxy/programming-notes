### SSH
SSH协议的开源实现:dropbear,openssh

### OpenSSH
客户端：ssh，scp，sftp  
版本：1中MAC使用CRC-32不安全，2中协商MAC，使用DH密钥交换算法  
配置：/etc/ssh/ssh_config  
用法：`ssh [-p port] [-l user] user@host commond ` 
strictHostKeyChecking第一次登陆检查密钥  
生成密钥对：`ssh-keygen -t rsa -b 2048 -C "user@mail.com" -P 'pass' -f 'file'`  
发送公钥：`ssh-copy-id -i file`  
**密钥登陆，攻破一台机器，所有能免密码登陆的都被攻破**  

服务端：sshd  
多实例：/usr/sbin/sshd -f config -p port
#### 客户端多个私钥配置
方法一：`eval $(ssh-agent -s)`，开启密钥代理，然后，`ssh-add key`添加多个key到缓存  
方法二：config文件中配置，个人的在`~/.ssh/config`  
```ini
Host github.com
    HostName github.com
    IdentityFile ~/.ssh/id_rsa_github
    PreferredAuthentications publickey
    User github_username
```
##### proxy
ssh -D 127.0.0.1:1080 -f -NT root@remote.server -p 22  
-D表示绑定本地端口，-f表示后台运行，-N表示不执行命令，-T表示不开启tty  
这样本地proxy端口1080，执行的命令，将在remote server执行  
另有-L本地端口转发，-R远程端口转发，三个host，有一个中转时使用  
#### dropbear
1. 生成服务端密钥：`dropbearkey -t rsa -f /etc/dropbear/dropbear_rsa_host_key -s 2048`  
`dropbearkey -t dss -f /etc/dropbear/dropbear_dss_host_key`   
2. 测试：`dropbear -p ip:port -F -E`，前台运行并显示错误  
3. 后台运行：`dropbear -p ip:port`  
