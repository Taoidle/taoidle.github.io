---
title: Git教程 —— Git的基本操作
tags:
  - git
  - 教程
id: '179'
categories:
  - - Git教程
comments: false
date: 2020-04-22 12:00:08
---

## 创建版本库

在前边的例子中，我们已经创建过的版本库，这里我们来回顾一下，首先我们找了一个合适的地方创建了一个空目录，然后我们使用git init 命令让该目录变成git仓库，这里我们看到git 输出信息Initalized empty Git repository，这是告诉我们这是一个空仓库。 注意，细心的你可能发现在git\_demo目录下有一个.git目录（该文件是隐藏的，需要在查看中勾选隐藏的项目选项） .git目录是git用来管理版本库的，如果乱改该目录下的文件git仓库会被破坏。

```shell
命令：git init
```

![](/assets/images/4aef82c398791fa.png) ![](/assets/images/e9079bfaf294f0e.png)

### 文件提交

文件提交是将暂存区中所有的内容添加到分支中，在上面的例子中我们使用命令git commit -m “create 1.c” 将暂存区的文件提交到分支master，这里我们使用了参数 -m 参数后引号的内容是此次提交的标注。然后我们用git status查看状态，此时工作区内容已经干净了，暂存区的文件已经提交到了分支上，所以git 告诉我们 working tree clean，并且我们用git log命令查看git 的日志信息，在日志信息中明确记录了我们有过一次提交记录，此记录中注明了”create 1.c”。

```shell
命令：git commit
```

![](/assets/images/121c09c74678b10.png)

### 版本回退

到目前位置，我们的git上只提交了一个版本，现在我们对文件进行修改，我们用编辑器在1.c中写一个hello world程序然后保存退出，然后我们用git status查看状态，这里git输出信息文件处于modified状态，表示文件仅被修改，但没有做其它任何操作。接着我们将文件添加到暂存区并提交，然后再git status 和 git log看一下。 ![](/assets/images/8d2cae96cabc9ca.png) ![](/assets/images/b1299c0dbb11d65.png) ![](/assets/images/4e592c7c30e6a94.png) 此时我们可以看到git log的输出信息上出现了两次commit 这就是我们提交的两次版本，第一次我们创建了1.c文件然后提交，第二次我们在1.c中写入了一个hello world程序提交。我们不妨对1.c文件做第三次修改，我们封装一个函数专门用来输出hello world，然后提交。 ![](/assets/images/2c7fee8816f8b8f.png) ![](/assets/images/c83237fb552a084.png) 这时，我们master分支上已经有三次提交记录了，我们看到这里我们的1.c有三个版本，分别为”create 1.c” “write hello world demo” “ add func\_p”。那么如果我们不想要这个封装好的函数，要从”add func\_p”这个版本直接回到上一个版本”write hello world demo”应该怎么做呢？别急，我们先看看git log给我们输出的信息都是什么。 首先我们看到日志的第一行

```shell
commit 8abb13d8fd51b4bd6f6b9aaedac4add3ed368de8 
```

这是commit id即版本号，当然你这串数字肯定和我的不一样，这一串数字是哈希值。 然后我们看第二行 Author ,Author是提交者的信息即我们安装Git后最先设置的 Username email@example.com 接着我们看第三行 Date ,很明显了，这一行就是提交的时间 最后我们看到了 add func\_p 这就是我们在提交的时候标注的信息 当然，聪明的你一定想说那关键信息不就是版本号和提交标注吗？你直接给我输出这两个不就可以了吗，输出这么大堆东西看着头都大了。这肯定是可以的，我们在git log后加个参数--pretty=oneline试一下 ![](/assets/images/49012411684ba30.png) 你看，加上参数之后git就只给我们输出版本号和标注了。了解了这些之后，我们回归正题，如何将版本退回到”wirte hello world demo” 在我们前面的学习中我们知道git里有个指针HEAD指向当前版本，我想你肯定猜到了，我们只需要让该指针指向上一个版本就可以了。那我们来学一个新命令git reset --hard HEAD^ 这个命令是将HEAD指针指向上一个版本，那如果是指向上上个版本就是git reset --hard HEAD^^。我们先来试一下，在命令行中输入这个命令 ![](/assets/images/fb52be0f128a23d.png) 这是git输出信息HEAD指针已经指向了write hello world demo 这个版本，为了验证是不是真的退回了上一个版本，我们cat 1.c一下看看 ![](/assets/images/b05ed2e618d23cf.png) 从输出结果我们看到已经成功退回上一版，这时候你就有一个疑问了，要是以后版本多了比如出现了上千的版本那一直HEAD^^(若干个^)岂不是非常麻烦，这不必担心，如果是退回非常多个版本比如20个，那其实并不需要打20个^，我们可以这么写HEAD~20，而且还有另一个办法，在前面我们说到了commit id ，那我们是不是也可以通过commit id来退回版本呢？答案是肯定的，我们试试用commit id来退回到最初的版本。我们先git log看一下，我们看到git log的输出信息只剩下两个版本了，刚刚退回前的”add func\_p”版本已经不见了。我们找到”create 1.c”版本，将commit后边的id号复制后填写到git reset --hard id。在图中你看到我并没有填写整一串id，这是因为git会自动帮我们找，所以我们填写前边几位就行了，当然也不能只填写一两位，在版本多了情况下git可能会找到多个版本，这时就无法确定是哪一个了。 ![](/assets/images/f65d43654ec665e.png) 这个时候我们再来看一下git log发现只剩下一个版本了，但是这个时候你后悔了，你想回到”add func\_p”版本，那怎么办呢？还有办法吗？ ![](/assets/images/92048a292d8d286.png) 当然办法还是有的，在你还没有关闭命令行的情况下你只需要往前翻，找到之前”add func\_p”的commit id就可以了，就像下边这样。我们找到了”add func\_p”的commit id并且跳了回去，此时我们不仅直接跳回了”add func\_p”版，而且顺带把中间版本也找回来了。 ![](/assets/images/e58c5e029225fe6.png) 那如果说你在下班前退回了某个版本，然后关了笔记本电脑，回到家的时候你后悔了那怎么办？还有办法吗？在git中当然有办法了，git给我们提供了一个命令叫git reflog

```shell
命令: git reflog
```

这个命令记录了你每次命令。我们先退回到最初版，然后用git reflog看看信息。 ![](/assets/images/665e04f99e70483.png) 在退回最初版后我们在git reflog中依然能够找到”add func\_p”版本的commit id：8abb13d，那我们就git reset --hard id回到”add func\_p”版本：git reset --hard 8abb13d 然后git log看一下，成功回到”add func\_p”版本。 ![](/assets/images/203af278603416a.png)

### 撤销修改

你在写完1.c后给这个hello world程序加了一行注释，正准备add的时候你觉得这行注释完全没有必要存在，这个时候你想撤销刚刚的修改应该怎么做呢？我们只需要对1.c checkout ，怎么写这行命令呢？我们在命令行输入

```shell
命令：git checkout -- 1.c 
```

注意，这里要用两个- 如果少了一个-的话表示的是切换到一个叫1.c的分支。 ![](/assets/images/e6beb1dae77c36e.png) ![](/assets/images/d1459e26da56b7f.png) 我们看到这里刚刚添加的注释已经被撤销了。那如果说这个文件已经add到了暂存区，但是还没commit，这个时候你想撤销之前添加的注释，首先我们应该把暂存区的修改退回到工作区，然后在撤销刚刚对1.c的修改。具体怎么做，我们用命令 git reset HEAD 1.c 将1.c从暂存区中重新放回工作区，然后再git checkout -- 1.c撤销工作区中1.c的修改。 ![](/assets/images/aff330497bacdaf.png) 在图中我们看到git reset HEAD 1.c之后暂存区中的1.c已经重新放回了工作区，而工作区的1.c是改动过的。这时我们在对1.c进行git checkout -- 1.c操作，撤销对1.c的修改，我们看看结果。 ![](/assets/images/7f6563dd39ec7e0.png) 我们看到1.c中的注释已经成功撤销了。

### 删除文件

某一天，你手滑把1.c给删了，那git是知道你把文件删了的，此时工作区和版本库是不一致，我们使用git status命令，git会告诉你1.c被删除了 ![](/assets/images/a6a0ba72ae4863b.png) 这时你想恢复文件，因为在版本库中1.c文件还是存在的，你只需要执行命令

```shell
命令：git checkout -- 1.c
```

将版本库中版本恢复到工作区就可以了。是不是这样呢？我们执行一下命令，执行git checkout -- 1.c命令后 1.c文件确实恢复了。 ![](/assets/images/a8ea5564ea97dd6.png) 如果说你不是手滑删错了而是确实想把1.c文件删掉，那你在工作区删除1.c后你需要执行命令

```shell
命令：git rm 1.c
```

然后commit才能将1.c从版本库中删除。