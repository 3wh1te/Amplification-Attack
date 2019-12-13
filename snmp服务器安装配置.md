

### SNMP服务器安装配置（反射放大攻击实验）

还是因为反射放大攻击那件事，搭个SNMP服务器，可能是最顺利的了，呵呵呵~

#### 一、服务器搭建

1. 安装

   snmp是客户端，snmpd是服务器端

   ~~~
   apt-get install snmp snmpd
   ~~~

2. 配置节点

    修改/etc/snmp/snmpd.conf文件，大概在45行，将下面的两行注释掉：

   ~~~
   view   systemonly  included   .1.3.6.1.2.1.1
   view   systemonly  included   .1.3.6.1.2.1.25.1
   ~~~

   增加一行：

   ~~~
   view   systemonly  included   .1
   ~~~

   样的话，我们就可以获取更多的节点信息，因为如果不这样做，我们能够获取的信息，仅仅是上面两个注释掉的节点所包含的信息。

   ​        修改之后，重启snmp服务，再使用命令观察一下：

   ~~~
   sudo service snmpd restart
   snmpwalk -v 2c -c public localhost .1.3.6.1.4.1.2021.4.3.0
   ~~~

3. 配置MIB库

    修改/etc/snmp/snmp.conf配置文件，将下面这一行注释掉：

   ~~~
   mibs :
   ~~~

4. 配置远程访问

   修改/etc/snmp/snmpd.conf配置文件，大概在15行，将下面一行注释掉：

   ~~~
   agentAddress  udp:127.0.0.1:161
   ~~~

   同时去掉下面这一行的注释（如果没有就加上）：

   ~~~
   #agentAddress udp:161,udp6:[::1]:161
   ~~~

   重启测试一下

   ~~~
   service snmpd restart
   ~~~

![](F:\python_work\amplification attack\pic\1576217802(1).jpg)

#### 二、测试

1. 本地测试

   ~~~
   snmpbulkget -v2c -c public localhost .1.3.6.1.4.1.2021.4.3.0
   snmpwalk -v 2c -c public 115.159.*.* .1.3.6.1.4.1.2021.9.1.6.1
   ~~~

2. 远程测试

   ~~~
   snmpbulkget -v2c -c public 192.168.19.129 .1.3.6.1.4.1.2021.4.3.0
   ~~~

   

#### 三、攻击

使用脚本攻击

脚本：<https://github.com/hacker900123/Amplifier>

#### 参考链接

<https://blog.csdn.net/xpleaf/article/details/51100725>
<https://blog.csdn.net/weixin_36485376/article/details/89438584>