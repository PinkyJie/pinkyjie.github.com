title: Git笔记(三)——[cherry-pick, merge, rebase]
date: 2014-08-10 19:37:24
categories:
- 工具相关
tags:
- Git
- SourceTree
- 版本控制
- git-cherry-pick
- git-merge
- git-rebase
---

书接[上回](/2014/08/09/git-notes-part-2/)，直入主题！这篇继续实践剩下的几个命令。

<!--more-->

现在的SourceTree状态如下：

![图24](/assets/images/git-notes-24.png)

### cherry-pick - 妈妈，我也要

cherry-pick其实在工作中还挺常用的，一种常见的场景就是，比如我在A分支做了几次commit以后，发现其实我并不应该在A分支上工作，应该在B分支上工作，这时就需要将这些commit从A分支复制到B分支去了，这时候就需要cherry-pick命令了，B分支指着这些commit说：妈妈，我也要！比如说，我们在master分支上继续做两次提交，第一次添加一行"test 10"，`git commit -am "commit 10"`，第二次添加“test 11”，到达如下图的状态：

![图25](/assets/images/git-notes-25.png)

这个时候我们发现，哦NO，我们不应该直接更改master分支，我们应该在自己的分支上做提交。这个时候先新建一个分支`git checkout -b branch3 1a222c3`，注意这里的最后一个参数是新分支的起点，也就是说，新的分支branch3是从“commit 8,9”开始的，现在我们需要把刚才的两次提交移动到新的分支上。运行`git cherry-pick 0bda20e 1a04d5f`，命令行会给出提示两个commit被复制到了当前分支上，此时SourceTree的状态如下图：

![图26](/assets/images/git-notes-26.png)

确定这两个commit被复制到指定分支以后，在master分支上将这两个commit删除。先切回master分支：`git checkout master`，运行`git reset --hard 1a222c3`，此时SourceTree的状态图为：

![图27](/assets/images/git-notes-27.png)

两个commit被成功的从master分支移动到了branch3分支。

### merge - 求合体

merge命令应该也是非常的常用，比如新开了一个分支去完成某个feature，然后完成了以后要merge到master分支来。就拿刚才的例子来讲，开了新分支branch3来存放“commit 10”和“commit 11”，这两个commit可以看成是新的feature，完事以后就要合并到master分支上了。但合并之前，一般要将master分支当前最新的commit合并到branch3上，因为你的branch3的起点此时可能已经不是master分支的最新commit了。切换到branch3分支上运行`git merge master`，Git提示“Already up-to-date.”，这说明当前所在的分支branch3比master分支还要新，branch3上的commit都是master最新commit点的子commit，故不需要合并。切回master分支`git checkout master`，将分支branch3合并到master分支上，`git merge branch3`，结果如下图：

![图28](/assets/images/git-notes-28.png)

可以看到branch3分支上的更改已经被合并到master上，其中“Fast-forward”是合并的一种类型，当当前分支（master）是目标分支（branch3）的祖先commit时会发生这种“Fast-forward”合并，其实可以理解为HEAD指针指向的快速移动。

前面看到的两种情况都是比较简单的合并，没有遇到任何“冲突”，我们来一次稍微复杂点的合并，比如我们需要将branch2合并到master上。在master分支上`git merge branch2`，哦NO，命令行提示“Automatic merge failed”，出现冲突了，Git无法判断如何merge，这个时候我们就需要手动解决冲突以后再提交。打开test.txt文件可以看到如下图的内容：

![图29](/assets/images/git-notes-29.png)

从图28中可以看到`<<<<<<<< HEAD`和`========`之间的内容是当前分支的内容，而`=======`和`>>>>>>>>> branch2`之间的内容是branch2分支的内容，由于改动了同一行所以Git无法自动合并了。这个时候你可以决定最后保留哪些内容，我们就简单的将“test aaa”插入到第4行，然后删除多余的标记保存，另外，需要手动做一次提交来解决冲突：`git add test.txt`，然后这次我们不用`-m`参数直接`git commit`，可以看到如下图的提示：

![图30](/assets/images/git-notes-30.png)

Git已经发现我们是在做一次merge的提交，并且是“解决冲突的merge”，可以看到Git默认给出的提交信息非常的友好，包含了合并的分支上的commit，也写明了冲突的文件是什么，我们就使用Git默认的提交信息，直接`:wq`保存退出。此时可以直观的从SourceTree里看到branch2分支的线已经合并到master上了，如下图所示：

![图31](/assets/images/git-notes-31.png)

总结一下merge的集中情况：
1. `git merge 目标分支`
    * 如果目标分支是当前分支的祖先commit节点，则merge什么也不会发生，因为当前分支已经是最新的了
    * 如果当前分支是目标分支的祖先commit节点，这时会发生Fast-forward的merge，merge的结果是简单的移动HEAD指针
2. 如果以上两种情况都不是的话，则其实是做的三方合并，除了这两个分支的最新commit以外，另外一个是这两个分支的共同祖先commit点。这种情况下如果没有冲突的话会自动生成一个merge的commit，如果有冲突则手动解决后还是会有一个merge的commit。

### rebase - 我是直男，不喜欢弯的

从图31可以看出，branch2分支（紫色的线）最终交汇到master分支上（蓝色的线），这还只是合并了一次，并且我们当前的分支才几个，如果分支很多并且频繁合并的话，这样弯弯曲曲的线会非常多，搞得你眼花缭乱，根本搞不清楚走向，显然，直男一向是不喜欢弯的，那能把它掰直吗？答案是肯定的，rebase就是干这个事情的！为了看清楚merge和rebase的区别，我们先将刚才branch2的合并取消，使master回滚到“commit 11”的状态：`git reset --hard dda0f7d`。这时我们位于master分支上，运行`git rebase branch2`，Git同样提示我们有冲突，只是这次提示非常长，如下图：

![图32](/assets/images/git-notes-32.png)

根据提示我们就可以发现rebase的流程，从“Applying: commit 3”这句就可以看出来，其实rebase的原理是先找到两个分支的共同祖先commit节点“commit 2,2.5”，然后把master分支上这个节点的儿子节点全部“应用”到branch2分支上。从图31中可以看到第一个儿子节点是“commit 3”，所以先apply这个，而“commit 3”和“commit aaa”编辑的都是第4行，所以立即出现了冲突！照例我们需要手动解决冲突。这次如果我们还是把“test aaa”放第4行，然后“test 3”放在第5行，那很明显后面的“commit 4”在apply时仍然会有冲突，所以为了方便，我们直接把“test 3”和“test aaa”都放在第4行，如下图：

![图33](/assets/images/git-notes-33.png)

然后保存，注意这时需要先把改动add以后再操作：`git add test.txt`。按照图32的提示运行`git rebase --continue`，结果怎么样呢？哦NO，又尼玛冲突了！！！为嘛啊！“commit 4”的第4行还是“test 3”，而我们现在的第4行是“test 3 test aaa”，所以还是冲突！看来rebase在apply每一个commit时并不只是apply这个commit上的变化（即“test 4”这一行），而是apply整个文件！可以想象这样走下去每一步都会有冲突，唯一的办法就是放弃“test aaa”这个更改。真的是这样吗？其实不然，先运行`git rebase --abort`放弃这次rebase。其实这种场景下并不适合rebase，一般我们把别的分支合并到master时用`merge`，而把master合并到别的分支时会用到`rebase`，那我们换个思路，切换到branch2分支上`git checkout branch2`，然后运行rebase命令`git rebase master`，显然，还是会有冲突的，这次我们直接把“commit aaa”加到最后一行保存。运行`git add test.txt`，然后继续进行`git rebase --continue`，可以看到结果成功了“Applying: commit aaa”。这时的SourceTree如下图：

![图34](/assets/images/git-notes-34.png)

可是这样与merge的结果恰恰相反，我们是把master分支merge到branch2上了，而我们初衷是将branch2分支merge到master上。简单！先切回master分支`git checkout master`然后运行`git merge branch2`即可，因为这时master分支是branch2分支的祖先commit节点，所以直接Fast-forward了！最终的状态图如下图所示：

![图35](/assets/images/git-notes-35.png)

从图35中可以看出，达到了和图31同样的效果，不同的是：首先，没有了“弯弯”的线；其次，多余的merge那一条commit没有啦！这才是直男喜欢的！

总结一下rebase：
1. `git rebase 目标分支`原理其实是先将HEAD指向目标分支和当前分支的共同祖先commit节点，然后将当前分支上的commit一个一个的apply到目标分支上，apply完以后再将HEAD指向当前分支。
2. rebase与merge的区别：
    * 把master分支合并到别的分支用rebase，把别的分支合并到master分支上用merge
    * rebase不会产生多余的commit，并且保持直线

_关于rebase其他要补充的_：

当然rebase还有其他很多很牛逼的功能，其“交互模式”可以让你干很多事情，比如调整commit的顺序啊，合并一些commit啊，删除一些commit啊等等，通过`-i`参数可以实现，当然这个命令有些复杂，我们可以使用SourceTree的图形化界面更直观的使用它。比如“commit 10”和“commit 11”太不和谐了，早就看你们不爽了，人家前面的“6,7”和“8,9”都成双成对，就你俩不和谐，我要“整顿”一下！在SourceTree中的“commit 8,9”上右击，点击子菜单中的“Rebase children of 1a222c3 interactively...”：

![图36](/assets/images/git-notes-36.png)

![图37](/assets/images/git-notes-37.png)

然后出现了图37中的对话框，我们想整理的2个commit显示在其中，“commit aaa”也是儿子节点，所以也显示在这里，我们可以通过红色方框中的按钮来进行操作。比如，我们想合并一下“commit 10”和“commit 11”，两个commit合并为“commit 10,11”。在这个对话框中可以非常直观的进行上面的操作，先选中“commit 11”这一行，选择红框中的“Squash with previous”按钮，可以看到出现了一条新的commit——“[2 commits]”，如图38所示：

![图38](/assets/images/git-notes-38.png)

选中这条commit，点击红框中的“Edit message”，出现了更改commit信息的对话框，如下图：

![图39](/assets/images/git-notes-39.png)

输入“commit 10,11”即可，点击OK保存，再次点击OK完成此次rebase。此时SourceTree的状态如图40所示：

![图40](/assets/images/git-notes-40.png)

可以看到，目的达到了，commit hash也跟原来的任何两个都不一样了。除了合并commit，利用图37的红色方框里的按钮，还可以实现删除commit，调整commit的顺序等功能，大家自己尝试吧，小心产生“冲突”哦！

好啦，到这里，感觉比较重要的git命令都介绍完了！！！其实这三篇都是小实践，要想融会贯通，还需要在真实项目中的大实践！



