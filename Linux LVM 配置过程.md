### Linux LVM 配置过程

---

**许多Linux使用者安装操作系统时都会遇到这样的困境：如何精确评估和分配各个硬盘分区的容量，如果当初评估不准确，一旦系统分区不够用时可能不得不备份、删除相关数据，甚至被迫重新规划分区并重装操作系统，以满足应用系统的需要。**

**LVM是Linux环境中对磁盘分区进行管理的一种机制，是建立在硬盘和分区之上、文件系统之下的一个逻辑层，可提高磁盘分区管理的灵活性。RHEL5默认安装的分区格式就是LVM逻辑卷的格式，需要注意的是/boot分区不能基于LVM创建，必须独立出来。**

#### 逻辑卷的创建

1. ###### 首先创建出两个分区 /dev/sdb1 /dev/sdb2，假设两个分区在不同的磁盘，并通过**t**选项设置分区标识为8e

   ```shell
   [root@localhost ~]# fdisk /dev/sdb
   
   WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
            switch off the mode (command 'c') and change display units to
            sectors (command 'u').
   
   Command (m for help): n
   Command action
      e   extended
      p   primary partition (1-4)
   p
   Partition number (1-4): 1
   First cylinder (1-2610, default 1): 
   Using default value 1
   Last cylinder, +cylinders or +size{K,M,G} (1-2610, default 2610): +1G
   
   Command (m for help): t
   Selected partition 1
   Hex code (type L to list codes): 8e
   Changed system type of partition 1 to 8e (Linux LVM)
   ```

   ###### 同样的方式新建另外一个分区，`fdisk -l` 可以看到两个分区已经创建

   ```shell
   Disk identifier: 0x6ba5d589
   
      Device Boot      Start         End      Blocks   Id  System
   /dev/sdb1               1         132     1060258+  8e  Linux LVM
   /dev/sdb2             133         264     1060290   8e  Linux LVM
   ```

   

2. ###### 把sdb1，sdb2两个分区转换为物理卷，并创建一个卷组myvg

   ```shell
   [root@localhost ~]# pvcreate /dev/sdb1 /dev/sdb2
     Physical volume "/dev/sdb1" successfully created
     Physical volume "/dev/sdb2" successfully created
   
   可以通过pvs和pvdisplay查看物理卷的状态
   [root@localhost ~]# pvs
     PV         VG         Fmt  Attr PSize  PFree
     /dev/sda2  vg_livedvd lvm2 a--u 19.51g    0 
     /dev/sdb1  myvg       lvm2 a--u  1.01g 1.01g
     /dev/sdb2  myvg       lvm2 a--u  1.01g 1.01g
   [root@localhost ~]# pvdisplay
     --- Physical volume ---
     PV Name               /dev/sdb1
     VG Name               myvg
     PV Size               1.01 GiB / not usable 3.41 MiB
     Allocatable           yes 
     PE Size               4.00 MiB
     Total PE              258
     Free PE               258
     Allocated PE          0
     PV UUID               rroWHG-w71l-XKZF-5AyO-i6fu-UUUh-NM2Alf
   
     --- Physical volume ---
     PV Name               /dev/sdb2
     VG Name               myvg
     PV Size               1.01 GiB / not usable 3.44 MiB
     Allocatable           yes 
     PE Size               4.00 MiB
     Total PE              258
     Free PE               258
     Allocated PE          0
     PV UUID               Sfwr9U-qYei-KAp1-eHNB-4Kwh-1p2T-9mHr27
   
   [root@localhost ~]# vgcreate myvg /dev/sdb1 /dev/sdb2
     Volume group "myvg" successfully created`
   
   可以通过vgs和vg命令查看卷组的状态
   
   [root@localhost ~]# vgs
     VG         #PV #LV #SN Attr   VSize  VFree
     myvg         2   0   0 wz--n-  2.02g 2.02g
     vg_livedvd   1   2   0 wz--n- 19.51g    0 
   [root@localhost ~]# vgdisplay
     --- Volume group ---
     VG Name               myvg
     System ID             
     Format                lvm2
     Metadata Areas        2
     Metadata Sequence No  1
     VG Access             read/write
     VG Status             resizable
     MAX LV                0
     Cur LV                0
     Open LV               0
     Max PV                0
     Cur PV                2
     Act PV                2
     VG Size               2.02 GiB
     PE Size               4.00 MiB
     Total PE              516
     Alloc PE / Size       0 / 0   
     Free  PE / Size       516 / 2.02 GiB
     VG UUID               3BZdPp-clgl-7qqu-k6AK-gQsc-yxOo-qdlzex
   ```

   

3. ###### 创建新的逻辑卷mylvm，并在卷组myvg中划分出1500M空间给新的逻辑卷

   ```shell
   [root@localhost ~]# lvcreate -L 1500M -n mylvm myvg
     Logical volume "mylvm" created.
   
   可以通过lvs和vgdisplay查看逻辑卷的状态
   
   [root@localhost ~]# lvs
     LV      VG         Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
     mylvm   myvg       -wi-a-----  1.46g                                                    
     lv_root vg_livedvd -wi-ao---- 17.51g                                                    
     lv_swap vg_livedvd -wi-ao----  2.00g                                                    
   [root@localhost ~]# lvdisplay
     --- Logical volume ---
     LV Path                /dev/myvg/mylvm
     LV Name                mylvm
     VG Name                myvg
     LV UUID                C3Rpbs-mxmp-HZ9O-hSJJ-754I-ZKk5-N8v7zz
     LV Write Access        read/write
     LV Creation host, time localhost.localdomain, 2019-04-04 19:00:25 +0800
     LV Status              available
   
     open                 0
   
     LV Size                1.46 GiB
     Current LE             375
     Segments               2
     Allocation             inherit
     Read ahead sectors     auto
   
     \-  currently set to     256
    Block device           253:2
   ```

   

4. ###### 这时mylvm就像一个随意扩大和缩小空间的分区一样，可以进行格式化，然后挂载到目录

   ```shell
   [root@localhost ~]# mkfs.ext4 /dev/myvg/mylvm 
   mke2fs 1.41.12 (17-May-2010)
   Filesystem label=
   OS type: Linux
   Block size=4096 (log=2)
   Fragment size=4096 (log=2)
   Stride=0 blocks, Stripe width=0 blocks
   96000 inodes, 384000 blocks
   19200 blocks (5.00%) reserved for the super user
   First data block=0
   Maximum filesystem blocks=394264576
   12 block groups
   32768 blocks per group, 32768 fragments per group
   8000 inodes per group
   Superblock backups stored on blocks: 
   	32768, 98304, 163840, 229376, 294912
   
   Writing inode tables: done                            
   Creating journal (8192 blocks): done
   Writing superblocks and filesystem accounting information: done
   
   This filesystem will be automatically checked every 29 mounts or
   180 days, whichever comes first.  Use tune2fs -c or -i to override.
   
   查看UUID ，修改/etc/fstab，然后mount
   
   [root@localhost ~]# blkid /dev/myvg/mylvm 
   /dev/myvg/mylvm: UUID="ca95f3e4-1c64-42e0-aa9a-2222459d8427" TYPE="ext4" 
   [root@localhost ~]# vim /etc/fstab
   
   UUID=ca95f3e4-1c64-42e0-aa9a-2222459d8427 /data ext4    defaults        0 0
   
   [root@localhost ~]# mount -a
   [root@localhost ~]# df -h
   Filesystem            Size  Used Avail Use% Mounted on
   /dev/mapper/vg_livedvd-lv_root
                          18G   11G  6.8G  60% /
   tmpfs                 1.9G   76K  1.9G   1% /dev/shm
   /dev/sda1             477M   42M  410M  10% /boot
   /dev/sr0              1.9G  1.9G     0 100% /media/CentOS-6.9-x86_64-LiveDVD
   /dev/mapper/myvg-mylvm
                         1.5G  2.3M  1.4G   1% /data
   
   [root@localhost ~]# cd /data/
   [root@localhost data]# ls
   lost+found
   ```

   

#### 逻辑卷的扩展

1. ###### 当卷组空间不够用的时候，可以扩展卷组

   首先我们新建一个5G的分区 /dev/sdb3，转换为物理卷，然后加入卷组myvg

   新建分区，转换为物理卷的步骤请参考逻辑卷的创建，我们这里直接扩展卷组

   ```shell
   [root@localhost ~]# vgextend myvg /dev/sdb3
     Volume group "myvg" successfully extended
   ```

   

2. ###### 当逻辑卷空间不够用的时候，可以对逻辑卷进行扩展

   ```shell
   [root@localhost ~]# lvextend -L +2G /dev/myvg/mylvm 
     Size of logical volume myvg/mylvm changed from 1.46 GiB (375 extents) to 3.46 GiB (887 extents).
     Logical volume mylvm successfully resized.
   [root@localhost ~]# resize2fs /dev/myvg/mylvm 
   resize2fs 1.41.12 (17-May-2010)
   Filesystem at /dev/myvg/mylvm is mounted on /data; on-line resizing required
   old desc_blocks = 1, new_desc_blocks = 1
   Performing an on-line resize of /dev/myvg/mylvm to 908288 (4k) blocks.
   The filesystem on /dev/myvg/mylvm is now 908288 blocks long.
   
   [root@localhost ~]# df -h
   Filesystem            Size  Used Avail Use% Mounted on
   /dev/mapper/vg_livedvd-lv_root
                          18G   11G  6.8G  60% /
   tmpfs                 1.9G   76K  1.9G   1% /dev/shm
   /dev/sda1             477M   42M  410M  10% /boot
   /dev/mapper/myvg-mylvm
                         3.4G  3.0M  3.3G   1% /data
   /dev/sr0              1.9G  1.9G     0 100% /media/CentOS-6.9-x86_64-LiveDVD
   ```

   ###### 一定注意要 resize2fs同步文件系统，否则不能识别出空间扩展

### 逻辑卷的释放

1. ###### 先umount挂载的目录，然后检查逻辑卷剩余空间的大小

   ```shell
   [root@localhost ~]# umoun /data/
   -bash: umoun: command not found
   [root@localhost ~]# umount /data
   [root@localhost ~]# e2fsck -f /dev/myvg/mylvm 
   e2fsck 1.41.12 (17-May-2010)
   Pass 1: Checking inodes, blocks, and sizes
   Pass 2: Checking directory structure
   Pass 3: Checking directory connectivity
   Pass 4: Checking reference counts
   Pass 5: Checking group summary information
   /dev/myvg/mylvm: 11/224000 files (9.1% non-contiguous), 23014/908288 blocks
   ```

   

2. ###### 重新定义文件系统的大小

   ```
   [root@localhost ~]# resize2fs /dev/myvg/mylvm 1G
   resize2fs 1.41.12 (17-May-2010)
   Resizing the filesystem on /dev/myvg/mylvm to 262144 (4k) blocks.
   The filesystem on /dev/myvg/mylvm is now 262144 blocks long.
   注意：如果是ext文件系统，使用resize2fs命令
   	 如果是xfs文件系统，使用xfs_growfs命令
   ```

   

3. ###### 重新定义逻辑卷的大小

   ```shell
   [root@localhost ~]# lvreduce -L 1G /dev/myvg/mylvm 
     WARNING: Reducing active logical volume to 1.00 GiB.
     THIS MAY DESTROY YOUR DATA (filesystem etc.)
   Do you really want to reduce myvg/mylvm? [y/n]: y
     Size of logical volume myvg/mylvm changed from 3.46 GiB (887 extents) to 1.00 GiB (256 extents).
     Logical volume mylvm successfully resized.
   ```

   

4. ###### 重新mount到目录就可以使用了

   ```shell
   [root@localhost ~]# mount -a
   [root@localhost ~]# df -h
   Filesystem            Size  Used Avail Use% Mounted on
   /dev/mapper/vg_livedvd-lv_root
                          18G   11G  6.8G  60% /
   tmpfs                 1.9G   76K  1.9G   1% /dev/shm
   /dev/sda1             477M   42M  410M  10% /boot
   /dev/sr0              1.9G  1.9G     0 100% /media/CentOS-6.9-x86_64-LiveDVD
   /dev/mapper/myvg-mylvm
                         977M  1.9M  924M   1% /data
   ```

   **（注意：文件系统大小和逻辑卷大小一定要保持一致才行。如果逻辑卷大于文件系统，由于部分区域未格式化成文件系统会造成空间的浪费。如果逻辑卷小于文件系统，哪数据就出问题了。）**

### 逻辑卷的删除

1. ###### umount挂载的目录

2. ###### 修改/etc/fstab的信息，否则会影响机器启动

3. ###### 然后依次删除 逻辑卷 卷组 物理卷

   ```shell
   [root@localhost ~]# umount /data/
   [root@localhost ~]# vim /etc/fstab 
   [root@localhost ~]# lvremove /dev/myvg/mylvm 
   Do you really want to remove active logical volume mylvm? [y/n]: y
     Logical volume "mylvm" successfully removed
   [root@localhost ~]# vgremove myvg
     Volume group "myvg" successfully removed
   [root@localhost ~]# pvremove /dev/sdb1 /dev/sdb2 /dev/sdb3
     Labels on physical volume "/dev/sdb1" successfully wiped
     Labels on physical volume "/dev/sdb2" successfully wiped
     Labels on physical volume "/dev/sdb3" successfully wiped
   ```

   

