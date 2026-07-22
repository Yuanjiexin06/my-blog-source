---
title: '抽象数据类型'
date: 2026-7-22
updated: 2026-7-22
tags: 数据结构与算法
categories: 笔记
description: 抽象数据类型的笔记
---

#  初识并发

## 线程基础

python的并发编程依赖于threading库，主要用于创建多进程or多线程。

```python
#导入threading库
import threading

import time

url_list= [  
    ("赵本山电影.mp4", "https://www.douyin.com/jingxuan?modal_id=7662294912322260274"),  
    (" 回春丹.mp4", "https://www.douyin.com/jingxuan?modal_id=7653445918758391103"),  
    ("暑假工.mp4", "https://www.douyin.com/jingxuan?modal_id=7657573507723374065")  
]

# print(time.time())  
# for file_name, url in url_list:  
#     res = requests.get(url)  
#     with open(file_name, "wb") as f:  
#         f.write(res.content)  
# print(time.time())

def task(filename, url):  
    res = requests.get(url)  
    with open(filename, "wb") as f:  
        f.write(res.content)  
    print(time.time())  

#threading.Threa用来创建新的线程。target指定线程要执行的函数，args指定函数的参数。
print(time.time())  
for name, url in url_list:  
    t = threading.Thread(target=task, args=(name, url))  
    t.start()

```

这段代码就是创建了一个线程，每个线程都会执行`print(time.time())`，最终会打印四次时间戳。可以清晰的看出来每次线程花费的时间。与注释掉的代码可以形成对比，注释的代码只有两个时间戳，分别是开始和结束的，可以判断程序从开始执行到程序结束的时间。

```python
import threading  

aim = 10000000
number = 0  
def _add(count):
	global number  
	for i in range(count):  
		number += 1
		
t = threading.Thread(target = _add, args=(aim,))
t.start()

print(number)
```

线程分为主线程和子线程，当程序运行起来的时候，会先创建一个python的进程，然后在进程中自动生成一个主线程，主线程是由上往下一行一行的执行代码，.当他执行到`t = Thread(target = _add, args = threading(aim,))`的时候，它会在同一个进程中创建一个子线程，子线程会执行`target`的函数，也就是循环`number += 1` `count`次，也就是传进去的`aim`。但是由于主线程和子线程是分开的，主线程会持续执行到`print(number)`的时候就会输出`number`，但是此时的值是都是不确定，因为线程不一样，但是程序是否退出还需要看进程`t`是否为守护进程。

但是使用`join()`函数可以解决这个问题。

```python
import threading  

aim = 10000000
number = 0  
def _add(count):
	global number  
	for i in range(count):  
		number += 1
		
t = threading.Thread(target = _add, args=(aim,))
t.start()

t.join     #等待子程序执行完毕。

print(number)
```

在执行`print(number)`之前先执行了`t.join()`函数，会让主线程执行到这行代码的时候等待子程序执行完毕才会继续进行，这样下来，在最后输出的`number`就一定会是`aim`的值。

## 守护进程

```python
import threading  
  
aim = 10000000  
number = 0  
  
  
def _add(count):  
    global number  
    for i in range(count):  
        number += 1  
  
  
t = threading.Thread(target=_add, args=(aim,))  
t.setDaemon(True) #设置守护进程。  
t.start()  
   
  
print(number)
```

使用`setDaemon(bool)`或者`t.daemon = False/True`可以设置守护进程或者普通进程。

```python
import threading
import time

number = 0

def add():
    global number
    for _ in range(10000000):
        number += 1
    print("子线程结束", number)

t = threading.Thread(target=add)
t.daemon = False      # 改成True试试
t.start()

print("主线程打印：", number)
```

- `daemon = False`：程序会一直运行到 `"子线程结束"` 打印出来。
- `daemon = True`：如果主线程很快结束，程序会直接退出，可能永远看不到 `"子线程结束"`。

当`daemon = False`的时候，当主线程执行完之后，其实后台的程序并没有退出，因为此时的子线程是普通进程，不会跟随主线程的结束而结束。相反，`daemon = True`的时候，主线程执行完会带走子线程。

## 线程安全--锁

线程安全主要依赖于线程锁。线程锁有两种类型：
- `.RLock()`：递归锁，简而言之就是可以实现锁的嵌套。
- `.Lock()`：非递归锁，不能完成锁的嵌套，如果强行嵌套就会出现死锁，导致程序崩溃。
```python
import threading  
import time  
  
lock_object = threading.RLock()  
  
lock = 10000000  
  
number = 0  
  
def _add(count):  
    global number  
    for i in range(count):  
        lock_object.acquire()  # 获取锁/加锁  
        global number  
        number += 1  
        lock_object.release()  # 释放锁/解锁  
  
def _sub(count):  
    global number  
    for i in range(count):  
        lock_object.acquire()  # 获取锁/加锁  
        global number  
        number -= 1  
        lock_object.release()  # 释放锁/解锁  
  
t1 = threading.Thread(target=_add, args=(lock,))  
t2 = threading.Thread(target=_sub, args=(lock,))  
print(time.time())  
t1.start()  
t2.start()  
  
t1.join()  
t2.join()  
  
print(number)
```

线程锁的作用就是保护线程正常运行。上段代码中有两个线程，他们分别实现自增和自减函数，但是线程的运行依赖于CPU的自行的调度，所以会出现一种情况：

*假设经过一段时间的运行，`number`在某个时间点的值为`100`。此时CPU在计算自增线程，但是可能没有完成自增的过程，就跳转到执行自减的过程了，然后`number`变成了`99`，然后CPU又跳回去执行自增线程，但是在自增线程中的`number`还卡在刚刚的状态，所以`number`变成了`101`，因为多个线程共享同一块内存，因此访问同一个变量时可能产生竞争。例如 `number += 1` 并不是原子操作，它包含读取、计算、写回三个步骤。如果CPU在线程执行过程中切换，可能导致多个线程读取到相同的旧值，最终覆盖其他线程的修改，使结果错误。。*

这就导致了结果变得很混乱。
另外举个例子：

```
时间        A线程              B线程

t1       读取number
         得到100

t2                         读取number
                           得到100

t3       计算100+1

t4                         计算100-1

t5       写入101

t6                         写入99
```

最后：
`number = 99`

但实际上：
`100 + 1 - 1 = 100`

这叫数据竞争。

加入锁情况就不一样了，在代码中，两个进程使用的是同一把锁，假设当自增线程先申请了锁，然后开始执行自增函数，此时，如果自减函数想要执行需要先申请锁，但是锁已经被自增线程拿走了，只有当自增线程释放锁，自减函数才能申请锁再执行代码。

## 线程池

线程池需要`from concurrent.futures import ThreadPoolExecutor`。

```python
import time  
from concurrent.futures import ThreadPoolExecutor

def task(vedio_url, num):  
    print("开始执行任务", vedio_url)
    
pool = ThreadPoolExecutor(10)   # 创建一个线程池，最大线程数为10
url_list = ["www.xxxx-{}.com".format(i) for i in range(300)]  # 模拟20个视频url  
  
for url in url_list:  
    pool.submit(task, url, 1)  # 提交任务到线程池，task为要执行的函数，url为传递给函数的参数  
  
pool.shutdown(True)  # 关闭线程池，等待所有任务完成  
print("主线程执行完毕")
```

循环再主线程中进行，而`submit`逐步将循环传进来的`url`提交给线程池，



