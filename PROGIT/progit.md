- [1. 起步](#1-起步)
	- [1.1 版本控制](#11-版本控制)
	- [1.3 Git](#13-git)
	- [1.6 初次运行Git前的配置](#16-初次运行git前的配置)
- [2. Git基础](#2-git基础)
	- [2.1 获取Git仓库](#21-获取git仓库)
	- [2.2 Git文件生命周期及操作](#22-git文件生命周期及操作)
	- [2.3 查看提交历史](#23-查看提交历史)
	- [2.4 撤销操作](#24-撤销操作)
	- [2.5 远程仓库的使用](#25-远程仓库的使用)
	- [2.6 打标签](#26-打标签)
- [3. Git分支](#3-git分支)
	- [3.1 分支简介](#31-分支简介)
	- [3.2 分支的新建与合并](#32-分支的新建与合并)
	- [3.3 分支管理](#33-分支管理)
	- [3.4 分支开发工作流](#34-分支开发工作流)
	- [3.5 远程分支](#35-远程分支)
	- [3.6 变基](#36-变基)
- [4. 服务器上的Git](#4-服务器上的git)
- [5. 分布式Git](#5-分布式git)
- [6. GitHub](#6-github)
	- [6.2 对项目做出贡献](#62-对项目做出贡献)
	- [6.3 维护项目](#63-维护项目)
	- [6.4 管理组织](#64-管理组织)
	- [6.5 脚本GitHub](#65-脚本github)
- [7](#7)

# 1. 起步
### 1.1 版本控制
版本控制系统（VCS）可以将某个文件回溯到之前的状态，可以比较文件的变化细节，查出最后是谁修改了哪个地方，从而找出导致怪异问题出现的原因，又是谁在何时报告了某个功能缺陷等等

### 1.3 Git

- 直接记录快照，而非差异比较
	- 其它版本管理系统：存储每次更改的差异信息![](https://github.com/limbo-note/books/blob/master/PROGIT/1-1.jpg)
	- git：每次提交，都对当时的全部文件制作一个快照并保存这个快照的索引。如果文件没有修改，Git 不再重新存储该文件，而是只保留一个链接指向之前存储的文件![](https://github.com/limbo-note/books/blob/master/PROGIT/1-2.jpg)
- 近乎所有操作都是本地执行
	- 本地有项目的完整历史，速度快，可离线工作
- 保证完整性
	- 所有数据在存储前都计算校验和（SHA-1散列），然后以校验和来引用。所以，不可能在Git不知情时更改任何文件内容或目录内容 
	- Git数据库中保存的信息都是以文件内容的哈希值（校验和）来索引，而不是文件名
- 三种状态：
	- 已提交commited（已保存在数据库中）
	- 已修改modified（已修改，但还没保存到数据库）
	- 已暂存staged（已对修改的文件做了版本标记）
- 三个工作区域：
	- Git仓库（保存项目的元数据和对象数据）
	- 工作目录（对某个版本独立提取出的内容，放在磁盘供用户修改和使用）
	- 暂存区域 （也叫索引，是一个文件，保存了下次将提交的文件列表信息）
- 基本工作流程：
	- 在工作目录中修改文件
	- 暂存文件，将文件快照放入暂存区域(`git add`)
	- 提交，永久保存到Git仓库(`git commit`)

### 1.6 初次运行Git前的配置

- 三个级别的配置文件
	1. /etc/gitconfig：系统上每个用户的通用配置（`git config --system`）
	2. ~/.gitconfig或 ~/.config/git/config：只针对当前用户(`git config --global`)
	3. 当前仓库的Git目录中的config文件,.git/config：只针对该仓库(`git config`)
	- 每个级别都覆盖上一个级别的配置
- 用户信息
	- `git config --global user.name/user.email`
- 文本编辑器 
	- `git config --global core.editor vim/emacs`
- 查看配置
	- `git config --list`
	- 可能会看到重复的变量名，因为会从多个级别配置文件读，以最后一个为准
	- `git config <key>`查看某一项配置

# 2. Git基础

### 2.1 获取Git仓库

- 创建仓库，初始化:`git init`
- 克隆已有仓库:`git clone`（支持https协议/git协议/ssh协议）
	- 在克隆远程仓库的时候，自定义本地仓库的名字:`git clone https://github.com/xxx/xxx mylibgit`

### 2.2 Git文件生命周期及操作

文件分为已跟踪/未跟踪，已跟踪又分为未修改/已修改/放入暂存区。未跟踪文件，它们既不存在于上次快照的记录中，也没有放入暂存区。 初次克隆某个仓库的时候，工作目录中的所有文件都属于已跟踪文件，并处于未修改状态。

生命周期变化：![](https://github.com/limbo-note/books/blob/master/PROGIT/2-1.jpg)

- 查看文件的状态（会显示未跟踪的文件和修改未提交的文件）：`git status`
- 跟踪新文件和暂存已修改的文件（git commit提交的是最后一次git add的那个版本）：`git add`
- 状态简览：`git status -s`
	- `??`: 未跟踪
	- `A`: 新添加到暂存区的文件，还没被修改
	- `M`: 在左边表示被修改并放入了暂存区，在右边表示被修改但是没放入暂存区
- 忽略文件(.gitignore)
	- 所有空行或者以 # 开头的行都会被忽略
	- 使用标准的 glob 模式匹配(shell使用的简化的正则表达式)
		- 星号（ * ）匹配零个或多个任意字符
		- [abc] 匹配任何一个列在方括号中的字符
		- 问号（ ? ）只匹配一个任意字符
		- 短划线,两个字符范围内的都可以匹配
		- 两个星号（ * ）表示匹配任意中间目录
	- 可以以（ / ）开头防止递归
	- 可以以（ / ）结尾指定目录
	- 惊叹号（ ! ）取反
	- 常用的标准gitignore文件[https://github.com/github/gitignore](https://github.com/github/gitignore )
- 查看文件的修改
	- `git diff`：修改了但是没暂存的内容（即当前内容和暂存区快照的差异）
	- `git diff --cached/--staged`: 修改了而且暂存了的内容（即暂存区快照和上一次提交的差异）
- 提交更新
	- 提交暂存区里的内容到Git仓库: `git commit [-m]`
- 跳过使用暂存区
	- `git commit -a [-m]`: 和先add再commit有些不同，不会把新文件加入暂存区
- 移除文件
	- `git rm`: 取消跟踪文件，同时删除磁盘中的文件
	- `git rm --cached`: 取消跟踪，但是不删除磁盘中的文件
- 移动文件
	- `git mv`

### 2.3 查看提交历史

- 查看提交历史: `git log`![](https://github.com/limbo-note/books/blob/master/PROGIT/2-2.jpg)
- 限制输出![](https://github.com/limbo-note/books/blob/master/PROGIT/2-3.jpg)

### 2.4 撤销操作

- 补救上一次的提交:`git commit --amend`，可修改提交信息和补上忘记放入暂存区的文件
- 取消暂存的文件: `git reset HEAD [file]`
- 撤销对文件的修改: `git checkout -- [file]`，把文件还原成上次提交后未修改的样子（**危险命令**，会覆盖当前文件数据）

### 2.5 远程仓库的使用

- 查看远程仓库: `git remote [-v]`, (push)代表有推送权限
- 添加远程仓库: `git remote add <shortname> <url>`, shortname可以给自己定义远程仓库名(clone的仓库默认名为origin)
- 抓取与拉取
	- `git fetch [remote-name]`: 拉取所有你还没有的数据, 将会拥有那个远程仓库中**所有分支**的引用，可以随时合并或查看
	- `git pull`: 自动抓取然后合并远程分支到当前分支(默认本地master分支跟踪了远程的master分支)
- 推送到远程
	-  `git push [remote-name] [branch-name] `: **只有当你有所克隆服务器的写入权限，并且之前没有人推送过时，这条命令才能生效**。 如果有别人已经推送过，**必须先将他们的工作拉取下来并将其合并进你的工作后才能推送**
- 查看远程仓库： `git remote show [remote-name] `， 关于本地和远程仓库的一些关联信息，很有用
- 远程仓库的移除与重命名: `git remote rename <oldname> <newname>`, `git remote rm <remote-name>`

### 2.6 打标签

- 查看标签`git tag [-l] [pattern]`
- 创建标签
	- 附注标签`git tag -a [tagname]`，查看标签信息`git show [tagname]`
	- 轻量标签`git tag [tagname]`，不加其它选项，`git show`时没有额外信息
	- 在tagname后加上指定提交的校验和，可给历史提交打标签
- 共享标签（把标签推到远程仓库）
- 检出标签


# 3. Git分支
### 3.1 分支简介

使用 git commit 进行提交操作时，先计算每个子目录的校验和，然后在 Git 仓库中这些校验和保存为树对象。；随后，会创建一个提交对象，包含指向这个树对象（项目根目录）的指针、提交信息等；每个文件都会有一个文件快照，保存在blob对象中

三个文件首次提交，对象结构如下：
![](https://github.com/limbo-note/books/blob/master/PROGIT/3-1.jpg)

每次提交的提交对象会指向上次的提交对象：
![](https://github.com/limbo-note/books/blob/master/PROGIT/3-2.jpg)
分支其实就是指向提交对象的指针。如图，分支(如master)就是一个指向提交对象的指针![](https://github.com/limbo-note/books/blob/master/PROGIT/3-3.jpg)


### 3.2 分支的新建与合并

- 分支创建
	- `git branch testing`会在当前提交对象上创建一个testing分支指针；而HEAD指针表示当前所在的分支，创建新分支并不会改变HEAD指针
	![](https://github.com/limbo-note/books/blob/master/PROGIT/3-4.jpg)
	- `git log --decorate` 命令查看各个分支当前所指的对象

- 分支切换
	- `git checkout`其实就是改变HEAD指针的指向，分支切换并修改时对象结构变化如下：
		1. `git checkout testing`![](https://github.com/limbo-note/books/blob/master/PROGIT/3-5.jpg)
		2. `git commit -a -m 'made a change'`![](https://github.com/limbo-note/books/blob/master/PROGIT/3-6.jpg)
		3. `git checkout master`切换到一个较旧的分支，工作目录会恢复到该分支最后一次提交时的样子![](https://github.com/limbo-note/books/blob/master/PROGIT/3-7.jpg)
		4. `git commit -a -m 'made other changes'`在旧分支上再做修改时![](https://github.com/limbo-note/books/blob/master/PROGIT/3-8.jpg)

- 一个实际使用分支的案例：
	1. 开发某个网站
	2. 为实现某个新的需求，创建一个分支
	3. 在这个分支上开展工作
	4. 正在此时，你突然接到一个电话说有个很严重的问题需要紧急修补
	5. 切换到你的线上分支（production branch）
	6. 为这个紧急任务新建一个分支，并在其中修复它
	7. 在测试通过之后，切换回线上分支，然后合并这个修补分支，最后将改动推送到线上分支
	8. 切换回你最初工作的分支上，继续工作


- 分支合并
	- `git merge <branchname>`使当前所在的分支和指定的分支合并
	- 在合并的时候，可能有"快进（fast-forward）"这个词。 表示当前的分支所指向的提交是你指定想合并的分支的直接上游，所以Git只是简单的将指针向前移动，即fast-forward
	- 当前分支与指定合并分支不属于直接上游关系时，就不能只是指针移动了。如图，master分支和iss53分支需要合并，Git 会使用两个分支的末端所指的快照（ C4 和 C5 ）以及这两个分支的工作祖先（ C2 ），做一个简单的三方合并![](https://github.com/limbo-note/books/blob/master/PROGIT/3-9.jpg) Git将此次三方合并的结果做了一个新的快照并且自动创建一个新的提交指向它。 这个被称作一次合并提交，它的特别之处在于他有不止一个父提交![](https://github.com/limbo-note/books/blob/master/PROGIT/3-10.jpg)
	- 合并之后，两个分支指向同一个对象，`git branch -d`可删除多余的分支
- 遇到冲突的分支合并
	- 两个分支对同一个文件的同一个部分进行了不同的修改，会产生合并冲突，Git会合并，但是没有创建新的合并提交
	- `git status`查看因冲突而未合并的文件
	- 为了解决冲突，你必须选择使用由 ======= 分割的两部分中的一个
	- 解决了所有文件里的冲突之后，对每个文件使用 `git add` 命令来将其标记为冲突已解决

### 3.3 分支管理

- `git branch` 列出所有分支
- `git branch -v` 查看每支分支的最后一次提交
- `git branch --merged/--no-merged`过滤已经合并或尚未合并到**当前分支**的分支
- 未合并的分支，在`git branch -d `删除时会失败，可用`git branch -D`强制删除

### 3.4 分支开发工作流

- 长期分支
	- master分支用来发布稳定版本，所有其它测试版本都在其右侧。当测试版本稳定后，合并至master分支内![](https://github.com/limbo-note/books/blob/master/PROGIT/3-11.jpg) ![](https://github.com/limbo-note/books/blob/master/PROGIT/3-12.jpg)
- 特性分支
	- 你在 master 分支上工作到 C1 ，这时为了解决一个问题而新建 iss91 分支，在 iss91 分支上工作到 C4 ，然而对于那个问题你又有了新的想法，于是你再新建一个 iss91v2 分支试图用另一种方法解决那个问题，接着你回到 master 分支工作了一会儿，你又冒出了一个不太确定的想法，你便在 C10 的时候新建一个 dumbidea 分支，并在上面做些实验。如图：![](https://github.com/limbo-note/books/blob/master/PROGIT/3-13.jpg)
	- 你决定使用第二个方案来解决那个问题，即使用在 iss91v2 分支中方案；另外，你将 dumbidea 分支拿给你的同事看过之后，结果发现这是个惊人之举。 这时你可以抛弃 iss91 分支（即丢弃 C5 和 C6 提交），然后把另外两个分支合并入主干分支。如图：![](https://github.com/limbo-note/books/blob/master/PROGIT/3-14.jpg)

### 3.5 远程分支

`git remote show <remotename>`来获得远程仓库的分支信息

- 本地和远程基本通信流程
	1. 克隆之后的服务器与本地仓库![](https://github.com/limbo-note/books/blob/master/PROGIT/3-15.jpg)
	2. 本地和远程都做了一些新的提交![](https://github.com/limbo-note/books/blob/master/PROGIT/3-16.jpg)
	3. 把远程同步到本地`git fetch origin`![](https://github.com/limbo-note/books/blob/master/PROGIT/3-17.jpg)这会在本地生成一个远程origin/master分支指向服务器的master分支，且此远程origin/master分支指针不能被修改
	4. 此时需要添加另一个远程仓库：`git remote add teamone git://git.team1.ourcompany.com`
	5. 同步第二个远程仓库，抓取本地没有的数据`git fetch teamone`![](https://github.com/limbo-note/books/blob/master/PROGIT/3-18.jpg)
	6. 同步至本地后，`git merge origin/master`可将远程分支合并至当前本地分支中

- 推送
	- `git push <remotename> <localbranch:remotebranch>`将本地分支localbranch推送到远程仓库remotename的远程分支remotebranch
	- `git config --global credential.helper cache`避免每次都输入密码
- 跟踪分支
	- 从一个远程跟踪分支中`git checkout -b [branch] [remote]/[branch]`新建出一个本地分支叫做**跟踪分支**，跟踪分支是与远程分支有直接关系的本地分支，在`git push/git pull`等命令无参数时，默认以跟踪分支中的关联规则来更新远程分支、当前本地分支
	- `git branch -u [remote]/[branch]`设置当前本地分支跟踪远程分支，本地没有该远程分支时，可`git push -u [remote] [branch]`进行推送并自动跟踪
	- `git branch -vv`可查看分支跟踪情况
- 删除远程服务器的分支
	- `git push [remote] --delete [branch]`

### 3.6 变基

整合不同分支的方法有：`merge`和`rebase`，`merge`合并会把两个分支的最新快照以及二者最近的共同祖先进行三方合并，并生成一个新的合并提交，即:![](https://github.com/limbo-note/books/blob/master/PROGIT/3-19.jpg) ![](https://github.com/limbo-note/books/blob/master/PROGIT/3-20.jpg)

- 变基的基本操作
	- 提取在 C4 中引入的补丁和修改，然后在 C3 的基础上再应用一次
	- `git checkout experiment`, `git rebase master`即为变基操作
	- 找到这两个分支最近共同祖先 C2 ，对比当前分支相对于该祖先的历次提交，提取相应的修改并存为临时文件，然后将当前分支指向目标基底 C3 , 最后以此将之前另存为临时文件的修改应用于 C3 ![](https://github.com/limbo-note/books/blob/master/PROGIT/3-21.jpg)
	- 此时可回到master，再进行合并. `git checkout master`,`git merge experiment` ![](https://github.com/limbo-note/books/blob/master/PROGIT/3-22.jpg)
	- 变基合并的结果和普通的三方合并的结果是一样的
- `git rebase --onto`
	- 如图，想将在client 分支里但不在 server 分支里的修改合并到master，这时候就必须用到变基操作`git rebase --onto master server client`![](https://github.com/limbo-note/books/blob/master/PROGIT/3-23.jpg)
	- 以上命令的意思是：“取出 client 分支，找出处于 client 分支和 server 分支的共同祖先**之后**的修改，然后把它们在 master 分支上重演一遍”，即最后会变成![](https://github.com/limbo-note/books/blob/master/PROGIT/3-24.jpg) 此操作没有将 C3 合并至master
- 变基的风险 
	- 不要对在你的仓库外有副本的分支执行变基
- 变基和合并
	- 变基会改变提交历史，而合并不会，各有优劣，酌情选择

# 4. 服务器上的Git
(.............)
# 5. 分布式Git
(.............)

# 6. GitHub

### 6.2 对项目做出贡献

- fork需要贡献的项目
- 将派生出的副本克隆到本地
- 创建出名称有意义的分支
- 修改代码
- 将改动提交到本地分支中
- 将新分支推送到 GitHub 的副本中
- 在github的副本中，可以创建合并请求，发送给原拥有者一个合并请求

### 6.3 维护项目
(........)
### 6.4 管理组织
(........)
### 6.5 脚本GitHub
(........)

# 7
(........)
