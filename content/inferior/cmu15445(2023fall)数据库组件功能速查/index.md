---
title: "CMU15445(2023 Fall)数据库组件功能速查"
date: 2024-04-29T14:09:51+08:00
draft: false
tags:
  - C++
  - 水文
  - 数据库
categories: 其他计算机科学
mathjax: true
markup: goldmark
image: cover.jpg
---

为了应对面试官的拷打，写下此文，方便我复习这个数据库的实现方法。

# Project 1

## Task 1

这一部分主要是要求实现一个LRU-K的替换算法。

LRU是最近最少使用算法，也就是把buffer里面，上次使用时间距离现在最远的元素换出去。但是LRU的问题在于，如果有很多“偶发性”数据访问，即只访问一两次的数据，那么LRU会替换掉几乎所有元素，从而降低命中率。

LRU-K的想法是，访问过K次及以上的数据和其他数据分开来算。显然，前者是更需要放在缓存里的，而后者是“偶发性”数据。

这里同样定义了距离，当访问次数大于等于$K$时，该元素的距离为当前时间点减去之前的第$K$次访问的时间点。如果一个元素的访问次数不足$K$，则距离无限大。换出的规则仍然是，把距离最大的元素换出。如果有多个元素距离无限大，则把最近访问最远的那个元素换出。相当于对它们进行朴素的LRU。

下面具体解析一下在`src/include/buffer/lru_k_replacer.h`里面的东西的作用

- `LRUNode`：这里记录我们刚刚提到的元素的信息。其中：
    - `history_`记录了最近的至多$K$次访问的时间戳，表头为最远的，表尾为最近的。
    - `k_`代表累计有几次访问，最大等于LRU-K算法设定的$K$值，不能超过。
    - `fid_`代表在buffer pool中的帧号，其作用在于通过帧号找到页号，具体会用在buffer pool中。
    - `is_evictable_`，为真时代表此元素可以换出。
- `LRUKReplacer`：这个就是具体实现LRU-K的地方。
    - 先看看它的成员变量：
        - `node_store_`，用于存储帧号对应的LRUNode。
        - `current_timestamp_`用于记录现在的时间戳，不需要用unix时间戳什么的，我们只要每次访问后将其加一即可。
        - `curr_size_`指的是目前有多少个evictable的帧。
        - `replacer_size_`指LRU-K能容纳多少Node
        - `k_`即代表LRU-K中的$K$值。
        - `latch_`即我们使用的互斥锁。
    - 再来看看成员函数：
        - 构造析构略
        - `Evict`指换出一个帧，参数为换出的帧其帧号的指针，返回真时换出成功，否则换出失败。换出规则即之前介绍的规则。
        - `RecordAccess`记录给定帧的一次访问。每次访问都要增加`current_timestamp_`，如果不在`node_store_`中还需要新建。然后修改`frame_id`对应的node的数据，包括`k_`和`history_`。如果已经有$K$次访问了，还需要注意不要自增超过$K$，以及`history_`不要多于$K$个记录。
        - `SetEvictable`即修改给定帧的可换出属性，同时会影响`curr_size_`的值。
        - `Remove`即删除给定帧的访问记录。注意只有evictable的帧才能换出。同时会影响`curr_size_`的值。

这里面所有的`frame_id`的范围都在`[0,replacer_size_)`中，否则为非法访问。

## Task 2

这里要求实现一个磁盘调度算法。不过这个算法并不高深，就是一个简单的FIFO，使用一个并发安全的队列实现。

`src/include/storage/disk/disk_scheduler.h`中，有：

- `DiskRequest`：代表一次磁盘访问的属性，包括
    - `is_write_`即是读还是写
    - `data_`一个指针。读磁盘时，指向要读入的内存数组的开头。写磁盘时，指向要写入的内存数组的开头。大小是固定的，为Page的大小，在bustub中为4096，不足的都会补0
    - `page_id_`即写入磁盘的第几页。在后面会详细介绍，我们只用知道bustub的`xxx.db`文件是一页一页顺着排列的就行。页号从`0`开始。
    - `callback_`一个线程同步的变量，后面（task 3）介绍。
- `DiskScheduler`：即实现磁盘调度算法的部分
    - 其成员变量有：
        - `disk_manager_`磁盘调度算法要向其发送磁盘操作请求，之后介绍
        - `request_queue_`即FIFO算法所用到的队列
        - `background_thread_`即在后台不断把队列里的请求拿出来，发送给`disk_manager`的线程
    - 其成员函数有：
        - 构造函数，在构造的时候，就启动了`background_thread_`线程，其执行`StartWorkerThread`
        - `Schedule`，即简单地把一个`DiskRequest`放入队列中
        - `StartWorkerThread`，可以看作是生产者-消费者模型中的消费者。其不断地从队列中拿出请求，向`disk_manager_`发送对应请求，然后将`callback_`赋值。无限循环直到拿出的请求是一个`std::nullopt`（队列中存的是`std::optional<DiskRequest>`）
        - 析构函数，主要负责向队列中放入`nullopt`，让其停止。然后和后台的线程进行`join`

这个队列在这个项目里是已经提供好了的，不需要自己实现。不过为了防止被面试官拷打，我们来解析一下这个队列的实现方式。其被称作`Channel`，在`src/include/common/channel.h`。其底层是STL的`queue`，在此基础上加入了一个互斥锁和一个条件变量。只支持两个方法，即`Put`在队尾放入元素，`Get`取出队头元素（相当于front和pop二合一）。实现也非常经典，值得抄下来学习

```cpp
void Put(T element){
    std::unique_lock<std::mutex> lk(m_);
    q_.push(std::move(element));
    lk.unlock();
    cv_.notify_all();
}

T Get(){
    std::unique_lock<std::mutex> lk(m_);
    cv_.wait(lk, [&](){return !q_.empty()})
    T element = std::move(q_.front());
    q_.pop();
    return element;
}
```

然后我们来看看`DiskManager`具体怎么访问硬盘的。抛去一些不重要的，有这些成员变量：

- `db_io_`，它是一个`std::fstream`，至此真相大白，根本没有什么更底层的东西，只是一个STL自带的文件流而已。
- `file_name_`，数据库文件的名字。
- `db_io_latch_`，在硬盘读写的时候持有锁，防止冲突。

再来看看其中比较重要的几个成员函数：

- 构造函数，传入文件名，打开fstream。
- `WritePage`，传入页号，以及data的指针。前面也提到过，bustub的数据文件就是把页顺序排列，我们知道页号、知道页大小，就知道页的开头在文件中的位置。这里需要使用`fstream`的`seekp`先定位，然后再`write`写出数据。
- `ReadPage`，传入页号，以及data的指针。基本上同上。只是需要注意，如果文件的最后一页不足4096，那么需要在内存中把后面全部填充0

## Task 3

这里就是真正实现buffer pool的地方。buffer pool对于数据库的作用，类似于cache对于cpu的作用。cpu一般是先把内存中的东西放到cache中，然后之后读取就会更快了。基于空间局部性和时间局部性，虽然cache比内存小，但是也可以提升访问速度。而buffer pool所做的，就是把磁盘中的东西拿到内存。

buffer pool中的东西分为两部分，第一部分是Project 1中实现的，其只实现了基本的操作。而Project 2中实现的下半部分主要是负责更好的自动控制，不再需要手动进行Unpin等操作，解放程序员。

还是先介绍`BufferPoolManager`中的成员变量：

- `pool_size_`，代表这个buffer pool中能容纳多少个page。LRUKReplacer的大小将会和它一致
- `next_page_id_`，其为一个`std::atomic<page_id_t>`，也就是说可以并发地访问。其中`page_id_t`在这里定义为了`uint32_t`。之后创建一个新页的时候需要用到这个值。
- `pages_`，一个数组，用于存放所有page的
- `disk_scheduler_`，即task 2实现的东西
- `page_table_`，通过页号找到帧号
- `replacer_`，即我们的LRU-K算法
- `free_list_`，其最初的大小等于`pool_size_`，存储了所有空闲的帧的帧号
- `latch_`，自己的互斥量。

成员函数：

- 构造函数，主要就是传入大小、$K$值，`DiskManager`等。这里要初始化`pages_ = new Page[pool_size_]`，初始化`replacer_`，同时把所有帧号放到`free_list_`里
- 析构函数，主要进行`delete[] pages_`
- `NewPage`，分配一个新页，返回这个新页的指针，新页的页号通过参数中的指针返回。由于我们是在内存中分配新页的，所以要先找一个位置，如果有空闲的帧就直接获取一个帧号，否则换出一个帧到硬盘上（即判断是否为脏页，为脏则要向`DiskManager`发出一个写请求，然后从`page_table_`中删掉这一项。否则直接从`page_table_`中删掉。这里删除的时候，需要`auto future = r.callback_.get_future()`，然后再发送请求，然后`future.wait()`，这样在删除完成之前就会被阻隔，从而线程同步。），然后继续使用这个换出的帧号。在`pages_`数组上，直接对这个元素进行初始化。例如重置数据、分配页号、设置脏位、设置`pin_count_`。然后写入`page_table_`，并在`replacer_`中记录一次访问，以及设定`evictable`为`false`。这里可能所有页面都被设置为不能换出，这里要返回`nullptr`
- `FetchPage`，根据页号获取一个页。这个页可能已经在buffer pool里了，那么直接读这个页。否则，要去硬盘里找。同前，有空闲帧就分配、否则换出，等等操作。在之后，我们要给这个page的`pin_count_`加一，表明又有一个新的线程在访问这个页。以及，同样地更新`page_table_`、`replacer_`等。
- `UnpinPage`，一个线程不再需要这个页时，需要将其unpin，也就意味着对应的`pin_count_`减一。同时`UnpinPage`的参数里会带有`is_dirty`，用于标注这个页的脏位。注意，如果页不在buffer pool里，或者`pin_count_`已经归零，就不需要进行操作了。页的这些元信息只存在于内存中，硬盘中只有data，所以`pin_count_`不需要从硬盘里拿出来减一。如果成功进行了减一，并且计数器归零，就可以在`replacer_`中把`evictable`设置为`true`了
- `FlushPage`，给定页号，无论是否为脏页，都写回硬盘中。之后设置为非脏页。其他信息不做改动。
- `FlushAllPages`，刷新所有页。
- `DeletePage`，这里说的不是删掉硬盘上的页。是删掉buffer pool中的页，并标记为空闲帧。当然，如果给出的页号本来就不在buffer pool里，则什么改动都不做。删除的时候已经假定引用计数归零了，所以如果非零，则是非法操作。需要从`page_table_`、`replacer_`中删掉相应的元素。然后插入到`free_list_`中，把原来`pages_`这个位置上的页元信息重置。
- `AllocatePage`，其实就是分配了一个新页号`return next_page_id_++`
- `DeallocatePage`，这就是个空函数，bustub里根本不考虑这个。

之后我们来关注一下这个`Page`里面都有什么东西，在`src/include/storage/page/page.h`中，成员变量：

- `data_`，一个指针，和读写硬盘的那个`data`指针类似，就是指向一个数组的头。大小固定为4096，即页大小。在构造函数中分配内存。析构中释放。
- `page_id_`，页号。
- `pin_count_`，表示有多少线程正在使用这个页。
- `is_dirty_`，脏位。
- `rwlatch_`，读写锁。实际上是用`std::shared_mutex`实现的，写的时候进行`lock()`和`unlock()`，读的时候进行`lock_shared()`和`unlock_shared()`。

成员函数可说的反而不多，除了一大堆getter和setter、读写上锁之外，有：

- `ResetMemory`，就是一个`memset`，把`data_`的内容初始化。

这里面还有两个叫`GetLSN`和`SetLSN`的函数，不知道干嘛的，整个项目里倒也没用过这两个函数。
