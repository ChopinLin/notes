# 如何同步 fork 的代码

fork 源码作者仓库代码只会将当时的代码复制一份到自己的仓库下。当原作者更新远程仓库时，fork的代码不会同时更新。那如何将自己的代码和原仓库保持一致？

## Git 命令行手动同步

### 为本地代码添加远程库。指向源码作者的远程仓库

- 将 fork 远程代码 clone 到本地

- 给 fork 配置一个 remote，上游（本地跟踪的远程库）指向源码作者远程仓库

  `git remote add <remote name> <author branch url>`

  ![1535438854593](C:\Users\LINXIA~1.MUC\AppData\Local\Temp\1535438854593.png)

- `git remote -v` 查看是否配置成功

  ![1535438871959](C:\Users\LINXIA~1.MUC\AppData\Local\Temp\1535438871959.png)

- 拉取源码作者远程库代码

  `git fetch <remote name>` 

  ![1535438965032](C:\Users\LINXIA~1.MUC\AppData\Local\Temp\1535438965032.png)

### 合并代码，并推送到自己的远程库

- 切换到本地需推送的分支
  `git checkout <branch name>`

- 把 <remote name>/<branch name> 分支合并到本地分支上，这一步相当于同步代码。如果本地有修改，会合并为一个新提交。
  `git merge <remote name>/<branch name>`

- 最后直接 `git push origin <branch name> 直接推送即可。

脚本化

```shell
#!/bin/bash
function sync_fork() {
    orgin_git=""
    echo -n  "please enter the source repository url ->  "
    read  orgin_git
}

function sync_source() {
   current_branch=$(git rev-parse --abbrev-ref HEAD)
  
   git fetch upstream
   git checkout master
   git commit -m 'update'
   git merge upstream/master
   git push # origin
   git checkout $current_branch
}

my_remote_repository=$(git remote -v)
echo $my_remote_repository
if [[ $my_remote_repository =~ "upstream" ]]
then
   sync_source
else
   git remote add upstream $orgin_git
   sync_source
fi
}

sync_fork
```

参考

[Configuring a remote for a fork](https://help.github.com/articles/configuring-a-remote-for-a-fork/#platform-linux)

[Syncing a fork](https://help.github.com/articles/syncing-a-fork/)

[不用命令行的方法](https://www.zhihu.com/question/20393785/answer/30725725)