title: linux文件权限
tags: linux
categories: linux
toc: true
date: 2016-11-05 12:12:12
---

linux的文件权限，可以从三个方面来总结，即**基本权限**、**默认权限**和**特殊权限**。

在真正理解这些权限之前，我们先来理解下**用户和用户组**的基本概念。

## 用户和用户组

linux下有不同的身份等级，包括以下三种身份：

---
1. 文件所有者（user）
2. 用户组（group）
3. 其他人（others）
---

另外，linux中所有的用户账号，包括root相关信息，都记录在`/etc/passwd`文件内。

至于用户账号的密码，则是记录在`/etc/shadow`这个文件中。

而所有的用户组信息则记录在`/etc/group`这个文件内。

---

下面我们使用`ls -al`这个命令来查看下某个文件下的文件属性：

![](http://7xvlvo.com1.z0.glb.clouddn.com/shell_ls.png)

其中第一列即代表文件的类型和权限，其中**第一位说明了文件类型，后面9个字符，每3位为一组**，分别代表`user`，`group`，`others`这三种身份的文件权限。

比如`add.sh`这个文件来说，其文件所有者（user）对应的权限为`rw-`，用户组（group）对应的文件权限为`rw-`，而其他人（others）对应的用户权限为`r--`。

另外，第二列和第三列分别对用该文件的文件所有者和用户组，这里都为xwj。

## 基本权限

上面提到的`rwx`即为文件的三种基本权限，它们对文件和目录具有不同的意义：

### 对文件的意义

`r（read）`：可读取此文件的内容
`w（write）`：可以编辑此文件内容（并不代表可以删除该文件）
`x（execute）`：可以执行该文件

### 对目录的意义

文件用户存放实际数据，而目录用于**记录文件名列表**。

基本权限对于目

`r（read contents in directory）`：具有读取目录结构列表的权限，所以当你具有某个目录的r权限时，可以**利用ls来将该目录的所有文件名显示出来**（如果仅有r权限，仅可以显示文件名）。

`w（modify contents of directory）`：具有更改目录结构列表的权限，比如**在该目录下新建、删除、重命名文件等**。

`x（access directory）`：具有进入该目录的权限，即可以cd来将此目录当作工作目录。

### chmod修改文件权限

以上简要介绍了3中基本文件权限，我们可以使用`chmod`命令来修改文件权限。

我们可以使用两种方法来改变文件权限：

**一、符号类型改变文件权限**

比如还是上图的`add.sh`，它的权限为`rw-rw-r--`，即文件所有者和用户拥有该文件的读写权限，而其他人仅具有读的权限。

我们现在要将它的权限更改为`rwxr-x--x`，要如何更改呢？

可以通过以下这条命令：

{% codeblock lang:sh %}
$ chmod u=rwx,g=rx,o=x add.sh
{% endcodeblock %}

或者这样也可以：

{% codeblock lang:sh %}
$ chmod u+x,g-w+x,o-r+x add.sh
{% endcodeblock %}

**二、 数字类型改变文件权限**

linux中每个权限对应一个数字分值，如下：

---
- r：4
- w：2
- x：1
---

假如同样要将某个文件的权限由`rw-rw-r--`更改为`rwxr-x--x`，那么我们可以使用以下命令：

{% codeblock lang:sh %}
$ chmod 751 add.sh
{% endcodeblock %}

### 改变文件所有者和用户组

除了更改文件权限之外，我们当然也可以更改文件的**文件所有者**以及**所属用户组**。

**一、更改文件所属用户组**

使用`chgrp`可以更改文件的用户组，比如我们要将`add.sh`这个文件的用户组从`xwj`更改为`root`。

{% codeblock lang:sh %}
$ chgrp root add.sh
{% endcodeblock %}

更改结果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/chgrp.png)

注意，需要使用`su`来切换账号到`root`才可以完成上面操作。

另外，如果想将整个文件夹及其子目录下的所有文件和目录都改变用户组的话，需要加上`-R`参数。

**二、更改文件所有者**

如果我们想将`add.sh`的文件所有者也改成`root`呢？

可以使用`chown`进行如下操作：

{% codeblock lang:sh %}
$ chown root add.sh
{% endcodeblock %}

结果如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/chown.png)

同样，这条命令也可以加上`-R`参数来对目录进行递归操作。

另外，chown还可以**同时更改文件所有者和用户组**，如下：

{% codeblock lang:sh %}
$ chown xwj:xwj add.sh
{% endcodeblock %}

以上命令，又将`add.sh`的文件所有者和用户组改回xwj。

## 默认权限

除了上面提到的文件基本权限`rwx`之外，在linux中，还存在**默认权限**。

**默认权限代表：目前用户在新建文件或者目录时候的默认权限值**。

我们可以先新建一个文件和目录查看一下默认权限：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E9%BB%98%E8%AE%A4%E6%9D%83%E9%99%90.png)

以上我们看到了新建文件和目录时的默认权限，当然，我们也可以通过`umask`这条命令来查看用户默认权限：

![](http://7xvlvo.com1.z0.glb.clouddn.com/umask.png)

对比上面两张图，我们可以明白：**umask分数是指最大权限需要减去的权限**。

---
- 对于一般文件来说，其最大权限为666，即`rw-rw-rw-`，而umask的后三位是`002`，则代表**从others的权限中拿掉分数为2的写权限w**。所以，文件的默认权限为`rw-rw-r--`。

- 对于目录来说，由于x与是否可以进入该目录有关，所以其最大权限为777，即`rwxrwxrwx`。根据umask的值，也很容易得出目录的默认权限为`rwxrwxr-x`。
---

假设两个用户属于同一个用户组，并且他们不想自己新建的文件或者目录被其他人修改。

那么，他们如何修改新建文件和目录的默认权限呢？方法如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%9B%B4%E6%94%B9umask.png)

注意：对于umask来说，有效的设定值为后三位，第一位恒为0。

## 特殊权限

linux中，除了`rwx`这三个基本文件权限之外，还会有特殊权限，比如：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E7%89%B9%E6%AE%8A%E6%9D%83%E9%99%90.png)

下面就来总结一下三种特殊的文件权限：`SUID`、`SGID`、`SBIT`。

### SUID

当`s`的标志出现在文件所有者的`x`权限上时，比如`/usr/bin/passwd`这个文件的权限状态`rwsr-xr-x`，此时就被称为`set UID`，简称为`SUID`的特殊权限。

它具有以下几点限制和功能：

---
1. SUID权限**仅对二进制文件有效**。
2. 执行者对于该程序**需要有x的可执行权限**。
3. SUID权限**仅在执行该程序的过程中有效**。
4. 执行者**将具有程序所有者的权限**。
---

就拿`passwd`这个命令来说，它会修改`/etc/shadow`中的第二个字段，即用户密码字段。

而`/etc/shadow`这么重要的文件，自然为root所有，普通用户没有读写权限。

但是实际上每个用户都可以通过passwd`这个命令来修改自己账户的密码，同时也就修改了`/etc/shadow` 这个文件的内容。

为什么呢？这就是SUID的功能！是**SUID让用户在执行passwd这个命令的过程中，暂时获得了root的权限了**。

以上，用一张图来展示：

![](http://7xvlvo.com1.z0.glb.clouddn.com/SUID.png)

### SGID

当s标志在用户所有者的x位置时为SUID，那么**s在用户组的x位置时成为Set GID，简称SGID**。

![](http://7xvlvo.com1.z0.glb.clouddn.com/locate.png)

与SUID不同，**SGID不仅可以用在二进制程序上，还可以用在文件或者目录上**。

SGID对二进制文件具有以下限制或作用：

---
1. 程序执行者**需要对二进制程序具备x权限**。
2. 与SUID类似，执行者在执行的过程中将**获得改程序用户组的支持**。
---

除了二进制文件之外，SGID也经常用在目录上，这使得目录具有以下功能：

---
> 1. 用户在此目录下的**有效用户组（effictive group）**将会变成该目录的用户组。
> 2. 用户在此目录下**创建新文件的用户组与此目录的用户组相同**。
---

关于GID的应用见最后的案例。

### SBIT

SBIT是`Strick Bit`的简称。

**它只针对目录有效**：当用户在目录下创建文件或者目录时，**仅有自己与root才有权力删除该文件**。

所以，当给某个目录加上SBIT权限之后，某个普通用户再也无法随便删除别人创建的文件了，比如我们的`/tmp`目录。

### SUID/SGID/SBIT的权限设置

跟`rwx`权限具有不同的分值一样，**SUID**，**SGID**，**SBIT**也分别具有不同的分值：

---
**SUID**：4
**SGID**：2
**SBIT**：1

---
假如我们想给某个目录`/test`加上`SGID`的权限，使其权限变为`rwxr-sr-x`。

则执行以下命令：

{% codeblock lang:sh %}
$ chmod 2755 /test
{% endcodeblock %}

## 一个案例

最后来看一个小小的案例：

在我的系统中有一个用户组`users`，并且该用户组下有两个用户`xwj`和`hyy`。
现在要求`root`账户为这两个普通用户新建一个用于共同开发的目录`/srv/project`，要求该目录不允许其他人查阅，那么应该如何设置呢？

首先我们使用`root`新建目录`/srv/project`，并且将其支持用户组改为`users`，权限设置为`rwxrwx---`。

![](http://7xvlvo.com1.z0.glb.clouddn.com/SGID%E5%BA%94%E7%94%A8.png)

然后先用`xwj`去创建一个文件`init.sh`，然后用账户`hyy`去尝试修改这个文件：

![](http://7xvlvo.com1.z0.glb.clouddn.com/SGIDtest.png)

我们发现，`xwj`新建的文件，`hyy`是没办法修改的！因为`init.sh`对于其他人的权限是只读。

**这时候，SGID就派上用场啦！**

我们回到`root`账户，将目录`/srv/project`加上`SGID`的权限：

{% codeblock lang:sh %}
$ chmod 2770 /srv/project
{% endcodeblock %}

然后使用`xwj`账户再新建一个文件`init2.sh`：

![](http://7xvlvo.com1.z0.glb.clouddn.com/SGIDtest2.png)

我们看到文件`init2.sh`所支持的用户组变为了`users`，这样账户`hyy`因为属于`users`用户组，自然可以更改`init2.sh`了。

两个人终于可以愉快地一块开发了！















