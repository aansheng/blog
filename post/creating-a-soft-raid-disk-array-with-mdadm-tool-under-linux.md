# Linux下mdadm工具创建软RAID磁盘阵列

RAID（Redundant Array of Independent Disks，独立磁盘冗余阵列）通过将多个硬盘设备组合成一个容量更大、安全性更好的磁盘阵列，并把数据切割成多个区段后分别存放在各个不同的物理硬盘设备上，然后利用分散读写技术来提升磁盘阵列整体的性能，同时把多个重要数据的副本同步到不同的物理硬盘设备上，从而起到了非常好的数据冗余备份效果，Linux下面可以通过mdadm这个指令来管理和创建软RAID。

## RAIA级别

下面是几个比较常用的RAID简单介绍，可以参考下

|级别|最低磁盘个数|空间利用率|说明|
|:--|:--|:--|:--|
|0	|2	|100%	|读写速度快，不容错|
|1	|2	|50%	|读写速度一般，容错|
|5	|3	|(n-1)/n	|读写速度快，容错，允许坏一块盘|
|10	|4	|50%	|读写速度快，容错|

## RAID 0测试

- 环境

```
$ cat /etc/redhat-release
CentOS Linux release 8.4.2105
# 这台机器上面有6块2T的NVME
$ lsblk | grep nvme | sort
nvme0n1     259:2    0   1.8T  0 disk
nvme1n1     259:5    0   1.8T  0 disk
nvme2n1     259:0    0   1.8T  0 disk
nvme3n1     259:1    0   1.8T  0 disk
nvme4n1     259:4    0   1.8T  0 disk
nvme5n1     259:3    0   1.8T  0 disk

```

- 安装mdadm

```
$ sudo dnf install mdadm -y

```

- 创建

```
$ sudo mdadm --create --verbose /dev/md0 --chunk=128 --level=0 --raid-devices=6 /dev/nvme0n1 /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1 /dev/nvme4n1 /dev/nvme5n1
......
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
# --create       创建
# --verbose      显示详细信息
# --chunk        可用于修改默认的区块大小
# --level        级别
# --raid-devices 设备数量

```

- 查看详情

```
$ sudo mdadm --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Thu Aug 12 22:32:41 2021
        Raid Level : raid0
        Array Size : 11720294400 (10.92 TiB 12.00 TB)
      Raid Devices : 6
     Total Devices : 6
       Persistence : Superblock is persistent

       Update Time : Thu Aug 12 22:32:41 2021
             State : clean
    Active Devices : 6
   Working Devices : 6
    Failed Devices : 0
     Spare Devices : 0

            Layout : -unknown-
        Chunk Size : 128K

Consistency Policy : none

              Name : fil-jn-p1-001:0  (local to host fil-jn-p1-001)
              UUID : ce9236f5:c29c536a:a4dad85b:2721fdd8
            Events : 0

    Number   Major   Minor   RaidDevice State
       0     259        2        0      active sync   /dev/nvme0n1
       1     259        5        1      active sync   /dev/nvme1n1
       2     259        0        2      active sync   /dev/nvme2n1
       3     259        1        3      active sync   /dev/nvme3n1
       4     259        4        4      active sync   /dev/nvme4n1
       5     259        3        5      active sync   /dev/nvme5n1

```

- 更新配置文件

```
$ sudo mdadm -Ds
ARRAY /dev/md0 metadata=1.2 name=fil-jn-p1-001:0 UUID=ce9236f5:c29c536a:a4dad85b:2721fdd8

$ sudo vim /etc/mdadm.conf
......
ARRAY /dev/md0 metadata=1.2 name=fil-jn-p1-001:0 UUID=ce9236f5:c29c536a:a4dad85b:2721fdd8

```

- 格式化

```
$ sudo mkfs.xfs -f -d agcount=128,su=128k,sw=6 -r extsize=640k /dev/md0
# su=128k保持不变，sw=5为硬盘数量，extsize=640k ，为su*sw

```

- 挂载

```
# 创建挂载目录
$ sudo mkdir -p /mnt
# 挂载
$ sudo mount /dev/md0 /mnt
# 查看挂载点
$ df -h | grep /mnt
/dev/md0              11T   78G   11T   1% /mnt

```

- 设置开机自动挂载

```
# 查看设备ID
$ sudo blkid | grep md0
/dev/md0: UUID="8f66ed54-a304-4720-8e98-d8a0c3b5b7e4" BLOCK_SIZE="512" TYPE="xfs"

# 写入fstab
$ sudo vim /etc/fstab
UUID=8f66ed54-a304-4720-8e98-d8a0c3b5b7e4 /mnt xfs defaults 0 0

# 挂载
$ sudo mount -a

# 查看挂载点
$ df -h | grep /mnt
/dev/md0              11T   78G   11T   1% /mnt

```

- 模拟磁盘损坏

```
$ sudo mdadm /dev/md0 -f /dev/sdc  #sdc盘损坏

```

查看磁盘阵列信息

```
$ sudo mdadm -D /dev/md0

```

### 从阵列中移除设备

要从阵列中移除一个设备，先将这个设备标记为faulty（故障）

```
$ sudo mdadm --fail /dev/md0 /dev/sdxx

```

现在从阵列中移除这个设备

```
$ sudo mdadm --remove /dev/md0 /dev/sdxx

```

永久移除设备（比如想把一个设备拿出来单独使用），先使用上述两个命令，然后：

```
$ sudo mdadm --zero-superblock /dev/sdxx

```

### 停止使用某个阵列

1. 卸载 (umount) 目标阵列
2. 用这个命令停止磁盘阵列运行：mdadm --stop /dev/md0
3. 将本节开头的三个命令在每块硬盘上都运行一遍。
4. 将 /etc/mdadm.conf 中的相关行移除。

### 向阵列中添加设备

向阵列中添加新设备

```
$ sudo mdadm --add /dev/md0 /dev/sdc1

```

检查已经添加的设备

```
$ sudo mdadm --misc --detail /dev/md0

```

### 其他级别创建示例

- RAID 5

使用4块磁盘作为工作 (active) 磁盘，1块作为备用 (spare) 磁盘建立 RAID5 阵列

```
$ sudo mdadm --create --verbose --level=5 --metadata=1.2 --chunk=256 --raid-devices=4 /dev/md1 /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1 --spare-devices=1 /dev/sdf1
# --metadata=1.2 协议版本

```

## 使用fio对磁盘性能测试

```
$ sudo dnf install fio -y
$ sudo fio -filename=/mnt/1.txt -direct=1 -iodepth 1 -thread -rw=randwrite -ioengine=psync -bs=4k -size=100G -numjobs=50 -runtime=180 -group_reporting -name=rand_100write_4k
rand_100write_4k: (g=0): rw=randwrite, bs=(R) 4096B-4096B, (W) 4096B-4096B, (T) 4096B-4096B, ioengine=psync, iodepth=1
...
fio-3.19
Starting 50 threads
rand_100write_4k: Laying out IO file (1 file / 102400MiB)
Jobs: 50 (f=50): [w(50)][100.0%][w=128MiB/s][w=32.9k IOPS][eta 00m:00s]
rand_100write_4k: (groupid=0, jobs=50): err= 0: pid=10261: Thu Aug 12 22:55:06 2021
  write: IOPS=30.7k, BW=120MiB/s (126MB/s)(21.1GiB/180005msec); 0 zone resets
    clat (usec): min=11, max=15246, avg=1624.60, stdev=1626.99
     lat (usec): min=11, max=15246, avg=1624.81, stdev=1627.02
    clat percentiles (usec):
     |  1.00th=[   19],  5.00th=[   32], 10.00th=[   43], 20.00th=[   56],
     | 30.00th=[   67], 40.00th=[  112], 50.00th=[  947], 60.00th=[ 2409],
     | 70.00th=[ 3032], 80.00th=[ 3392], 90.00th=[ 3785], 95.00th=[ 4113],
     | 99.00th=[ 5014], 99.50th=[ 5407], 99.90th=[ 6063], 99.95th=[ 6390],
     | 99.99th=[ 7439]
   bw (  KiB/s): min=88885, max=165577, per=100.00%, avg=122935.40, stdev=272.79, samples=17900
   iops        : min=22217, max=41380, avg=30706.91, stdev=68.26, samples=17900
  lat (usec)   : 20=1.60%, 50=14.30%, 100=23.54%, 250=1.95%, 500=3.25%
  lat (usec)   : 750=3.58%, 1000=2.27%
  lat (msec)   : 2=6.61%, 4=36.50%, 10=6.40%, 20=0.01%
  cpu          : usr=0.20%, sys=4.32%, ctx=13363977, majf=0, minf=362488
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued rwts: total=0,5529108,0,0 short=0,0,0,0 dropped=0,0,0,0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: bw=120MiB/s (126MB/s), 120MiB/s-120MiB/s (126MB/s-126MB/s), io=21.1GiB (22.6GB), run=180005-180005msec

Disk stats (read/write):
    md0: ios=0/5526298, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=0/922288, aggrmerge=0/0, aggrticks=0/22445, aggrin_queue=22445, aggrutil=97.75%
  nvme0n1: ios=0/921542, merge=0/2, ticks=0/22445, in_queue=22446, util=97.65%
  nvme3n1: ios=0/920791, merge=0/0, ticks=0/22353, in_queue=22353, util=97.67%
  nvme2n1: ios=0/923160, merge=0/0, ticks=0/22402, in_queue=22402, util=97.69%
  nvme5n1: ios=0/922497, merge=0/0, ticks=0/22611, in_queue=22611, util=97.67%
  nvme1n1: ios=0/923405, merge=0/0, ticks=0/22469, in_queue=22469, util=97.75%
  nvme4n1: ios=0/922333, merge=0/0, ticks=0/22391, in_queue=22391, util=97.65%

```

## 参考文献

- [RAID (简体中文)](https://wiki.archlinux.org/title/RAID_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))