# GIT 教程

author : Chopin

[TOC]

## 关于版本控制与GIT

1. 什么是版本控制 
2. 本地版本控制，集中式版本控制，分布式版本控制
3. git 特性
   - 版本快照
   - 几乎全本地操作
   - SHA-1 hash (40 个16进制字符) check-summed
   - 三种状态  modified, staged, committed

![1533177876634](C:\Users\LINXIA~1.MUC\AppData\Local\Temp\1533177876634.png)

4. 配置文件
   - system，global，local
   - `git config --list`

## GIT基本操作

### GIT仓库

   1. 建立本地仓库
   ```shell
   $ cd /home/user/my_project
   $ git init
   ```
   2. 拉取远程仓库

   ```shell
   $ git clone https://github.com/remote/repo newname
   ```
### 记录仓库变更

总的来说，git仓库里的文件包含两种主要的状态。tracked 和 untracked。untracked表示文件未加入git管理。untracked文件的所有变更，git都是无法管理的，一般是上一个版本不存在的文件，属于新增的文件。tarcked 文件来自上一个版本。在git的管理下又分为未修改，修改，暂存三种状态。![1533282953441](C:\Users\LINXIA~1.MUC\AppData\Local\Temp\1533282953441.png)

   1. 状态查看

      查看仓库中文件的状态可以使用命令 `git status`
   ```shell
   $ git status
   On branch master
   Your branch is up-to-date with 'origin/master'.
   nothing to commit, working directory clean
   ```

   也可以使用 `git status -s` 显示简易信息

   ```shell
   $ git status -s
    M README
   MM Rakefile
   A lib/git.rb
   M lib/simplegit.rb
   ?? LICENSE.txt
   ```

   - M 表示修改
   - MM 表示修改后，添加到暂存后，又再次修改
   - A 表示添加到暂存
   - ?? 表示untracked 文件

   如何理解暂存呢？可以认为，添加到暂存的文件，表示需要将该文件添加到下一个版本中。

   ### 忽略文件

   在软件开发中，有些文件不适合让git管理（比如一些编译文件，日志文件等），如何让git忽略这些文件呢？ 可以使用 .gitignore 文件。

   ```shell
   $ cat .gitignore
   *.[oa]
   *~
   ```

   ==.gitignore== 文件的语法：

   - (#) 开头的行为注释，忽略空行
   - (/) 结尾表示文件夹
   - (!) 表示排除匹配规则
   - 使用 glob patterns 匹配文件列表 (递归遍历整个仓库目录)
   - (/) 开头表示禁用递归匹配

   a/**/z 可以匹配多层嵌套文件夹 ，如： a/z, a/b/z, a/b/c/z, 等

例子中的  .gitignore 文件表示：忽略所有 .a,.o,~ 结尾的文件。[常用的例子]( https://github.com/github/gitignore)

.gitignore 文件管理的文件列表是  .gitignore 文件所在的根目录下的所有文件。一个git仓库不一定只有一个.gitignore 文件。可以有多个在不同目录。（linux 内核库有 206 个 .gitignore 文件）


### 查看暂存和非暂存的变更

```   shell
$ git diff
$ git diff --staged
$ git diff --cached #（--staged and --cached are synonyms）
$ git difftool
```

`git diff`    暂存区与工作目录相比。结果未提交到暂存区的修改。git add 后为 0

`git diff --staged`  上一版本与暂存区相比。比较结果是未提交到下一版本的修改。git commit 后为0

`git difftool` 可以选择第三方的比较工具。

### 提交修改

```shell
$ git commit -a -m "Story 182: Fix benchmarks for speed"
$ git commit
$ git commit -v
```

-m 自己编写提交信息

-a 暂存所有工作目录的修改 (相当于 git add . )

-v 自动提交并附带 git diff 结果

没有任何参数，git会生成一个默认的提交信息 

### 移动和删除文件

```shell
$ git rm PROJECTS.md
$ git rm -f PROJECTS.md
$ git rm --cached README
```

对于 未修改，或者未暂存的文件 直接删除，提交后生效。

对于已经修改，并且已经暂存的文件，需要加上 -f 参数。提交后生效



git 命令删除与操作系统 rm 命令 从本地删除的区别



## GIT 高级操作



### 快速定位提交

1. 单个定位

   ```shell
   # Short SHA-1
   $ git log --abbrev-commit --pretty=oneline #显示简洁的 SHA-1 ID
   $ git show 1c002d #查看具体的提交
   
   # 分支引用
   $ git show <branch-name> #查看分支的最后一个提交，分支名只是最后一个提交的引用。
   $ git rev-parse topic1
   ca82a6dff817ec66f44342007202690a93763949 #解析分支的名字，输出具体提交的ID
   
   # reflog 简洁id
   $ git reflog #git版本的 shell history 只在本地有效，本地命令的历史记录。
   $ git show HEAD@{5} #查看最近命令中的第五个命令
   
   # 查找父引用 '^'
   $ git show HEAD^ #HEAD 的前一个引用
   ```

   linux 内核上超过 700 000 的提交，约有 6.5 million的对象，只需要 SHA-1 的前11位就能保证 ID 唯一。

2. 范围定位

   ```shell
   # ..
   $ git log master..exp #列出 exp上不在 master 上的提交。 （exp - exp 与master 的交集）
   $ git log origin/master..HEAD # 列出当前分支中没有提交远程的 commit
   $ git log ^refA refB #等价 找出 ^ 后指定分支所没有的提交
   $ git log refB --not refA #等价 找出 -not 后指定分支所没有的提交
   $ git log refA refB --not refC # A,B中找出 C中没有的提交。
   
   # ...
   $ git log --left-right master...exp #找出两个分支差异的提交。(master - exp 与master 的交集)<
   < F
   < E #只存在master (master - exp 与master 的交集)<
   > D
   > C #只存在exp (exp - exp 与master 的交集)>
   ```

   











## 附录

### SHA-1 冲突的概率有多低

A lot of people become concerned at some point that they will, by random
happenstance, have two distinct objects in their repository that hash to the same
SHA-1 value. What then?
If you do happen to commit an object that hashes to the same SHA-1 value as a
previous different object in your repository, Git will see the previous object already
in your Git database, assume it was already written and simply reuse it. If you try
to check out that object again at some point, you’ll always get the data of the first
object.
However, you should be aware of how ridiculously unlikely this scenario is. The
SHA-1 digest is 20 bytes or 160 bits. The number of randomly hashed objects
needed to ensure a 50% probability of a single collision is about 2
80 (the formula
for determining collision probability is p = (n(n-1)/2) * (1/2^160)). 2
80 is 1.2 x 10 24
or 1 million billion billion. That’s 1,200 times the number of grains of sand on the
earth.
Here’s an example to give you an idea of what it would take to get a SHA-1
collision. If all 6.5 billion humans on Earth were programming, and every second,
each one was producing code that was the equivalent of the entire Linux kernel
history (6.5 million Git objects) and pushing it into one enormous Git repository, it
would take roughly 2 years until that repository contained enough objects to have
a 50% probability of a single SHA-1 object collision. Thus, a SHA-1 collision is less
likely than every member of your programming team being attacked and killed by
wolves in unrelated incidents on the same night. 













   

   

   