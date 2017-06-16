# 多线程

> http://python.jobbole.com/86909/

虽然`python`中由于`GIL`的机制致使多线程不能利用机器多核的特性，但是多线程对于我们理解并发模型以及底层操作非常有用。

线程的有两种使用方法，**一种是在函数使用，一种是放在类中使用**



## 在函数中使用

语法：

```python
thread.start_new_thread(function, args[, kwargs] )
```

- function - 线程函数。
- args - 传递给线程函数的参数,必须是个tuple类型。
- kwargs - 可选参数。



```python
import threading


def run(num):
    print 'hi , i am a thread.', num


def main():
    threads = []
    for i in range(5):
        t = threading.Thread(target=run, args=(i,))
        threads.append(t)
        t.start()

    for t in threads:
        t.join()


if __name__ == '__main__':
    main()
```

运行结果：

```
hi , i am a thread. 0
hi , i am a thread. 1
hi , i am a thread. 2
hi , i am a thread. 3
hi , i am a thread. 4
```



## 在类中使用

```python
class MyThread(threading.Thread):
    def __init__(self, num):
        self.num = num
        super(MyThread, self).__init__()

    def run(self):
        print 'i am a thread,', self.num
        time.sleep(1)


if __name__ == '__main__':
    threads = []
    for i in range(5):
        t = MyThread(i)
        threads.append(t)
        t.start()
    for t in threads:
        t.join()
```

- **run()**，需要重写，编写代码实现所需要的功能。
- **getName()**，获得线程对象名称
- **setName()**，设置线程对象名称
- **start()**，启动线程
- **join([timeout])**，等待另一线程结束后再运行。若指定timeout则超时之后不再阻塞当前线程。
- **setDaemon(bool)**，设置子线程是否随主线程一起结束，必须在`start()` 之前调用，默认为`False`。
- **isDaemon()**，判断线程是否随主线程一起结束。
- **isAlive()**，检查线程是否在运行中。



## 线程同步与互斥锁

```python
var = 0
lock = threading.Lock()  # 创建锁


class IncrThread(threading.Thread):
    def run(self):
        global var
        lock.acquire()  # 获取锁
        print 'before,var is ', var
        var += 1
        print 'after,var is ', var
        lock.release()  # 释放锁


def use_incre_thread():
    threads = []
    for i in range(50):
        t = IncrThread()
        threads.append(t)
        t.start()
    for t in threads:
        t.join()
    print 'After 10 times,var is ', var


if __name__ == '__main__':
    use_incre_thread()
```



## 可重入锁

为了支持在同一线程中多次请求同一资源，`python` 提供了可重入锁（`RLock`）。`RLock`内部维护着一个`Lock`和一个`counter`变量，`counter`记录了`acquire`的次数，从而使得资源可以被多次`require`。直到一个线程所有的`acquire`都被`release`，其他的线程才能获得资源。



```python
mutex = threading.RLock()


class MyThread(threading.Thread):
    def run(self):
        if mutex.acquire(1):
            print 'threading gte mutex:', self.name
            time.sleep(1)
            mutex.acquire()
            mutex.release()
            mutex.release()


def main():
    print 'start main threading:'
    threads = [MyThread() for i in range(2)]
    for t in threads:
        t.start()
    for t in threads:
        t.join()
    print 'end main threading.'
```



## 后台线程

使用多线程默认情况下，当主线程退出之后，即使子线程没有 `join`，子线程也依然会继续执行。如果希望主线程退出后，其子线程也退出而不再执行，则需要设置子线程为后台线程。`python`提供了`setDaemon`方法，将子线程与主线程进行绑定，当主线程退出时子线程的生命也随之结束。

```python
class MyThread(threading.Thread):
    def run(self):
        wait_time = random.randrange(1, 10)
        print 'thread %s will wait %s s' %(self.name, wait_time)
        time.sleep(wait_time)
        time.sleep(30)
        print 'thread %s finished.' % self.name
 
 
def main():
    print 'start thread:'
    for i in range(3):
        t = MyThread()
        t.setDaemon(1)
        t.start()
    print 'end thread.'
 
if __name__ == '__main__':
    main()
```

运行结果：

```python
start thread:
thread Thread-1 will wait 9 s
thread Thread-2 will wait 1 s
thread Thread-3 will wait 7 s
end thread.
```

