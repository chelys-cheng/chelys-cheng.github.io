---
title: Linux新硬盘分区及开机挂载
date: 2018-09-05 13:12:41
categories: [Linux]
tags: [Linux]
---
云服务器上需要做MySQL数据备份，然后发现云服务器上有一块200G的硬盘并没有进行挂载使用，于是重新进行分区，计划分为两个区，分别100G，其中一个盘做MySQL数据备份。
完整步骤记录如下：
<!--more-->
 ## 查看硬盘信息
 ```bash
	[root@localhost ~]$ fdisk -l
	Disk /dev/xvda: 42.9 GB, 42949672960 bytes, 83886080 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk label type: dos
	Disk identifier: 0x000a9cf3
	
	Device Boot          Start         End      Blocks   Id  System
	/dev/xvda1            2048     8390655     4194304   82  Linux swap / Solaris
	/dev/xvda2   *     8390656    83886079    37747712   83  Linux
	
	Disk /dev/xvde: 214.7 GB, 214748364800 bytes, 419430400 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
```
   可以看见有硬盘`/dev/xvde`未分区。
 ## 创建新硬盘分区
```bash
    [root@localhost ~]$ fdisk /dev/xvde
    Welcome to fdisk (util-linux 2.23.2).
	
    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.

    Device does not contain a recognized partition table
    Building a new DOS disklabel with disk identifier 0x670a5625.

    Command (m for help): m
    Command action
        a   toggle a bootable flag                  # 设定可启动标记
        b   edit bsd disklabel
        c   toggle the dos compatibility flag
        d   delete a partition                      # 删除一个分区
        l   list known partition types              # 各分区类型所对应的ID
        m   print this menu                         # 菜单
        n   add a new partition                     # 添加一个分区
        o   create a new empty DOS partition table
        p   print the partition table               # 显示该磁盘下的当前分区信息
        q   quit without saving changes             # 不保存退出
        s   create a new empty Sun disklabel
        t   change a partition's system id
        u   change display/entry units
        v   verify the partition table
        w   write table to disk and exit            # 保存退出
        x   extra functionality (experts only)

    Command (m for help): p                         # 打印分区信息,可以看到当前并没有分区

    Disk /dev/xvde: 214.7 GB, 214748364800 bytes, 419430400 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x670a5625

    Device Boot          Start         End      Blocks   Id  System

    Command (m for help): n                         # 创建一个新的分区
    Partition type:
        p   primary (0 primary, 0 extended, 4 free) # 创建逻辑分区
        e   extended                                # 创建扩展分区
    Select (default p): p                           # 选择创建逻辑分区
    Partition number (1-4, default 1): 2            # 分区个数，我这边分成了两部分
    First sector (2048-419430399, default 2048):    # 开始的扇区
    Using default value 2048
    Last sector, +sectors or +size{K,M,G} (2048-419430399, default 419430399): 209717247 # 结束扇区
    Partition 2 of type Linux and of size 100 GiB is set

    Command (m for help): n                         # 创建第二个分区
    Partition type:
        p   primary (1 primary, 0 extended, 3 free)
        e   extended
    Select (default p): p
    Partition number (1,3,4, default 1):
    First sector (209717248-419430399, default 209717248):
    Using default value 209717248
    Last sector, +sectors or +size{K,M,G} (209717248-419430399, default 419430399):
    Using default value 419430399
    Partition 1 of type Linux and of size 100 GiB is set

    Command (m for help): p

    Disk /dev/xvde: 214.7 GB, 214748364800 bytes, 419430400 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x670a5625

        Device Boot      Start         End      Blocks   Id  System
    /dev/xvde1       209717248   419430399   104856576   83  Linux
    /dev/xvde2            2048   209717247   104857600   83  Linux

    Partition table entries are not in disk order

    Command (m for help): w                         # 保存新分区
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.
```
   再次使用`fdisk -l`命令查看磁盘信息
```bash
    [root@localhost ~]$ fdisk -l
    Disk /dev/xvda: 42.9 GB, 42949672960 bytes, 83886080 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x000a9cf3

    Device Boot          Start         End      Blocks   Id  System
    /dev/xvda1            2048     8390655     4194304   82  Linux swap / Solaris
    /dev/xvda2   *     8390656    83886079    37747712   83  Linux

    Disk /dev/xvde: 214.7 GB, 214748364800 bytes, 419430400 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk label type: dos
    Disk identifier: 0x670a5625

    Device Boot          Start         End      Blocks   Id  System
    /dev/xvde1       209717248   419430399   104856576   83  Linux
    /dev/xvde2            2048   209717247   104857600   83  Linux

    Partition table entries are not in disk order
```

   可以看到磁盘/dev/xvda已经分区好了，但是可以看见，最后有出现了
`Partition table entries are not in disk order`
   这是由于分区表中分区的顺序的硬盘物理顺序不一致，我们重新进入`fdisk`中进行修复
```bash
    [root@localhost ~]$ fdisk /dev/xvde
    Welcome to fdisk (util-linux 2.23.2).

    Changes will remain in memory only, until you decide to write them.
    Be careful before using the write command.


    Command (m for help): x                         # 输入x，进入专家命令行

    Expert command (m for help): m
    Command action
        b   move beginning of data in a partition
        c   change number of cylinders
        d   print the raw data in the first sector
        e   list extended partitions
        f   fix partition order                     # 这一项用于修复分区顺序
        g   create an IRIX (SGI) partition table
        h   change number of heads
        i   change the disk identifier
        m   print this menu
        p   print the partition table
        q   quit without saving changes
        r   return to main menu
        s   change number of sectors/track
        v   verify the partition table
        w   write table to disk and exit

    Expert command (m for help): f
    Done.

    Expert command (m for help): p                  # 再次查看分区，可以看到已经没有了提示

    Disk /dev/xvde: 255 heads, 63 sectors, 26108 cylinders

    Nr AF  Hd Sec  Cyl  Hd Sec  Cyl     Start      Size ID
        1 00  32  33    0  75  13  766       2048  209715200 83
        2 00  75  14  766  85  25  508  209717248  209713152 83
        3 00   0   0    0   0   0    0          0          0 00
        4 00   0   0    0   0   0    0          0          0 00

    Expert command (m for help): w
    The partition table has been altered!

    Calling ioctl() to re-read partition table.
    Syncing disks.
```
 ## 格式化分区
```bash
    [root@localhost ~]# mkfs.ext3 /dev/xvde1        # 格式化第一分区
    mke2fs 1.42.9 (28-Dec-2013)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    6553600 inodes, 26214144 blocks
    1310707 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=4294967296
    800 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
            4096000, 7962624, 11239424, 20480000, 23887872

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done
    
    [root@localhost ~]# mkfs.ext3 /dev/xvde2        # 格式化第二分区
    mke2fs 1.42.9 (28-Dec-2013)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    6553600 inodes, 26214400 blocks
    1310720 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=4294967296
    800 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks:
            32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
            4096000, 7962624, 11239424, 20480000, 23887872

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done
```

 ## 挂载磁盘
 
   创建目录`/storage/diskone`和`/storage/disktwo`分别挂载`/dev/xvde1`和`/dev/xvde2`分区
```bash
    [root@localhost ~]# mkdir /storage/diskone
    [root@localhost ~]# mkdir /storage/disktwo
    [root@localhost ~]# mount /dev/xvde1 /storage/diskone/
    [root@localhost ~]# mount /dev/xvde2 /storage/disktwo/
```
   再次查看硬盘挂载情况
```
    [root@localhost ~]# df -TH
    Filesystem     Type      Size  Used Avail Use% Mounted on
    /dev/xvda2     ext3       38G  7.6G   29G  21% /
    devtmpfs       devtmpfs  2.0G     0  2.0G   0% /dev
    tmpfs          tmpfs     2.0G     0  2.0G   0% /dev/shm
    tmpfs          tmpfs     2.0G  211M  1.8G  11% /run
    tmpfs          tmpfs     2.0G     0  2.0G   0% /sys/fs/cgroup
    tmpfs          tmpfs     395M     0  395M   0% /run/user/1001
    /dev/xvde1     ext3      106G   63M  101G   1% /storage/diskone
    /dev/xvde2     ext3      106G   63M  101G   1% /storage/disktwo
```
   可以发现，两个新分区已经挂载上了。

   **注：**卸载磁盘使用命令umount /dev/xvde1。

 ## 设置开机自动挂载

   编辑`/etc/fstab`文件，在最后添加
```
	/dev/xvde1  /storage/diskone    ext3    defaults    0 0
	/dev/xvde2  /storage/disktwo    ext3    defaults    0 0
```
   重启服务器，就可以了。
