### Linux LVM 配置过程

---

#### 逻辑卷的创建

1. 首先创建出两个分区 /dev/sdb1 /dev/sdb2，并通过**t**选项设置分区标识为8e

   `fdisk  /dev/sdb`

2. 把sdb1，sdb2两个分区转换为物理卷，并创建一个卷组myvg

   `pvcreate /dev/sdb1 /dev/sdb2`

   `vgcreate myvg /dev/sdb1 /dev/sdb2`

   可以通过vgs和vgdata命令查看卷组的状态

3. 在myvg中划分出1G空间给新的逻辑卷mylv

   lvcreate -L 500M -n mylv myvg

   可以通过lvs和vgdisplay查看逻辑卷的状态

4. 这时mylv就可以和分区一样进行格式化，然后挂载了

   mkfs.ext4 /dev/myvg/mylv

   mount  /dev/myvg/mylv /data

   然后修改fstab

#### 逻辑卷的扩展

当逻辑卷空间不够用的时候，可以对逻辑卷进行扩展

1. 扩展逻辑卷的空间

   lvextend -L +1G /dev/myvg/mylv

2. 同步文件系统

   resize2fs /dev/myvg/mylv

   df -h 检查下

当卷组空间不够用的时候，可以扩展卷组

1. 新建一块分区sdb3,加入myvg中，然后扩展myvg

   vgextend myvg /dev/sdb3

### 逻辑卷的释放

1. 先umount挂载的目录

   umount  /data

2. 检查逻辑卷的剩余空间

   e2fsck -f /dev/myvg/mylv

3. 重新定义文件系统的大小

   resize2fs /dev/myvg/mylv 1G

4. 重新定义逻辑卷的大小

   lvreduce /dev/myvg/mylv 1G

5. 重新mount到目录就可以使用了

   **（注意：文件系统大小和逻辑卷大小一定要保持一致才行。如果逻辑卷大于文件系统，由于部分区域未格式化成文件系统会造成空间的浪费。如果逻辑卷小于文件系统，哪数据就出问题了。）**

### 逻辑卷的删除

1. umount挂载的目录
2. 修改/etc/fstab的信息，否则会影响机器启动
3. lvremove /dev/myvg/mylv
4. vgremove myvg
5. pvremove /dev/sdb1 /dev/sdb2 /dev/sdb3

![](http://note.youdao.com/noteshare?id=b6ea120595eecc514e9a7351e6d8ec2a&sub=9ADFBE24BE2B46F482D60E3811F0C94C)