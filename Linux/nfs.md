### 文件共享服务
- 应用层：ftp
- 内核：nfs
- 跨平台：samba（smb，cifs）

网络存储模型：DNS（直接附加存储，如主板上），NAS（网络附加存储），SAN（块级别共享，内核中）  
### FTP
命令连接：文件管理类命令，始终在线  
数据连接：数据传输，按需建立，开启子进程  
服务端软件：wu-ftpd，proftpd，pureftpd，vsftpd，servU  
客户端软件：ftp，lftp，lftpget，wget，curl，filezilla，gftp，flashfxp，cuteftp，axel（多线程下载）  
传输模式：二进制，文本  
##### 主动模式：由服务器创建连接
- 命令连接：client：5xxx→server：21  
- 数据传输：server：20→client：5xxx+1  
由于大多数情况下客户端都安装了防火墙，主动模式很少使用  
##### 被动模式Passive：由客户端创建连接  
- 数据传输：client：5xxx+1→server：随机端口  
随机端口：121，23：121*256+23  
服务端防火墙通过连接追踪开放随机端口  
##### 响应码
- 1xx：信息  
- 2xx：成功  
- 3xx：需要补充信息  
- 4xx：客户端错误  
- 5xx：服务端错误  

##### 认证
用户类型：系统用户，匿名用户，虚拟用户  
nsswitch：名称解析框架，配置文件`/etc/nsswitch.conf`,模块`/lib64/libnss*`,`/usr/lib64/libnss*`  
pam:用户认证框架，配置文件`/etc/pam.conf,/etc/pam.d/*`,模块`/lib64/security/`  
#### vsftp
用户认证配置文件：/etc/pam.d/vsftpd  
主配置文件：/etc/vsftpd/vsftpd.conf，行前不能有空格  
系统用户配置：/etc/vsftpd/ftpusers，user_list  
匿名用户资源目录：/var/ftp/pub  
默认允许匿名用户但不允许上传，系统用户  
不允许修改主目录/var/ftp权限  

##### 配置
匿名用户配置：  
`anon_mkdir_write_enable=YES`  
`anon_upload_enable=YES`  
`anon_other_write_enable=YES`删除权限  
本地用户权限：  
写权限：write_enable  
禁锢到家目录：chroot_local_user  
chroot_list_file=/etc/vsftp/chroot_list  


dirmessage_enable：切换到目录显示附加信息，文件名`.message`  
xferlog_enable：传输日志  
chown_uploads：修改上传文件属主  

ftpusers：禁用的系统用户名  
userlist_enable,userlist_deny,userlist_allow  
max_clients  
max_per_ip  
anon_max_rate单位是字节  
local_max_rate  

虚拟用户权限：  
存储方式：文本，数据库（pam_mysql在epel中）  
guest_enable  
guest_user  
user_config_dir  
在上述目录中创建同名文件，文件中添加anon_开头的相关权限  

#### 其他FTP
FTP为明文协议，传输都是明文  
FTPS：SSL  
SFTP：SSH  
### NFS
4.2开始支持并行的NFS（pNFS），元数据和数据分别存储  
nfsd本地磁盘读取文件  
mountd基于IP的认证  
idmapd文件属主属组映射  
#### 安装
CentOS7：`yum install nfs-utils rpcbind`  
服务端和客户端都要安装，仅服务端需要启动NFS服务，都使用systemctl管理  

portmapper:`rpcinfo -p`  
查看NFS服务器端共享的文件系统：`showmount -e NFSSERVER_IP`

挂载NFS文件系统：`mount -t nfs SERVER:/path/to/sharedfs  /path/to/mount_point`

配置文件/etc/exports：`directory (or file system)   client1(option1, option2) client2(option1, option2)` 
##### 配置选项
- 读写：rw，可读写；ro：只读
- 压缩用户：all_squash：所有用户都压缩为nfsnobody，root_squash，no_root_squash
- 匿名用户：anonuid：一般为默认65534，anongid：匿名用户gid
- 同步：sync，安全高；async，性能高，no_wdelay关闭写延时，需sync

exportfs：维护exports文件导出的文件系统表的专用工具：  
-ar: 重新导出所有的文件系统  
-au: 关闭导出的所有文件系统  
-u FS: 关闭指定的导出的文件系统
##### 挂载选项
`mount`查看所有挂载选项  
开机自动挂载nfs（/etc/fstab）：`192.168.1.100:/shared/upload /shared/upload nfs defaults,_netdev 0 0`
- noexec：禁止运行二进制程序（PHP等需要解释程序的除外）  
- nosuid：禁止SUID特殊权限  
- nodev：禁止设备文件  
- rsize：读缓冲，默认131072
- wsize：写缓冲，默认131072
- defaults：默认选项`rw,  suid, dev, exec, auto, nouser, async, and relatime.`  
- _netdev：防止无网络重复尝试挂载
- noatime：不更新访问时间
- nodiratime：不更新目录访问时间
##### 强制卸载
`umount -lf /mnt`：强制并采用慵懒方式卸载
### samba
跨平台的NFS，文件系统类型CIFS  
端口众多：137/udp，138/udp，139/tcp，445/tcp
NetBIOS：137/138通信，广播方式获取主机和IP，15个字符主机名，WINS是非广播  
服务：nmbd（NetBIOS），smbd（cifs），winbindd（域）  
##### 客户端
探测：smbclient -L HOST -U USERNAME  
登陆：smbclint //SERVER/shared_name -U USERNAME
挂载：`mount -t cifs //SERVER/shared_name  /mount_point -o username=USERNAME,password=PASSWORD`
##### 服务端
仅能使用系统用户，密码为samba自定义密码，添加`smbpasswd -a USERNAME`  
安装：`yum -y install samba`  
服务：nmb，smb  
配置文件：/etc/samba/smb.conf  
windows中工作组默认为workgroup，修改配置，mygroup换成workgroup  
smbpasswd：-d禁用，-e启用，-x删除  
自定义共享
``` ini
[shared_name]
path = /path/to/share_directory
comment = Comment String
guest ok = {yes|no}
public = {yes|no}
writable = {yes|no}
read only = {yes|no}
write list = +GROUP_NAME #组内用户可写
```
testparm语法检测  
用户权限和smb权限都有才能写  
samba-swat