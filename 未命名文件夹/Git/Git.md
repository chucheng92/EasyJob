## Git常用命令备忘

### 基本配置

```sh
git config --global user.name
git config --global user.email
```

### 创建版本库 mkdir -> pwd -> git init

添加，查看状态，比较git add -> git status -> git diff

提交 git commit -m 'description'

显示历史记录 git log 带参git log --pretty=oneline

### 版本回退 
git reset --hard HEAD~n
版本回退（指定的版本号） git reflog -> git reset --hard 版本号

### 工作区和暂存区

git add 把文件添加到暂存区
git commit把暂存区的所有内容提交到当前分支上

### Git撤销修改和删除文件操作

1.撤销修改

方法：
第一：如果我知道要删掉那些内容的话，直接手动更改去掉那些需要的文件，然后add添加到暂存区，最后commit掉。
第二：我可以按以前的方法直接恢复到上一个版本。使用 git reset  –hard HEAD^

更好的方法：
git checkout -- readme.txt丢弃工作区的修改（撤销）
命令 git checkout -– readme.txt 

意思就是，把readme.txt文件在工作区做的修改全部撤销，这里有2种情况，如下：

1.readme.txt自动修改后，还没有放到暂存区，使用撤销修改就回到和版本库一模一样的状态。
2.另外一种是readme.txt已经放入暂存区了，接着又作了修改，撤销修改就回到添加暂存区后的状态。

2.删除文件

rm b.txt 接下来：直接commit或者git checkout -- filename撤销

### 远程仓库

创建SSHKey ssh-keygen -t rsa –C "932191671@qq.com"

1.创建远程库 

```sh
git remote add origin https://github.com/lemongjing/testgit.git
```

把本地master分支的最新修改推送到

```sh
github上 git push -u origin master
```

2.远程库存在
git clone

### 创建与合并分支

每次提交，Git都把它们串成一条时间线，这条时间线就是一个分支。在Git里，这个分支叫主分支，即master分支。HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。 

创建

**git branch dev + git checkout dev = git checkout -b dev 创建并切换到dev**

合并

在master分支下执行git merge dev然后删除dev 

```sh
git branch -d dev
```

总结

```sh
查看分支：git branch

创建分支：git branch name

切换分支：git checkout name

创建+切换分支：git checkout –b name

合并某分支到当前分支：git merge name

删除分支：git branch –d name
```

1.如何解决冲突

Git用

```sh
<<<<<<<,=======,>>>>>>>
```

标记出不同分支的内容，其中<<<HEAD是指主分支修改的内容，>>>>>fenzhi是指fenzhi上修改的内容，我们可以修改下保存。

2.分支管理策略

通常合并分支时，git一般使用”Fast forward”模式，在这种模式下，删除分支后，会丢掉分支信息，现在我们来使用带参数 –no-ff来禁用”Fast forward”模式。首先我们来做demo演示下：

```sh
创建一个dev分支。
修改readme.txt内容。
添加到暂存区。
切换回主分支(master)。
合并dev分支，使用命令 git merge –no-ff  -m “注释” dev
查看历史记录
```

禁用fast-forward模式

```sh
git merge –-no-ff -m "注释" dev
```

查看分支日志

```sh
git log --graph --pretty=oneline --abbrev-commit
```

### 工作现场

暂存工作现场git stash

列出工作现场：git stash list

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，可以使用如下2个方法：

1.git stash apply恢复，恢复后，stash内容并不删除，你需要使用命令git stash drop来删除。

2.另一种方式是使用git stash pop,恢复的同时把stash内容也删除了。

### 多人协作

创建本地分支与远程分支的链接

git branch --set-upstream-to=origin/<branch> dev

例：

```sh
git branch --set-upstream dev origin/dev
```

提交
git push origin dev
拉取
git pull

### 本地远程分支相关

查看远程分支git branch -a

删除本地分支：git branch -d name

例，删除了本地dev

```sh
* master
  remotes/origin/dev
  remotes/origin/master
```

删除远程分支（两种方法）

git push --delete origin dev

git push origin :dev(冒号前面一个空格)