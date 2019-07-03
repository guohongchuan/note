### samba服务配置教程

**什么是samba？**

samba服务是一种可以让Unix-Like平台和Windows共享文件的一种NAS服务

**配置过程**

samba用户名：smb1（确保本机已经有这个用户）

共享目录：/data（确保本机已经有这个目录）

安装samba服务端软件

yum install samba

配置防火墙，允许samba服务通过

firewall-cmd --add-service=samba

firewall-cmd --add-service=samba --permanent

修改配置文件 /etc/samba/smb.conf

添加一段配置（更多详细配置，请参考帮助文档）

```shell
[data]						#共享的名称
comment = Public Stuff			#共享的说明注释
hosts allow = 192.168.12.0/24	#允许访问共享的网段
path = /data					#共享目录的路径
writable = no					#是否可写，如果设置yes，所有用户都可写入，
write list = smb1				#可写入用户的列表
```

修改global中的   workgroup = WORKGROUP ，与windows的工作组相同

配置selinux，修改新建目录的type

chcon -R -t samba_share_t /data

确保smb1用户有对/data的读写权限

重启samba服务并设置开机启动

systemctl restart smb

systemctl enable smb

把用户smb1添加到samba服务，按照提示输入两遍密码

```shell
[root@localhost samba]# smbpasswd -a smb1
New SMB password:
Retype new SMB password:
Added user smb1.
```

**Linux 下挂载共享的目录**

安装samba客户端软件

yum install samba-client

```shell
[root@localhost ~]# smbclient -L //192.168.253.130/data -U smb1
Enter SAMBA\smb1's password:
Sharename       Type      Comment
---------       ----      -------
print$          Disk      Printer Drivers
data            Disk      Public Stuff
IPC$            IPC       IPC Service (Samba 4.8.3)
smb1            Disk      Home Directories
Reconnecting with SMB1 for workgroup listing.
Server               Comment
---------            -------

Workgroup            Master
---------            -------

```

```shell
[root@localhost ~]# smbclient  //192.168.253.130/data -U smb1
Enter SAMBA\smb1's password:
Try "help" to get a list of possible commands.
smb: \> mkdir test
```

验证一下，data目录是否已经共享出来，权限是否正确

挂载前需要先安装cifs相关的软件，yum install cifs*

编辑/etc/fstab文件进行挂载

```shell
//192.168.253.130/data  /mnt                    cifs    defaults,username=smb1,password=123456,multiuser,sec=ntlmssp    0 0
```

mount -a

