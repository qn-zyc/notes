# 安装路径

mac 上是 `/Library/Python/2.7/site-packages`

# MySQLdb

- 下载 <https://pypi.python.org/pypi/MySQL-python/1.2.5#downloads>
- 查找 `mysql_config` : `whereis mysql_config`
- - 如果没有的话可能是没有安装 `mysql-devel` : `yum install mysql-devel`, 之后在 `/usr/bin` 应该存在 mysql_config 了。
- 修改 site.cfg
  - 修改mysql_config为mysql配置文件的路径 /usr/bin/mysql_config
  - threadsafe = False
- `python setup.py build`
- `sudo python setup.py install`



## centos

`yum install mysql-python`



# pip

## centos 安装

下载 [get-pip.py](https://pip.pypa.io/en/stable/installing/)

执行: `python get-pip.py`


## 更新包

更新pip:

```bash
pip install -U pip
```

使用 `pip install -h` 查看 install 的帮助信息。

## 查看已安裝包列表

```bash
pip freeze
```

## 卸载包

```bash
pip uninstall package_name
```

## 查询包

```bash
pip search "package_name"
```
## 查看需要更新的包

```python
pip list --outdated	
```

