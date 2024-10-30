
## 前言


线程是操作系统能够进行运算调度的最小单位，它被包含在进程之中，是进程中的实际运作单位。由于CPython的GIL限制，多线程实际为单线程，大多只用来处理IO密集型任务。


Python一般用标准库`threading`来进行多线程编程。


## 基本使用


* 方式1，创建`threading.Thread`类的示例



```
import threading
import time

def task1(counter: int):
    print(f"thread: {threading.current_thread().name}, args: {counter}, start time: {time.strftime('%F %T')}")
    num = counter
    while num > 0:
        time.sleep(3)
        num -= 1
    print(f"thread: {threading.current_thread().name}, args: {counter}, end time: {time.strftime('%F %T')}")

if __name__ == "__main__":
    print(f"main thread: {threading.current_thread().name}, start time: {time.strftime('%F %T')}")
    # 创建三个线程
    t1 = threading.Thread(target=task1, args=(7,))
    t2 = threading.Thread(target=task1, args=(5,))
    t3 = threading.Thread(target=task1, args=(3,))

    # 启动线程
    t1.start()
    t2.start()
    t3.start()

    # join() 用于阻塞主线程, 等待子线程执行完毕
    t1.join()
    t2.join()
    t3.join()
    print(f"main thread: {threading.current_thread().name}, end time: {time.strftime('%F %T')}")

```

执行输出示例



```
main thread: MainThread, start time: 2024-10-26 12:42:37
thread: Thread-1 (task1), args: 7, start time: 2024-10-26 12:42:37
thread: Thread-2 (task1), args: 5, start time: 2024-10-26 12:42:37
thread: Thread-3 (task1), args: 3, start time: 2024-10-26 12:42:37
thread: Thread-3 (task1), args: 3, end time: 2024-10-26 12:42:46
thread: Thread-2 (task1), args: 5, end time: 2024-10-26 12:42:52
thread: Thread-1 (task1), args: 7, end time: 2024-10-26 12:42:58
main thread: MainThread, end time: 2024-10-26 12:42:58

```

* 方式2，继承`threading.Thread`类，重写`run()`和`__init__()`方法



```
import threading
import time

class MyThread(threading.Thread):
    def __init__(self, counter: int):
        super().__init__()
        self.counter = counter

    def run(self):
        print(f"thread: {threading.current_thread().name}, args: {self.counter}, start time: {time.strftime('%F %T')}")
        num = self.counter
        while num > 0:
            time.sleep(3)
            num -= 1
        print(f"thread: {threading.current_thread().name}, args: {self.counter}, end time: {time.strftime('%F %T')}")

if __name__ == "__main__":
    print(f"main thread: {threading.current_thread().name}, start time: {time.strftime('%F %T')}")
    # 创建三个线程
    t1 = MyThread(7)
    t2 = MyThread(5)
    t3 = MyThread(3)

    # 启动线程
    t1.start()
    t2.start()
    t3.start()

    # join() 用于阻塞主线程, 等待子线程执行完毕
    t1.join()
    t2.join()
    t3.join()
    print(f"main thread: {threading.current_thread().name}, end time: {time.strftime('%F %T')}")

```

继承`threading.Thread`类也可以写成这样，调用外部函数。



```
import threading
import time

def task1(counter: int):
    print(f"thread: {threading.current_thread().name}, args: {counter}, start time: {time.strftime('%F %T')}")
    num = counter
    while num > 0:
        time.sleep(3)
        num -= 1
    print(f"thread: {threading.current_thread().name}, args: {counter}, end time: {time.strftime('%F %T')}")

class MyThread(threading.Thread):
    def __init__(self, target, args: tuple):
        super().__init__()
        self.target = target
        self.args = args
    
    def run(self):
        self.target(*self.args)

if __name__ == "__main__":
    print(f"main thread: {threading.current_thread().name}, start time: {time.strftime('%F %T')}")
    # 创建三个线程
    t1 = MyThread(target=task1, args=(7,))
    t2 = MyThread(target=task1, args=(5,))
    t3 = MyThread(target=task1, args=(3,))

    # 启动线程
    t1.start()
    t2.start()
    t3.start()

    # join() 用于阻塞主线程, 等待子线程执行完毕
    t1.join()
    t2.join()
    t3.join()
    print(f"main thread: {threading.current_thread().name}, end time: {time.strftime('%F %T')}")

```

## 多线程同步


如果多个线程共同对某个数据修改，则可能出现不可预料的后果，这时候就需要某些同步机制。比如如下代码，结果是随机的（个人电脑用python3\.13实测结果都是0，而低版本的python3\.6运行结果的确是随机的）



```
import threading
import time

num = 0

def task1(counter: int):
    print(f"thread: {threading.current_thread().name}, args: {counter}, start time: {time.strftime('%F %T')}")
    global num
    for _ in range(100000000):
        num = num + counter
        num = num - counter
    print(f"thread: {threading.current_thread().name}, args: {counter}, end time: {time.strftime('%F %T')}")

if __name__ == "__main__":
    print(f"main thread: {threading.current_thread().name}, start time: {time.strftime('%F %T')}")
    # 创建三个线程
    t1 = threading.Thread(target=task1, args=(7,))
    t2 = threading.Thread(target=task1, args=(5,))
    t3 = threading.Thread(target=task1, args=(3,))
    t4 = threading.Thread(target=task1, args=(6,))
    t5 = threading.Thread(target=task1, args=(8,))

    # 启动线程
    t1.start()
    t2.start()
    t3.start()
    t4.start()
    t5.start()

    # join() 用于阻塞主线程, 等待子线程执行完毕
    t1.join()
    t2.join()
    t3.join()
    t4.join()
    t5.join()
    
    print(f"num: {num}")
    print(f"main thread: {threading.current_thread().name}, end time: {time.strftime('%F %T')}")

```

### Lock\-锁


使用互斥锁可以在一个线程访问数据时，拒绝其它线程访问，直到解锁。`threading.Thread`中的`Lock()`和`Rlock()`可以提供锁功能。



```
import threading
import time

num = 0

mutex = threading.Lock()

def task1(counter: int):
    print(f"thread: {threading.current_thread().name}, args: {counter}, start time: {time.strftime('%F %T')}")
    global num
    mutex.acquire()
    for _ in range(100000):
        num = num + counter
        num = num - counter
    mutex.release()
    print(f"thread: {threading.current_thread().name}, args: {counter}, end time: {time.strftime('%F %T')}")

if __name__ == "__main__":
    print(f"main thread: {threading.current_thread().name}, start time: {time.strftime('%F %T')}")
    # 创建三个线程
    t1 = threading.Thread(target=task1, args=(7,))
    t2 = threading.Thread(target=task1, args=(5,))
    t3 = threading.Thread(target=task1, args=(3,))

    # 启动线程
    t1.start()
    t2.start()
    t3.start()

    # join() 用于阻塞主线程, 等待子线程执行完毕
    t1.join()
    t2.join()
    t3.join()
    
    print(f"num: {num}")
    print(f"main thread: {threading.current_thread().name}, end time: {time.strftime('%F %T')}")

```

### Semaphore\-信号量


互斥锁是只允许一个线程访问共享数据，而信号量是同时允许一定数量的线程访问共享数据。比如银行有5个窗口，允许同时有5个人办理业务，后面的人只能等待，待柜台有空闲才可以进入。



```
import threading
import time
from random import randint

semaphore = threading.BoundedSemaphore(5)

def business(name: str):
    semaphore.acquire()
    print(f"{time.strftime('%F %T')} {name} is handling")
    time.sleep(randint(3, 10))
    print(f"{time.strftime('%F %T')} {name} is done")
    semaphore.release()

if __name__ == "__main__":
    print(f"main thread: {threading.current_thread().name}, start time: {time.strftime('%F %T')}")
    threads = []
    for i in range(10):
        t = threading.Thread(target=business, args=(f"thread-{i}",))
        threads.append(t)

    for t in threads:
        t.start()

    for t in threads:
        t.join()
    
    print(f"main thread: {threading.current_thread().name}, end time: {time.strftime('%F %T')}")

```

执行输出



```
main thread: MainThread, start time: 2024-10-26 17:40:10
2024-10-26 17:40:10 thread-0 is handling
2024-10-26 17:40:10 thread-1 is handling
2024-10-26 17:40:10 thread-2 is handling
2024-10-26 17:40:10 thread-3 is handling
2024-10-26 17:40:10 thread-4 is handling
2024-10-26 17:40:15 thread-2 is done
2024-10-26 17:40:15 thread-5 is handling
2024-10-26 17:40:16 thread-0 is done
2024-10-26 17:40:16 thread-6 is handling
2024-10-26 17:40:19 thread-3 is done
2024-10-26 17:40:19 thread-4 is done
2024-10-26 17:40:19 thread-7 is handling
2024-10-26 17:40:19 thread-8 is handling
2024-10-26 17:40:20 thread-1 is done
2024-10-26 17:40:20 thread-9 is handling
2024-10-26 17:40:21 thread-6 is done
2024-10-26 17:40:23 thread-7 is done
2024-10-26 17:40:24 thread-5 is done
2024-10-26 17:40:24 thread-8 is done
2024-10-26 17:40:30 thread-9 is done
main thread: MainThread, end time: 2024-10-26 17:40:30

```

### Condition\-条件对象


`Condition`对象能让一个线程A停下来，等待其他线程，其他线程通知后线程A继续运行。



```
import threading
import time
import random

class Employee(threading.Thread):
    def __init__(self, username: str, cond: threading.Condition):
        self.username = username
        self.cond = cond
        super().__init__()

    def run(self):
        with self.cond:
            print(f"{time.strftime('%F %T')} {self.username} 到达公司")
            self.cond.wait()  # 等待通知
            print(f"{time.strftime('%F %T')} {self.username} 开始工作")
            time.sleep(random.randint(1, 5))
            print(f"{time.strftime('%F %T')} {self.username} 工作完成")

class Boss(threading.Thread):
    def __init__(self, username: str, cond: threading.Condition):
        self.username = username
        self.cond = cond
        super().__init__()

    def run(self):
        with self.cond:
            print(f"{time.strftime('%F %T')} {self.username} 发出通知")
            self.cond.notify_all()  # 通知所有线程
        time.sleep(2)

if __name__ == "__main__":
    cond = threading.Condition()
    boss = Boss("老王", cond)
    
    employees = []
    for i in range(5):
        employees.append(Employee(f"员工{i}", cond))

    for employee in employees:
        employee.start()
    boss.start()

    boss.join()
    for employee in employees:
        employee.join()


```

执行输出



```
2024-10-26 21:16:20 员工0 到达公司
2024-10-26 21:16:20 员工1 到达公司
2024-10-26 21:16:20 员工2 到达公司
2024-10-26 21:16:20 员工3 到达公司
2024-10-26 21:16:20 员工4 到达公司
2024-10-26 21:16:20 老王 发出通知
2024-10-26 21:16:20 员工4 开始工作
2024-10-26 21:16:23 员工4 工作完成
2024-10-26 21:16:23 员工1 开始工作
2024-10-26 21:16:28 员工1 工作完成
2024-10-26 21:16:28 员工2 开始工作
2024-10-26 21:16:30 员工2 工作完成
2024-10-26 21:16:30 员工0 开始工作
2024-10-26 21:16:31 员工0 工作完成
2024-10-26 21:16:31 员工3 开始工作
2024-10-26 21:16:32 员工3 工作完成

```

### Event\-事件


在 Python 的 `threading` 模块中，`Event` 是一个线程同步原语，用于在多个线程之间进行简单的通信。`Event` 对象维护一个内部标志，线程可以使用 `wait()` 方法阻塞，直到另一个线程调用 `set()` 方法将标志设置为 `True`。一旦标志被设置为 `True`，所有等待的线程将被唤醒并继续执行。


`Event` 的主要方法


1. **`set()`**：将事件的内部标志设置为 `True`，并唤醒所有等待的线程。
2. **`clear()`**：将事件的内部标志设置为 `False`。
3. **`is_set()`**：返回事件的内部标志是否为 `True`。
4. **`wait(timeout=None)`**：如果事件的内部标志为 `False`，则阻塞当前线程，直到标志被设置为 `True` 或超时（如果指定了 `timeout`）。



```
import threading
import time
import random

class Employee(threading.Thread):
    def __init__(self, username: str, cond: threading.Event):
        self.username = username
        self.cond = cond
        super().__init__()

    def run(self):
        print(f"{time.strftime('%F %T')} {self.username} 到达公司")
        self.cond.wait()  # 等待事件标志为True
        print(f"{time.strftime('%F %T')} {self.username} 开始工作")
        time.sleep(random.randint(1, 5))
        print(f"{time.strftime('%F %T')} {self.username} 工作完成")

class Boss(threading.Thread):
    def __init__(self, username: str, cond: threading.Event):
        self.username = username
        self.cond = cond
        super().__init__()

    def run(self):
        print(f"{time.strftime('%F %T')} {self.username} 发出通知")
        self.cond.set()

if __name__ == "__main__":
    cond = threading.Event()
    boss = Boss("老王", cond)
    
    employees = []
    for i in range(5):
        employees.append(Employee(f"员工{i}", cond))

    for employee in employees:
        employee.start()
    boss.start()

    boss.join()
    for employee in employees:
        employee.join()


```

执行输出



```
2024-10-26 21:22:28 员工0 到达公司
2024-10-26 21:22:28 员工1 到达公司
2024-10-26 21:22:28 员工2 到达公司
2024-10-26 21:22:28 员工3 到达公司
2024-10-26 21:22:28 员工4 到达公司
2024-10-26 21:22:28 老王 发出通知
2024-10-26 21:22:28 员工0 开始工作
2024-10-26 21:22:28 员工1 开始工作
2024-10-26 21:22:28 员工3 开始工作
2024-10-26 21:22:28 员工4 开始工作
2024-10-26 21:22:28 员工2 开始工作
2024-10-26 21:22:30 员工3 工作完成
2024-10-26 21:22:31 员工4 工作完成
2024-10-26 21:22:31 员工2 工作完成
2024-10-26 21:22:32 员工0 工作完成
2024-10-26 21:22:32 员工1 工作完成

```

## 使用队列


Python的`queue`模块提供同步、线程安全的队列类。以下示例为使用queue实现的生产消费者模型



```
import threading
import time
import random
import queue


class Producer(threading.Thread):
    """多线程生产者类."""

    def __init__(
        self, tname: str, channel: queue.Queue, done: threading.Event
    ):
        self.tname = tname
        self.channel = channel
        self.done = done
        super().__init__()

    def run(self) -> None:
        """Method representing the thread's activity."""

        while True:
            if self.done.is_set():
                print(
                    f"{time.strftime('%F %T')} {self.tname} 收到停止信号事件"
                )
                break
            if self.channel.full():
                print(
                    f"{time.strftime('%F %T')} {self.tname} report: 队列已满, 全部停止生产"
                )
                self.done.set()
            else:
                num = random.randint(100, 1000)
                self.channel.put(f"{self.tname}-{num}")
                print(
                    f"{time.strftime('%F %T')} {self.tname} 生成数据 {num}, queue size: {self.channel.qsize()}"
                )
                time.sleep(random.randint(1, 5))


class Consumer(threading.Thread):
    """多线程消费者类."""

    def __init__(
        self, tname: str, channel: queue.Queue, done: threading.Event
    ):
        self.tname = tname
        self.channel = channel
        self.done = done
        self.counter = 0
        super().__init__()

    def run(self) -> None:
        """Method representing the thread's activity."""
        while True:
            if self.done.is_set():
                print(
                    f"{time.strftime('%F %T')} {self.tname} 收到停止信号事件"
                )
                break
            if self.counter >= 3:
                print(
                    f"{time.strftime('%F %T')} {self.tname} report: 全部停止消费"
                )
                self.done.set()
                continue
            if self.channel.empty():
                print(
                    f"{time.strftime('%F %T')} {self.tname} report: 队列为空, counter: {self.counter}"
                )
                self.counter += 1
                time.sleep(1)
                continue
            else:
                data = self.channel.get()
                print(
                    f"{time.strftime('%F %T')} {self.tname} 消费数据 {data}, queue size: {self.channel.qsize()}"
                )
                time.sleep(random.randint(1, 5))
                self.counter = 0


if __name__ == "__main__":
    done_p = threading.Event()
    done_c = threading.Event()
    channel = queue.Queue(30)
    threads_producer = []
    threads_consumer = []

    for i in range(8):
        threads_producer.append(Producer(f"producer-{i}", channel, done_p))

    for i in range(6):
        threads_consumer.append(Consumer(f"consumer-{i}", channel, done_c))

    for t in threads_producer:
        t.start()

    for t in threads_consumer:
        t.start()

    for t in threads_producer:
        t.join()

    for t in threads_consumer:
        t.join()


```

## 线程池


在面向对象编程中，创建和销毁对象是很费时间的，因为创建一个对象要获取内存资源或其他更多资源。在多线程程序中，生成一个新线程之后销毁，然后再创建一个，这种方式就很低效。池化多线程，也就是线程池就为此而生。


将任务添加到线程池中，线程池会自动指定一个空闲的线程去执行任务，当超过最大线程数时，任务需要等待有新的空闲线程才会被执行。Python一般可以使用`multiprocessing`模块中的`Pool`来创建线程池。



```
import time

from multiprocessing.dummy import Pool as ThreadPool

def foo(n):
    time.sleep(2)


if __name__ == "__main__":
    start = time.time()
    for n in range(5):
        foo(n)
    print("single thread time: ", time.time() - start)

    start = time.time()
    t_pool = ThreadPool(processes=5)  # 创建线程池, 指定池中的线程数为5(默认为CPU数)
    rst = t_pool.map(foo, range(5))  # 使用map为每个元素应用到foo函数
    t_pool.close()  # 阻止任何新的任务提交到线程池
    t_pool.join()  # 等待所有已提交的任务完成
    print("thread pool time: ", time.time() - start)

```

## 线程池执行器


python的内置模块`concurrent.futures`提供了`ThreadPoolExecutor`类。这个类结合了线程和队列的优势，可以用来平行执行任务。



```
import time
from random import randint
from concurrent.futures import ThreadPoolExecutor

def foo() -> None:
    time.sleep(2)
    return randint(1,100)

if __name__ == "__main__":
    start = time.time()
    futures = []
    with ThreadPoolExecutor(max_workers=5) as executor:
        for n in range(10):
            futures.append(executor.submit(foo))  # Fan out
            
    for future in futures:  # Fan in
        print(future.result())
    print("thread pool executor time: ", time.time() - start)

```

执行输出



```
44
19
86
48
35
74
59
99
58
53
thread pool executor time:  4.001955032348633

```

`ThreadPoolExecutor`类的最大优点在于：如果调用者通过`submit`方法把某项任务交给它执行，那么会获得一个与该任务相对应的`Future`实例，当调用者在这个实例上通过`result`方法获取执行结果时，`ThreadPoolExecutor`会把它在执行任务的过程中所遇到的异常自动抛给调用者。而`ThreadPoolExecutor`类的缺点是IO并行能力不高，即便把`max_worker`设为100，也无法高效处理任务。更高需求的IO任务可以考虑换异步协程方案。


## 参考


* 郑征《Python自动化运维快速入门》清华大学出版社
* Brett Slatkin《Effective Python》(2nd) 机械工业出版社


 本博客参考[slower加速器](https://chundaotian.com)。转载请注明出处！
