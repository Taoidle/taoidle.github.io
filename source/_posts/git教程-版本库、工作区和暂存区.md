---
title: Git教程 —— 版本库、工作区和暂存区
tags:
  - git
  - 教程
id: '113'
categories:
  - - Git教程
date: 2020-03-16 10:17:41
---

## Git的基本工作流程

1.  创建Git工作目录和版本库
2.  添加修改删除文件
3.  git add 命令将文件修改添加到暂存区
4.  git commit 命令将暂存区所有内容提交到当前分支（创建版本库时git自动创建一个master分支）

## 版本库

版本库又名仓库，英文名repository，可简单理解为一个文件目录，目录里所有的文件都受Git管理。Git会记录该目录下文件的修改删除，这些记录在将来需要恢复某个记录文件时起到至关重要的作用。

## 文件添加

文件添加指的是将文件添加到暂存区，这里我们回顾一下前面的例子，在前边的例子中，我们使用了git add 1.c命令将我们的文件1.c 添加到了暂存区，然后我们使用了git status命令来查看项目状态，这里git给我们输出的信息在分支master中没有任何提交记录，但是1.c文已经添加到了暂存区。 命令：git add filename ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/edb84957b223d6a.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/034069cb3a181ff.jpeg)

## 暂存区

英文名stage或index，一般在.git目录下的index文件中，所以暂存区有时也被称为索引（index）。

## 工作区

就是你在电脑里能看到的目录。

## 一个简单的示例

我们通过一个简单的示例来理解上面几个概念。 首先，我们打开Git Bash，在桌面上创建一个目录文件叫git\_demo，然后进入git\_demo目录。这个时候git\_demo就是我们的工作目录。此时我们的Git自动帮我们创建了第一个分支master 以及一个指向master的指针HEAD。 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/e9a350a6a586253.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/3702b46d7173cbf.jpeg) 接着，我们输入git init命令，在git\_demo目录下创建版本库，此时你会发现在git\_demo目录下多了一个隐藏文件夹 .git ,该文件夹就是我们的版本库。 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/63d96c143a063ca.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1c84c81e4ca26e5.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/572ecd978d78681.jpeg) 然后，我们使用touch 1.c命令在工作区里创建一个.c文件，并用git add 1.c将文件添加到暂存区。我们使用命令git status查看文件状态，git log查看git日志，看到文件尚未被提交，在git日志中master分支没有任何提交记录，得知我们添加的1.c文件仍在暂存区。 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/e437e7368ffb667.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/d6ebcb7047443eb.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/bf1a70e8ce730db.jpeg) 最后，我们使用git commit命令将暂存区内容提交到分支master中。这里git commit -m “create 1.c”中””中内容为此次提交的说明。我们用git status 和 git log 看一下提交之后文件处于什么状态。通过git status我们看到当我们一旦提交后没有做任何修改，那么工作区就是干净的。在git log中我们的master分支上出现了第一次提交记录，该提交记录的说明是 create 1.c。 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/8f1c4bdc7d73fe6.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1f5f2e6a346d0e0.jpeg) 到这里，我们的版本库暂存区和工作区的内容就讲完了，但是我们不妨继续做实验加深对git的理解。接下来我们对1.c进行修改，比如我们写个hello world程序，然后我们用git add 1.c把它添加到暂存区，然后我们继续对1.c程序进行修改，比如写一行注释，最后我们git commit一下把东西提交。 此时我们使用git status查看状态发现我们对1.c的第二次修改并没有被提交，而且使用git diff HEAD -- 1.c命令查看文件不同时git明确的给我们指出来这多了一行//hello world，这是为什么呢？这里我们需要清楚一个概念，在git的管理中，git对于版本的控制实质上是对修改进行追踪。什么意思呢？你对文件的增删改对git来说都只是修改，git并不会追踪文件的本身。我们回到刚刚的实验，最初我们在1.c中写了一个hello world程序，然后我们把1.c添加到了暂存区，这里实际上并不是把1.c文件添加到了暂存区，而是将1.c文件的修改提交到了暂存区，然后我们后面对1.c写了一行注释，这个改动并没有被添加，所以commit后版本库和工作区是不一致的。 ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/46086c6030d0a2f.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/1846795b60e6210.jpeg) ![](https://cdn.jsdelivr.net/gh/taoidle/taoidle.github.io@master/assets/images/ac72b20e5bce2da.jpeg)