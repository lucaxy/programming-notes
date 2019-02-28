### 简介
同步网络时间，时间服务器分层，0层一般是GPS时间，10层是局域网时间服务器  
Linux时间分为系统时间和硬件时间，系统开机读取硬件时间  
NTP服务使用123/udp端口，软件包由NTP和chrony，chrony更新，更智能  
不规范的同步方式是定时任务，每隔5分钟同步一次，时间有跳跃性，可能导致计划任务执行多次，但安全性高  
`ntpupdate -u ntp1.aliyun.com`  
### chrony
安装：`yum install chrony`  
服务端配置：`/etc/chrony.conf`中开启`allow`和`local stratum 10`，即使服务端的时间没有同步也提供服务  
客户端配置：`server`中填入服务端IP即可  
##### 时间查看
`timedatectl status`
##### 同步状态
`chronyc tracking`