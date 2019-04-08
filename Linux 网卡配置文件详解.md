### Linux 网卡配置文件详解

配置文件路径 /etc/sysconfig/network-scripts/ifcfg-eth0

```shell
DEVICE=eth0               #网络接口名称
TYPE=Ethernet             #网络接口类型
ONBOOT=                   #系统启动时，是否激活  yes：系统启动时激活  no：系统启动时不激活
IPADDR=192.168.30.128     #IP地址
BOOTPROTO=                #系统启动地址协议 none：不使用启动地址协议 dhcp：DHCP动态地址协议 static：静态地址协议 bootp：BOOTP协议
NETMASK=255.255.255.0     #子网掩码
GATEWAY=                  #网关地址
BROADCAST=                #广播地址
HWADDR=                   #MAC地址
PEERDNS=                  #是否指定DNS，如果使用DHCP协议，默认为yes。yes：如果设置DNS1或DNS2，会把DNS设置写入 /etc/resolv.conf  no：不修改/etc/resolv.conf中的DNS
DNS{1,2}=                 #DNS地址
NM_CONTROLLED=            #是否由Network Manager控制该网络接口,修改保存后立即生效，无需重启。yes：由Network Manager控制 no：不由Network Manager控制  （有坑，建议设置为no）
USERCTL=                  #用户权限控制。yes：非root用户允许控制该网络接口 no：非root用户不允许控制该网络接口
IPV6INIT=                 #yes:支持ipv6 no：不支持ipv6
IPV6ADDR=                 #IPv6地址

```

##### 附 Linux修改网卡名称的方法

修改配置文件  vim /etc/udev/rules.d/70-persistent-net.rules

```shell
#PCI device 0x1022:0x2000 (pcnet32)

SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:0c:29:73:cd:d6", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

#PCI device 0x1022:0x2000 (pcnet32)

SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:0c:29:ba:f6:52", ATTR{type}=="1", KERNEL=="eth*", NAME="eth1"

#PCI device 0x1022:0x2000 (pcnet32)

SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="00:0c:29:d2:4e:9b", ATTR{type}=="1", KERNEL=="eth*", NAME="eth3"
```

找到网卡地址对应的记录，修改NAME （注意NAME不可重复，注释掉不需要的记录，并且需要重启操作系统），然后修改网卡配置文件即可