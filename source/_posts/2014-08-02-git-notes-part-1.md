title: Git笔记(一)——[commit, checkout]
date: 2014-08-02 16:11:25
categories:
- 工具相关
tags:
- Git
- SourceTree
- 版本控制
- git-commit
- git-checkout
---

其实一直觉得自己是会用Git的，毕竟咱也是用Github的人啊！可是三月份找工作时候的一次面试颠覆了我的看法：

>Q: 用过Git吗？平常怎么用的？
>A: 用过的，一般就是add，commit，push嘛
>Q: branch用的多吗？git rebase这命令使用过吗？
>A: 一般都是自己的项目用的，就一个人，没涉及到这么复杂的使用
>Q: 那换个话题吧。。。

这么一搞才发现，其实我只会皮毛啊。。。正好现在新公司是用Git的，我也趁机恶补一下！本文其实就是对[图解Git](http://marklodato.github.io/visual-git-guide/index-zh-cn.html)这篇文章的一个实践。

<!--more-->

### 三个目录

谈到Git，最先需要明确的就是这三个概念：

* Working Directory：工作目录，这个可以简单的理解为你在文件系统里真实看到的文件
* Stage（Index）：暂存“目录”，用git add命令添加的文件就到了这里，即将被commit的文件
* Repository：项目“目录”，用git commit提交的文件就到了这里

后两个“目录”之所以加上引号，是因为其实它们并不是真实存在于文件系统中的目录，是个抽象的概念！为了方便后面的表述，我们就动手建立一个Git工程，并添加一个`test.txt`的文件。

``` bash
mkdir GitNotes
cd GitNotes
git init
touch test.txt
```

为了更直观的观看，后续将采用[SourceTree](http://www.sourcetreeapp.com/)这个软件来讲解。将这个目录添加到SourceTree中可以看到：

![图1](/assets/images/git-notes-1.png)

从图1中我们可以直观的看到三个目录，其中区域1即为Repository目录，默认显示的是所有commit的log记录，由于我们还没有进行commit，所以此时看到的就是ncommitted changes。区域2的标题为Staged files，就是Stage目录，目前我们没有git add，所以这里是空的。区域3就是Working目录，与文件系统对应的，我们新建的文件`test.txt`就在这里，可以看到前面有个问号，表示Not Tracked，就是说这个文件从来没有被add过。

有了这些基本概念，我们来做第一次commit吧，看看文件是怎么在这三个目录下转移的吧。运行`git add test.txt`，观察SourceTree的面板，发现文件`test.txt`已经来到了区域2，如图2。运行`git commit -m "add test file"`，文件已经到达了区域1。点击区域1的commit message，即可看到文件。标签master表示我们当前处于master分支，最新的commit为“add test file”，这个commit可以用cde6c09来唯一的标识。

![图2](/assets/images/git-notes-2.png)

### commit - 我会好几种姿势呢

前面的第一次提交用的是比较常规的姿势，其实提交代码还有好多种姿势哦！我们对test.txt文件做一下简单的修改，加一行“test 1”，然后运行`git commit -am "commit 1"`，通过加`-a`参数，和先运行`git add .`再commit的效果是一样的，也就是文件直接从Working目录到了Repo里。换个姿势再来一次，给文件加一行“test 2”，然后运行`git commit test.txt -m "commit 2"`，这次文件也是从Working直接到Repo去了。我们还可以再换个姿势，给文件再添加一行“test 2.5”，运行`git commit --amend -am "commit 2, 2.5"`，观察SourceTree，发现提交最后的一次提交记录被修改了，并且包含了最近的两次更改，如图3所示。

![图3](/assets/images/git-notes-3.png)

总结一下commit的几种姿势：
1. 传统姿势：先`git add file`再`git commit -m "xxx"`
2. 快速提交当前所有文件的更改：`git commit -am "xxx"`会先add所有的更改然后提交
3. 快速提交单个文件的更改：`git commit file -m "xxx"`只提交这个文件的更改
4. 修改最后一次提交：`git commit --amend -am "xxx"`将当前的更改加入最后一次commit中并更改最后一次commit的信息。其实观察可发现新的commit是替换了原先的commit，因为commit的hash已经变了。

关于commit还有几点想说的：
* `git commit -a`和`git commit file`这两个命令对Untracked的文件是无效了，也就是说只对add过的文件的更改才有效。比如我们新建一个文件`touch test1.txt`，然后运行`git commit -am "add test1"`或`git commit test1.txt -m "add test1"`都是无效的，如图4所示。

![图4](/assets/images/git-notes-4.png)

* 尽量不要使用`-m`标签，`-m`标签只适用于单行的提交信息，而提交信息最好越详细越好，方便别人，更方便自己。举个例子，给test.txt添加一行“test 3”，运行`git add test.txt`，再运行`git commit`，这时会打开Git中默认的编辑器（一般是vim），推荐像图5这样添加commit信息。其中第一行是简短的信息，第三行是详细的解释，标准就是第一行一目了然，第三行越详细越好。这样做的另一个好处是，Github默认是支持这种书写方式的，在Github的pull request里，默认显示第一行，第三行被折叠，非常方便。并且如果你的pull request只包含这一个commit的话，Github会默认将第一行作为标题，第三行作为内容，如图6。

![图5](/assets/images/git-notes-5.png)

![图6](/assets/images/git-notes-6.png)

* 尽量也不要使用`git add .`或`git commit -a`，这两条命令都会将当前所有的更改进行Stage或commit，这表面看来没什么大问题，其实是很危险的，有的时候会将未注意的更改错误的提交。Git的最佳实践还是强调小步提交，也就是说提交频繁一点，每次提交包含的更改少一点，这样不仅方便跟踪，更能避免多人合作时产生冲突。使用SourceTree这样的工具可以做到行级别的提交，也就是说一个文件我修改了好多行，可以把这些更改放到不同的commit里去。比如，修改test.txt添加两行“test 4”和“test 5”，然后打开SourceTree观察，如图7所示，点击一行后，右上角的“Stage lines”图标就会出现，这个图标的作用就是将这一行更改进行add。用这种方法，我们可以把这两行作两次提交。最终结果如图8所示。

![图7](/assets/images/git-notes-7.png)

![图8](/assets/images/git-notes-8.png)

### checkout - 上得了厅堂，下得了厨房

哟哟切克闹，煎饼果子来一套！说到了吃，git checkout可以说是身兼多职——上得了厅堂，下得了厨房。一个是分支相关的操作，另一个是可以恢复文件到之前的某个状态。

平常最常用的功能就是创建和切换分支了。运行`git checkout -b branch1`新建一个名为“branch1”的分支并切换过去。给test.txt再添加一行“test 5.5”，做一次commit，`git commit test.txt -m "commit 5.5"`。这时的状态如下图：

![图9](/assets/images/git-notes-9.png)

可以看到当前的分支上有个小对号，当前所处的commit前面的圆点是白色的。此时运行`git checkout master`就会切回master分支。“master”这个位置不仅可以放分支的名称，还可以放commit的hash。比如`git checkout cc59b55`（commit 2，2.5所对应的hash），这个时候Git会给出提示信息，如下图：

![图10](/assets/images/git-notes-10.png)

此时Git处在Detached HEAD状态，从SourceTree里也可以看出有一个HEAD的tag指向对应的commit。HEAD可以理解为时一个指针，指向当前所在的分支当前的commit。其实这个时候Git处在一个“游离的匿名分支”上，Git提示说你可以做修改，做提交，但一旦你checkout到别的地方，这些提交将无法再引用到，如果你想保存，必须在此基础上创建一个新的分支。什么意思呢？我们跟着提示一步一步做。首先给test.txt文件（此时文件只有三行，最后一行是test 2.5）再添加一行“test aaa”并提交`git commit test.txt -m "commit aaa"`。这时，看一下SourceTree的状态图：

![图11](/assets/images/git-notes-11.png)

按照Git给的提示，此时我们checkout到别的地方去：`git checkout master`切回master分支，再去SourceTree看一眼，我擦泪，刚才的修改丢了！！如下图：

![图12](/assets/images/git-notes-12.png)

不听“提示”言，吃亏在眼前，难道真的像提示说的那样，刚才的修改再也找不到了吗？非也，只要记得commit的hash，我们是可以切回去，从图11中找到commit aaa所对应的hash，运行`git checkout 6382c7d`，再看看SourceTree，OK，都回来了！“游离的匿名分支”的取名就来源于此，这些commit目前不属于任何分支，不能通过切分支的方式找到他们，只能记住hash才能切回来。为了保存这些提交，我们按照提示新建一个分支：`git checkout -b branch2`。这样以后就可以通过`git checkout branch2`切到该分支找到这些提交了。

除了分支相关的操作，checkout的另一个作用就是恢复文件了。既然说到恢复，那肯定有source（从哪里恢复）和target（恢复到哪里去），个人感觉一般checkout的target就是指你的工作目录，而source自然是其他两个目录了。也就是说，可以从暂存目录往工作目录里恢复文件，也可以从Repo里的各个commit记录里往工作目录恢复文件。为了方便讲解，我们先切回master分支：`git checkout master`，然后给test.txt加一行“test temp”，这个时候可以在SourceTree里看到存在Uncommited changes，运行`git checkout -- test.txt`，可以看到刚才的更改被取消了，也就是说我们从Repo的最新commit里将test.txt恢复到了我们的工作目录里。这里的`--`符号主要是为了避免歧义，其实这里我们不要`--`，直接运行`git checkout test.txt`也是可以的，但试想这么一种情况：我们的文件名称与分支名称一样，比如有一个叫做“master”的文件，如果不加`--`则Git会认为你想切换分支，所以必须使用`--`来告诉Git你想恢复文件而不是切换分支，多才多艺的人就是这样闹心啊！我们重复同样的操作，给test.txt添加一行“test temp”并做add操作：`git add test.txt`，这样文件就到了暂存目录。这时候运行`git checkout test.txt`是没有作用的，因为你的工作目录和暂存目录是一样，继续更改test.txt，添加一行"test temp1"并保存，这里运行`git checkout test.txt`，则会发现新加的这一行temp1没有了，也就是暂存目录被恢复到工作目录了。实践了“Repo最新commit => 工作目录”和“暂存目录 => 工作目录”，下面试试历史commit作为source吧。为了方便，将当前暂存区的更改（“test temp”）进行提交：`git commit -m "commit temp"`，此时的commit记录如下图：

![图13](/assets/images/git-notes-13.png)

这里我们试着将某个历史commit的test.txt恢复回来，比如将commit 3状态下对应的文件恢复，则找到对应的commit hash，运行`git checkout 9fc9896 test.txt`，会发现当前工作目录最后一行已经是“test 3”了，与commit 3时的文件状态一致，并且这个更改（最新的commit与commit “test 3”的差别）已经自动被add到暂存区域，如下图：

![图14](/assets/images/git-notes-14.png)

运行`git reset HEAD test.txt`（git add的反操作，后面会讲）和`git checkout test.txt`（最新的commit恢复到工作目录）来使test.txt恢复到“test temp”的状态。除了hash，还可以使用一些“快捷方式”来引用各个commit，比如刚才的操作用`git checkout HEAD~3 test.txt`也是一样的，其中`HEAD~3`表示比当前的commit早3个commit（爷爷的爸爸。。。）。说到快捷方式，其实分支名称也可以理解成一个快捷方式，代表所在分支的最新commit。比如运行`git checkout branch2 test.txt`，同样可以看到branch2的最新commit的状态被恢复到工作目录并添加到了暂存区。

总结一下checkout的几个功能：
1. 分支相关操作：`git checkout 分支名/commit hash`切换到相应的分支或commit，加上`-b`参数则会创建分支并切换过去
2. 恢复文件相关操作：`git checkout [分支名/commit hash/HEAD快捷方式] -- 文件名`恢复指定分支的最新commit或指定commit或快捷方式指向的commit的文件到工作目录，若省略中间的参数，则
    * 暂存区有内容且暂存区内容与工作目录不同，则恢复暂存区的状态到工作目录
    * 暂存区无内容，则恢复HEAD（最新的commit）的状态到工作目录

P.S. 本来准备把所有命令写在一篇里，不过感觉篇幅太长了也不好，这篇就到这里吧，剩下的下篇再写。
