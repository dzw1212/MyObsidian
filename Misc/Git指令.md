
# 基本操作

## 本地仓库

```c++
//设置个人信息
git config --global user.name "Your Name"
git config --global user.email "email@example.com"

git init //初始化当前目录为git仓库

git clone [repository address] //或直接克隆已有仓库

git add README.md //添加文件
git add *         //添加所有文件
git add -f README.md //或略gitignore强制添加文件

git rm README.md //从版本库中删除文件

git checkout -- README.md //撤销文件的本地工作区修改
git reset HEAD README.md //撤销文件的本地暂存区修改

git commit -m "commit readme file" //提交到本地仓库

//自动add被修改和被删除的文件，可省略git add操作，但不适用于新增文件
git commit -a

git status //查看当前仓库状态，包括文件修改、待提交等

git log //查看提交日志(仅限HEAD指针及其之前的版本信息，如果回退过版本则显示不全)
git log --pretty=online //简洁版
git log --graph //查看分支合并图
git reflog //查看所有历史版本信息(一般是为了版本回退后查询commit ID)

git reset --hard HEAD^ //回退到上一个版本，HEAD表示当前版本

git branch //查看所有本地分支
git branch -r //查看所有远程分支
git branch -a //查看所有本地分支和远程分支

git checkout [branch] //切换分支
git checkout -b [branch] //创建并切换分支
git switch [branch] //切换分支
git switch -b [branch] //创建并切换分支

git merge [branch] //合并指定分支到当前分支
git merge --no-ff -m [msg] [branch] //禁用Fast-Forward

git branch -d [branch] //删除分支
git branch -D [branch] //如果分支还未合并就要删除，则需要强制删除
```

## 远程仓库

```c++
//将本地仓库与远程仓库进行关联
git remote add origin git@github.com:xxxxx

//根据远程仓库克隆一个本地仓库
git clone git@github.com:xxxxx

git push origin master //将本地分支master推送到远程
git push -u origin master //推送，并关联本地master与远程master
6
git remote -v //查看远程库信息

git remote rm origin //解绑本地仓库与远程仓库

git checkout -b dev origin/dev //创建本地分支dev与远程分支dev
git branch --set-upstream-to=origin/dev dev //关联两个dev分支
git push origin dev //推送到远程dev分支

```

# 基本概念

## 分支

### Fast Forward模式

默认情况下，git会使用Fast-Forward模式进行分支合并；
```c++
git merge [branch]
//等同于
git merge -ff [branch]
```

在Fasr-Forward模式下，如果合并没有冲突，则会直接移动HEAD指针，从`git log`上看本次分支合并和普通的版本更新没有区别；
```cmd
$ git log --graph --pretty=oneline --abbrev-commit
*   e1e9c68 (HEAD -> master, ff) merge with ff
*   cf810e4 origin with ff
```

如果禁用Fast-Forward模式，git会在分支合并的时候生成一个新的commit
```cmd
$ git log --graph --pretty=oneline --abbrev-commit
*   e1e9c68 (HEAD -> master) merge with no-ff
|\  
| * f52c633 (dev) add merge
|/  
*   cf810e4 conflict fixed
```

修改默认分支合并模式：
```c
git config branch.master.mergeoptions  "--no-ff" //仅针对当前分支有效
git config --add merge.ff false //仅针对当前版本库的所有分支有效
git config --global --add merge.ff false //针对所有版本库的所有分支有效
```

### 分支暂存

如果当前处于分支branch1，想要切换到master分支处理文件，但是当前的修改还不能提交，此时分支无法直接切换，需要先将当前分支的修改内容暂存至git栈，暂存之后，使用`git status`会发现没有任何修改；
```c++
git stash //暂存
git stash save [msg] //暂存并添加注释

git stash list //查看所有暂存信息

git stash show //查看最近的一次stash与当前目录的差异

git stash apply stash@{$num} //恢复至某次statsh，不删除任何stash

git stash drop stash@{$num} //删除指定的stash

git stash pop //恢复至最近的一次stash并删除(建议仅在只有一次stash时使用)

git stash clear //移除所有的stash
```

### cherry-pick

将其他分支的某次修改同步到当前分支；
```c++
git cherry-pick [commitId]
```

## 标签

标签与commit ID相对应，相当于给commit ID一个更具可读性的代称；
```c++
git tag [name] //给当前分支打上标签
git tag -a [name] -m [msg] //带有msg的标签
git tag [name] f52c633 //给某个commit ID打上标签

git tag //查看所有标签
git show [name] //查看某个标签的详细信息

git tag -d [name] //删除某个标签

git push origin [name] //推送某个标签到远程
git push origin --tags //推送所有本地标签到远程

//删除远程标签
git tag -d [name] //先删除本地
git push origin :refs/tags/[name] //再推送到远程
```

## gitignore

一个常用`.gitignore`文件的集合；
*https://github.com/github/gitignore

自动生成`.gitignore`文件的在线工具：
*https://gitignore.itranswarp.com/

语法示例：
```c++
.* //忽略所有以.开头的文件
!.gitignore //除了.gitignore
```

```c++
git check-ignore -v [filename] //查看是哪个规则导致文件的忽略
```


# 附录

## 专有名词解释

`origin`：远程仓库链接的默认名称，也是`.git/refs/remotes/`下的一个文件夹名称；也可以自定义远程仓库链接的名称，使用命令：
```c++
git remote add [xxx] [git@github.com:xxxxx]
```

`HEAD`：对某个commit ID的引用，一般表示最新的版本；
	当使用`git commit`时，HEAD会移动并指向最新的commit ID；
	当使用`git checkout`切换分支时，HEAD会移动并指向对应分支的最新commit ID；
	当使用`git reset`回退版本时，HEAD会移动到对应的commit ID；


## 查看所有配置

```c++
git config --list
```

或者直接打开`.gitconfig`文件查看；

## git diff调用BeyondCompare

方法1、通过指令配置；
```c++
git config --global diff.tool bc4
git config --global difftool.bc4.path "C:\\Program Files\\Beyond Compare 4\\BCompare.exe"
```


修改`.gitconfig`文件，添加以下内容：
```c++
[diff]
	tool = bc4
[difftool "bc4"]
	path = "C:\\Program Files\\Beyond Compare 4\\BCompare.exe"
```