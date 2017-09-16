<!-- TOC -->

- [GOPATH设置](#gopath设置)
- [将GOPATH写入到.hash_profile](#将gopath写入到hash_profile)
- [添加GOBIN到PATH](#添加gobin到path)

<!-- /TOC -->


前面我们在安装Go的时候看到需要设置GOPATH变量，Go从1.1版本开始必须设置这个变量，而且不能和Go的安装目录一样，这个目录用来存放Go源码，Go的可运行文件，以及相应的编译之后的包文件。所以这个目录下面有三个子目录：src、bin、pkg

# GOPATH设置

go 命令依赖一个重要的环境变量：$GOPATH

Windows系统中环境变量的形式为%GOPATH%，本书主要使用Unix形式，Windows用户请自行替换。

（注：这个不是Go安装目录。下面以笔者的工作目录为示例，如果你想不一样请把GOPATH替换成你的工作目录。）

在类似 Unix 环境大概这样设置：

	export GOPATH=/home/apple/mygo

为了方便，应该把新建以上文件夹，并且把以上一行加入到 .bashrc 或者 .zshrc 或者自己的 sh 的配置文件中。

Windows 设置如下，新建一个环境变量名称叫做GOPATH：

    GOPATH=c:\mygo

GOPATH允许多个目录，当有多个目录时，请注意分隔符，多个目录的时候Windows是分号，Linux系统是冒号，当有多个GOPATH时，默认会将go get的内容放在第一个目录下。

以上 $GOPATH 目录约定有三个子目录：

- src 存放源代码（比如：.go .c .h .s等）
- pkg 编译后生成的文件（比如：.a）
- bin 编译后生成的可执行文件（为了方便，可以把此目录加入到 $PATH 变量中，如果有多个gopath，那么使用${GOPATH//://bin:}/bin添加所有的bin目录）

# 将GOPATH写入到.hash_profile

1. 进入用户目录
  
  	`cd ~`
  	
2. 创建.bash_profile,如果没有的话
  
    `touch .bash_profile`
  
3. 打开并编辑.bash_profile
    
    `open -e .bash_profile`
    
4. 更新刚配置的环境变量
    
    `source .bash_profile`

# 添加GOBIN到PATH

在.bash_profile加入下面的内容：

```
export GOPATH=/Users/zhangyuchen/go/pro
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
```


