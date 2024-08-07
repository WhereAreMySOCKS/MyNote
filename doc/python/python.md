# asyncio
## 前置内容
### 协程（Coroutine）
协程是 Python 中的一种特殊的生成器，可以挂起和恢复执行。可以使用 async def 来定义协程函数，并且协程函数在调用时返回一个协程对象。例如：
```python
async def my_coroutine():
    await asyncio.sleep(1)
    return "Done"
```
### 任务（Task）
任务是对协程的进一步封装，用于调度和管理协程的执行。任务是在事件循环中执行协程的更高层次的抽象。可以使用 asyncio.create_task() 或 loop.create_task() 将协程转换为任务。例如：
```python
task = asyncio.create_task(my_coroutine())
```
## 正文
asyncio 是python3.7之后引入的新语法。由于GIL的限制，python中无法利用多核并行计算；因此在python中多个任务同时创建后，需要进行一定的调度来提升任务执行效率。<br><b>asyncio的主要工作就是负责在单进程环境下调度任务</b>,每当一个任务结束后，会通过  系统，并执行下一任务。

## Example 1
```python
import asyncio

async def main():
        print('hello')
        await asynico.sleep(1)
        print('world')
        await asynico.sleep(2)

coro = main()
```
上述代码中，使用async关键字定义方法main为一个coroutine，main方法被直接调用后，会返回一个coroutine对象，而不会执行真正的方法逻辑。

想要调用被async修饰的方法，有两种方式：
- 进入async模式，让event loop控制程序。


- 用await将协程封装为task，并由event loop决定何时执行该task。

```python
import asyncio

asyncio.run(main())  # 建立event loop 并且将main方法转换成task
```
通过asyncio.run方法将main转换为task后，event loop中此时只有一个task (即main方法)。此时main方法开始执行，直到遇到关键字await。

await将asynico.sleep(1)从coroutine转换成了task，存放在event loop中，并且之前的task(main方法)必须等到该task完成后才能继续执行，后续的await也同理，最终程序执行过程大概为：
- 输出hello，等待一秒
- 输出world，等待两秒

在上面这个例子中，两个task是串行执行的。如何让两个task个同时等待呢？

## Example 2
```python
import asyncio

async def say_after(delay,what):
        await asyncio.sleep(delay)
        print(what)

async def main():
        print('hello')
        task1 = asynico.create_task(say_after(1,'hello')) # 创造task
        print('world')
         task2 = asynico.create_task(say_after(2,'world'))

        await task1
        await task2     


asyncio.run(main())
```

运行该程序，发现耗时两秒便输出了hello world。通过asynico.create_task创建了task，并交给event loop进行调度。event loop发现task1阻塞后，会自动调用其他任务，因此只需要两秒就完成了输出。

当task数量很大时，可以通过gather方法将所有task注册到event loop上
```python
import asyncio

async def say_after(delay,what):
        await asyncio.sleep(delay)
        print(what)

async def main():
        print('hello')
        task1 = asynico.create_task(say_after(1,'hello')) # 创造task
        print('world')
         task2 = asynico.create_task(say_after(2,'world'))

        asyncio.gather(task1,task2) # 注册task

asyncio.run(main())
```

也可以简化创造task这一步骤，直接使用gather进行注册

```python
import asyncio

async def say_after(delay,what):
        await asyncio.sleep(delay)
        print(what)

async def main():
        asyncio.gather(say_after(1,'hello'),
                        say_after(2,'world')) # 注册task

asyncio.run(main())
```
