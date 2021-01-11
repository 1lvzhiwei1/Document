# Git

## 安装

==sudo apt-get install git==

## 创建仓库

==初始化一个仓库：git init==

**只能跟踪文本文件，不能跟踪二进制文件**

==添加文件到仓库==

* 使用命令`git add <file>`，注意，可反复多次使用，添加多个文件
* 使用命令`git commit -m <message>`，完成

## 基本操作

==git status==

**功能：查看仓库当前状态，查看是否有文件被修改，不显示具体修改内容**

==git diff==

**功能：查看仓库文件修改的具体内容**

### 版本回退

==git log==

**功能：查看最近的git提交日志**

==回退指定版本==

**语法：**

* **git reset --hard HEAD^**

* **git reset --hard 命令版本号**

==git reflog==

**功能：查看历史命令版本号**

### 工作区和暂存区

==`git add`命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210109164309869.png" alt="image-20210109164309869" style="zoom:80%;" />

* 用`git add`把文件添加进去，实际上就是把文件修改添加到暂存区
* 用`git commit`提交更改，实际上就是把暂存区的所有内容提交到当前分支

### 管理修改

**git跟踪管理的是修改，而非文件**

**每次修改，如果不用`git add`到暂存区，那就不会加入到`commit`中**

==第一次修改 -> `git add` -> 第二次修改 -> `git commit`==

git add后，第一次修改放入了暂存区，第二次修改没有add，所以commit后只提交暂存区的内容，第二次修改的内容没有上传

**解决方法：**

==第一次修改 -> `git add` -> 第二次修改 -> `git add` -> `git commit`==

### 撤销修改

* 当修改还未add，仍然处在工作区的时候，使用==git checkout -- < filename >==来撤销修改
* 修改已经add，处于暂存区的时候
  * 先使用==git reset HEAD < filename >==回退到工作区状态
  * 再使用==git checkout -- < filename >==来撤销修改
* 已经commit后，使用==git reset --hard 命令版本号==来撤销修改，**前提是没有提交到远程库**

### 删除文件

==删除步骤==

* **git rm < filename >**
* **git commit -m "xxx"**

==误删恢复==

**git checkout -- < filename >：**用版本库里的版本替换工作区的版本，无论工作区是修改还是删除，都可以“一键还原”；只能恢复文件到最新版本，会丢失**最近一次提交后修改的内容**

## 远程仓库

### 添加远程库

==关联远程库==

**git remote add origin git@server-name:path/repo-name.git**

==本地推送到远程分支==

**git push origin 分支名**

### 克隆远程库

==克隆==

**git clone git@github.com:username/repositoryname.git**

### git修改协议

**https协议每次推送远程时，需要输入用户名和密码，ssh效率更高**

* **git remote rm origin**
* **ssh协议：git remote add origin git@github.com:username/分支名.git**
* **https协议：git remote add origin https://github.com:username/分支名.git**

## 分支管理

### 创建和合并分支

==当前分支在master上==

![image-20210110204028030](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210110204028030.png)

==创建新分支dev，HEAD指向dev，代表当前分支在dev上==

**git checkout -b dev**

**参数-b代表创建并切换，相当于两条命令**

* git branch dev：创建dev分支
* git checkout dev：切换分支

**git branch：查看所有分支**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210110204043004.png" />

==在dev分支上提交修改==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210110204059730.png" />

==把dev合并到master上：master指向dev的最新提交==

**git merge dev：合并指定分支到当前分支**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210110204118966.png" />

==删除dev分支==

**git branch -d dev**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210110204043004.png" style="zoom:80%;" />

### 解决冲突

==master和featurel分支各自分别有新的提交，git无法进行快速合并，必须手动解决冲突==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210110210424043.png" style="zoom:80%;" />

==手动修改冲突的文件==

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210110210651896.png" alt="image-20210110210651896" style="zoom:80%;" />

**git log --graph：查看分支合并图**

### 分支管理策略

**fast forward合并模式：删除分支后，会丢掉分支信息**

==禁用fast forward模式==

**git merge --no-ff -m "merge with no-ff" dev**

==分支策略==

* **master应该是非常稳定的，仅用来发布新版本**
* **团队成员平时修改自己的分支，合并到dev上**
* **发布新版本时，再把dev合并到master上**

<img src="C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210110211436055.png" alt="image-20210110211436055" style="zoom:80%;" />

### Bug分支

**需要新建分支修复bug，但当前dev分支工作还未完成**

* 先保存dev的工作现场
* 新建分支修复bug
* 切回dev，恢复工作现场

==保存工作现场==

**git stash**

==查看保存的工作现场==

**git stash list**

恢复现场有两个办法：

* git stash apply：恢复后，stash内容并不删除，需要用`git stash drop`来删除
* git stash pop：恢复的同时把stash内容也删了

==在master分支上修复的bug，想要合并到当前dev分支==

**git cherry-pick <commit\>：bug提交的修改“复制”到当前分支，避免重复劳动**

### Feature分支

**开发一个新功能，最好新建一个分支**

==强行删除一个没有被合并过的分支==

**git branch -D <name\>**

### 多人协作

工作模式：

* **试图用`git push origin <branch-name>`推送自己的修改**
* **如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并**
* **如果合并有冲突，则解决冲突，并在本地提交**
* **没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功**

==如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`建立链接关系==

==更新本地库==

**git pull**

==查看远程库信息==

**git remote -v**

==在本地创建和远程分支对应的分支==

**git checkout -b branch-name origin/branch-name**

### Rebase

==rebase==

**把本地未push的分叉提交历史整理成直线**

**git rebase**

## 标签管理

**tag就是版本库的快照，是指向commit的指针，能够快速的根据标签找到某个commit**

### 创建标签

==创建新标签==

* **git tag <name\>：在HEAD指向的commit打上标签**

* **git tag <name\> commitid：给指定commit打上标签**

* **git tag -a tagname -m "备注信息" commitid**

==查看所有标签==

**git tag**

==查看标签信息==

**git show <tagname\>**

### 操作标签

==删除标签==

**git tag -d tagname**

==推送标签到远程==

* **git push origin <tagname\>**
* **git push origin --tags**

==删除远程标签==

* **先删除本地，git tag -d 版本号**
* **git push origin :refs/tags/tagname**

## 自定义Git

### 忽略特殊文件

**在Git的工作区的根目录下创建一个特殊的.gitignore文件，填入需要忽略的文件名**

**配置文件浏览：https://github.com/github/gitignore**

==原则==

* 忽略操作系统自动生成的文件，比如缩略图等
* 忽略编译生成的中间文件、可执行文件等，例如.class文件
* 忽略带有敏感信息的文件，例如存放口令的配置文件

### 配置别名

==配置全局别名==

**git config --global alias.别名 命令**

==删除别名==

**修改.ignore文件**

