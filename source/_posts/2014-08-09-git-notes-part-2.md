title: Git笔记(二)——[diff, reset]
date: 2014-08-09 22:32:41
categories:
- 工具相关
tags:
- Git
- SourceTree
- 版本控制
- git-diff
- git-reset
---

书接[上回](/2014/08/02/git-notes-part-1/)，直入主题！如果你是接着上篇来的，那么先运行`git reset HEAD test.txt`和`git checkout test.txt`来放弃当前的更改，使最新的commit回到“commit temp”，这个时候运行`git status`，会看到“nothing to commit, working directory clean”。这里，“nothing to commit”说明暂存目录是空的，“working directory clean”说明你的工作目录也没有任何修改。

<!--more-->

回到这种状态是为了方便我们下面的讲解，此时的SourceTree状态为：

![图13](/assets/images/git-notes-13.png)

### diff - 来，叔叔给你检查身体

好好的一个`diff`命令能让我想到的就是这句猥琐的经典台词了，diff可以让你比较项目中任意两个状态的差别。说到比较，自然就又有source和target了，那么diff命令最直观的用法其实就是`git diff source target`。这里的source和target与checkout中的类似，可以是“commit的hash”，“分支名”，“快捷方式”。比如，我们想比较图13中前两个commit，运行`git diff ce81811 6382c7d`即可，得到的结果如下图：

![图15](/assets/images/git-notes-15.png)

可以看到，比较的结果其实是以target为基准的，也就是说target相比于source有了哪些变化，图15的结果中commit “6382c7d”比“ce81811”少了三行，多了一行，分别用减号和加号来表示。同样，使用`git diff master branch2`和`git diff HEAD branch2`得到的结果与上面是一致的，分支名和“HEAD”一类的都可以看做是commit的快捷方式。

这种source和target都给的情况是最容易理解的，复杂就复杂在如果我们省略一个参数会怎么样呢？比如运行`git diff branch2`结果如下图所示：

![图16](/assets/images/git-notes-16.png)

可以看到，图16的结果与图15是相反的。也就是说`git diff branch2`与`git diff branch2 HEAD`的结果是一样的，即如果只给一个参数，则这个参数为source，target默认为当前所在分支的最新的commit。现在就下这个结论对吗？注意我们现在处在“暂存目录为空”+“工作目录clean”的状况下，现在我们把工作目录搞成dirty试试，给test.txt再加一行“test 6”并保存。这时再试试`git diff branch2`，结果如图17所示：

![图17](/assets/images/git-notes-17.png)

可以看到新建的“test 6”也进去了。可以断定，在工作目录不clean的情况下，target默认表示的是工作目录。这个结论是否还是为时尚早呢，如果暂存目录有东西会怎么样？运行`git add test.txt`，然后继续`git diff branch2`，发现结果与图17是一致的，这还是不能说明问题，因为此时工作目录与暂存目录是一致的（都到test 6）。那么我们再加一行“test 7”，这时工作目录为“test 7”，暂存目录还是“test 6”，此时运行`git diff branch2`，发现“test 7”这一行也被加了进去。这个时候我们基本可以断定，target默认显示的确实是工作目录。

前面看了省略一个参数的情况，那俩参数都省略会咋样呢？运行`git diff`结果如下图：

![图18](/assets/images/git-notes-18.png)

可以看到比较结果为只增加了“test 7”，所以这个时候的source是暂存目录，而target还是工作目录。为了验证这个推测，我们运行`git add test.txt`，将“test 7”的修改也add到暂存目录，这时运行`git diff`，返回结果为空，因为此时暂存目录和工作目录是一致。我们做一次提交`git commit -m "commit 6,7"`，这时，暂存目录为空，工作目录clean，继续运行`git diff`，还是空的。到这里我们可以断定，如果两个参数都省略，那么默认source为暂存目录，默认target为工作目录。

前面的情况涉及到“各个commit之间的比较”，“各个commit与工作目录的比较”，“暂存目录与工作目录的比较”，那么只差一种情况了，我想比较“暂存目录”和“各个commit”怎么整呢？为了实现这个，我们先给暂存目录来点东西：加一行“test 8”并保存，然后`git add test.txt`，然后在编辑test.txt加一行“test 9”，这么做的原因是让暂存区有东西而且暂存区与工作目录不同。这时运行`git diff --cached branch2`，可以发现结果为下图：

![图19](/assets/images/git-notes-19.png)

从图中可以发现“test 8”在而“test 9”不在，说明此时的target已经变成暂存区了。

总结一下diff的各种情况：
1. `git diff source target`返回的结果是target相对于source的变化，这里的source和target可以是`commit的hash/分支名/快捷方式`
2. 如果只给一个参数，则这个参数就是source，而默认的target是工作目录，如果工作目录clean的话，则target为当前所在分支的最新commit
3. 如果一个参数都不给，默认的source是暂存目录，而target还是工作目录
4. 如果想要使暂存目录作为target的话，需要使用`--cached`参数

在继续往下走之前，先将刚才的更改全部提交，运行`git add test.txt`和`git commit -m "commit 8,9"`。

### reset - 有了我你随便咋折腾都行

版本控制最大的好处就是可以方便的找到以前的版本并恢复，所以从这个角度来说reset命令的地位还是比较重要的，可以让你无所顾忌的随便蹂躏整个项目。说到恢复，也有source和target的概念，这里的source肯定就是各个commit（包含分支名和快捷方式），而target根据不同的参数可能是暂存目录或工作目录或两者同时都是target。比如我们选定当前commit的父commit作为source，运行`git reset HEAD~ test.txt`，提示有Unstaged change，此时SourceTree里的Uncommitted changes的状态如下图：

![图20](/assets/images/git-notes-20.png)

从图20中可以发现，暂存区域的文件状态与父commit时一致，而改变是减掉了“test 8”和“test 9”两行，说明工作目录并没有发生变化（工作目录含有这两行）。可以看到这种情况下的target其实是暂存目录，它并没有改变工作目录。说到这里，把前面欠的课补上，还记得前面我们做git add的反操作时用了`git reset HEAD test.txt`，其实也是将HEAD状态的文件恢复到了暂存区，工作目录保持不变，而那时最新commit的文件状态和工作目录是一致的，所以最终产生的效果就是“git add反操作”。其实这里的`HEAD`也可以省略，因为默认的source就是当前所在分支的最新commit。更进一步，文件名`test.txt`也可以省略，默认会将Repo里的所有文件恢复，因为此时我们就只有这一个文件，所以效果是一样的。

再介绍reset的其他参数之前，我们想把刚才的reset再给reset掉，很简单，只要再运行一遍`git reset`即可，因为我们需要的其实是“git add的反操作”。然后加参数运行reset，`git reset --soft HEAD~`，注意这里我们并不是省略文件名，而是一旦加了`--soft`就不能跟文件路径，而是恢复整个项目的所有文件了，结果如下图所示：

![图21](/assets/images/git-notes-21.png)

可以看到，这次reset直接改变了HEAD，原先的“commit 8,9”消失了，最新的commit变成了原先的`HEAD~`，但这次reset仍然没有修改工作目录，只是将“commit 8,9”的文件状态add到了暂存区。既然有`--soft`参数，那肯定会有`--hard`参数，这次我们保持当前的状态，直接运行`git reset --hard ce81811`（commit temp所对应的hash），发现当前最新的commit变成“commit temp”，并且暂存区域是空的，然后工作目录也是clean的，说明`--hard`参数不管运行命令前处于什么状态，都直接将工作目录恢复到“commit temp”的状态，清空暂存区域。这也比较符合`--hard`这个单词强硬的意思。此时的SourceTree状态图为：

![图22](/assets/images/git-notes-22.png)

从图中可以看出，“commit temp”之前的commit都已经丢失了，整个项目被强制恢复到了“commit temp”所在的状态。

总结一下reset的用法：
1. `git reset [commit hash/分支名/快捷方式] [文件名]`类似“git add的反操作”，直接将所在commit的文件状态恢复到暂存区域。省略commit则默认为HEAD，省略文件名默认为所有文件。只改变暂存目录，不改变工作目录，当前commit不变。
2. `git reset --soft [commit hash/分支名/快捷方式]`软恢复，将恢复前所在commit的文件状态恢复到暂存区，当前最新commit为参数中的commit。只改变暂存目录，不改变工作目录，当前commit改变。
3. `git reset --hard [commit hash/分支名/快捷方式]`硬恢复，强制将整个项目恢复为参数中的commit时的文件状态，清空暂存目录，工作目录clean。暂存目录和工作目录同时被改变，当前commit改变。

关于reset命令的其他补充：当前HEAD已经位于“commit temp”，是不是前面的commit都找不回来了？当然不会，reset过的操作也是可以被reset的。有两种方法：
* 如果记得“commit 8,9”的hash（从图20中可以看到），则直接`git reset --hard 1a222c3`，则项目直接强制恢复到“commit 8,9”所在的状态。
* 如果不记得的话，运行`git reflog`，这个命令会输出一个列表，包含HEAD发生的所有变化。如下图：

![图23](/assets/images/git-notes-23.png)

在图23中可以发现“commit 8,9”所对应的条目为`1a222c3 HEAD@{9}: commit: commit 8,9`，第一项就是commit hash，第二项自然是快捷方式了。那么只要我们运行`git reset --hard HEAD@{9}`即可。注意，我的reflog输出结果可能与你的不同，因为写教程的需要我可能做了很多额外的操作。

P.S. 显然两篇也不够啊，发现主要是Retina屏的截图太尼玛大了。。。下篇再讲剩下的吧！
