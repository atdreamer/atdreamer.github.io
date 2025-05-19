关于异步IO这个概念，可能有些伙伴不是非常明白，那就先来看看异步IO是怎么回事儿。
为了大家能够更形象得理解这个概念，我们拿放羊来打个比方：

下载请求开始，就是放羊出去吃草；
下载任务完成，就是羊吃饱回羊圈。
同步放羊的过程就是这样的：
羊倌儿小同要放100只羊，他就先放一只羊出去吃草，等羊吃饱了回来在放第二只羊，等第二只羊吃饱了回来再放第三只羊出去吃草……这样放羊的羊倌儿实在是……

再看看异步放羊的过程：
羊倌儿小异也要放100只羊，他观察后发现，小同放羊的方法比较笨，他觉得草地一下能容下10只羊（带宽）吃草，所以它就一次放出去10只羊等它们回来，然后他还可以给羊剪剪羊毛。有的羊吃得快回来的早，他就把羊关到羊圈接着就再放出去几只，尽量保证草地上都有10只羊在吃草。

很明显，异步放羊的效率高多了。同样的，网络世界里也是异步的效率高。那么简单的异步程序怎么实现呢，请看接下来的示例源代码：
`python
"""
desc:测试讲解python[版本3.8***]的异步[任务调度](https://zhida.zhihu.com/search?content_id=175153659&content_type=Article&match_order=1&q=%E4%BB%BB%E5%8A%A1%E8%B0%83%E5%BA%A6&zhida_source=entity)执行
欢迎添加qq群:492656241
针对内容:python、爬虫场景
"""
import asyncio
import time

task_count = 0  # 系统支持最大任务并发数
task_list = [{'name': 'task a', 'time': 1},
             {'name': 'task b', 'time': 2},
             {'name': 'task c', 'time': 3},
             {'name': 'task d', 'time': 4},
             {'name': 'task e', 'time': 5},
             {'name': 'task f', 'time': 6},
             {'name': 'task g', 'time': 7},
             {'name': 'task h', 'time': 8},
             {'name': 'task i', 'time': 9},
             {'name': 'task j', 'time': 10},
             {'name': 'task k', 'time': 11},
             {'name': 'task l', 'time': 12}]


async def delay_sleep(sleep_time):
    await asyncio.sleep(sleep_time)
    return sleep_time


async def request(unit):
    print("开始执行任务单元:", unit['name'])
    global task_count
    sleep_time = await delay_sleep(unit['time'])
    task_count -= 1  # 当前任务单元执行完成 系统正在执行任务数 -1
    print("任务单元执行完成:", unit['name'], sleep_time)


async def init():
    global task_count
    temp = task_list[:5]
    for each in temp:
        task_list.remove(each)  # 删除任务单元
        asyncio.create_task(request(each))
        task_count += 1


async def main():
    global task_count
    await init()  # 创建初始化任务 执行并发
    while True:
        print("系统当前正在执行任务数量:{}".format(task_count))
        if task_count < 5:
            if task_list:
                each = task_list[0]
                del task_list[0]
                print("添加新的任务单元:{}".format(each))
                asyncio.create_task(request(each))
                task_count += 1
            else:
                if not task_count:
                    print("所有任务单元执行完成...终止程序")
                    break
                else:
                    #  休眠等待所有任务执行完成
                    await asyncio.sleep(0.5)
        else:
            await asyncio.sleep(0.5)


if __name__ == '__main__':
    start_time = time.time()
    asyncio.run(main())
    print("总共消耗时间:{}".format(time.time() - start_time))
`
以上代码主要通过sleep来模拟在耗时较长的场景中怎么通过asyncio来实现异步发起任务请求以及并发执行。逻辑说明：

1、首先再次申明python的版本为3.8，因为不同的版本在python中是存在写法不同的，但就目前的版本来看，3.8版本写异步因该是最简单的了。

2、程序执行， 首先通过init()方法创建5个（系统最大并发支持任务数）协程任务。注意，在此只是创建了5个协程任务单元而已，但是还并没有执行。

3、进入while True无限循环，判断当前系统已经存在的任务数量，若大于等于5则进入else逻辑，执行await asyncio.sleep(5),当执行到此时，程序将会通过await让出cpu当前的控制权，触发程序去执行init()函数创建的5个协程任务单元的启动执行。

4、每当上述触发执行的任务单元执行完成时，系统任务数量将会减1，如此将进入到当前系统执行任务数小于最大并发数的逻辑，即创建新的协程任务单元。

5、当循环把所有任务都成功创建完成之后，系统将循环判断task_count即当前任务数是否为0(全部执行完成)，如果全部执行完成则break终止，否则sleep(0.5)等待未完成的任务执行完成。

上述就是通过python来进行异步任务执行的基本逻辑了，大家颗针对自己的任务场景进行相应的调整切换即可。