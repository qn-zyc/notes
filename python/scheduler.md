# sche.scheduler

```python
import sched
import time


def print_time():
    print time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())


def scheduler():
    s = sched.scheduler(time.time, time.sleep)  # timefunc, delayfunc
    # 延迟5秒，优先级1，action, arguments(传给action的)
    s.enter(5, 1, print_time, ())
    s.enter(10, 1, print_time, ())
    # 阻塞直到队列为空，阻塞是调用上面指定的delayfunc
    s.run()


if __name__ == '__main__':
    scheduler()
```

多线程模式下不能在scheduler运行时添加任务。

scheduler会阻塞住主进程，直到任务队列为空。



# threading.Timer

```python
from threading import Timer

Timer(5, print_time, ()).start()
Timer(10, print_time, ()).start()
time.sleep(11)  # sleep while time-delay events execute
```

