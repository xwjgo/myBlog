title: 02-github基本使用
tags: github
categories: github使用培训
toc: true
date: 2016-7-08 12:12:12
---
接下来，通过一个案例来介绍github的基本使用，希望你能够跟着案例边学边做！

## 案例介绍

在这个案例中，你完成以下几件事情：

1. 创建github账户并且登陆
2. 创建一个新的仓库
3. 新建文件
4. 创建分支
5. 在分支中修改文件
6. 发送Pull Request 
7. 合并Pull Request 

以上任务不用写任何代码，在gihub网站上就可以在线完成。

现在开始，跟着案例走一遍吧！

## 创建github账户并且登录

这个流程很简单，就不多说了，点击[这里](https://gtihub.com)进入github官网，注册一个帐号并且登录上去吧！

## 创建新的仓库

什么是仓库(repository)？

一个仓库通常就是一个项目。这个仓库里可以放一些文本文件，文件夹，图片，视频等任何你的项目中所需要的东西。

在github中创建一个仓库非常简单：

1. 在首页的右上角，点击 `＋` ，然后选择 `New repository`。
2. 给你的仓库起一个名字，比如 `Test`。
3. 为这个仓库写一个简短的描述。
4. 为这个项目初始化一个 `README.md` 文件。

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/02-%E5%88%9B%E5%BB%BA%E4%BB%93%E5%BA%93.png)
__---

点击 `Create repository`，恭喜，你的仓库创建完成！

## 新建文件

创建了仓库之后，在里面添加你的第一个文件吧！

1. 点击仓库首页中`Create new file`。
2. 输入文件名，比如 `github.txt`。
3. 添加文件内容。
4. 添加提交信息，比如 `创建了一个新文件`。

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/03-%E6%96%B0%E5%BB%BA%E6%96%87%E4%BB%B6.png)
__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/04-%E6%8F%90%E4%BA%A4%E4%BF%A1%E6%81%AF.png)
__---

点击`Commit new file`,创建新文件成功，并且回到仓库首页。

这时候，你会发现你的仓库中已经有了两个文件：一个是README.md，是在你创建仓库时自动创建的；另一个是刚才新建的文件 github.txt。

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/05-%E4%B8%A4%E4%B8%AA%E6%96%87%E4%BB%B6.png)
__---

## 创建分支

什么是分支(branch)？

默认情况下，你的仓库只有一个分支`master`，我们之前创建新文件的操作就在这个分支上。

当你创建一个新的分支，比如`newB`分支时，其实是将分支`master`中的所有文件克隆了一份到`newB`分支,以后你就可以在这个分支中做开发。当在这个分支中进行了一些改动后，可以将之合并到`master`分支。

利用分支，你可以实现多人协同开发，比如为开发团队中的每个人都创建一个分支，大家在自己分支中完成开发后，再提交到`master`分支中。

用图来说明一下github分支的理念：

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/06-%E5%90%88%E5%B9%B6%E5%88%86%E6%94%AF.png)
__---

现在，你可以创建`newB`分支了。到你的Test仓库首页，进行如下操作：

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/07-%E6%96%B0%E5%BB%BA%E5%88%86%E6%94%AF.gif)
__---

在创建完成之后，将会自动切换到`newB`分支。

## 在分支中修改文件

现在你可以来修改`newB`分支中的`github.txt`。直接点击`github.txt`，会进入以下界面：

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/07-%E5%9C%A8%E7%BA%BF%E7%BC%96%E8%BE%91.png)
__---

我们可以在文件中作出以下修改：删除第一行，在最后加上两行，或者其他任意修改。

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/08-%E5%9C%A8%E7%BA%BF%E7%BC%96%E8%BE%91.gif)
__---

修改结束后，可以为这次修改加上个一个标题和描述：

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/09-%E6%9B%B4%E6%94%B9%E4%BF%A1%E6%81%AF.png)
__---

点击`Commit changes`，现在你已经成功了修改了`newB`分支中的`github.txt`文件。但是`master`分支中的`github.txt`却不受影响。

## 发送Pull Request

假设现在你已经在`newB`分支上做了非常漂亮的修改，现在要把它合并到`master`分支中，你要怎么做？

当然是 open a `Pull Request`！。

`Pull Request`是github实现团队协作开发的核心。当你在完成自己分支的改动并且发出`Pull Request`后，目标仓库管理者能够看到`Pull Request`的改动内容，并且通过讨论决定要不要将改动合并到自己的仓库中。

在`newB`分支的主页中，点击`Branch:newB`后的`New pull request`即可进入以下界面：

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/10-open%20a%20pull%20request.png)
__---

点击 `Create pull request`,完成发送。

## 合并Pull Request

最后一步，是时候把你分支`newB`上的内容合并到`master`中了。

在进行完上一步的操作之后，会进入标签`Pull requests`下面：

1. 点击绿色的`Merge pull request`按钮。
2. 点击`Confirm merge`
3. 删除`newB`分支，因为它的更改已经合并到`master`分支了。

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/13-%E5%90%88%E5%B9%B6%E5%88%86%E6%94%AF.png)
__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/14-Confrim%20merge.png)
__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/12-%E5%88%A0%E9%99%A4%E5%88%86%E6%94%AF.png)
__---

这时候再回到Test仓库的首页，打开查看github.txt，发现它的内容也已经改变了。

__---
![](http://7xvlvo.com1.z0.glb.clouddn.com/15-%E6%94%B9%E5%8F%98%E6%88%90%E5%8A%9F.png)
__---

Celebrate！你已经成功将`newB`分支上的改动合并到了分支`master`上!

至此，github的基本使用就先告一段落。

## 总结&任务

我们总结本节所学内容如下，希望你能亲自实践一遍：

1. 创建github账户并且登陆
2. 创建一个新的仓库
3. 新建文件
4. 创建分支
5. 在分支中修改文件
6. 发送Pull Request 
7. 合并Pull Request 



