<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [git](#git)
	- [概念](#概念)
	- [配置](#配置)
		- [别名](#别名)
		- [输出](#输出)
	- [命令简介](#命令简介)
	- [git add](#git-add)
	- [git status](#git-status)
	- [git stash 储藏](#git-stash-储藏)
	- [git diff](#git-diff)
	- [git reset](#git-reset)
	- [git checkout](#git-checkout)
	- [git ls-tree](#git-ls-tree)
	- [git ls-files](#git-ls-files)
	- [git rm](#git-rm)
	- [git clean](#git-clean)
	- [git log](#git-log)
- [.gitignore](#gitignore)
- [github](#github)
	- [更新fork项目](#更新fork项目)
	- [开启二次验证，如何上传下载代码](#开启二次验证如何上传下载代码)
	- [拉取远程库的pr到本地分支](#拉取远程库的pr到本地分支)
- [参考](#参考)

<!-- /TOC -->



# git

## 概念
* 工作区, 暂存区, 版本库
* HEAD 是版本库的一个引用.



## 配置
* `git config` 对应git库的 `.git/config`.
* `git config --global` 对应 `~/.gitconfig`.
* `git config --system` 对应 `/etc/gitconfig`.
* 读取配置: `git config <section>.<key>`
* 设置配置: `git config <section>.<key> <value>`
* 操作其他INI文件: `GIT_CONFIG=test.ini git config a.b.c "hello, world"`
* 删除配置: `git config --unset --global user.name`

```shell
git config --global user.name "Your Name"
git config --global user.email YourEmail@example.com
```

### 别名

```shell
git config --system alias.st status
git config --global alias.ci "commit -s"
```

### 输出

开启颜色显示: `git config --global color.ui true`


## 命令简介
- `git init`: 创建版本库
- `git status`: 查看状态
- `git add`: 将工作区的改动提交到暂存区。
- `git commit`: 将暂存区的改动提交到版本库。



## git add

- `git add -f filename`: 强制将 filename 的修改添加到暂存区（一般用于 filename 被忽略的情况下）




## git status

- `git status -s`: 以简洁的形式显示变化。
- `git status --ignored -s`: 显示被忽略的文件（.gitignore 文件中的规则）


## git stash 储藏

* `git stash`: 储藏当前变更(暂存的和工作区的), 不包括 untracked 文件(未跟踪文件, 比如新建的)
* `git stash save "my message"`: 自定义储藏的消息.
* `git stash save -u`: 储藏当前变更, 包括 untracked 文件, `-u|--include-untracked`.
* `git stash list`: 查看储藏列表
* `git stash apply`: 应用最新的储藏，但不删除储藏.
* `git stash apply stash@{1}`: 应用指定的储藏.
* `git stash apply --index`: 重新暂存暂存区的文件.
* `git stash drop stash@{1}`: 从列表中移除储藏，不指定储藏则删除最新的.
* `git stash pop`: 应用最新的储藏并从列表中移除.
* `git stash pop <stash>`: 应用指定的储藏并从列表中删除。
* `git stash branch <branch_name> <stash>`: 使用储藏创建分支并删除储藏.
* `git stash clear`: 删除所有的储藏。


## git diff
* `git diff`: 工作区和暂存区比较
* `git diff --cached`: 暂存区和 HEAD 比较
* `git diff HEAD`: 工作区和 HEAD 比较



## git reset

- `git reset HEAD` 或者 `git reset`: 用版本库的目录树覆盖暂存区的目录树，工作区不受影响（add 到暂存区的内容变成了 modified 状态）。
- `git reset HEAD <paths>`: 用版本库中指定目录覆盖暂存区的目录树。
- `git reset --mixed <commit>`: 重置 HEAD 并覆盖暂存区，但不改变工作区。上面的命令就是省略了 `--mixed`。
- `git reset --hard HEAD^`: 将版本库重置到 HEAD 游标的上一次提交，并覆盖暂存区，`--hard` 表示也覆盖工作区。
- `git reset --hard 9e8a761`: 重置到任意一次提交上。
- `git reset -- filename` 或 `git reset HEAD filename`: 仅将 filename 的改动撤出暂存区，相当于 `git add filename` 的反向操作。



## git checkout

- `git checkout .` 或 `git checkout -- .`: 用暂存区全部的文件替换工作区的文件(修改的文件会撤销修改，新增的文件不受影响)。
- `git checkout -- <paths>`: 用暂存区指定的文件替换工作区的文件。
- `git checkout HEAD .`: 用 HEAD 指向的 master 分支替换暂存区和工作区的文件(只替换 master 分支中跟踪了的文件)。
- `git checkout HEAD <paths>`: 用 HEAD 指向的 master 分支中指定的文件替换暂存区和工作区的文件。
- `git checkout <commit> -- <paths>`: 用指定提交的文件覆盖暂存区和工作区对应的文件。
- `git checkout <branch>`: 切换到指定分支，这会改变 HEAD 头指针的指向。
- `git checkout`: 对工作区进行状态检查，汇总显示工作区、暂存区与 HEAD 的差异。


## git ls-tree

- `git ls-tree -l HEAD`: 查看 HEAD 指向的目录树，`-l` 表示显示文件大小(单位B)
    ```
    100644 blob ac3c1b0ba79940754a27c94253ae19dd40f69f76      12	a.txt
    ```
    从左到右分别表示 文件属性(rw-r--r--)，blob对象(文件)，ID，文件大小(12B)，文件名。


## git ls-files

- `git ls-files -s`: 查看暂存区的目录树。
    ```
    100644 ac3c1b0ba79940754a27c94253ae19dd40f69f76 0	a.txt
    ```
    第三个字段 0 表示暂存区编号。



## git rm

- `git rm file1 file2 ...`: 删除文件并将删除动作加入暂存区，之后直接使用 commit 提交就可以从版本库最新提交中删除文件了。



## git clean

- `git clean -nd`: 显示哪些文件和目录会被删除，不会真的删。
- `git clean -fd`: 删除未被跟踪的文件和命令(不在暂存区，也不在版本库)，文件已经被跟踪了，不管是不是修改了都不会删除。



## git log

- `git log --graph --oneline`   : 每个提交占一行，以图的方式展示。
- `git log --oneline <filename>`: 查看某个文件的修改历史，获取到 id 后可用 `git show <id>` 查看详细情况。




# .gitignore

语法规则：
- 注释使用 `#` 开头。
- 可以使用通配符，`*` 任意多字符，`?` 一个字符，`[abc]` 可选字符范围等。
- 如果以 `/` 开头，表明忽略的文件在此目录下，而非子目录下。
- 如果以 `/` 结尾，表明要忽略的是目录，同名文件不忽略，否则同名文件和目录都忽略。
- 在名称前加 `!` 表示不忽略。

示例：

```
# 这是注释
*.a       # 忽略以 .a 为扩展名的文件
!lib.a    # 但是 lib.a 文件或目录不要忽略
/TODO     # 只忽略此目录下的 TODO 文件
build/    # 忽略所有 build/ 目录下的文件
doc/*.txt # 忽略文件如 doc/notes.txt, 但是 doc/server/arch.txt 不被忽略。
```








# github

## 更新fork项目

1. 增加源分支地址到你项目远程分支列表中(此处是关键)，先得将原来的仓库指定为upstream，命令为：
    ```bash
    git remote add upstream https://github.com/被fork的仓库.git
    ```
    此处可使用 `git remote -v` 查看远程分支列表

2. fetch源分支的新版本到本地
   ```bash
   [master]> git fetch upstream
   ```
3. 合并两个版本的代码
   ```bash
   [master]> git merge upstream/master
   ```
4. 将合并后的代码push到github上去
   ```bash
   [master]> git push origin master
   ```




## 开启二次验证，如何上传下载代码

> http://shigongbo.blog.163.com/blog/static/976090201542983536987

如果开起了二次验证（Two-factor authentication两个要素认证），命令行会一直提示输入用户名和密码。查找了一下解决方法如下：

1. 准备Token信息。

登陆GitHub，通过右上角的设置按钮进入设置页面，点击Personal access tokens，为你的账号创建一个Token，

创建好以后，保存这个Token，最好保存到你本地文件，因为离开页面后这个将会找不到了。

然后回到电脑的命令行界面。

2. 设置git保存认证信息

执行 `git config --global credential.helper store`

3. 使用git clone代码

```bash
git clone XXXX.git
```

此时会提示你输入UserName 和Password， 如：

Username for 'https://github.com': yourname（此处名称为你在GitHub上的UserNmae，而不是你GitHub的邮箱）

Password for 'https://hainuo@github.com':此处即为你获得的Token。
到此OK。

如果你没有设置 `git config --global credential.helper store`，那么你每次git pull或者 git push时候都会提示你输入UserName和Password。

如果设置了该选项，则UserName和Password将会被保存，下次直接git pull或者git push即可。

其实该命令会在用户根目录下生成一个名为.git-credentials的文件，里面保存了你的UserName和Token。



## 拉取远程库的pr到本地分支

- [Checking out pull requests locally](https://help.github.com/articles/checking-out-pull-requests-locally/)

```bash
git fetch origin pull/ID/head:BRANCHNAME
# 比如
git fetch origin pull/1122/head:review1122

# 之后本地就多了个 review1122 的分支
git checkout review1122
```

上面的 origin 换成 upstream 也可以。



# 参考
* [git权威指南]()
