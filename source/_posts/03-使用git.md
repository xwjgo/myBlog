title: 03-使用git
tags: github
categories: github使用培训
toc: true
date: 2016-7-10 12:12:12
---

到现在为止，你已经掌握了github的基本使用，学会了在github中新建自己的项目仓库，完成对项目文件的增删改查以及对分支的创建合并等操作。

## 那为什么还要使用git？

但是在实际项目开发中，不可能所有操作都在线完成。比如没有网络的时候，或者项目文件很多且很大时,在线操作将非常不方便。

那么，有没有什么工具能够将github上的操作都搬到本地呢？或者说我能不能在本地修改完后再提交到github呢？

答案是有的，git可以提供所有的这些功能。

在[ 01-认识github ]( http://xwjgo.github.io/2016/06/20/01-%E8%AE%A4%E8%AF%86github/ )一节中，我们提到过，github是作为git的远程仓库而存在的。所以，在接下来的案例中，我们将把远程仓库即github中的`Test`仓库克隆(复制)到本地进行一系列的操作，然后再提交到github的远程仓库中。

本节你将学习git的简单使用，并且这将加深你对github以及git关系的理解，同样，边学边练！

## 案例介绍

在这个案例中，你将完成以下几件事情：

1. 下载并安装git
2. 克隆远程仓库`Test`到本地
3. 创建本地分支`newB`
4. 在本地分支`newB`中修改文件
5. 将修改提交到本地git仓库
6. 将`newB`分支合并到`master`分支
7. 添加`SSH key`
8. 将本地修改推送到远程github仓库

注：本节所有命令均在 Ubuntu 16.04 环境下运行，并且所有能在终端运行的命令前面都用 `$` 标识。

## 下载并安装git

**1.下载**

点击[这里](https://git-scm.com/downloads)下载适合自己系统的git安装包。

**2.Window平台安装**

window平台安装很简单，直接下一步就可以。
安装完成之后，在开始菜单找到`Git`->`Git Bash`，会弹出Git命令窗口，你可以在该窗口进行Git操作。

**3.Linux平台安装（以Ubuntu为例）**

在终端中输入以下命令安装：

{% codeblock lang:javascript %}
$ sudo apt install git
{% endcodeblock %}

**4.检验是否安装成功**

在终端中输入命令可以查看git版本号：

{% codeblock lang:python %}
$ git --version
git version 2.7.4
{% endcodeblock %}

这代表git安装成功！

**5.配置git**

安装完git后，每个机器都需要自报家门，告诉git你是谁，执行以下两条命令：

{% codeblock lang:html %}
$ git config --global user.name <yourName>        // yourName 即你在github上注册的帐号
$ git config --global user.email <yourEmail@example.com>        // yourEmail 即你在注册github时用到的邮箱
{% endcodeblock %}

执行完以上内容之后，你就可以尽情的使用git了！

## 克隆远程仓库Test到本地

还记得我们之前在github上新建的仓库`Test`吗？现在你可以将它从github的远程仓库中克隆到本地的`gitProject`文件夹中，你需要做以下两件事：

第一步，复制远程`Test`仓库的地址,操作如下：

1. 进入github中`Test`仓库首页。
2. 点击仓库右侧的绿色按钮`Clone or download`按钮。
2. 复制本仓库的url(或者点击url右侧图标)。

---

![](http://7xvlvo.com1.z0.glb.clouddn.com/16-%E5%A4%8D%E5%88%B6URL.gif)

---

第二步，打开终端，依次运行以下命令：

{% codeblock lang:html %}
$ cd gitProject        // 进入gitProject文件夹
$ git clone https://github.com/xwjlearning/Test.git     // 最后的url即你刚刚复制的url
{% endcodeblock %}

如果clone成功，则会出现以下消息：

{% codeblock lang:html %}
Cloning into 'Test'...
remote: Counting objects: 28, done.
remote: Compressing objects: 100% (22/22), done.
remote: Total 28 (delta 3), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (28/28), done.
Checking connectivity... done.
{% endcodeblock %}

OK! 现在你打开`gitProject`文件夹，已经可以看到里面多了一个`Test`的文件夹，这样就完成了将github远程仓库克隆到本地的任务！

## 创建本地git分支 newB

现在你就可以在本地操作仓库`Test`中的文件了，即使没有网络，也不影响你的正常工作，因为你可以在本地修改，等到有网络的时候再提交到github上。

另外，现在本地的这个`Test`文件夹已经成为了一个本地的git仓库，也就是说，它已经是一个git可以管理的文件夹了，你所有的改动都会被它记录下来，并且你可以随时回到之前的某一个版本！这就是git的有用之处。

接下来，假设你想要修改`Test`文件夹下的`github.txt`文件。但是，和在github上直接操作一样，你需要新建一个分支`newB`用于开发，这样能够避免对`master`分支造成破坏。

在终端通过`cd`命令进入`Test`文件夹后，执行以下命令，查看当前所有分支：

{% codeblock lang:html %}
$ git branch        // 查看所有分支
* master        // 当前只有一个分支，master, 星号表示当前所在分支
{% endcodeblock %}

因为当前只有一个分支，所以，你需要通过以下命令新建并且转到分支`newB`:

{% codeblock lang:html %}
$ git branch newB        // 创建分支 newB
$ git checkout newB      // 切换到分支 newB
{% endcodeblock %}

以上两条命令可以合并为一条命令：

{% codeblock lang:html %}
$ git checkout -b newB      // 创建并转到分支 newB
{% endcodeblock %}

在运行完以上命令后，再用`git branch`命令查看分支，会有以下结果：

{% codeblock lang:html %}
  master
* newB        // 这表示当前分支已经在newB上了
{% endcodeblock %}

另外，你可以用`git status`来查看当前仓库的状态：

{% codeblock lang:html %}
$ git status        // 查看当前仓库状态
On branch newB        // 表示当前在分支 newB 上
nothing to commit, working directory clean        // 表示没有什么需要提交
{% endcodeblock %}

## 在本地分支newB中修改文件

经过以上的操作，现在你可以放心大胆的修改项目中的文件了！

第一次修改，`github.txt`的末尾添加了两行新内容。现在`github.txt`文件内容如下：

{% codeblock lang:html %}
ONE
TWO
THREE
I'm the added line.
I'm alse the added line.
我是在本地添加的！
我也是在本地添加的！
{% endcodeblock %}

做完第一次修改后，你可以再用`git status`查看当前仓库的状态：

![](http://7xvlvo.com1.z0.glb.clouddn.com/git%20status)

其中红色的部分`modified:    github.txt`表示文件`github.txt`已经修改，并且还没有加到暂存区。

## 将修改提交到本地git仓库

将修改文件提交到git仓库需要经过以下两步：
第一步，执行以下命令，将`github.txt`加入git的暂存区：

{% codeblock lang:html %}
$ git add github.txt        // 将 github.txt 加入暂存区
{% endcodeblock %}

如果将`github.txt`改成`.`，则将所有改动文件加入暂存区

{% codeblock lang:html %}
$ git add .        // 将所有改动的文件加入暂存区
{% endcodeblock %}

这样再次查看当前仓库状态：

![](http://7xvlvo.com1.z0.glb.clouddn.com/Screenshot%20from%202016-06-29%2022-06-12.png)

你可以发现，`modified:    github.txt`这行信息变成了绿色，这表示你对`github.txt`文件的修改已经提交到了缓存区。

第二步，你可以使用`git commit`命令将所有缓存区的文件提交到git仓库中了！

{% codeblock lang:html %}
$ git commit -m "末尾增加了两行"        // 将缓存区保存的改动提交到git仓库中
[newB 53949b6] 增加了两行
1 file changed, 2 insertions(+)
{% endcodeblock %}

-m 之后的参数，是为这次提交所加的标签。

这样你就将缓存区的改动文件提交到git仓库中了！现在你可以再次查看git仓库的状态：

{% codeblock lang:html %}
$ git status        // 查看 git 仓库状态
On branch newB
nothing to commit, working directory clean
{% endcodeblock %}

这时候你会发现，缓存区是干净的，没有什么需要提交。

恭喜，你完成了将改动提交到git仓库的流程，总结来说只有以下两条命令。

{% codeblock lang:html %}
$ git add <filename>              // 将改动文件加入到缓存区
$ git commit -m <message>         // 将缓存区的文件提交到git仓库
{% endcodeblock %}

## 将 newB 分支合并到 master 分支
 
假设你已经做过了很多次修改，并且都`commit`到了git仓库中，不过这些改动都还是在`newB`分支中，`master`分支中还是原来的旧文件。

你可以通过以下命令将`newB`分支合并到`mater`分支：

{% codeblock lang:html %}
$ git checkout master          // 切换到 mater 分支
$ git merge mewB               // 合并 newB 分支
Updating a17fea6..53949b6
Fast-forward
 github.txt | 2 ++
 1 file changed, 2 insertions(+)
{% endcodeblock %}

在合并完newB分支之后，newB分支就可以删除了：

{% codeblock lang:html %}
$ git branch -D newB         // 删除 newB 分支
Deleted branch newB (was 53949b6).
{% endcodeblock %}

## 添加 SSH key

通过以上操作，你已经完成了在本地的所有操作，现在是时候将本地的文件同步到远程 github 仓库中了，不过，在正式推送之前，你需要一点点设置。

因为你本地的git仓库和github仓库之间的传输是通过SSH加密的，所以，你需要告诉github你的公钥，这样github才能确定发起推送的机器是谁，防止冒充。

这个过程有两部分组成：

**1.创建SSH key**

在终端中运行以下命令：

{% codeblock lang:html %}
$ ssh-keygen -t rsa -C "yourEmail@example.com"        // 创建 SSH key
{% endcodeblock %}

你需要将eamil换成你注册github时使用的邮箱，然后一路回车，使用默认值即可。

注意，在刚才的终端中，你需要稍稍留意以下两行内容：

{% codeblock lang:html %}
Your identification has been saved in /tmp/guest-3fktb1/.ssh/id_rsa.
Your public key has been saved in /tmp/guest-3fktb1/.ssh/id_rsa.pub.
{% endcodeblock %}

进入`.ssh`文件夹，你会发现里面有两个文件，一个是`id_rsa`，这是私钥，不要泄露出去；另一个是`id_rsa.pub`，这个是公钥，可以放心的告诉他人。

打开`id_rsa.pub`，并复制其中的内容，`id_rsa.pub`中内容大致如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/index2.png)

**2.在github上添加SSH key**

登录你的github，点击右上角你的头像，然后选择`Settings`，进入以下界面进行操作：

![](http://7xvlvo.com1.z0.glb.clouddn.com/%E6%B7%BB%E5%8A%A0sshkey.png)

点击`Add SSH key`，完成SSH key的添加。这样，你的机器就可以向这个github帐号推送修改了！

## 将本地修改推送到远程github仓库

最后一步，开始推送！

当你从远程仓库克隆时，实际上git自动把本地的`master`分支和远程的`master`分支对应起来了，并且，远程仓库的默认名字是`origin`。

你可以用`git remote -v`来查看远程仓库的信息：

{% codeblock lang:html %}
$ git remote -v
origin https://github.com/xwjlearning/Test.git (fetch)        // 克隆的地址
origin https://github.com/xwjlearning/Test.git (push)         // 推送的地址
{% endcodeblock %}

如果没有推送权限，你就看不到`push`的地址。

现在你可以执行`git push`命令，把本地的所有提交推送到远程仓库。推送时，要制定本地分支名，这样，git就会把该分支推送到远程仓库对应的远程分支上：

{% codeblock lang:html %}
$ git push origin master         // 将本地的master分支推送之远程仓库的master分支
{% endcodeblock %}

注意，在推送到远程分支的时候，需要你输入github帐号和密码。

如果出现以下消息，恭喜，推送成功！

{% codeblock lang:html %}
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 370 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/xwjlearning/Test.git
   a17fea6..53949b6  master -> master
{% endcodeblock %}

这时候，你通过浏览器登录github网站，查看`github.txt`的文件内容，会发现，它已经发生了改变！

![](http://7xvlvo.com1.z0.glb.clouddn.com/20-%E6%88%90%E5%8A%9F%E6%8E%A8%E9%80%81%E8%BF%9C%E7%A8%8B.png)

至此，整个案例结束！

## 总结&任务

我们总结本节所学的知识如下，希望你能够亲自实际操作一遍：

1. 下载并安装git
2. 克隆远程仓库`Test`到本地
3. 创建本地分支`newB`
4. 在本地分支`newB`中修改文件
5. 将修改提交到本地git仓库
6. 将`newB`分支合并到`master`分支
7. 添加`SSH key`
8. 将本地修改推送到远程github仓库


