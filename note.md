git log -p -2 -stat

调整git log 格式

git log --pretty=oneline,short,full,fuller

--pretty=format:"%h - %an, %ar : %s"

--pretty=format:"%h %s" --graph

$ git log --oneline --decorate #查看各个分支所指向的提交



过滤git log 的输出

-n

--grep 查找提交message字符串

-S 在更改的代码里查找

--committer



所有提交了的信息都可以恢复，不论删除还是覆盖。但是未提交的就不行。

undoing change

```shell
$ git commit --amend  #重做上一次提交，推送可以加 -f

$ git reset HEAD <file>  #Unstaging file
$ git checkout -- <file> #丢弃更改，不可恢复。危险
```



远程仓库操作

```shell
$ git remote -v #查看远程仓库信息（注意与分区区分）

$ git remote add <shortname> <url> #添加远程仓库

$ git remote remove <shortname>

$ git fetch <remote> #拉取远程分支(仅下载)

$ git fetch origin #clone 命令拉取的远程仓库默认叫 origin

$ git pull  #拉取远程分支，并自动合并到本地当前分支上

#推送前，最好先fetch，看是否有新提交，合并到新提交上，再推送
$ git push origin master #推送分支到远程。

$ git remote rename pb paul #将远程仓库 pb 改名为 paul
```



获取远程仓库的详细信息

```shell
$ git remote show origin
```





git tag

有两种 lightweight and annotated.

lightweight tag 就像一个不可变的分支，只是指向某个提交的义个指针。轻量级。

annotated tag 存储了所有信息的git对象。就像一次提交

```shell
$ git tag -l "v1.8.5*" #模糊搜索

$ git tag -a v1.4 -m "my version 1.4" #创建一个tag，并添加tag name 和 tag message，-a 指定创建一个annotated tag。

$ git show v1.4 #显示 tag v1.4 的详细信息

$ git tag -a v1.2 9fceb02<checksum> #为某次提交指定tag
```

注意，tag默认只在本地有效，并不会推动到远程库，如果你想其他commiter 也看到这个tag，需要 `$ git push origin <tagName>`推送某个tag 或者 `$ git push origin --tags` 推送所有tag



切换到某个tag

`$ git checkout <tagName>` 可以切换至tag name的tag 上。浏览本次tag的文件。此时，仓库所在的状态会变成 detached HEAD 状态。在这个状态下的所有提交，都不会属于任何分支，并且只有当前这个tag指向的提交才能查看，其他提交无法获取tag上的修改信息。

如果想查看并修改某个tag的代码，用于bug修复。应该创建基于某个tag 的分支。`$ git checkout -b <branch> <tagName>`



branch

```shell
$ git branch testing #
$ git checkout testing #
```







开启binlogmy.cnf配置中设置：log_bin="存放binlog路径目录",binlog文件开启binlog后，会在数据目录（默认）生产host-bin.n（具体binlog信息）文件及host-bin.index索引文件（记录binlog文件列表）。当binlog日志写满(binlog大小max_binlog_size，默认1G),或者数据库重启才会生产新文件，但是也可通过手工进行切换让其重新生成新的文件（flush logs）；另外，如果正使用大的事务，由于一个事务不能横跨两个文件，因此也可能在binlog文件未满的情况下刷新文件

```mysql
mysql> show variables like; '%log_bin%'; #查看binlog配置
mysql> show binary logs; #查看binlog文件列表
mysql> show master status; #查看当前的binlog信息，写入的pos
mysql> reset master #清空binlog日志文件
```

如何查看binlog内容（二进制内容，不能直接查看）

1. mysqlbinlog 工具

   ```shell
   $ /usr/bin/mysqlbinlog  mysql-bin.000007 # mysqlbinlog是mysql官方提供的一个binlog查看工具，
   # 也可使用–read-from-remote-server从远程服务器读取二进制日志，
   # 还可使用--start-position --stop-position、--start-time= --stop-time精确解析binlog日志
   ```

2. mysql 命令行解析

   ```mysql
   # SHOW BINLOG EVENTS [IN 'log_name'] //要查询的binlog文件名 [FROM pos]  
   # [LIMIT [offset,] row_count]
   mysql> show binlog events in 'mysql-bin.000007' from 1190 limit 2
   ```



三种binlog格式

Row level:  

Statement level:  

Mixed level:  ``

binlog 两大作用 ：恢复和复制。

```
恢复是binlog的两大主要作用之一，接下来通过实例演示如何利用binlog恢复数据：
    
    a.首先，看下当前binlog位置
        mysql> show master status;
        +------------------+----------+--------------+------------------+-------------------+
        | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
        +------------------+----------+--------------+------------------+-------------------+
        | mysql-bin.000008 |     1847 |              |                  |                   |
        +------------------+----------+--------------+------------------+-------------------+
    b.向表tb_person中插入两条记录：
        insert into tb_person  set name="person_1", address="beijing", sex="man", other="test-1";
        insert into tb_person  set name="person_2", address="beijing", sex="man", other="test-2";
    c.记录当前binlog位置：
        mysql> show master status;
        +------------------+----------+--------------+------------------+-------------------+
        | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
        +------------------+----------+--------------+------------------+-------------------+
        | mysql-bin.000008 |     2585 |              |                  |                   |
        +------------------+----------+--------------+------------------+-------------------+
    d.查询数据 
        mysql> select *  from tb_person where name ="person_2" or name="person_1";
        +----+----------+---------+-----+--------+
        | id | name     | address | sex | other  |
        +----+----------+---------+-----+--------+
        |  6 | person_1 | beijing | man | test-1 |
        |  7 | person_2 | beijing | man | test-2 |
        +----+----------+---------+-----+--------+
    e.删除一条: delete from tb_person where name ="person_2";
        mysql> select *  from tb_person where name ="person_2" or name="person_1";
        +----+----------+---------+-----+--------+
        | id | name     | address | sex | other  |
        +----+----------+---------+-----+--------+
        |  6 | person_1 | beijing | man | test-1 |
        +----+----------+---------+-----+--------+
    f. binlog恢复（指定pos点恢复/部分恢复）
        mysqlbinlog   --start-position=1847  --stop-position=2585  mysql-bin.000008  > test.sql
        mysql> source /var/lib/mysql/3306/test.sql
    d.数据恢复完成 
        mysql> select *  from tb_person where name ="person_2" or name="person_1";
        +----+----------+---------+-----+--------+
        | id | name     | address | sex | other  |
        +----+----------+---------+-----+--------+
        |  6 | person_1 | beijing | man | test-1 |
        |  7 | person_2 | beijing | man | test-2 |
        +----+----------+---------+-----+--------+
    e.总结
        恢复，就是让mysql将保存在binlog日志中指定段落区间的sql语句逐个重新执行一次而已
```