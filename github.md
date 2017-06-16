<!-- TOC -->

- [1. 更新fork项目](#1-更新fork项目)
- [2. 开启二次验证，如何上传下载代码](#2-开启二次验证如何上传下载代码)

<!-- /TOC -->

# 1. 更新fork项目

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




# 2. 开启二次验证，如何上传下载代码

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