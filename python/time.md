# 获取时间

```python
import time

timestamp = time.time()
print "时间戳：", timestamp  # 1466735398.95

localTime = time.localtime(timestamp)
print "时间元组：", localTime  # time.struct_time(tm_year=2016, tm_mon=6, tm_mday=24, tm_hour=10, tm_min=33, tm_sec=11, tm_wday=4, tm_yday=176, tm_isdst=0)
print localTime[0]  # 2016
```

时间元组

| 序号   | 字段     | 值                        |
| ---- | ------ | ------------------------ |
| 0    | 4位数年   | 2008                     |
| 1    | 月      | 1 到 12                   |
| 2    | 日      | 1到31                     |
| 3    | 小时     | 0到23                     |
| 4    | 分钟     | 0到59                     |
| 5    | 秒      | 0到61 (60或61 是闰秒)         |
| 6    | 一周的第几日 | 0到6 (0是周一)               |
| 7    | 一年的第几日 | 1到366 (儒略历)              |
| 8    | 夏令时    | -1, 0, 1, -1是决定是否为夏令时的旗帜 |



## 获取时间戳

```python
# time.struct_time(tm_year=2100, tm_mon=12, tm_mday=12, tm_hour=12, tm_min=12, tm_sec=12, tm_wday=6, tm_yday=346, tm_isdst=-1)
time_arr = time.strptime('2100-12-12 12:12:12', '%Y-%m-%d %H:%M:%S')
time.mktime(time_arr)  # 4132267932.0
```





# 格式化时间

```python
import time

# 格式化成2016-03-20 11:45:39形式
print time.strftime("%Y-%m-%d %H:%M:%S", time.localtime()) 

# 格式化成Sat Mar 28 22:24:24 2016形式
print time.strftime("%a %b %d %H:%M:%S %Y", time.localtime()) 
  
# 将格式字符串转换为时间戳
a = "Sat Mar 28 22:24:24 2016"
print time.mktime(time.strptime(a,"%a %b %d %H:%M:%S %Y"))
```

python中时间日期格式化符号：

| 符号   | 说明                      |
| ---- | ----------------------- |
| %y   | 两位数的年份表示（00-99）         |
| %Y   | 四位数的年份表示（000-9999）      |
| %m   | 月份（01-12）               |
| %d   | 月内中的一天（0-31）            |
| %H   | 24小时制小时数（0-23）          |
| %I   | 12小时制小时数（01-12）         |
| %M   | 分钟数（00=59）              |
| %S   | 秒（00-59）                |
| %a   | 本地简化星期名称                |
| %A   | 本地完整星期名称                |
| %b   | 本地简化的月份名称               |
| %B   | 本地完整的月份名称               |
| %c   | 本地相应的日期表示和时间表示          |
| %j   | 年内的一天（001-366）          |
| %p   | 本地A.M.或P.M.的等价符         |
| %U   | 一年中的星期数（00-53）星期天为星期的开始 |
| %w   | 星期（0-6），星期天为星期的开始       |
| %W   | 一年中的星期数（00-53）星期一为星期的开始 |
| %x   | 本地相应的日期表示               |
| %X   | 本地相应的时间表示               |
| %Z   | 当前时区的名称                 |
| %%   | %号本身                    |



# datetime

## 获取当前时间

```python
In [11]: datetime.datetime.now()
Out[11]: datetime.datetime(2016, 12, 1, 15, 32, 16, 512545)
    
In [3]: datetime.date.today()
Out[3]: datetime.date(2016, 12, 14)
    
# 获取毫秒值
In [33]: datetime.datetime.now().microsecond / 1000
Out[33]: 434
    
# 从时间戳获取时间
datetime.datetime.fromtimestamp(1480929026)

# 从datetime获取时间戳
In [10]: time.mktime(datetime.datetime.now().timetuple())
Out[10]: 1481694783.0
```



## 时间运算

```python
In [13]: d2 = d1 + datetime.timedelta(seconds=10)

In [14]: d1
Out[14]: datetime.datetime(2016, 12, 1, 15, 33, 7, 172147)

In [15]: d2
Out[15]: datetime.datetime(2016, 12, 1, 15, 33, 17, 172147)
    
# 昨天
In [4]: datetime.date.today() - datetime.timedelta(days=1)
Out[4]: datetime.date(2016, 12, 13)
```



## 格式化

```python
In [16]: d1 = datetime.datetime.now()

In [17]: d1.strftime('%Y-%m-%d %H:%M:%S')
Out[17]: '2016-12-01 15:34:16'
```

