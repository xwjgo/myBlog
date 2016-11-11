title: 04-github团队协作
tags: github
categories: github使用培训
toc: true
date: 2016-7-13 12:12:12
---

在你学会了git的基本使用之后，你可以完成将github中项目文件克隆到本地，在本地完成修改后再提交到github的任务。

## 另一名小伙伴

现在假如又多了一名小伙伴`小A`要跟你一起来完成项目，所以你和`小A`两个人都同时拥有对该仓库的修改权。

这时候，工作流程就会发生一些变化，比如你们两个同时将仓库文件`newFile.txt`克隆到本地，并且都做了自己的修改，那么推送到远程的时候以谁的为准呢？

还有很多类似的情况，所以，如何利用github来进行多人协作，就是你必须要解决的问题。

## 案例介绍

假设我们在github的`Test`仓库中又新建了一个文件`newFile.txt`，并且该仓库只有`master`分支。

![](http://7xvlvo.com1.z0.glb.clouddn.com/21-%E6%96%B0%E5%88%9B%E5%BB%BA%E7%9A%84%E6%96%87%E4%BB%B6.png)

`newFile.txt`文件刚开始只有3行内容：

{% codeblock lang:html %}
one
two
three
{% endcodeblock %}

下面将会发生以下几个事件：

**1.同时克隆**

你和另外一个小伙伴`小A`依次将远程仓库`Test`克隆到了本地，准备要修改。

**2.小A 的修改**

`小A`在文件`newFile.txt`的末尾加了一行，最后将修改提交到了远程仓库`master`分支中。

现在远程仓库中`newFile.txt`内容如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/22-%E5%B0%8FA%E6%96%B0%E5%8A%A0%E7%9A%84%E4%B8%80%E8%A1%8C.png)

**3.你的修改**

在`小A`将修改推送到远程之后，你也对`newFile.txt`作出了如下修改：

{% codeblock lang:html %}
one
two
three
four
five
{% endcodeblock %}

做完修改之后，你试图通过`git push`命令将修改提交到远程仓库，可是，错误发生了：

{% codeblock lang:html %}
$ git push origin master        // 将修改推送到远程
Username for 'https://github.com': xwjlearning
Password for 'https://xwjlearning@github.com': 
To https://github.com/xwjlearning/Test.git
 ! [rejected]        master -> master (fetch first)
error: failed to push some refs to 'https://github.com/xwjlearning/Test.git'
hint: Updates were rejected because the remote contains work that you do
hint: not have locally. This is usually caused by another repository pushing
hint: to the same ref. You may want to first integrate the remote changes
hint: (e.g., 'git pull ...') before pushing again.
hint: See the 'Note about fast-forwards' in 'git push --help' for details.
{% endcodeblock %}

发生这样因`并行操作`而发生`提交冲突`的的问题，如何解决呢？

## 解决冲突

其实从上面的错误信息中，git就已经告诉了我们解决办法：先用`git pull`命令将`小A`的提交从远程抓下来，然后，再本地解决完冲突之后，再提交。

首先，你可以先从远程拉取`小A`的更新：

{% codeblock lang:html %}
$ git pull origin master        // 从远程拉取更新
remote: Counting objects: 7, done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 7 (delta 1), reused 7 (delta 1), pack-reused 0
Unpacking objects: 100% (7/7), done.
From https://github.com/xwjlearning/Test
 * branch            master     -> FETCH_HEAD
   6f8259b..543b076  master     -> origin/master
Auto-merging newFile.txt
CONFLICT (content): Merge conflict in newFile.txt
Automatic merge failed; fix conflicts and then commit the result.
{% endcodeblock %}

因为两个人同时对一个文件进行了修改，所以git在自动合并`newFile.txt`的时候，发生了错误。

我们这时候打开本地的`newFile.txt`，你会发现它的内容如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/Screenshot%20from%202016-06-30%2021-02-31.png)

合并冲突即修改这个文件，最终修改如下：

![](http://7xvlvo.com1.z0.glb.clouddn.com/Screenshot%20from%202016-06-30%2021-06-44.png)

解决完冲突之后，我们再依次执行以下3条命令，将更改提交至远程：

{% codeblock lang:html %}
$ git add newFile.txt
$ git commit -m "合并了两个人的修改"
$ git push origin master
{% endcodeblock %}

最终出现如下提示，则表示推送成功：

{% codeblock lang:html %}
Counting objects: 6, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 608 bytes | 0 bytes/s, done.
Total 6 (delta 2), reused 0 (delta 0)
To https://github.com/xwjlearning/Test.git
   543b076..b4a8d65  master -> master
{% endcodeblock %}

至此，一个简单的两人通过github实现协同开发的例子就结束了！

## 更多技能

这里还总结了几条在实战中常用的git命令，希望你能亲自实践一遍：

**1.查看提交日志**

{% codeblock lang:html %}
$ git log             // 查看提交的历史记录
{% endcodeblock %}

运行截图如下：

![]()

**2.版本回退**

注意`commit`后边的一大串字符是`版本号`，这个可以帮你回退到某一个历史版本。

比如你要回退到`小A的修改`这一个版本，就可以使用以下命令：

{% codeblock lang:html %}
$ git reset --hard 543b076a9447c25429a4fce0ae2032bb4c0b0772   // 回到某一历史版本，版本号只需要前几位就行
HEAD is now at 543b076 小A 的修改
{% endcodeblock %}

或者使用以下命令，来回到前一个版本：

{% codeblock lang:html %}
$ git reset --hard HEAD^     // 回退到当前版本的上一个版本
HEAD is now at b0793d9 我的修改
{% endcodeblock %}

`HEAD`后面`^`加入有`N`个，那就代表回到当前版本的前`N`个版本。

**3.关于push和pull的参数**

在使用`git push`和`git pull`时，他们的参数都是一样的，这里以`git push`为例：

{% codeblock lang:html %}
$ git push <远程主机名> <本地分支名>:<远程分支名>        // 将本地分支推送到远程分支
{% endcodeblock %}

## 总结&任务

通过这个案例，你应该通过实践来掌握以下知识：

1. 找到一个小伙伴，共同来修改某个项目文件。
2. 从远程抓取更新
3. 处理冲突



