title: 搭建hardware RAID5服务器
tags: linux
categories: linux
toc: true
date: 2017-3-15 12:12:12
---

最近要开始整毕业设计了，老师给了一台dell的`PowerEdge T620`服务器，`24G`内存，配有三块`300G`的希捷硬盘，正好可以用来搭建一个RAID5。

折腾了一下午，终于完成了基本的配置，安装了`centos 7`，并且配置了静态IP，以后打算用来提供www服务和数据库服务。

![](http://7xvlvo.com1.z0.glb.clouddn.com/dell-server.png)

## RAID

RAID，英文全程为`Redundent Array of Independent Disks`，即**独立硬盘冗余阵列**，一般简称为**磁盘阵列**。旧称`Redundent Array of Indexpensive Disks`，即**廉价磁盘冗余阵列**。

其基本思想就是将多个较为廉价的小容量硬盘，组合称为一个大容量硬盘，并且在操作系统看来，其就像一个单独的硬盘或者逻辑存储单元。

RAID有以下几点好处：

1. 增强数据集成度
2. 增强容错功能
3. 增强读写性能

RIAD具有不同的等级，这里将主要介绍`RAID-0`，`RAID-1`，`RAID 0+1`，`RAID 1+0`，`RAID 5`。

### RAID-0

RAID-0采用**等量（stripe）模式**，将磁盘先切分出等量的区块（chunk），当一个文件要些人RAID时，该文件会依据chunk的大小切割好，然后被等量地放置在各个磁盘中。

举例来说，当你使用两块磁盘组成RAID-0，并且有100M的数据需要写入磁盘时，那么每个磁盘会被写入50M的存储量。

![](http://7xvlvo.com1.z0.glb.clouddn.com/raid-0.png)

RAID-0的优点是每个磁盘负责的数据量变小了，因此读写可以并行处理，性能较好。

当然缺点也很明显，任何一块磁盘坏掉，整个RAID上所有数据均会丢失。

### RAID-1

RAID-1采用**映像（mirror）模式**，让同一份数据同时完整地保存在多块磁盘上。这样，每个磁盘上都将保存相同的内容。

![](http://7xvlvo.com1.z0.glb.clouddn.com/raid-1.png)

RAID-1具有数据保护功能，只要有任何一块磁盘没有坏掉，数据就不至于丢失。但是，其写性能会很差，因为需要将同一份数据写入到多个不同的磁盘，并且整体容量，只有原来的`1/N`。


### RAID 0+1 & RAID 1+0

RAID-0读写性能好，但是数据不安全。RAID-1数据安全，但是写性能不好。那我们可不可以将这两种模式结合一下呢？

RAID 0+1或者RAID 1+0就可以办到。其模式如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/raid-01.png)

### RAID-5

RAID-5综合考虑和性能和备份功能，并且至少需要3块及以上的磁盘，才能构建一个RAID-5。

这种磁盘阵列的数据写入类似RAID-0，不过每个写入过程中，将会保存一个同位检查数据（Parity）到磁盘中，这个同位检查数据类似于其他磁盘数据XOR的结果，当在RAID-5中有任何一个磁盘损坏时，都可以通过其他磁盘保存的数据来进行还原。

![](http://7xvlvo.com1.z0.glb.clouddn.com/raid5.png)

其最多只允许一块磁盘损坏，并且总容量将变为原来的`N-1/N`。

RAID-6于RAID-5类似，不过RAID-6中使用了两块磁盘作为parity的存储，所以其整体容量减少两块，同时其允许出错的磁盘数量也增加到两块。

### Spare Disk

当磁盘阵列中有磁盘坏掉时，就需要将坏掉的磁盘拔除，然后换上新的硬盘，这时，磁盘阵列就开始主动重建（rebuild）原本坏掉的磁盘数据了，这就是磁盘阵列的优点之一。

不过，我们还需要手动拔插硬盘，如果不是支持热插拔的磁盘阵列，还需要关机才可以这样做。

如果我们想让系统实时地重建坏掉的硬盘，那么就需要预备磁盘（spare disk）。

所谓spare disk，就是一块或者多块没有包含在原本磁盘阵列等级中的磁盘，正常情况下，spare disk不会被磁盘阵列的使用。当磁盘阵列中有磁盘坏掉时，spare disk会被主动拉进磁盘阵列中，并且将坏掉的那块磁盘移除磁盘阵列，然后立即重建数据。

## software RAID & hardware RAID

硬件磁盘阵列（hardware RAID）就是通过磁盘阵列卡来完成处理RAID的任务，不需要CPU参与运算，因此其读写性能较好。比如，在处理RAID-5中同位检查码（parity）计算的时候，磁盘阵列并不会占用服务器的I/O总线。其常用与RAID-5和RAID-6中。

因为hardware RAID用到的磁盘阵列卡价格昂贵，所以，软件磁盘阵列（software RAID）被发明出来。software RAID主要通过cpu来处理RAID的任务，会占用很多的cpu资源。在centos上，可以使用`mdadm`这套软件来管理磁盘阵列。

## 使用PREC在PowerEdge T620上配置RAID5

PREC，即`PowerEdge Expanable RAID Controller`，是dell服务器上专门用于配置RAID的工具，在开机时注意提示，按下`ctrl + r`即可进入PREC的工作环境。

接下来我们需要依次进行如下操作：

1. 删除系统默认创建的RAID
2. 创建新的虚拟磁盘（virtual disk）
3. 配置RAID参数，包括RAID等级、加入的磁盘等
3. 快速初始化虚拟磁盘
4. 重启计算机，安装系统

## centos分区

最后，顺便提一下服务器的centos分区情况，因为配置了RAID5模式，所以真正可用的磁盘空间只剩下570G左右。我拿出了500G的空间进行了分区，结果如下；

- / 50G
- /boot 500M
- /home 300G
- /usr 100G
- /var 30G
- swap 20G

还剩下70G没有分区，有需要的时候再拿来用。
