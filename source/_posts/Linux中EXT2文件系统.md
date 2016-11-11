title: EXT2文件系统
tags: linux
categories: linux
toc: true
date: 2016-11-10 12:12:12
---

之前总结了linux下的磁盘分区，分区完成之后，下一步就需要将分区**格式化(format)**。

**为什么要格式化呢？**这是因为每种操作系统所设置的文件属性/权限并不相同，为了存放这些文件所需的数据，就要将分区进行格式化，以成为操作系统能够利用的文件系统。

今天，就整理下linux的正规文件系统：`Ext2`。

## Ext2文件系统特征

我们知道，在Linux中，一个文件除了本身的数据内容之外，通常含有非常多的属性，比如文件权限、所有者、群组、创建时间等。

Ext2文件系统中，文件属性存放在`inode`中，而实际数据则存放到`data block`当中。另外，还有一个`superblock`来记录整个文件系统的整体信息，包括inode和block的总量、使用量、剩余量等。

每个inode和block都有编号，更详细的说明如下：

---
- superblock：记录文件系统整体信息，包括inode/block的总量、使用量、剩余量，以及文件系统的格式等相关信息。
- inode：记录文件的属性，一个文件占用一个inode，同时记录此文件数据所在的block号码。
- block：记录文件的实际数据，可能会占用多个block。
---

下图来帮助理解：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E7%B4%A2%E5%BC%95%E5%BC%8F%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F.png)

以上这种数据访问的方法我们称为**索引式文件系统**，Ext2就是这样的一个文件系统。

## Ext2文件系统结构

如果我们格式化完某一个分区之后，文件系统将会自动把`inode`和`block`规划好。

但是，假如某个文件系统高达几百GB，那么**把所有的inode和block放在一起，将非常不容易管理**。

所以，Ext2文件系统在格式化的时候，是将分区划**分为多个块组(block group)**的，每个块组都有独立的`inode/block/superblock`系统。

图示如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/Ext2%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E7%BB%93%E6%9E%84.png)

下面针对上图依次做出说明。

### boot sector

文件系统的最前面有一个启动扇区**(boot sector)**，这个启动扇区可以安装引导**装载程序(boot loader)**。

如此一来，我们就可以将不同的`boot loader`安装到个别文件系统的最前端，而不用覆盖整块硬盘唯一的`MBR`。

这样也能做出**多重引导的环境**。

### data block

`data block`是用来存放文件内容的地方，在Ext2文件系统中，所支持的block大小有`1KB`、`2KB`以及`4KB`三种。

承上，每个block只能存放一个文件的数据。若文件较大，可能占用多个block；若文件小于block大小，剩下的空间也不能再使用了。

所以，假设你文件系统的block设置为4KB，并且你有很多很多小文件，那么将造成很大的空间浪费。

但是，如果将block大小设置为1KB，并且有很多大文件，那么一个文件将需要占据多个block，此时可能导致文件读写性能下降。

### inode table

`inode table`是inode的集合，某个inode记录的是某个文件的属性以及该文件的数据实际放在哪些block之中。

下面罗列出了一个inode记录的基本数据项：

---
- 该文件的访问模式（read/write/excute）
- 该文件的所有者与组（owner/group）
- 该文件的大小
- 该文件创建或状态改变的时间（ctime）
- 最近一次读取时间（atime）
- 最近修改时间（mtime）
- 特殊权限（SUID/SGID）
- 该文件的真正内容指向（block 号码）

---

另外，每个inode的固定大小为`128bytes`，而inode记录一个block号码要花费`4bytes`。

假设一个文件大小为400M，并且文件系统的block大小为4KB，那么该文件至少需要10万个block来存储器数据，该文件的inode哪儿能存储这么多block号码呢？

为了解决以上问题，Ext2文件系统将inode记录block号码的区域定义为**12个直接、一个间接、一个双间接**与**一个三间接记录区**。

下图来辅助理解：

![](http://7xvlvo.com1.z0.glb.clouddn.com/inode%E7%BB%93%E6%9E%84%E5%9B%BE.png)

这样假设我们的文件系统的block大小设置为1KB，那么该文件的inode的三间接记录区所能保存的block大小为：`256×256×256×1KB`，约为2的34次方个byte，大约为`16GB`。

承上，当文件系统**将block大小设置为1KB时，能够容纳的单个最大文件为16GB**。

但是这种计算方法，不能用在block大小为2KB和4KB时，因为这将会收到Ext2文件系统本身的限制。

### superblock

`superblock`记录的是整个文件系统的相关信息，大小为`1024bytes`，记录信息主要包括：

---
- block与inode的总量。
- 未使用与已使用的inode/block数量。
- block与inode的大小（block为1K、2K、4K，inode为128bytes）。
- 文件系统的挂载时间、最近一次写入数据的时间、最忌一次检验磁盘的时间等。
- 一个`valid bit`数值，若该值为0，表示该文件系统已经挂载，若为1，表示未被挂载。
---

以上信息，我们可以通过`dumpe2fs`命令来查看，由于显示内容比较多，就不上图片了。

另外，我们知道，**一个文件系统应该只会有一个superblock而已**。

但事实上，除了第一个`block group`内会含有`superblock`之外，后续的`block group`不一定含有`superblock`，若含有的话，则是第一个`block group`中`superblock`的备份，这样可以进行`superblock`的救援。

### file system description

即文件系统描述说明，这个区段描述每个`block group`的开始与结束block号码，以及说明每个区段（superblock、inode table、data block等）分别介于哪一个block号码之间。

这部分也可以通过`dumpe2fs`来查看。

### block bitmap

即块对照表，记录哪些block是空的，哪些已经被使用。

比如你删除某个文件时，此时`block bitmap`中想对应该文件block的号码就被修改成“未使用中”。

### inode bitmap

即inode对照表，与`block bitmap`类似，`inode bitmap`记录使用与未使用的inode号码。

### 信息查询

使用`dumpe2fs`可以查询到以上所有信息。

## 如何记录目录与文件

以上对linux下的Ext2文件系统做了简单的介绍，那么目录与文件在Ext2文件系统中是如何如何被记录的呢？

### 目录

当我们在linux的Ext2文件系统中，新建一个目录时，Ext2会分配一个inode与至少一块block给该目录。

其中，inode记录该目录的相关权限和属性，以及分配到的那块block的号码。

而那块block则记录该目录下的文件名，以及每个文件对应的inode号码。

那么我们如何查看某个目录下保存的文件名和inode号码呢？用`ls -li`即可：

![](http://7xvlvo.com1.z0.glb.clouddn.com/ext2%E7%9B%AE%E5%BD%95.png)

上图第一列就是每个文件对应的inode号码！

另外，我们如何查看给某个目录分配的空间大小呢？使用`ls -ld`:

![](http://7xvlvo.com1.z0.glb.clouddn.com/ext2%E7%9B%AE%E5%BD%95%E4%BA%8C.png)

上图可以看出，文件系统为每个目录分配的空间大小都是`4096Bytes`的整数倍。这是因为，我的block就设置为`4KB`（可以使用`dumpe2fs`来查看）。

但是，我们也发现目录`/proc`所占磁盘空间为0，这是因为该目录不占用硬盘容量（**占用内存**），所以消耗的block大小自然为0。

还有`/lost+found`目录，其占用了4个block。这是因为该目录下文件数量太多，导致**一个block不能够记录该目录下的所有文件名及其对应inode号码**，因此会多一些block来记录相关数据。

### 文件

当我们在linux下的Ext2文件系统中，新建一个文件时，Ext2会**分配一个inode与相对于该文件大小的block数量给该文件**。

比如，当文件系统block大小为4KB，我新建了一个100KB的文件，那么将会分配1个inode和25个block来存储该文件。

但是，由于inode仅有12个直接指向，因此还要多一个block来作为作为块号码的记录。

### 文件内容读取过程

由上面总结可知：**无论是文件还是目录，inode本身并不记录文件名；而文件名是记录在其所在目录的block中**。

因此当我们试图读取一个文件时，务必会先读取其所在目录的inode和block，在block中找到待读取文件的inode号码，继而再读取该文件的block内容。

另外我们需要直到，通常一个文件系统最顶层的inode号码，是从`2号`开始（参考上一张图片）。

下面假设我们要读取`/etc/passwd`文件的内容，大致会经过以下几个步骤：

---

1. /的inode : 查看用户是否具有对应权限，若有，则可以读取对应block内容
2. /的block : 在block中找到/etc目录的inode号码
3. /etc的inode : 根据上一阶段拿到的inode号码，找到并读取/etc的inode，查看用户是否具有相应权限
4. /etc的block : 读取/etc的block，找到文件passwd对应的inode号码
5. /etc/passwd的inode : 根据文件passwd具有的权限，决定是否可以读取block内容
6. /etc/passwd的block : 读取文件passwd的内容

## 两个查询命令

最后，我们来看与文件系统相关的两个信息查询命令：`df`和`du`。

### df

df可以列出当前磁盘的挂载情况以及磁盘使用量，主要来自`superblock`中，所以速度很快。

使用方法如下：

{% codeblock lang:sh %}
$ df [-kmhi] [目录或文件名]
参数：
-k：以KB为容量显示各文件系统使用量
-m：以MB为容量显示各文件系统使用量
-h：以人类较易理解的GB、MB、KB等格式显示
-i：不使用硬盘容量，而以inode的数量来表示
-T：连同该文件系统的类型（如Ext3）也列出来
{% endcodeblock %}

以下有几个使用场景：

后面不加文件或者目录名，将列出系统内所有的文件系统使用和挂载信息：

![](http://7xvlvo.com1.z0.glb.clouddn.com/df-1.png)

如果后面跟上文件名，将自动分析该目录或者文件所在分区，并且列出该分区的使用和挂载信息：

![](http://7xvlvo.com1.z0.glb.clouddn.com/df-2.png)

列出某文件系统的的inode个数使用量：

![](http://7xvlvo.com1.z0.glb.clouddn.com/df-3.png)

### du

du主要用与评估**文件大小**或者**目录容量**（注意，即使一个文件中只有一个字符`a`，那么该文件也占用一个block，即4K）。


与df读取superblock不同，du这个命令会直接到文件系统内去查找所有的文件数据，所以速度稍慢。

du的用法如下：

{% codeblock lang:sh %}
$ du [-akmhs] [文件或者目录名称]
参数：
-a：列出所有的文件大小和目录容量，因为默认du仅列出目录容量。
-k：以KB显示
-m: 以MB显示
-h：以人类较易理解的KB、MB、GB来显示
-s：仅列出当前（指定）目录的block总量
{% endcodeblock %}

现在假设我们有这样一个名为`test`的文件夹：

![](http://7xvlvo.com1.z0.glb.clouddn.com/du-1.png)

来看几个场景：

列出该目录下，所有目录容量（注意：仅显示目录）：

![](http://7xvlvo.com1.z0.glb.clouddn.com/du-2.png)

列出该目录下，所有文件大小和目录容量（包含文件和目录）：

![](http://7xvlvo.com1.z0.glb.clouddn.com/du-3.png)

今天就写到这里吧！

下一篇将总结一下linux中的连接文件，因为与文件系统相关性较高，所以要趁热打铁，才能记得牢啊O\_o。

