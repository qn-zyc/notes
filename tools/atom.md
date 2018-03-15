
<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->
<!-- code_chunk_output -->

* [快捷键](#快捷键)
* [拼写检查](#拼写检查)
* [代码片段](#代码片段)
	* [多行代码](#多行代码)
* [设置](#设置)
	* [UI 设置](#ui-设置)
		* [字体](#字体)
* [安装](#安装)
	* [插件安装](#插件安装)
	* [插件列表](#插件列表)
		* [编辑器](#编辑器)
			* [seti](#seti)
			* [terminal](#terminal)
			* [sync-settings](#sync-settings)
		* [vim](#vim)
		* [golang](#golang)
		* [python](#python)
		* [rust](#rust)
		* [markdown](#markdown)
		* [git](#git)
		* [openresty](#openresty)

<!-- /code_chunk_output -->


# 快捷键
- `cmd + \`: 折叠 tree view
- `ctl + 0`: 焦点移到目录树, 按 ESC 焦点转换到编辑器. 连续按可以来回切换.
- `cmd + k, cmd + n`: 切换到下一个pane, 在 keymap 中搜索 "focus pane"
- `cmd + shift + [`, `cmd + alt + left`: 向左切换 item (标签页)
- `cmd + 1`: 切换到第一个 item.
- `cmd + t` | `cmd + p`: 在项目中搜索文件.
- `cmd + b`: 搜索已经打开的文件.
- `cmd + r`: 搜索方法.
- `ctrl + tab`: 在打开的文件列表中切换
- `cmd + .`: 在下面显示执行的快捷键的绑定情况.

查找:
- `cmd + shift + f`: 在工程中查找.




# 拼写检查

拼写检查在 `spell-check` 包, 想禁用某些文件类型的拼写检查, 可以在 `/Setting/Grammars` 中去掉.

查看文件类型的方式在 `spell-check` 包的 README 中: `cmd+shift+p, Editor: Log Cursor Scope`.


# 代码片段

## 多行代码

```cson
'.source.lua':
    'nginx log':
        'prefix': 'ngxlog'
        'body': """
            local log = ngx.log
            local INFO = ngx.INFO
            local ERR = ngx.ERR
            """
```


# 设置

## UI 设置

### 字体
设置 UI 字体大小 https://github.com/atom/atom/issues/2530

我的设置:

```less
@font-size: 1rem;
@font-family: "Operator Mono";

html,
body,
.tree-view,
.tooltip,
.tab-bar .tab,
.find-and-replace .find-meta-container,
atom-text-editor[mini],
.btn,
.status-bar,
.list-tree li.list-nested-item > .list-item {
  font-size: @font-size;
  font-family: @font-family;
}
```

设置字体粗细？
- https://gist.github.com/james2doyle/9260736
- https://developer.mozilla.org/zh-CN/docs/Web/CSS/font-smooth
- https://developer.mozilla.org/zh-CN/docs/Web/CSS/text-rendering

```less
body {
  -webkit-font-smoothing: antialiased;
  text-rendering: optimizeLegibility;
  -moz-osx-font-smoothing: grayscale;
}
```



# 安装

## 插件安装

该死的 gfw.

[方案1](http://blog.csdn.net/qianghaohao/article/details/52331432):

使用国内的源, 创建 `~/.atom/atomrc` 文件, 输入一下内容:

```
registry=https://registry.npm.taobao.org/  
strict-ssl=false
```

[方案2](https://www.zhihu.com/question/50859713/answer/153110096):

从 github 上 clone 插件到 `~/.atom/packages`, 然后运行 `npm install` 或者 `apm install`, 下面以 atom-beautify 为例:

```bash
git clone git@github.com:Glavin001/atom-beautify.git
cd atom-beautify
npm install
```

或者：

```bash
git clone https://github.com/atom-minimap/minimap
apm install minimap
```


## 插件列表

### 编辑器
- [atom-ide-ui](https://github.com/facebook-atom/atom-ide-ui)
- [ide-json](https://github.com/atom/ide-json)
- [minimap](https://github.com/atom-minimap/minimap)
- [minimap-bookmarks](https://github.com/atom-minimap/minimap-bookmarks)
- [minimap-highlight-selected](https://atom.io/packages/minimap-highlight-selected)
- [project manager](https://github.com/danielbrodin/atom-project-manager)
- [highlight-selected](https://github.com/richrace/highlight-selected)
- [hyperclick](https://github.com/facebook-atom/hyperclick)
- [terminal](https://github.com/platformio/platformio-atom-ide-terminal)
- [activate power mode](https://github.com/JoelBesada/activate-power-mode)
- [linter](https://github.com/steelbrain/linter)
- [todo show](https://github.com/mrodalgaard/atom-todo-show)
- [seti-ui](https://github.com/jesseweed/seti-ui)
- [seti-syntax](https://github.com/jesseweed/seti-syntax)
- [file-icons](https://atom.io/packages/file-icons)
- [alignment](https://github.com/Freyskeyd/atom-alignment)
- [sync-settings](https://github.com/atom-community/sync-settings)
- [pretty-json](https://github.com/federomero/pretty-json)

#### seti

使用 seti-ui 报错: variable @font-family is undefined, 切换到其他 ui 就没事, 参考下面两个 issue:

- https://github.com/facebook-atom/atom-ide-ui/issues/14
- https://github.com/jesseweed/seti-ui/pull/459/files

解决方法, 修改 `styles/ui-variables.less`, 添加 `font-family`:

```less
@seti-font-family: 'Roboto', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif;
@font-family: 'Roboto', 'Helvetica Neue', 'Helvetica', 'Arial', sans-serif;
```


#### terminal

修改字体, 参考
- https://github.com/platformio/platformio-atom-ide-terminal/issues/33

在 `~/.atom/packages/platformio-ide-terminal/lib/platformio-ide-terminal.coffee` 中查找 fontFamily 修改.


#### sync-settings
- [ATOM同步插件和配置，使用sync-settings](http://www.jianshu.com/p/bd006b349d03)


### vim
- [vim mode plus](https://github.com/t9md/atom-vim-mode-plus)
- [ex-mode](https://github.com/lloeki/ex-mode)
- [relative numbers](https://github.com/justmoon/relative-numbers)


### golang
- [go-plus](https://github.com/joefitzgerald/go-plus)
- [go-signature-statusbar](https://github.com/wndhydrnt/go-signature-statusbar) 在状态栏显示函数信息.
- [linter-golinter](https://github.com/AtomLinter/linter-golinter)
- [ide-go](https://github.com/ckaznocha/ide-go)
- [go-quick-import](https://github.com/mastercactapus/atom-go-quick-import)


### python
- [ide python](https://github.com/lgeiger/ide-python)


### rust
- [language-rust](https://atom.io/packages/language-rust)
- [ide-rust](https://atom.io/packages/ide-rust)


### markdown
- [markdown toc](https://github.com/nok/markdown-toc)
- [markdown table editor](https://github.com/susisu/atom-markdown-table-editor)
- [markdown-scroll-sync](https://github.com/vincentcn/markdown-scroll-sync)
- [markdown-preview-enhanced](https://github.com/shd101wyy/markdown-preview-enhanced)


### git
- [split-diff](https://github.com/mupchrch/split-diff)
- [git-time-machine](https://github.com/littlebee/git-time-machine)


### openresty
- [language-lua](https://github.com/FireZenk/language-lua)
- [atom-language-nginx](https://github.com/hnagato/atom-language-nginx)
