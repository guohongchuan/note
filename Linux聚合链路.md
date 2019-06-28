### Linux 聚合链路

什么是聚合链路？

链路聚合，是指将多个物理网络端口捆绑在一起，成为一个逻辑网络端口，目的是实现增加网络带宽，负载均衡，主备等功能。网卡的链路聚合一般常用的有"bond"和"team"两种模式，"bond"模式最多可以添加两块网卡，"team"模式最多可以添加八块网卡。一般用team模式。

如何配置？

我们以两张网卡 eth0 eth1,team模式,主备策略为例在RHEL7系统下配置网卡链路聚合

示意图如下

![](<https://raw.githubusercontent.com/guohongchuan/picture/master/1561716216(1).png>)

team0就是操作系统使用的逻辑端口，我们需要对team0配置IP地址等操作

根据系统自带配置文件模板	/usr/share/doc/teamd-1.9/example_ifcfgs/1/

修改配置文件，并复制到网卡配置文件目录 /etc/sysconfig/network-scripts

ifcfg-eth1

```shell
DEVICE="eth1"   #网卡的设备名称
DEVICETYPE="TeamPort"
ONBOOT="yes" #设置网卡开机启动
TEAM_MASTER="team0"   #逻辑网卡的名称
```

ifcfg-eth2

DEVICE="eth2"
DEVICETYPE="TeamPort"
ONBOOT="yes"
TEAM_MASTER="team0"

ifcfg-team0

DEVICE="team0"    #逻辑网卡的名称
DEVICETYPE="Team"
ONBOOT="yes"
BOOTPROTO=none
NETMASK=255.255.255.0
IPADDR=192.168.23.11   
TEAM_CONFIG='{"runner": {"name": "activebackup"}}'   

#聚合链路的策略，共四种。广播容错(broadcast)，轮询（roundrobin），主备（activebackup），负载均衡（loadbalance）

重启网络服务  sytemctl restart network

ifconfig 查看网络信息，会发现eth1，eth2，team0的mac地址是相同的。

如果需要新增一张网卡，比如eth3，新增配置文件 ifcfg-eth3 然后重启网络服务即可

常见问题

重启网络服务可能会失败，常见的问题有

配置文件名称 ifcfg-设备名称 设备名称要与配置文件内容 DEDVICE字段一致

多次修改DEVICE，可能会导致mac地址冲突，找到有问题的网卡并关闭，ifconfig 设备名称  down，然后再重启网络服务

关闭networkmanager

systemctl stop NetworkManager







