### 防火墙
隔离工具，分为主机防火墙和网络防火墙，工作于主机或网络的边缘，根据事先定义的规则过滤报文。  
主机防火墙在内核空间TCP/IP协议栈中实现功能，在多个位置设置卡点。  
网络防火墙中入侵检测系统（包括HIDS主机入侵检测和NIDS网络入侵检测）通知防火墙从而形成IPS（入侵防御系统），  
另外有蜜罐（HoneyPot）  
### iptables/netfilter
规则管理工具框架包括四表（filter，nat，mangle，raw）五链（INPUT，FORWARD，OUTPUT，PREROUTING，POSTROUTING，即位置）  
#### 四表
- filter，过滤到本机的报文
- nat，SNAT源地址转换，DNAT目标地址转换，PNAT端口转换  
- mangle，拆分报文修改并封装  
- raw，关闭nat中连接追踪功能  
#### 五链  
- PREROUTING，路由选择之前进行，如DNAT
- INPUT
- FORWARD
- OUTPUT 
- POSTROUTING,发出报文之前，如SNAT  
##### 数据报文流程
- 进入：PREROUTING, INPUT
- 出去：OUTPUT, POSTROUTING  
- 转发：PREROUTING, FORWARD, POSTROUTING  
##### 各功能中的链
- filter：INPUT，FORWARD，OUTPUT
- nat：PREROUTING，OUTPUT，POSTROUTING
- mangle：PREROUTING,INPUT，FORWARD,OUTPUT，POSTROUTING 
- raw：PREROUTING，OUTPUT
##### 添加规则时的法则
- 同类规则，匹配范围小的放上面
- 不同类规则，匹配报文频率大的放上面
- 设置默认策略，黑名单和白名单
- 必要时可将一条规则描述的多个规则合并为一个  
##### 功能优先级次序：raw，mangle，nat，filter
### 命令
`iptables -t table 子命令 链 规则 -j 目标`
自定义链只能通过在内置链中引用生效  
计数器，有pkts和bytes，默认是filter表  
链管理：  
`-F` flush清空  
`-N` 创建自定义链  
`-X` 删除自定义的空链  
`-Z` 置零计数器  
`-P` 设置默认策略(ACCEPT,DROP,REJECT)  
`-E` 重命名自定义链，引用计数不为0的不能重命名，也不能删除  
规则管理：  
`-A`追加  
`-I`插入，在指定条目前面  
`-D`删除，指定条件或编号  
`-R`替换指定规则  

查看：  
`-L` 查看，`-n`数字格式不反解，`-v`,`-vv`,`-vvv`详细信息，`--line-numbers`显示规则编号，`-x`计数器精确值  

匹配条件：  
`-s`源地址，`!`取反  
`-d`目标地址，`!`取反  
`-p`协议，tcp,udp,icmp  
`-i`流入接口，仅能用于INPUT，PREROUTING，FORWARD  
`-o`流出接口，仅能用于OUTPUT，FORWARD，POSTROUTING  

扩展匹配：  
`-m`扩展名，`-p`指明协议后可省略`-m`,例如：`-m tcp --dport 22`源端口22  
`-m tcp --sport 22`目标端口22  `--tcp-flags LIST1 LIST2`:检查LIST1中标志位，其中LIST2中必须为1，其余为0  
TCP六种标志位（SYN,ACK,FIN,RST,PSH,URG），例如` --tcp-flags SYN,ACK,FIN,RST SYN`第一次握手，简写为`--syn`  
`-m udp`udp,`--sport`,`--dport`  
`-m icmp`icmp,`--icmp-type`可用数字表示，0表示echo-reply，8表示echo-request  
目标：  
`-j`ACCEPT,DROP,REJECT,RETURN,REDIRECT(端口重定向)，LOG记录日志，MARK做防火墙标记，DNAT目标地址转换，  
SNAT源地址转换，MASQUERADE地址伪装，或自定义链规则等  

#### 扩展
`rpm -ql iptables | grep "[[:lower:]]\+\.so$"`,大写为target，小写是可使用的扩展  
`man iptables-extension`Centos7  
`multiport`最多15个离散端口，`--sports 22,1024:1080`22端口和1024到1080的所有端口，`--dports`  
`iprange`,`--src-range`,`--dst-range`指定ip范围，用`-`分割  
`string`检查报文中字符串，`-algo`比对算法，bm和kmp，`--string patern`,`--hex-string`,`--from --to`  
`time`,根据报文到达时间和指定时间进行匹配`--datestart`,`--datestop`,`--timestart`,`--timestop`  
`connlimit`根据客户端IP进行并发连接数限制，`--connlimit-above`,`--connlimit-upto`  
`limit`收发报文速率，令牌桶过滤器，`--limit`,`--limit-burst`  
`state`跟踪连接请求，检查连接的状态，修改`/proc/sys/net/nf_conntrack_max`,查看已追踪的连接`/proc/net/nf_conntrack`  
常用状态：NEW,ESTABLISHED,RELATED(ftp),INVALID,对于出站不放行NEW可以防范反弹式木马,  
`/proc/sys/net/netfilter/`不同协议追踪超时设置
##### ftp防火墙规则
- 装载nf_conntrack_ftp模块
`/lib/modules/KERNEL_VERSION/kernel/net/netfilter/`,依赖nf_conntrack  
`modprobe nf_conntrack_ftp`  
`lsmod`
- 放行命令端口的NEW，ESTABLISHED
`iptables -A INPUT -d localIP -p tcp --dport 21 -m state NEW,ESTABLISHED -j ACCEPT`  
- 放行数据端口的RELATED，ESTABLISHED
`iptables -A INPUT -d localIP -p tcp -m state RELATED,ESTABLISHED -j ACCEPT`  
- 放行出站端口
`iptables -A OUTPUT -s localIP -p tcp -m state ESTABLISHED -j ACCEPT` 
##### 保存规则
`iptables-save > file`  
`iptables-restore < file`
CentOS6中`service iptables save`相当于`iptables-save > /etc/sysconfig/iptables`  
`service iptables restart`  
CentOS7中`yum install  iptables-services`  
并非真的服务，只是重新加载配置文件或清空配置文件  
firewalld:https://www.ibm.com/developerworks/cn/linux/1507_caojh/  
##### 日常规则
```shell
/sbin/iptables -I INPUT 1 -i lo -j ACCEPT
/sbin/iptables -I INPUT 2 -m state --state ESTABLISHED,RELATED -j ACCEPT
/sbin/iptables -I INPUT 3 -p tcp -m multiport --dports 22,80,443 -j ACCEPT
/sbin/iptables -I INPUT 4 -p tcp --dport 3306 -j DROP
/sbin/iptables -I INPUT 5 -p icmp -m icmp --icmp-type 8 -j ACCEPT
```
##### 网络防火墙
开启核心转发：`sysctl -w net.ipv4.ip_forward=1`  
开放内网ftp服务，同主机防火墙  
开放samba服务：137/udp，138/udp，139/tcp，445/tcp  
开放dns服务：53/udp（INPUT，OUTPUT）  
nat为安全而生，在网络层和传输层实现  
`iptables -t nat -A POSTROUTING -s 192.168.20.0/24 ! -d 192.168.20.0/24 -j SNAT --to-source 172.16.100.9`  
`iptables -t nat -A PREROUTING -d 172.16.100.9 -p tcp --dport 80 -j DNAT --to-destination 192.168.20.2:8080`  
地址伪装，自动替换合适IP（ADSL）  
`iptables -t nat -A POSTROUTING -s 192.168.20.0/24 ! -d 192.168.20.0/24 -j MASQUERADE`  
ip地址工作在内核空间，能访问其中一个IP地址，其他的也能访问
proxy在应用层实现  
###### tcp_wrapper
类似于iptables，工作于库调用，修改配置即时生效，必须调用libwrap，并且只能控制tcp访问  
vsftp，sshd，telnet，denyhosts等可用  
判断有没有调用libwrap，动态编译使用ldd查看依赖，例如`` ldd `which sshd` ``，静态编译，  
使用strings查找hosts.allow和hosts.deny  
判断流程：先读取判断hosts.allow再读取判断hosts.deny  
语法：`daemon_list: client_list [:options]`  
daemon_list,应用程序程序名，逗号分隔，ALL表示所有  
client_list,客户端IP，`172.16.`等同于172.16.0.0/255.255.0.0，KNOWN表示已知，UNKNOWN未知，PARANOID，客户端正反解不一致  
ALL表示所有客户端  
EXCEPT排除：在hosts.allow中`vsftp:172.16. EXCEPT 172.16.100.0.255.255.255.0 EXCEPT 172.16.100.1`，172.16.100.1可以访问  
spawn: 启动一个额外命令  
``in.telnetd: ALL : spawn /bin/echo `date` login attempt from %c to %s, %d >> /var/log/telnet.log ``  
`%s`服务端IP，`%c`客户端IP，`%d`程序名称  
deny：allow文件中拒绝；allow:deny文件中allow  
`vsftp:172.16.:deny`

#### 无公网IP虚机上外网
原理：通过SNAT代理上网，即日常局域网通过网关上网  
1. 网关关闭firewalld，并禁止开机启动，打开iptables服务  
2. 打开核心转发：net.ipv4.ip_forward=1，文件：/etc/sysctl.conf，命令：sysctl -p  
3. 添加规则：`iptables -t nat -A POSTROUTING -o eth0 -s 192.168.1.0/24 -j SNAT --to 192.168.1.1`  
4. 客户端添加路由：`route add default gw 192.168.1.1`  

注意事项：  
- 云上关闭虚机的MAC与IP绑定  
- 网关与客户机要在同一个安全组  