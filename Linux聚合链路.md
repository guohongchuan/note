Linux 聚合链路

什么是聚合链路？

链路聚合，是指将多个物理网络端口捆绑在一起，成为一个逻辑网络端口，实现增加网络带宽，负载均衡，主备等功能。网卡的链路聚合一般常用的有"bond"和"team"两种模式，"bond"模式最多可以添加两块网卡，"team"模式最多可以添加八块网卡。

如何配置？

我们以两张网卡 eth0 eth1,team模式,主备策略为例在RHEL7系统下配置网卡链路聚合

示意图如下



根据系统自带配置文件模板	/usr/share/doc/teamd-1.9/example_ifcfgs/1/

修改成如下内容

ifcfg-eth1

DEVICE="eth1"
DEVICETYPE="TeamPort"
ONBOOT="no"
TEAM_MASTER="team_test0"   #逻辑端口的名称

ifcfg-eth2

DEVICE="eth2"
DEVICETYPE="TeamPort"
ONBOOT="no"
TEAM_MASTER="team_test0"

ifcfg-team_test0

DEVICE="team_test0"
DEVICETYPE="Team"
ONBOOT="no"
BOOTPROTO=none
NETMASK=255.255.255.0
IPADDR=192.168.23.11
TEAM_CONFIG='{"runner": {"name": "roundrobin"}}'



