# git分支部分回滚方案

前提：
需求的所有提交备注的前缀最好是需求的标题，这样更容易区分需求的提交。否则需要根据图形线路找对应分支的提交。

案例：
有3个需求：需求1、需求2、需求3，分支分别为：feature1、feature2、feature3。
按顺序依次合并到qa1分支，现在要回滚需求2的代码。

步骤：
1、查看qa1分支图形化提交记录
```bash
git log qa1 --graph --oneline --decorate=short
```
如下：
```
*   9cde889 (HEAD -> qa1) Merge branch 'feature3' into qa1
|\  
| * bb5439a (origin/feature3, feature3) 需求3---2
| * b486794 需求3---1
* |   a72bf8b Merge branch 'feature2' into qa1
|\ \  
| * | df7fb59 (origin/feature2, feature2) 需求2---2
| * | 808553b 需求2---1
| |/  
* | 6de1d9f (origin/feature1, feature1) 需求1---2
* | 8d27596 需求1---1
|/  
* 9127461 (origin/master, master) 开发环境配置文件
* 370e145 上传回调
* 1c245bf 部分接口-删除文件
* 7787dae 部分接口
* 7a43cf3 初始化
```

2、根据备注找到需求2的最后一个commitId，然后执行如下命令
```bash
git reset --mixed a72bf8b
```

3、目前需求3的修改已经变成未提交状态，执行如下命令暂存代码
```bash
git stash save
```

4、根据备注找到需求1的最后一个commitId，然后执行如下命令
```bash
git reset --hard 6de1d9f
```

5、目前需求2的代码已经被清除，然后执行如下命令，把需求3的代码恢复
```bash
git stash pop
git commit -am '需求3'
```
再查看日志，看看效果，可以看到需求2的代码已经回滚了，需求3的代码变成了qa1分支的提交。
```
* eb5a69b (HEAD -> qa1) 需求3
* 6de1d9f (origin/feature1, feature1) 需求1---2
* 8d27596 需求1---1
* 9127461 (origin/master, master) 开发环境配置文件
* 370e145 上传回调
* 1c245bf 部分接口-删除文件
* 7787dae 部分接口
* 7a43cf3 初始化
```

6、然后要取消qa1分支的保护，执行如下命令强行推送到远程分支，推送完成之后再加上qa1分支的保护。
```bash
git push origin qa1 --force
```
