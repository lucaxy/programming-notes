### IO模型
##### 多道处理模块（MPM）
prefork：一个主进程，多个子进程，每个子进程处理一个请求，使用select的IO模型  
worker：一个主进程，多个子进程，每个子进程生成多个线程，每个线程处理一个请求，也使用select的IO模型  
event：一个主进程，多个子进程，每个进程响应多个请求，使用事件驱动IO或异步IO  

- 同步（synchronous）：  
调用发出，不会立即返回，一旦返回，返回最终结果  
- 异步（asynchronous）：  
调用发出，立即返回消息，返回的并非最终结果，被调用者通过状态、通知机制等通知调用者，或通过回调函数处理结果  
- 阻塞（block）：  
调用结果返回之前，调用者被挂起，得到返回结果之后才能继续  
- 非阻塞（non-block）：  
调用结果返回之前，调用者不会被挂起  

Read具体操作：进程向内核发起请求，内核读取文件到内核内存中，内核拷贝到进程内存中。  
1. 阻塞式IO（blocking IO）  
挂起，直到数据复制到进程内存完成  
2. 非阻塞式IO（nonblocking IO）  
不停轮询，是否准备好，然后阻塞在复制内核空间内存  
3. 复用型IO（IO Multiplexing）（select，poll）  
请求提交给内核中的代理人，第一阶段阻塞在代理人上，代理人可以接受其他请求，第二阶段阻塞在调用上  
4. 事件驱动式IO（signal driven IO）（epoll）  
第一阶段完成，内核通知进程数据准备好，非阻塞，回调，第二阶段依然阻塞，内核复制数据到进程  
边缘触发：只通知一次，以后须回调  
水平触发：多次通知  
5. 异步IO（asynchronous IO）  
第二阶段完成后通知进程，完全非阻塞  
### Apache
2.2不支持同时编译多个MPM模块，rpm包中有三个MPM模块，在/etc/sysconfig/httpd  
httpd -l查看编译的静态模块，httpd -M查看所有模块  
特性：Indexs列出目录，FollowSymLinks符号链接，AllowOverride是否允许使用目录下的.htaccess，性能差一般都不使用  
认证：basic，digest  
htpasswd -c /usr/local/apache/passwd/.htpasswd rbowen
```ini
AuthType basic
AuthName "comment"
AuthUserFile "/etc/httpd/conf.d/.htpasswd"
AuthGroupFile ""
require valid-user
require group GroupName
```
##### 2.4中的访问控制
https://httpd.apache.org/docs/2.4/mod/mod_authz_core.html
RequireAll中条件必须都要满足，RequireAny中满足一个即可，RequireNone满足其中条件的都是禁止的  
其中RequireAny和RequireNone中不允许使用否定条件（require not），三者可以嵌套  
不使用RequireXX标签时，表示RequireAny  
```
<RequireAll>
    #Require all denied
    Require all granted
    Require not ip 10 172.20
    Require not host public.company.com
    Require expr "!(%{QUERY_STRING} =~ /secret/)"
</RequireAll>
```
##### 路径重写
http://httpd.apache.org/docs/current/rewrite/
### Nginx
二次开源版：Tengine（支持动态模块）、Registry  
特性：  
- 模块化  
- 高可靠
- 低内存消耗
- 支持热部署  
- 支持AIO，event，mmap（将文件映射为内存），sendfile  
- 功能丰富：缓存服务器，HTTP服务器（支持fastcgi，uwsgi），http，smtp，pop3的反向代理，负载均衡服务器，SSI及图片裁剪等  
- 支持自定义变量  

sendfile：直接从磁盘发送到网卡，不经过Nginx，仅支持小文件，sendfile64支持稍大的文件  
flv伪流媒体：配合客户端播放器可实现拖拽播放  

#### 常用参数
worker_rlimit_nofile：worker打开最大文件数  
worker_processes：worker数量，支持auto，一般为CPU数减一  
worker_cpu_affinity：CPU绑定，每个hash表示一个worker使用的CPU掩码，如：00000001 00000010  
timer_resolution：时间解析度，可以减少查询系统时间，提高性能  
worker_priority：nice值，值越小优先级越大，-20到19，默认是0，优先级是120  
worker_connections：单个worker最大并发连接数  
error_log：使用debug级别，需要编译时开启调试  
tcp_nodelay：多次小请求合并成一个，off时延时会加大，需要开启Keep-Alive  
tcp_nopush：与tcp_nodelay有相反的效果，on时有利于大文件的传输，需要开启sendfile  
keepalive_timeout：默认75s  
keepalive_disabled：禁用  
##### Nagle算法：  
正常情况下，每发送一个报文，就需要一个确认报文（ACK），且立即发送，启用该算法后，当报文满足【MSS最大报文长度，FIN标识，  
TCP_NODELAY，超时（200ms），所有ACK都已收到】之一，则允许发送，否则仅放入缓冲中，只允许一个未被ACK的包存在于网络，提高网  
络吞吐量，降低了实时性。如果设置了TCP_CORK（tcp_nopush），即使所有ACK都已收到，依然放入缓冲中，等待适时发送，更进一步降  
低了实时性。  
典型Web服务器配置：  
```ini
sendfile   on;
tcp_nopush on;
keepalive_timeout 60;
tcp_nodelay on;
```

##### 防盗链：
```
location ~* \.(jpg|png|gif|jpeg)$ {
    valid_referer none blocked www.baidu.com;
    if($invalid_referer){
        rewrite ^/ http://www.baidu.com/404.html;
    }
}
```

### Web服务器
location：①精确匹配`=`，②非正则`^~`，③正则`~`或`~*`，自上而下，不区分是否带`*`，④一般路径，最长优先，顺序无关  
alias：路径别名，即路径替换  
error_page 404 404.html：定义响应码返回文件  
rewrite：路径重写  
if：条件判断，可以使用正则，-f文件，-d目录，-e文件存在性，-x执行权限  
auth_basic：认证说明  
auth_basic_user_file：用户密码文件  
##### rewrite标志：
- last，最后一个规则，让浏览器重新请求新路径，地址栏不显示新路径，用于内部跳转  
- break，让浏览器重新请求新路径并不再检查该location内的rewrite规则，地址栏不显示新路径，用于内部跳转  
- redirect，以302临时重定向返回新路径，地址栏显示新路径  
- permanent，以301永久重定向返回新路径，地址栏显示新路径
#### 反向代理
 proxy_pass：当location为正则时，后端只能是host，不能加路径，否则视为语法错误    
 proxy_set_header：为后端设置header，例如：X-Real-IP→$remote_addr
 #### 缓存服务器
 proxy_cache_path /path/to/cache/ levels=1:2 keys_zone=one:32m;(http中)
 proxy_cache one;  
 proxy_cache_valid 200 1d;  
 chown www:www /path/to/cache  
 #### 负载均衡
 upstream（http中）  
 proxy_pass  
 #### fastcgi
 fastcgi_pass  
 fastcgi_cache  
 
 