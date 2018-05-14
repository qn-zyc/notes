# govendor
* [GitHub](https://github.com/kardianos/govendor)
* Copy existing dependencies from $GOPATH with `govendor add/update`.
* If you ignore `vendor/*/`, restore dependencies with `govendor sync`.
* Pull in new dependencies or update existing dependencies directly from remotes with `govendor fetch`.
* Migrate from legacy systems with `govendor migrate`.

常用命令：

```bash
# 从 GOPATH 更新所有的 vendor, 只查看要做的动作不执行
govendor update -n +vendor
# 从 GOPATH 更新所有的 vendor
govendor update +vendor
# 从 GOPATH 更新特定的包, 只查看要做的动作不执行
govendor update -n <import-path>
# 从 GOPATH 更新特定的包
govendor update <import-path>
```

对于 govendor 来说，主要存在三种位置的包：项目自身的包组织为本地（local）包；传统的存放在 $GOPATH 下的依赖包为外部（external）依赖包；被 govendor 管理的放在 vendor 目录下的依赖包则为 vendor 包。

包的状态：
* `+local`(缩写:l): 本地包，即项目自身的包组织。
* `+external`(缩写:e): 外部包，即被 $GOPATH 管理，但不在 vendor 目录下。
* `+vendor`(缩写:v): 已被 govendor 管理，即在 vendor 目录下。
* `+std`(缩写:s): 标准库中的包。
* `+unused`(缩写:u): 未使用的包，即包在 vendor 目录下，但项目并没有用到。
* `+missing`(缩写:m): 代码引用了依赖包，但该包并没有找到。
* `+program`(缩写:p): 主程序包，意味着可以编译为执行文件。
* `+outside`: 外部包和缺失的包。
* `+all`: 所有的包。


命令列表:
* `init`: 初始化 vendor 目录。
* `list`: 列出所有的依赖包。
* `add`: 添加包到 vendor 目录，如 `govendor add +external` 添加所有外部包。
* `add PKG_PATH`: 添加指定的依赖包到 vendor 目录。
* `update`: 从 $GOPATH 更新依赖包到 vendor 目录。
* `remove`: 从 vendor 管理中删除依赖。
* `status`: 列出所有缺失、过期和修改过的包。
* `fetch`: 添加或更新包到本地 vendor 目录。
* `sync`: 本地存在 vendor.json 时候拉取依赖包，匹配所记录的版本。
* `get`: 类似 go get 目录，拉取依赖包到 vendor 目录。

Quick Start：

```bash
# Setup your project.
cd "my project in GOPATH"
govendor init

# Add existing GOPATH files to vendor.
govendor add +external

# View your work.
govendor list

# Look at what is using a package
govendor list -v fmt

# Specify a specific version or revision to fetch
govendor fetch golang.org/x/net/context@a4bbce9fcae005b22ae5443f6af064d80a6f5a55
govendor fetch golang.org/x/net/context@v1   # Get latest v1.*.* tag or branch.
govendor fetch golang.org/x/net/context@=v1  # Get the tag or branch named "v1".

# Update a package to latest, given any prior version constraint
govendor fetch golang.org/x/net/context

# Format your repository only
govendor fmt +local

# Build everything in your repository only
govendor install +local

# Test your repository only
govendor test +local

```

FAQ:

Q: How do I test only my packages?
A: Run `govendor test +local`.

Q: How do I build install all my vendor packages?
A: Run `govendor install +vendor,^program`.

Q: How do I pull all my dependencies from network remotes?
A: Run `govendor fetch +out`.

Q: I have a working program with dependencies in $GOPATH. I want to vendor now.
A: Run `govendor add +external`.

Q: I have copied dependencies into "vendor". I want to update from $GOPATH.
A: Run `govendor update +vendor`.

Q: I'm getting missing packages from appengine but I don't care about appengine. How do I ignore these packages?
A: Edit the `vendor/vendor.json` file. Update the "ignore" field to include "appengine". If you are already ignoring tests, it will look like: "ignore": "test appengine",.

Q: I have modified a package in $GOPATH and I want to try the changes in vendor without committing them.
A: Run `govendor update -uncommitted <updated-package-import-path>`.

Q: I've forked a package and I haven't upstreamed the changes yet. What should I do?
A: Assuming you've pushed your changes to an accessable repository, run `govendor fetch github.com/normal/pkg::github.com/myfork/pkg`. This will fetch from "myfork" but place package in "normal".

Q: I have C files or HTML resources in sub-folders. How do I ensure they are copied as well?
A: Run either `govendor fetch github.com/dep/pkg/^` or `govendor add github.com/dep/pkg/^`. This is the same as using the `-tree` argument.

Q: How do I prevent vendor source from being checked in?
A: Add `vendor/*/` to your ignore file.

Q: How do I populate the vendor folder if it has not been checked in?
A: Run `govendor sync`.

参考：
* [go 依赖管理利器 -- govendor](https://blog.csdn.net/yeasy/article/details/65935864)
* [GitHub](https://github.com/kardianos/govendor)


