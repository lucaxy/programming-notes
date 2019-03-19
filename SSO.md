# 单点登陆
### 正常登陆
通过Header中特定的cookie，如：PHPSESSID等，该cookie唯一表示一个会话，通过该cookie可以查询到用户信息  
该cookie也就是Session Id，Session Id的存储方式有很多种，可以是文件，可以是MySQL，也可以是redis  
### 二级域名登陆
设置cookie时将cookie的域名，设置为`.test.local`，这样app1.test.local和app2.test.local都可以获取登陆信息  
```php
//app1.php
session_set_cookie_params(0,'/','.test.local',false,false);
session_start();
$_SESSION['user_id']=502;
echo 'This is app1';

//app2.php
session_set_cookie_params(0,'/','.test.local',false,false);
session_start();
if(!empty($_SESSION['user_id'])){
    echo "My user id is {$_SESSION['user_id']}";
}else{
    echo 'No user id';
}
```
结果如图：  
![](https://picabstract-preview-ftn.weiyun.com/ftn_pic_abs_v3/b5f0a620c22292b1fc12169192d974526f3037d3bf5b14acc7da9f888be5a27c2396d4cae5f7522eda0bcc419e34e640?pictype=scale&from=30113&version=3.3.3.3&uin=730116539&fname=95%5D%292%247EA%7DLC6QQL48%24%7DSPB.png&size=750)  
![](https://picabstract-preview-ftn.weiyun.com/ftn_pic_abs_v3/0666766c5bd6b7ebaf3d3cbd8505f60fef48e1f9d1efe208293765f7ab09a79ca772ae80d0179e8cc67df91f3eb800a9?pictype=scale&from=30113&version=3.3.3.3&uin=730116539&fname=GRF1%24HCPBEK3LB8%5D4%40%7B_%247A.png&size=750)  
### SSO/CAS
上面是一个顶级域名的情况，现实是公司业务线众多，就需要不同域名也能单点登陆，这时就要一个登陆中心，即SSO  
分两种情况：一种是SSO也没有登陆，就需要先登陆SSO，二是SSO已经登陆，无需再次登陆  
```php
//app1.php
session_start();
#⑤验证SSO返回的Ticket是否有效
if(!empty($_GET['token'])){
    #验证方式一般为curl，这里简单md5
    if(md5('app1-'.$_GET['uid'])==$_GET['token']){
        $_SESSION['user_id']=$_GET['uid'];
    }
}
#①判断是否登陆，否则跳到SSO登陆；⑥进行登陆后的操作
if(!empty($_SESSION['user_id'])){
    echo 'My user id is '.$_SESSION['user_id'];
}else{
    header('location:http://sso.test.local/sso.php?app=app1');
}

//sso.php
session_start();
#③SSO登陆
if(!empty($_POST['user_id'])){
    $_SESSION['user_id']=$_POST['user_id'];
}

#如果SSO已经登陆，则③可跳过
#②判断SSO是否登陆，否则弹出登陆框；④SSO登陆后，带上ticket，跳转到原App
if(!empty($_SESSION['user_id'])){
    if($_GET['app']=='app1'){
        $url="http://app1.test.local/app1.php?uid={$_SESSION['user_id']}&token=".md5('app1-'.$_SESSION['user_id']);
    }else if($_GET['app']=='app2'){
        $url="http://app2.test.local/app2.php?uid={$_SESSION['user_id']}&token=".md5('app2-'.$_SESSION['user_id']);
    }
    header("location:$url");
}else{
    echo <<<EOF
    <form method='post'>
        <input type='hidden' name='user_id' value='403'>
        <input type='submit'>
    </form>
EOF;
}

//app2.php，与app1逻辑相同
session_start();
if(!empty($_GET['token'])){
    if(md5('app2-'.$_GET['uid'])==$_GET['token']){
        $_SESSION['user_id']=$_GET['uid'];
    }
}
if(!empty($_SESSION['user_id'])){
    echo 'My user id is '.$_SESSION['user_id'];
}else{
    header('location:http://sso.test.local/sso.php?app=app2');
}
```
结果：  
![](https://picabstract-preview-ftn.weiyun.com/ftn_pic_abs_v3/f3ff3bf0b04b30a4f688f9b35fc41aebfd620b06193b4bacc496286b369db59a75956da8cfc8c8c6d24778a892b5d0a2?pictype=scale&from=30113&version=3.3.3.3&uin=730116539&fname=EBL%28OW_QB%28%7D8Z71FM5A6MZ9.png&size=750)  
![](https://picabstract-preview-ftn.weiyun.com/ftn_pic_abs_v3/2bb1914994b12dfda3553fb139e54f8e076c0e99a2435d00c1a3c00a417f791039463a61b2fdeb0532ddb65d0d8a757a?pictype=scale&from=30113&version=3.3.3.3&uin=730116539&fname=M%25KEA%5B5P%7D%5DY%25PI%25L~%405A%7DLG.png&size=750)  