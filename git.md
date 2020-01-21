## 1. 版本号控制

<kbd>工作区</kbd> -git add--> <kbd>暂存区</kbd> -git commit--> <kbd>本地库</kbd> 

+ 团队协作

本地库 -- push --> <kbd>远程库</kbd>

远程库 -- clone --> 本地库

+ 跨团队协作

远程库 -- fork --> 自己的远程库 -- clone --> 本地库-- push -->自己远程库 -- pull request --> 审核 & merge --> 远程库

## 2. 简介

linux 中以.开始的是隐藏文件

**.git目录中存放的本地库相关文件**

设置签名：

**~/.gitconfig**

git config --global user.name yangzl

git config --global user.email yangzl@git.com

## 3. 命令行

git --help

+ 工作区 -- add --> 暂存区

+ 暂存区 -- git restore --staged files --> 工作区

+ 工作区 -- git restore files --> 放弃做的修改

#### 3.1 git查看版本

git log：less打开 b上一页，f下一页，space下一页，q退出

git log --pretty=oneline

git log --oneline

git reflog

![image-20200121115629960](F:\Program Files\Typora\workspace\image-20200121115629960.png)

#### 3.2 基于索引操作版本号

gir reset --hard commit hash

git reset --hard HEAD^^ 后退2步

git reset --hard HEAD~n 后退n步

参数说明：

<kbd>soft</kbd> 移动本地库HEAD指针

<kbd>mixed</kbd> 移动本地库HEAD指针，重置暂存区

<kbd>hard</kbd> 移动本地库HEAD指针，重置暂存区，工作区

#### 3.3 比较文件差异

git diff test.txt **工作区 diff 暂存区**

git diff HEAD test.txt **工作区 diff 本地库版本**

git diff --cached test.txt **暂存区 diff 本地库**

#### 3.4 分支操作

git branch -v

git branch hot_fix

**git checkout -b hot_fix** 创建并切换到hot_fix分支

**分支合并**

+ git checkout master(**将其它分支合并到master**)
+ git merge [**有新内容的分支**]

![image-20200121130458579](F:\Program Files\Typora\workspace\image-20200121130458579.png)

解决冲突：

**删除特殊字符，git add apple.txt** 

**git commit -m 'master & hot_fix both commit' 这里不能带文件名**

#### 3.5 远程操作

git remote -v

<kbd>git remote add origin address</kbd>

<kbd>git clone -b 5.1.x spring-framework ./spring-framework</kbd>= git init + git remote add origin ...

<kbd>git push -u origin master</kbd>

<kbd>git pull origin master</kbd> = git fetch + git merge

<kbd>git fetch origin master</kbd>

<kbd>git checkout origin/master</kbd> 切换到远程分支查看内容，<kbd>git checkout master</kbd> 切换到本地master

merge冲突解决同分支合并冲突解决

<kbd>git merge origin/master</kbd> 将远程库分支合并到本地master分支

## 4.图形化

.gitignore文件

~/.gitconfig文件中添加配置

[core]
	excludesfile = c:/Users/yangzl/Java.gitignore

## 5.git工作流

+ **集中式工作流** 同SVN，所有提交到master分支
+ **GitFlow工作流** 多分支

![image-20200121141353905](F:\Program Files\Typora\workspace\image-20200121141353905.png)

+ **Forking工作流** 跨团队协作



## GitHub

进大厂 TMD

``` javascript
// github搜索
zookeeper in:name,description,readme
springboot starts:>=5000
springcloud forks:>1000
springboot starts:>5000 forks :>1000
awesome redis 用于收集学习，工具，书籍
高亮显示 #L13-L50

// 带版本号fork
git clone -b 5.1.x git@github.com:spring-projects/spring-framework.git ./spring-framework

// 提交错误文件夹|文件
git rm -r --cached dirname|filename

// git pull & git fetch & git merge
pull == fetch + merge
git pull origin master == git fetch origin master & git merge
```

