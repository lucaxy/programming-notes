### 安全
攻击方法：窃听，伪装，重放，篡改，拒绝服务  
安全机制：加密，数字签名，访问控制，数据完整，认证交换，流量填充，路由控制，公证  
数据保密：连接保密，无连接保密（UDP），选择域保密，流量保密  
算法协议：对称加密，公钥加密，单向加密（哈希），认证协议  
公钥加密：公钥加密的数据只能，与之配对的私钥解密，反之亦然，常用于数字签名（加密哈希），密钥交换（对称密钥），数据加密  
DH密钥交换：  
1. 协商一致的大素数和模数，  
2. 各自使用一个隐私数通过求次方取模发给对方，  
3. 获得对方发送的数字，用自己隐私数字求次方，即双方一直的密钥  

dhparam：用于快速计算出安全素数p，即q是素数，同时`p=2q+1`依然是素数，这样的p就是安全素数  
### openssl
ssl/tls历史版本：ssl2.0，ssl3.0，tls1.0，tls1.1，tls1.2，tls1.3  
三大组件：openssl命令行工具，libcrypto加密库，libssl协议库  
HTTPS通讯建立大致过程：  
1. TCP建立，以及协商SSL版本，加密算法及密钥长度  
2. 服务器发送公钥及客户端验证服务器公钥  
3. 通过RSA或DH算法交换初始密钥，以及算出主通信密钥（还有CBC的IV，报文效验码密钥等）  
4. 开始HTTP通信  

#### 命令
enc加密，-e加密，-d解密  
dgst数字签名，`openssl dgst -md5 file`  
passwd生成密码，`openssl passwd -1 -salt SALT`,md5加密  
rand生成随机数，`openssl rand -hex 4`，8位16进制数  
genrsa生成rsa私钥，`openssl genrsa -out private.key 2048`  
`(umask 077;openssl genrsa -out private.key 2048)`，同时修改权限  
从私钥获取公钥，`openssl rsa -in private.key -pubout`  
#### 生成强密码
random与urandom区别：urandom伪随机数，熵池用尽，软件算法生成，非阻塞  
`tr -dc A-Za-z0-9_ < /dev/urandom | head -c 32 | xargs`  
#### 私有CA
配置文件：`/etc/pki/tls/openssl.conf`  
CA目录：`/etc/pki/CA`  
数据库文件：index.txt  
序列号：serial  
吊销序列号：crlnumber  

1. 创建CA证书  
openssl genrsa -out ca.key 2048  
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt -days 36500  
2. 签发证书  
openssl genrsa -out server.key 2048  
openssl rsa -in server.key -pubout -out server.pem  
openssl req -new -key server.key -out server.csr  
openssl x509 -req -CA ca.crt -CAkey ca.key -CAcreateserial -in server.csr -out server.crt -days 36500  

方法二（需要在默认位置生成CA证书）：  
openssl req -new -x509 -key ca.key -out ca.crt -days 36500  
openssl ca -in server.csr -out server.crt -days 36500  