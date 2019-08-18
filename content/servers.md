# 少院服务器工作手册(2015级工作记录)
作者：徐弈
文档链接：[gitlab](https://git.ustclug.org/turin_turambar/scgy-tech-handbook/edit/xuyi-servers/servers.md)
## 主控室硬件概述
### 常用主机
|  主机名   | ip地址                                            | 操作系统 | 位置和外观                                    | 主要功能                                   | 备注                      |
| :-------: | :------------------------------------------------ | :------- | :-------------------------------------------- | :----------------------------------------- | :------------------------ |
| scgy-conn | 202.38.70.99/24 202.38.70.159/24 202.38.70.181/24 | Debian 7 | Dell旧主机，桌上                              | 各类网页服务，反向代理，短信大炮指定服务器 |                           |
|  scgy-gw  | 202.38.70.7/24 202.38.70.12/24 192.168.0.2/22     | Debian 6 | 旧DEll主机，靠里的机柜的最下层，从外往里第3个 | 少院网关，一些端口转发                     |                           |
|  scgysu   | 202.38.70.1/24                                    | Debian 8 | 刀片机，靠内的机柜的第二层                    | 少院新FTP                                  | kvm编号：03               |
|    vps    | 202.38.70.19/24                                   | Debian 7 | Dell旧主机，桌上                              | Xen VPS服务器                              |                           |
|  newscgy  | 192.168.3.115/22                                  | Debian 8 | 联想新主机                                    | 待建的网页服务，以及其他功能               | 70.7的6022端口有ssh的转发 |

*更多服务器的信息参见[技术部wiki](http://ourscgy.ustc.edu.cn/tech/index.php?title=%E6%9C%8D%E5%8A%A1%E5%99%A8)，笔者不很熟悉的虚拟机和windows server未在上面列出*
### 其他硬件设备
|      名称      | 特征与位置                                            | 备注                                                         |
| :------------: | :---------------------------------------------------- | :----------------------------------------------------------- |
|      空调      | 台式空调                                              | 空调可能会坏并显示E1，此时应该打空调上的电话维修，然后找代镭报销 |
|     交换机     | 桌子旁边的机柜上有一个，FreeShell的黑色机柜里面有一个 | 设置交换机时用网线连上，手动设置ip到交换机的同一网段，然后用浏览器登录，帐号密码在交换机上 |
|     云存储     | 桌子底层的盒子                                        | ip、帐号及密码在机器上                                       |
| 爱乐协会硬盘盒 | 桌子中间层的盒子                                      | 接在vps虚拟机gewu上，主机信息参见[技术部wiki](http://ourscgy.ustc.edu.cn/tech/index.php?title=%E6%9C%8D%E5%8A%A1%E5%99%A8) |
|      KVM       | 里面的机柜里                                          | 按右边的键切换，**请不要按左边的键**，据说会锁定KVM          |

## 少院网络服务手册
### 网页服务
目前少院提供的网页服务基于经典的LNMP架构，即Linux+Nginx+MySQL+php。
大部分网页位于scgy-conn（202.38.70.99）上，部分位于scgysu（70.1）上并在70.99上做反向代理。
网页的日常维护工作包括新建或者设置服务，对应于新建一个nginx server配置文件。
下面提供了两个nginx配置文件的实例。
```
# 一个典型nginx站点的配置文件
server {
	listen 80; #监听的端口
	listen [::]:80;
	server_name scgypf.ourscgy.ustc.edu.cn; # 服务的域名，用于分辨不同的服务
	access_log /var/log/nginx/scgypf/access.log; # 访问日志位置
	error_log /var/log/nginx/scgypf/error.log; # 错误日志位置
	root /srv/scgypf; # 服务的根目录，即搜索http请求的资源的根路径
	location / { # 指定请求/目录资源的搜索路径
		try_files $uri $uri/ /index.php/$args;
		index index.html; # 指定以http://server_name访问时返回的默认页面
	}
	location ~ \.php { # 指定请求php资源时nginx的行为
		fastcgi_pass 127.0.0.1:9000; # 将php请求转接给php解释服务
		include fastcgi_params;
	}
}
```

```
# 一个经典的nginx反向代理配置文件
server {
	listen 80;
	listen [::]:80;
	server_name tour.ourscgy.ustc.edu.cn;
	access_log /var/log/nginx/scgy-tour/access.log;
	error_log /var/log/nginx/scgy-tour/error.log;
	location / {
		proxy_pass http://202.38.70.1:8001; # 将请求转接给另一台主机
		include /etc/nginx/proxy_params;
	}
}
```
对上面的例子的几点说明

+ nginx主配置文件位于/etc/nginx/nginx.conf，但一般不做修改，
  添加或编辑站点时修改/etc/nginx/sites-available/目录下的文件。
  一个站点对应一个文件。
  配置完成后用软链接链接到/etc/nginx/sites-enabled/目录下以启用。
  此时还需要让nginx重新载入配置文件以生效。链接和配置文件载入的命令如下。
```
    ln -s /etc/nginx/sites-available/site-example /etc/nginx/sites-enabled/ # 软链接
    # 下面的命令二选一，用于使nginx重新载入配置文件
    /etc/init.d/nginx reload # 可用于sysVinit和systemd
    systemctl reload nginx # 仅用于systemd
    # 可用start，stop，restart，status替换reload执行开始服务，停止服务，重启服务和查看服务运行状态
```
+ 少院的域名为ourscgy.ustc.edu.cn，dns解析允许泛解析，也可以将xxx.ourscgy.ustc.edu.cn，和yyy.ourscgy.ustc.edu.cn分开。dns的设置位于[https://www.cloudxns.net/](https://www.cloudxns.net/)。
  cloudxns还可以设置主机监控。**请记住这一工具。**

### ftp 服务
ftp服务用于资料的存贮和共享，
技术部提供的ftp服务主要有面向学生会内部和面向其他社团两种。
详情见下表。

|    用户    | 域名 (http访问)                                              | 主机ip (ftp访问) | 用户名    | 备注                         |
| :--------: | :----------------------------------------------------------- | :--------------- | :-------- | :--------------------------- |
|    少院    | N/A                                                          | 202.38.70.105    | scgy      | 请迁移到newftp               |
|    少院    | [http://202.38.70.1](http://202.38.70.1)  [http://ourscgy.ustc.edu.cn/newftp](http://ourscgy.ustc.edu.cn/newftp) | 202.38.70.1      | scgy-ftp  | 新ftp，网页在70.99上做的反代 |
| 少院技术部 | N/A                                                          | 202.38.70.105    | scgytech  |                              |
|   格物社   | N/A                                                          | 202.38.70.99     | anonymous | 70.99的默认ftp               |
|  爱乐协会  | N/A                                                          | 202.38.70.105    | classic   | 详情请与爱乐协会负责人联系   |

上述大部分ftp服务的信息可以在[技术部wiki](http://ourscgy.ustc.edu.cn/tech/index.php?title=FTP)找到。
架设在虚拟机上的ftp服务采用proftp，可以根据用户名提供不同的服务，
请自行阅读相关文档。
这类ftp服务的用户和密码信息存储在202.38.70.134上的mysql数据库中。
其他的ftp服务采用vsftp。
*建议之后的技术部成员将少院旧的ftp服务中的文件迁移到新的上，推行新ftp*

### iptables 简明指导
iptables时用于控制linux内核对不同ip协议网络包采取的行为的工具，可用于防火墙和端口转发。
具体的原理和详细的使用指南不在这里介绍，可以自行参考网上的材料。
这里同样以少院主控室的服务器为例介绍一些常用的iptables命令。
少院大量使用iptables的主要是网关202.38.70.7，因为需要屏蔽一些ip并对一些内网服务器做端口转发。
#### 屏蔽ip
用```iptables -L```可以列出当前的iptables规则，在202.38.70.7上执行结果如下

```
tur@scgy-gw:~$ sudo iptables -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
fail2ban-ssh  tcp  --  anywhere             anywhere            multiport dports ssh

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     tcp  --  anywhere             wlt.ustc.edu.cn     tcp dpt:www reject-with icmp-port-unreachable
REJECT     tcp  --  anywhere             202.38.70.12        tcp dpt:3389 reject-with icmp-port-unreachable

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain LOG_REJECT (0 references)
target     prot opt source               destination

Chain fail2ban-ssh (1 references)
target     prot opt source               destination
RETURN     all  --  anywhere             anywhere
```
值得注意的是Chain FORWARD中的第一行，这表示内核在做包的转发时如果判断包的目的地(destination)是网络通，
这一转发会被拒绝(REJECT)，然后返回给发送包的主机一个icmp-port-unreachable报文。
这是由于少院网关采用长期网络通，如果内网用户又通过网关登录了网络通会导致长期网络通被踢掉。

用```iptables -S```命令可以以命令的形式在屏幕上打印出当前的规则，如下所示。

```
tur@scgy-gw:~$ sudo iptables -S
-P INPUT ACCEPT
-P FORWARD ACCEPT
-P OUTPUT ACCEPT
-N LOG_REJECT
-N fail2ban-ssh
-A INPUT -p tcp -m multiport --dports 22 -j fail2ban-ssh
-A FORWARD -d 202.38.64.59/32 -p tcp -m tcp --dport 80 -j REJECT --reject-with icmp-port-unreachable
-A FORWARD -d 202.38.70.12/32 -p tcp -m tcp --dport 3389 -j REJECT --reject-with icmp-port-unreachable
-A fail2ban-ssh -s 50.56.80.27/32 -j DROP
-A fail2ban-ssh -j RETURN
```
由此可以看出其实我们要实现上面说的对wlt.ustc.edu.cn的屏蔽我们只需要以root权限执行
```
iptables -A FORWARD -d 202.38.64.59/32 -p tcp -m tcp --dport 80 -j REJECT --reject-with icmp-port-unreachable
```

需要知道的另一组命令是```iptables-save```和```iptables-restore```，分别用于保存设置和读取设置。
*iptables是不会自动存储你刚添加的rule的。*
更改后请用输出重定向把配置文件保存到/etc/iptables/rules，并把原文件重命名为rules+日期

#### NAT 转发
NAT是挺有趣的一个机制，路由器的一个核心功能就是NAT转发，有兴趣的同学可以自己去了解。
这里介绍一下怎么用NAT表实现端口转发。

首先之前的```iptables -L```命令打印出的是默认的filter table，可以加上-t选项打印出其他table。
比如如下命令
```
tur@scgy-gw:~$ sudo iptables -L -t nat
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DNAT       tcp  --  anywhere             202.38.70.7         tcp dpt:8889 to:192.168.0.253:3389
DNAT       tcp  --  anywhere             202.38.70.7         tcp dpt:8839 to:192.168.3.94:3389
DNAT       tcp  --  anywhere             202.38.70.7         tcp dpt:8890 to:192.168.0.253:8080
DNAT       tcp  --  anywhere             202.38.70.7         tcp dpt:1688 to:202.38.64.111:1688
DNAT       tcp  --  anywhere             202.38.70.7         tcp dpt:6022 to:192.168.3.115:22
DNAT       tcp  --  anywhere             202.38.70.7         tcp dpt:http-alt to:192.168.3.50:80

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
SNAT       tcp  --  192.168.0.0/22       192.168.0.253       tcp dpt:3389 to:192.168.0.2
SNAT       tcp  --  192.168.0.0/22       192.168.3.94        tcp dpt:3389 to:192.168.0.2
SNAT       tcp  --  192.168.0.0/22       192.168.0.253       tcp dpt:http-alt to:192.168.0.2
SNAT       all  --  192.168.0.0/22      !192.168.0.0/22      to:202.38.70.7
SNAT       all  --  10.0.0.0/24         !10.0.0.0/24         to:202.38.70.7
SNAT       tcp  --  anywhere             202.38.64.111       tcp dpt:1688 to:202.38.70.7
SNAT       tcp  --  192.168.0.0/22       192.168.3.115       tcp dpt:ssh to:192.168.0.2
SNAT       tcp  --  192.168.0.0/22       192.168.3.50        tcp dpt:www to:192.168.0.2

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
这里打印出的是nat table。其中
```
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
DNAT       tcp  --  anywhere             202.38.70.7         tcp dpt:6022 to:192.168.3.115:22
Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
SNAT       tcp  --  192.168.0.0/22       192.168.3.115       tcp dpt:ssh to:192.168.0.2
```
实现了把内网的newscgy服务器的ssh服务端口映射到202.38.70.7的6022端口，
也就是说把指向202.38.70.7:6022的包转发到192.168.3.115:22。
出于某种原因，仅执行下面命令添加第一条rule会造成内网无法访问192.168.3.115:22。
```
iptables -A PREROUTING -d 202.38.70.7/32 -p tcp -m tcp --dport 6022 -j DNAT --to-destination 192.168.3.115:22
```
所以必须添加第二条rule。具体原因感兴趣的可以自己了解。