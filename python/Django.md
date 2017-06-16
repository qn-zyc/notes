# 安装

## mac

从 [https://www.djangoproject.com/download/] 下载压缩包，解压，执行：

```bash
sudo python setup.py install
```

查看：`/Library/Python/2.7/site-packages`



# 创建测试项目

```sh
django-admin.py startproject testweb
cd testweb
python manage.py runserver
```

在 `localhost:8000` 查看。

指定端口： `python manage.py runserver 127.0.0.1:8000`



