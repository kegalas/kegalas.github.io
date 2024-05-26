---
title: "C++并发支持库用法速查"
date: 2024-05-21T23:42:42+08:00
draft: false
tags:
  - C++
  - 水文
  - 并发
categories: 其他计算机科学
mathjax: true
markup: goldmark
---

# thread

C++启动一个新线程，只要构造一个`std::thread`对象就可以了。使用`join()`方法可以等待线程汇入，使用`detach()`可以不等待。这里`join()`只能使用一次，可以用`joinable()`查询是否可以`join`

使用方法如下

```cpp
std::thread t(f, 1, "123", 0.0);
```

其中第一个参数是一个函数对象，其后面的都是函数参数（应当使得`f(args)`的调用合法）。

注意这里的参数，会被`decay-copy`，也就是说，如果`f`的形参是`f(A const & a)`之类的，用thread的构造函数传入参数的话，仍然会产生拷贝，并且不会引用原来的变量。此时需要使用`std::ref`来提供包装。

注意`thread`的函数是不能有返回值的，只能是`void`。返回值可以写在参数中，通过引用或者指针来进行传递。一般会调用`t.join()`汇入之后再去读取这个返回值。

可以有返回值的线程使用`std::async`，见后。

# mutex

C++提供的互斥锁。使用方法：

```cpp
std::mutex m;
m.lock();
//...
m.unlock();
```

但是我们一般不会直接使用`std::mutex`提供的方法，而是会使用`lock_guard`，其提供了RAII的包装，在析构时自动解锁。

```cpp
std::mutex m;
std::lock_guard<std::mutex> lock(m); // C++17之后也可以省略模板参数
```

可以使用`std::lock`同时锁住两个（多个）锁，避免条件竞争。

```cpp
std::mutex m1, m2;
std::lock(m1, m2);
std::lock_guard<std::mutex> lock_a(m1,std::adopt_lock);
std::lock_guard<std::mutex> lock_b(m2,std::adopt_lock);
```

如果是`C++17`则可以使用`std::scoped_lock`，就不需要再来两个`lock_guard`了

```cpp
std::scoped_lock lock(e1.m, e2.m);
```

`unique_lock`则会更灵活一点，`lock_guard`只能在析构时解锁，而`unique_lock`不仅可以在构造的时候上锁、析构的时候解锁，还可以提前解锁和重新上锁，甚至还可以推迟上锁。

```cpp
std::mutex m1;
std::unique_lock<std::mutex> lk(m1); // 构造时上锁，析构时解锁

std::mutex m2;
std::unique_lock<std::mutex> lk(m2);
lk.unlock();
lk.lock(); // 可以其他解锁和重新上锁

std::unique_lock<std::mutex> lk_a(m3, std::defer_lock);
std::unique_lock<std::mutex> lk_b(m4, std::defer_lock);
std::lock(lk_a, lk_b); // 和之前的std::lock作用相同
```

# 避免死锁的指导

1. 避免嵌套锁。持有一个锁，就不要再去尝试持有第二个。当需要持有多个锁时，使用`std::lock`
2. 避免在持有锁时调用其他函数。因为你不知道这个其他函数会不会也尝试获取锁。
3. 使用固定顺序获取锁。例如规定所有函数都要先获取A锁，再获取B锁。当`std::lock`不适用时应该考虑这种方法。
4. 使用层次锁结构。

# condition_variable

条件变量，用于线程同步。典型的使用例为生产者消费者模型

```cpp
std::mutex mut;
std::queue<data_chunk> data_queue; // 1
std::condition_variable data_cond;

void data_preparation_thread()
{
    while(more_data_to_prepare())
    {
        data_chunk const data=prepare_data();
        std::lock_guard<std::mutex> lk(mut);
        data_queue.push(data); // 2
        data_cond.notify_one(); // 3
    }
}

void data_processing_thread()
{
    while(true)
    {
        std::unique_lock<std::mutex> lk(mut); // 4
        data_cond.wait(
            lk,[]{return !data_queue.empty();}); // 5
        data_chunk data=data_queue.front();
        data_queue.pop();
        lk.unlock(); // 6
        process(data);
        if(is_last_chunk(data))
            break;
    }
}
```

其中生产者可以用`notify_one`和`notify_all`来实现通知等待的线程。消费者调用`wait`来等待通知，其中传入的参数首先是一个锁，然后是一个函数，作为等待的条件。`wait`会检查等待条件是否为真，当条件满足时，进行下一行代码。如果条件不满足，就会释放锁，等待下一次被唤醒。被唤醒时又重新获取锁，再次判断条件，以此类推。

可以用`wait_for`和`wait_until`设置等待时间。

# future

前面提到，`std::thread`的函数对象是不能带有返回值的，如果需要有返回值，应该使用`std::async`

```cpp
int foo(int);
void bar();

std::future<int> ans = std::async(foo, 1);
bar();
std::cout<<ans.get()<<"\n";
```

这里的`ans.get()`的调用，会一直阻塞，执行`foo`执行完毕。我们也可以使用`ans.wait()`，阻塞，直到值变为可用，稍后再`get`值。

`async`还可以进行惰性求值

```cpp
std::future<int> ans = std::async(std::launch::deferred, foo, 1);
```

具体这个`foo`的函数的执行时间，会推迟到`wait`或者`get`出现的时候，再去执行。

`std::packaged_task`可以将`future`与函数对象进行绑定。当调用`packaged_task`时，就会调用这个函数对象，当`future`状态就绪时，会存储返回值。典型的应用有线程池，见后。

```cpp
void task_bind()
{
    std::packaged_task<int()> task(std::bind(f, 2, 11));
    std::future<int> result = task.get_future();
 
    task();
 
    std::cout << "task_bind:\t" << result.get() << '\n';
}
 
void task_thread()
{
    std::packaged_task<int(int, int)> task(f);
    std::future<int> result = task.get_future();
 
    std::thread task_td(std::move(task), 2, 10);
    task_td.join();
 
    std::cout << "task_thread:\t" << result.get() << '\n';
}
```

如上，第一个函数是在当前线程执行任务，第二个函数则是将任务传递给别的线程。

`std::promise`提供了一种手动给`future`设定值的方式，例如

```cpp
void do_work(std::promise<void> barrier)
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
    barrier.set_value();
}

std::promise<void> barrier;
std::future<void> barrier_future = barrier.get_future();
std::thread new_work_thread(do_work, std::move(barrier));
barrier_future.wait();
new_work_thread.join();
```

当`do_work`执行完后，`barrier`会设置值，从而提醒`barrier_futuer`值可用，从而结束等待。也是一种线程同步的工具。

# atomic

C++标准库提供的原子类型，对其进行的操作是原子的。C++提供了内存序，来规定编译器进行命令重排的程度。例如

```cpp
std::atomic_int acnt;
int cnt;
 
void f()
{
    for (int n = 0; n < 10000; ++n)
    {
        ++acnt;
        ++cnt;
    }
}
 
int main()
{
    {
        std::vector<std::jthread> pool;
        for (int n = 0; n < 10; ++n)
            pool.emplace_back(f);
    }
 
    std::cout << "原子计数器为 " << acnt << '\n'
              << "非原子计数器为 " << cnt << '\n';
}
```

C++标准库中规定了许多类型别名，例如这里的`std::atomic_int`就是`std::atomic<int>`的别名。

C++的原子类型的内部实现有两种方法，一种是用锁实现的，另一种是无锁的原子指令。可以通过对原子类型的变量使用`.is_lock_free()`来查询。一个类型是否无锁取决于平台，但是`atomic_flag`总是无锁的。

这里`atomic_flag`不是`std::atomic`的特化的别名，他和`atomic<bool>`是不一样的。它不支持赋值、运算等操作。只支持`test_and_set(), test(), clear()`等更基础的操作。

虽然对`atomic_int`等类型直接使用`+,-,++,--,+=,-=`很方便，但是使用`fetch_add,fetch_sub,store,load,exchange`等函数可以更精细地控制内存序。

`C++`有六种内存序

```cpp
typedef enum memory_order {
    memory_order_relaxed,
    memory_order_consume,
    memory_order_acquire,
    memory_order_release,
    memory_order_acq_rel,
    memory_order_seq_cst
} memory_order;
```

- `relaxed`，其是最宽松的，只保证对该原子类型的操作是原始的，不限制重排。
- `acquire`，其不允许将在`load`该原子类型之后的命令重排到该命令之前。
- `release`，其不允许将在`store`该原子类型之前的命令重排到该命令之后。
- `consume`，类似于`acquire`，但是区别在于，只会限制和该原子类型相关的操作的重排。
- `acq_rel`，即结合二者，前后的命令都不能重排。
- `seq_cst`，最严格的，也是前后命令都不难重排，并且所有线程的语句都以全局的内存修改顺序为参照。

# 线程池

这里给出一个典中典极简实现：[https://github.com/progschj/ThreadPool/blob/master/ThreadPool.h](https://github.com/progschj/ThreadPool/blob/master/ThreadPool.h)
