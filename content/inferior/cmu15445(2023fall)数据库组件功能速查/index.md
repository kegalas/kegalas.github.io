---
title: "CMU15445(2023 Fall)数据库组件功能速查"
date: 2024-04-29T14:09:51+08:00
draft: true
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

还是先介绍`BufferPoolManager`（`src/include/buffer/buffer_pool_manager.h`）中的成员变量：

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

# Project 2

## Task 1

在上一部分，我们实现的`BufferPoolManager`实现的比较底层的操作。程序员需要手动创建页、获取页、删除页、Unpin页。本部分要求我们，实现一个wrapper，能够在构造的时候获取页，析构的时候Unpin、删除页。同时，实现要求我们保证并发安全，所以也要自动地控制锁。

首先关注`src/include/storage/page/page_guard.h`，其中有三个类：

- `BasicPageGuard`
- `ReadPageGuard`
- `WritePageGuard`

这三个类都是独占资源的类，所以类似于`unique_ptr`，它们的拷贝构造和拷贝赋值被禁用了。看看它们的成员变量：

- `BasicPageGuard`
    - `bpm_`指向一个`BufferPoolManager`的指针
    - `page_`指向自己所管理的页的指针
    - `is_dirty_`脏位
- `ReadPageGuard`
    - `guard_`是一个`BasicPageGuard`。本类的成员函数的实现方式，使得本页只读，后面介绍。
- `WritePageGuard`
    - `guard_`是一个`BasicPageGuard`。本类的成员函数的实现方式，使得本页可读写，后面介绍。

看看它们的前几个成员函数：

- `BasicPageGuard`
    - 默认构造函数，其传入`bpm`和`page`指针来初始化。
    - 移动构造函数，把另外一个guard的`page_`，`is_dirty_`，`bpm_`全部都移动过来，之前的清空。
    - `Drop`，即废弃掉这个页，或者说放弃控制权。这里就需要调用`bpm_`中的`UnpinPage`了，传入自己的页号和脏位。之后再把成员变量清空。
    - 移动赋值函数，和移动构造类似，但是要先把自己的资源`Drop`掉，再去考虑移动的事。
    - 析构函数，可以直接调用`Drop`
- `ReadPageGuard`
    - 默认构造函数，传入的也是`bpm`和`page`指针，其用来初始化`guard_`成员。其实我不是很明白这里为什么没有上读者锁（项目原版的代码），可能是因为项目里根本没有地方直接构造`ReadPageGuard`吧。
    - 移动构造函数，我们只用移动`guard_`即可。虽然我写了先把`that`解锁再把`this`加锁，但我想了一下好像没有必要，甚至可能是错的。不过评测没有问题。
    - `Drop`，废弃页。注意如果`this`已经有一个页，需要先解锁，然后再`Unpin`。之后再把`guard_`的信息清空。如果弄反了，可能`Unpin`后页立刻换出，然后我们再解锁，解锁的就是其他页的锁了。
    - 移动赋值函数，同前，如果需要，先考虑`Drop`掉自己。
    - 析构函数，调用`Drop`
- `WritePageGuard`，和`ReadPageGuard`基本一样，只不过加锁的时候加的是写者锁。

前面也提到，一般不会直接调用`ReadPageGuard`和`WritePageGuard`的默认构造函数，我们会通过`BasicPageGuard`中内置的两个函数来“升级”成另外两个：

- `UpgradeRead`，首先给`page_`上读者锁，然后记录下`page_`和`bpm_`的指针，清空自己的成员变量，调用默认构造函数返回一个`ReadPageGuard`
- `UpgradeWrite`，同上，只不过上的是写者锁。

这三个类还有几个读取数据的成员函数

- `BasicPageGuard`
    - `PageId`，获取`page_`的页号
    - `GetData`，获取`page_`的`data_`指针，一个`const char *`，也即不可修改内容
    - `As`，将`GetData`中获得的指针转化为另一个类型的指针，即`const T *`
    - `GetDataMut`，同`GetData`，只不过将`is_dirty_`设置为`true`，返回`char *`
    - `AsMut`，将`GetDataMut`获得的指针转为`T *`
- `ReadPageGuard`，只包含`PageId`，`GetData`，`As`
- `WritePageGuard`，包含全部五个函数。

之后我们回到`src/include/buffer/buffer_pool_manager.h`中，解决上个Project的遗留问题

- `FetchPageBasic`，首先调用自己的`FetchPage`获取页号，然后调用`BasicPageGuard`的默认构造函数构造，然后返回
- `FetchPageRead`，调用`FetchPageBasic`后进行`UpgradeRead`，返回
- `FetchPageWrite`，类似于上条。
- `NewPageGuarded`，类似于第一条，使用`NewPage`的页号。

## Task 2

从这里开始要求我们实现一个可扩哈希。我们先把可扩哈希的原理说明一下，从一张图开始

![1.jpg](1.jpg)

首先可以看到，它是一个三层结构。第一层是`header`，其大小固定。第二层是`directory`，其大小可以变化，从只有一项，到大小上限。而第三层是`bucket`。

`header`的每一项指向了一个`directory`（当然如果没有数据的话就指向空指针），而`directory`的每一项指向了一个`bucket`，数据实际上是存在`bucket`里。所以，如果我们要读写一个数据，我们要经过以下步骤：

1. 找到该数据对应与`header`中的哪一项。
2. 从`header`的这一项中找到其对应的`directory`，然后在找到该数据对应该`directory`中的哪一项
3. 从`directory`中的这一项找到其对应的`bucket`，然后在`bucket`中遍历（`bucket`可以看做是固定大小的数组），找到所需的数据位置。

具体是根据什么方法来找的呢？

首先，我们会获得该数据的哈希值。假设我们这里得到的哈希值是一个32位无符号整数，那么，根据`header`的`max_depth`，提取出哈希值的高`max_depth`位。如上图，其`max_depth`为$2$。如果我们的哈希值是`01...111`，那么就要映射到`header`中的第`1`项（从`0`开始计数）。如果是`10...111`就映射到第`2`项，以此类推。

接下来，我们在对应的`directory`里，根据其`global_depth`，找到哈希值的低`global_depth`位。例如在`global_depth`为$2$时，`01...111`就映射到`directory`的第`3`项。

可扩哈希具体指的是哪里可扩呢？实际上指的是`directory`可扩。我们画个简单的情况来介绍：

![2.jpg](2.jpg)

如图，我们省去了`header`，并且现在`directory`的`global_depth`的大小为`0`，也就是只有第`0`项。我们这里假设`bucket`大小为$2$。这里我们有一个放了两个数据的`bucket`，这里用哈希值表示数据。

整个`directory`有一个`global_depth`，主要是代表项数，也用来找哈希值对应的`bucket`。而每一项，有一个`local_depth`，代表的是，该项指向的`bucket`中，保证所有数据的哈希值最低的`local_depth`位是相同的。上图中，`local_depth`为`0`，即没有一位是保证相同的。

如果我们要再插入一个数据（假设是`...10`）到其中，怎么做呢？这里显然已经插入满了，所以我们要扩容。首先，把`global_depth`加一，同时`directory`的大小也就增加了（大小为$1<<{global\_depth}$），所以我们要增加`directory`的项数，其还是指向这个`bucket`。

![3.jpg](3.jpg)

现在，`directory`的`global_depth`是`1`，而两项的`local_depth`都是`0`。观察我们要插入的数据`...10`，其最低`1`位是`0`，所以要插入第`0`项。而第`0`项指向的`bucket`仍然是满的。所以我们现在要把`bucket`分裂。如下：

![4.jpg](4.jpg)

这里分裂的时候，需要按照`directory`的映射关系，将`bucket`的数据分别放到相应的新`bucket`中。这里，我们就可以增加`local_depth`了，都是`1`。然后我们再插入数据`...10`，就可以正常插入了。

![5.jpg](5.jpg)

再例如下图

![6.jpg](6.jpg)

我们插入`...100`，和之前一样，我们需要先扩容`directory`

![7.jpg](7.jpg)

这里`global_depth`变成了`2`，写在了上方。而`local_depth`都是`1`，写在左边。然后我们的数据`...100`是映射到第`0`项，其`bucket`是满的，所以要分裂`bucket`，再插入，如下

![8.jpg](8.jpg)

目前为止，插入和查询就说明白了。删除数据放到之后再说，先来看看代码，首先是`src/include/storage/page/extendible_htable_header_page.h`

首先，源代码注释中给出了这个`page`的布局

```cpp
/**
* Header page format:
*  ---------------------------------------------------
* | DirectoryPageIds(2048) | MaxDepth (4) | Free(2044)
*  ---------------------------------------------------
*/
```

这里我们也可以知道，无论是`header`、`directory`还是`bucket`，都是作为一个页存在硬盘上的。大小都依然是`4096`。

这段注释指出，`header`存储的数据有两个，一个是`MaxDepth`，一个是表项。通过阅读后面的代码可知，这个类有两个成员变量。

```cpp
page_id_t directory_page_ids_[HTABLE_HEADER_ARRAY_SIZE];
uint32_t max_depth_;
```

表项是用一个数组来表示的，其类型为`page_id_t`，被`using page_id_t = uint32_t`定义。而另一个就是代表表的最大大小。因为`uint32_t`是`4`字节的，而`DirectoryPageIds`占`2048`字节，所以最多有`512`项。这里也说明，实现上表项不是一个指向`directory`的指针，而是存储了对应的页号，找到`directory`需要读取硬盘上的另一个页。

接下来我们看看其中定义的三个静态变量

```cpp
static constexpr uint64_t HTABLE_HEADER_PAGE_METADATA_SIZE = sizeof(uint32_t);
static constexpr uint64_t HTABLE_HEADER_MAX_DEPTH = 9;
static constexpr uint64_t HTABLE_HEADER_ARRAY_SIZE = 1 << HTABLE_HEADER_MAX_DEPTH;
```

第一个定义了元信息的大小，第二个定义了`bustub`默认的`max_depth`，第三个定义了默认的表大小。接下来我们看`ExtendibleHTableHeaderPage`的成员函数

- 所有构造函数和析构函数都被禁用了，根据代码的注释说这样是为了保证内存安全。我水平不够暂时不了解原理。不过这样操作后就不能正常声明Header Page了，后续似乎都是通过强制类型转换来将普通页变成Header页。
- `Init`，传入`max_depth`，让我们对`header`进行初始化。我们照做即可，注意表项要全部初始化为`INVALID_PAGE_ID`，代表没有指向任何一个`directory`
- `HashToDirectoryIndex`，传入32位无符号整数的哈希值，找到对应的表项。就像我们之前说的一样，找到高`max_depth`位就行。`return hash>>(32-max_depth_);`。可能要特殊处理`0`深度的情况。
- `GetDirectoryPageId`，传入表项下标，返回页号。说白了就是根据数组下标返回数组元素。
- `SetDirectoryPageId`，说白了就是根据数组下标设置元素。
- `MaxSize`，返回表大小。

之后，我们来看`src/include/storage/page/extendible_htable_directory_page.h`，首先同样是看`page`的布局

```cpp
/**
* Directory page format:
*  --------------------------------------------------------------------------------------
* | MaxDepth (4) | GlobalDepth (4) | LocalDepths (512) | BucketPageIds(2048) | Free(1528)
*  --------------------------------------------------------------------------------------
*/
```

存储了四种数据，和成员变量一一对应，首先是`max_depth_`，代表`global_depth_`的上限，然后就是`global_depth_`自己，用于表示`directory`现在的大小，以及用于映射关系。之后是`local_depths_`，这是一个数组，每个元素都是`8`位无符号整数，代表对应表项的`local_depth`，可知directory最多有`512`个表项。最后的`bucket_page_ids_`也是一个数组，类型为`page_id_t`的数组，存储表项对应的`bucket`的页号。

```cpp
static constexpr uint64_t HTABLE_DIRECTORY_MAX_DEPTH = 9;
static constexpr uint64_t HTABLE_DIRECTORY_ARRAY_SIZE = 1 << HTABLE_DIRECTORY_MAX_DEPTH;
```

静态变量和之前相似。接下来我们看看成员函数

- 所有构造函数和析构函数都被禁用
- `Init`，传入`max_depth`，初始化`max_depth_`，`global_depth`，以及各个表项的信息。
- `HashToBucketIndex`，传入`32`位无符号哈希值，算出对应第几个表项。从前面的讨论可以得到，取`hash % (1<<global_depth_)`即可。
- `GetSplitImageIndex`，传入下标，获得其镜像`bucket`的下标。这里的镜像`bucket`指的是，前面进行`directory`扩容时，表项会翻倍，指向同一个`bucket`的表项数也会翻倍。每一个旧表项都会有一个新表项与它指向同一个`bucket`。这两个表项互为镜像。该项的`local_depth`已知，则当`local_depth`为零时，镜像为自己。否则镜像下标为`bucket_idx ^ (1<<(local_depth-1))`。无非是翻转一个二进制位。
- `IncrGlobalDepth`，我这里把扩容操作一起做进来了。具体来说，假设原来的大小是`sz`，那么，`global_depth_++`之后，新的大小变为`sz*2`。其中的表项有`element[sz+i]=element[i]`，顺着复制表项即可。
- `DecrGlobalDepth`，这里只需要`global_depth_--`。因为具体的缩容操作更复杂，需要在`bucket`层面才能执行。这里不需要清空数组多余的部分。
- `CanShrink`，具体怎么缩容后面再说。这里只要记住，所有的`local_depth`都小于`global_depth`，则可以缩容。

省略掉了一些简单的`getter`和`setter`。

之后，我们来看`src/include/storage/page/extendible_htable_bucket_page.h`，首先同样是看`page`的布局

```cpp
/**
* Bucket page format:
*  ----------------------------------------------------------------------------
* | METADATA | KEY(1) + VALUE(1) | KEY(2) + VALUE(2) | ... | KEY(n) + VALUE(n)
*  ----------------------------------------------------------------------------
*
* Metadata format (size in byte, 8 bytes in total):
*  --------------------------------
* | CurrentSize (4) | MaxSize (4)
*  --------------------------------
*/
```

首先是`METADATA`，八个字节组成，分别是`bucket`的当前大小和最大大小。之后就是实际存储的键值对（KV）。每个键值对的键和值的大小不定（括号后面的是序号），具体存的什么数据，之后会介绍。

成员变量也很简单，和页的内容一一对应。`size_`表示当前大小，`max_size_`表示最大大小，`array_`则是后面的KV对的数组，他的类型是`MappingType`，定义为模板`std::pair<KeyType, ValueType>`

来看成员函数

- 所有构造函数和析构函数都被禁用
- `Init`，负责初始化`max_size_`和`size_`
- `LookUp`，传入`key`和比较`key`的比较函数，查找这个`bucket`里有没有`key`相等的元素，返回`value`。（本课程不考虑`key`冲突，也不考虑`multimap`这样的情况）。这里直接遍历数组`array_`，依次比较每个`key`即可进行查找。
- `Insert`，传入`key, value, cmp`，插入到`bucket`中，只要`key`没出现过并且`array_`空间还有空闲就可以插入。
- `Remove`，传入`key, cmp`，删除`key`对应的元素。注意，删除的时候，要把后面的所有元素往前移动一个来填补空缺。因为我们的插入是顺着插入，找到第一个空闲就插入。

省略掉其他的`getter`和`setter`，现在我们来讨论一下，它`bucket`里面的KV究竟是在存什么。

在`extendible_htable_bucket_page.cpp`中，代码末尾有：

```cpp
template class ExtendibleHTableBucketPage<int, int, IntComparator>;
template class ExtendibleHTableBucketPage<GenericKey<4>, RID, GenericComparator<4>>;
template class ExtendibleHTableBucketPage<GenericKey<8>, RID, GenericComparator<8>>;
template class ExtendibleHTableBucketPage<GenericKey<16>, RID, GenericComparator<16>>;
template class ExtendibleHTableBucketPage<GenericKey<32>, RID, GenericComparator<32>>;
template class ExtendibleHTableBucketPage<GenericKey<64>, RID, GenericComparator<64>>;
```

这样的模板特化。首先我们知道了，KV中的V是`RID`（第一个除外，应该只是用来调试的）。什么是`RID`？实际上，又可以看做是一种指针。`RID`存储了两个成员变量`page_id_`和`slot_num_`，也就是说，具体的数据是存在别的页中，存在序号为`slot_num_`的部分。这一部分具体可以阅读`src/include/common/rid.h`

然后是前面的`GenericKey<>`是什么？他其实是一个包装的数组，当作哈希值来使用。这个数组可以用我们数据库里的具体的数据来设置（称作`Tuple`，以后再说），也可以直接整数值来设置（仅用于测试意义）。`GenericKey<4>`代表数组长度为`4`（数组类型为`char[]`）。具体可见`src/include/storage/index/generic_key.h`。

同样的这个文件里，定义了`GenericComparator<>`，具体做的事就是比较`GenericKey<>`之间的大小关系，当然，也就是通过比较数组，或者说比较`Tuple`来实现的。

这个`Tuple`其实就是我们在数据库表里看到的一行数据，具体展开就太大了，跟这里也关系没有太密切，我放到后面的附录来说。

## Task 3

TODO：重新用更好的语言组织这一段。

上一个task我们了解了可扩展Hash大概是怎么运行的，也介绍了一下三个层级的信息是如何存储的，接下来我们来具体实现可扩展Hash的功能。具体需要修改的代码在`src/container/disk/hash/disk_extendible_hash_table.cpp`

先介绍`DiskExtendibleHashTable`的成员变量：

- `index_name_`，似乎没有作用，只在构造函数里进行了赋值。
- `bpm_`，即一个BufferPoolManager，我们创建、读取各种page的时候会用到。
- `cmp_`，用于在查询等功能中比较数据键值的。
- `hash_fn_`，TODO
- `header_max_depth_, directory_max_depth_, bucket_max_size_`，前面介绍过了，不再重复。
- `header_page_id_`，显然，你要访问hash表必须得有个入口，才能找到header的内容，才能依次往下访问数据，所以我们必须存一下header所在的页的页号。

首先，在你创建这个hash表的时候（即调用构造函数），还没有任何数据被插入，所以我们只需要创建一个空的`header_page`即可，后面插入数据的时候才会按需要创建directory或者bucket。具体来说，我们要实现一堆功能，我们先讲比较简单的查询过程，再讲复杂的插入和删除过程。

**查询**

*找到header*

这个比较简单，像我们之前说的一样，通过将Key转为hash值，再用hash高位来获取对应的header项（即directory的下标），这里我们使用之前实现的`ExtendibleHTableHeaderPage`类中的`HashToDirectoryIndex`方法。至于获取header page，我们首先通过bpm中的`FetchPageRead`，输入`header_page_id_`来获取一个读页面，然后再用`.As<ExtendibleHTableHeaderPage>()`来将其类型转换为header page，然后就可以调用其方法了。

*找到header中的directory*

首先我们在获取到directory的下标之后，就要获取其页号，正好我们的header page中也已经实现了这个方法，即`GetDirectoryPageId`。当然，可能在查询的时候并不存在对应的directory页，即返回了无效页号，进行特判然后返回`false`。之后我们如法炮制`.As<ExtendibleHTableDirectoryPage>()`

*找到directory中的bucket*

在获取了Directory Page之后，我们就可以调用之前实现的`HashToBucketIndex`，来获取bucket的下标，然后调用`GetBucketPageId`来获取bucket的页号，当然，如果没有找到也要进行特判。之后我们如法炮制`.As<ExtendibleHTableBucketPage<K, V, KC>>()`

*找到bucket的对应数据*

这一步最简单，调用Bucket Page中实现的`Lookup`即可。

**插入**

插入要麻烦很多，涉及到扩充directory时的一系列新建、迁移操作。我们先看最简单的，

*不需要新建任何东西，有空位可以直接插入*

- 首先，插入时我们还是要获取header page，方法同前。
- 获取directory page，方法同前。
- 获取bucket page，方法同前。不过注意因为我们需要写入，所以需要调用`AsMut<ExtendibleHTableBucketPage<K, V, KC>>()`
- 进行插入，调用Bucket page的`Insert`函数。
    - `Insert`返回`true`，则说明插入成功
    - `Insert`返回`false`，则说明插入不成功，两种情况，一种情况是已经有一样的数据了，另一种是Bucket满了。这里我们讨论的是有空位的情况，所以只可能是有数据了，所以直接返回插入失败即可。

*需要新建bucket*

这种情况发生在获取bucket的页号时返回了无效页号的时候。

- `bpm_->NewPageGuarded`来获取一个新的页
- `directory->SetBucketPageId`来给directory中的存储bucket页号的数组赋值
- `bpm_->FetchPageWrite`、`.AsMut<ExtendibleHTableBucketPage<K, V, KC>>`来获取bucket page
- `bucket_page->Init`来初始化bucket
- `bucket_page->Insert`进行插入，通常来说会成功，可以return其返回值。

*需要新建directory*

这种情况发生在获取directory的页号时返回了无效页号的时候。

- `bpm_->NewPageGuarded`来获取一个新的页
- `header->SetDirectoryPageId`来给header中的存储directory页号的数组赋值
- `bpm_->FetchPageWrite`、`.AsMut<ExtendibleHTableDirectoryPage>`来获取directory page
- `dir_page->Init`来初始化directory
- 调用新建bucket来插入数据的函数，也就是上一部分所讲的内容。

*bucket满了，需要扩充、迁移*

这种情况同前所述，发生在bucket page调用`Insert`返回`false`，但用`Lookup`却找不到数据的时候。插入时扩充的流程已经在Task 2中介绍，这里就只说实现流程

- 先增加该bucket的local depth，即`dir_page->IncrLocalDepth`
- 如果该local depth大于了directory的global depth，则说明directory需要扩容。因为local depth指的是bucket中所有的数据的key的最低`local_depth_`位是相同的，global depth则被用于取下标，key的最低`global_depth_`位对应着directory中bucket数组的下标。当local depth等于global depth时，说明directory每一个位置都对应着不同的bucket，不能再进行bucket分裂，所以directory必须扩充，然后再进行分裂。扩充直接调用`dir_page->IncrGlobalDepth`即可，见前文的实现。
- 进行bucket的分裂。经过了上一条过程，保证了bucket一定可以分裂，（如果不满足自增后的local depth大于global depth，则代表本来就可以分裂）。首先通过前文实现的`dir_page->GetSplitImageIndex`来获取bucket下标的“镜像下标”。我们需要分配一个新的bucket page，进行初始化等等。
- 分裂之后进行数据迁移，因为local depth增加，通常情况下数据会根据多出来的那一位的`0`或`1`的取值分为两类，前一类留在旧的bucket中，后一类需要搬迁到新的bucket中。我们可以先将老的bucket数据复制一份，然后清空老bucket。再根据新的local depth mask分别将数据插入到两个bucket中。
- 重新映射directory。因为我们多出来了一个bucket，所以需要让directory中的一些项指向这个bucket。一般来说只需要将镜像下标的项指向这个bucket即可。

但是，有一种非常棘手的特殊情况。假设现在directory的大小为1，而bucket的大小为2，里面有两个数据的hash值分别为`11...000000`、`11...010000`，而我们需要插入的新数据是`11...100000`，这就导致我们需要将global depth首先扩充到5，才能将已有的两个数据映射到不同的下标上。也就是说我们之前虽然进行了bucket分裂，但是所有数据都还是留存在了老的bucket中，还是无法插入新数据（新数据由于映射关系也不会插入新bucket中）。

我们之前的代码只会分裂一次，并没有考虑到老数据全部待在老bucket中的情况。为了解决这个问题，我们可以使用递归的思路，直接在分裂和重映射之后再次从头进行一次插入过程即可。

这里的重映射也值得说一说，在我们反复的扩充时，可能就不只有两个下标指向同一个bucket了，可能有4个、8个更多个，而只用“镜像下标”是无法表示那么多的，需要使用local_depth_mask来依次遍历所有需要重映射的下标。

**删除**

删除也比较麻烦，主要是涉及到收缩directory

- 首先，插入时我们还是要获取header page，方法同前。
- 获取directory page，方法同前。
- 获取bucket page，方法同前。
- 调用`bucket_page->Remove`，先将数据从bucket中删除。
- 删除数据之后，理论上就有可能将之前分裂的两个bucket合并成一个。例如假设bucket的大小为2，现在有互为镜像下标的两个bucket分别存入了2个和1个数据，如果删掉数据，变成存了1个和1个数据的bucket，则可以合并。但是每次都这样检查并合并就太复杂了，在bustub里面，当且仅当该bucket为空或者镜像下标的bucket为空才会合并。于是我们就检查这个条件是否成立，如果不为空就直接不合并了。我们这里计这两个bucket中，数据不空的为$\alpha$，数据空的为$\beta$
- 除了检查是否为空bucket，还要检查global depth是否为0，如果为0也没有合并的需要。如果下标和其镜像下标相同，那么也没有合并的需要。另外，还需要检查bucket和其镜像下标bucket的local depth是否相等，相等的才能合并，因为我们总是假设共用一个bucket的两个directory项具有同样的local depth
- 如果所有的检测都通过，那么我们需要将所有的bucket中，页号等于$\alpha$或$\beta$的所有bucket，都更换其页号为$\alpha$的页号，并且将对应的local depth减一。这一步也就进行了bucket合并。然后再删除掉$\beta$页
- 注意我们合并完之后可能还是一个空bucket，此时就可以循环或者递归的方式检测还是否可以继续合并。
- 当我们递归式的合并结束后，就可以检测directory是否可以收缩了。当且仅当所有local depth都小于global depth时，可以进行收缩。因为此时所有bucket都至少有两个directory项指向他。
- 如果可以收缩，就进行收缩，由于directory的数组大小是固定的，所以我们简单一点可以直接对global depth减一，而不清除数组值。

## Task 4

这里要求我们处理并发安全，其实只要之前实现好了PageGuard，然后在访问页的时候`FetchPageWrite`和`FetchPageRead`就没有并发问题了。注意，页用完了要及时Drop，以提高并发度。

# Project 3

TODO: 关于SQL parser、binder、planner的介绍

![9.jpg](9.jpg)

本project负责将SQL语句转化为实际执行的数据操作。具体来说，SQL语句首先经过Parser和Binder将其转化为抽象语法树，之后经过Planner，将抽象语法树转化为转化成一颗由操作符组成的树，然后在经过Optimizer，将这棵树中多余的操作符去除。之后就按照树形结构执行剩下的操作符。下面是一颗典型的操作符树

![10.jpg](10.jpg)

本Project负责的实际上就是实现所有这些操作符，然后在此基础上完成一些Optimizer的算法。前面三个组件都是项目已经实现好的。

## 已实现部分介绍

有几个操作符已经实现了，可以作为参考。

### AbstractExecutor 

首先，所有的操作符都继承与这个类，我们看看他的变量和函数。

成员变量就一个，是`exec_ctx_`，它是一个`ExecutorContext*`，里面存储了执行操作时所需要的上下文内容，主要包括下个project要用的`Transaction`、`TransactionManager`等，之前实现过的`BufferPoolManager`，以及`Catalog`等等。（TODO：介绍）

成员函数除去构造析构，有

- `Init`，用于初始化，必须要在下面的`Next`之前调用
- `Next`，用于执行操作符，然后返回数据（`Tuple`及其`RID`）。具体来说，数据库对于数据的处理是一条一条（即`Tuple`）来执行的，例如最简单的遍历操作，我们就需要一直调用`Next`，获取所有`Tuple`，直到遍历结束。又比如累加，就需要操作符执行“子操作符”遍历之后，返回其累加和。
- `GetOutputSchema`，`Schema`这个东西其实就记载了数据库表中的每一列代表什么数据类型。（TODO：介绍）
- `GetOutputSchema`，返回成员变量`exec_ctx_`

### Projection

它的作用是实现表达式计算，对输入数据应用该表达式，得到返回值。

成员变量多了两个

- `plan_`，存储了一个`ProjectionPlanNode*`，即planner生产的节点，里面存储了将会用到的表达式，包含以下三种：`ColumnValueExpression`、`ConstantExpression`、`ArithmeticExpression`。都很简单，就是原样保留Tuple中一列的值的表达式，常数表达式和算术表达式。
- `child_executor_`，是一个指向子节点的指针，后续的几乎所有操作符都有这种变量。

后面的绝大部分都包含这两个成员变量，不再介绍。

成员函数的签名保持不变，介绍一下实现

- `Init`，由于Projection本身没有什么需要初始化的，所以只用调用子节点的初始化。
- `Next`，Projection是对输入应用操作符，所以要先得到输入，就执行子节点的`Next`来得到。然后对子节点返回的`Tuple`使用`plan_->GetExpressions`，得到表达式执行后的值，然后将值组合成一个新的`Tuple`进行返回即可。

### Filter

filter负责执行条件判断，满足条件的会返回给上层，不满足的则会被过滤。直接看成员函数

- `Init`，调用子节点的初始化。
- `Next`，返回子节点给出的数据中第一个满足条件的。具体来说，`FilterPlanNode`中有一个`GetPredicate`函数，其返回的谓词表达式可以判断一个`Tuple`是否满足条件。我们重复调用子节点的`Next`，执行判断，直到满足条件，然后就返回。当我们反复调用Filter的`Next`时，子节点返回的值就会从上一次的地方接着往下遍历返回。直到子节点全部遍历完，Filter也就遍历完了。

### Values

values用于插入的时候提供输入数据，没有子节点，自己的`ValuesPlanNode`中存储了所有要插入的数据。

由于可以插入多行数据，所以我们的成员变量中要加入一个`cursor_`表示该返回哪一行数据了。

同样的原因，我们的`Init`函数就需要初始化这个`cursor_`为`0`。

在执行`Next`时，我们通过`cursor_`访问`plan_->GetValues()`提供的数组，然后将其值组成一个新的`Tuple`进行返回即可。
## Task 1

### SeqScan

这里我们要实现遍历`Table`的功能。

需要注意的成员变量是`iter_`，他是一个`TableIterator`的指针，能够让我们遍历一个Table。来看成员函数

- `Init`，首先我们当然是要初始化迭代器，这个迭代器在`exec_ctx_`中找到`catalog`，再找到`table_info`，再找到其迭代器保存下来即可。
- `Next`，按照一般想法来说，只需要判断`iter_`是否遍历完，若没完就`GetRID`和`GetTuple`，然后迭代器自增就完了。但是这里的SeqScan和数据库支持一些特性。首先是`Table`中的数据在删除时不会直接从内存中移除，而仅仅是打上了一个已被删除的标签。另外是为了优化，`Filter`中的判断条件允许直接在`SeqScan`中进行判断。所以，我们建立一个循环，不断自增迭代器，直到第一个没有被删除，并且满足条件的数据。然后保存数据准备返回，之后再自增迭代器准备好下一次调用。

### Insert

插入的节点总是包含一个能够提供值的子节点。插入一个数据不仅要将数据本身插入到`Table`中，还要将`Index`进行更新（即Project 2）所实现的。回忆一下，我们放在可扩哈希里的是数据的Hash和其RID，我们将数据插入`Table`后就会得到这个RID。

来看成员函数

- `Init`，首先还是要初始化子节点，为了便于处理，我在成员变量中添加了一个标记，来确认该插入操作之前是否进行过，防止重复执行`Next`，这里对这个标记进行初始化。
- `Next`，首先还是通过`exec_ctx_`一路获取到`catalog, table_info, index_infos`。插入到`Table`的一条数据分为两部分，即`TupleMeta`和`Tuple`本身，`TupleMeta`存储了数据元信息，在Project3中我们只会用到标记是否删除的功能，所以我们需要将`is_delete_`设置为`false`。然后，我们调用子节点的`Next`来获取一个`Tuple`，然后通过`table_info->table_->InsertTuple`进行插入，其返回一个RID。之后我们调用该`Tuple`的`KeyFromTuple`函数来获取Key，然后调用`index_info->index_->InsertEntry`来插入到哈希表中。反复调用子节点的`Next`来获取数据并插入，直到全部插入。最后本`Next`返回的Tuple是一个`INTEGER`类型，表示一共插入了多少条数据。

### Update

这个节点会修改已存在`Tuple`的值。同样的，最后返回的Tuple是一个`INTEGER`类型，表示一共修改了多少条数据。该节点会修改子节点提供的所有值，所以当局部修改一个`Table`时，其子节点通常会是一个`Filter`，或者优化后是一个带有谓词的`SeqScan`。项目建议我们通过删掉老的，增加新的来实现修改。

- `Init`，基本和`Insert`节点一样。
- `Next`，前面也提到，先删除，后插入，我们就来讲一下删除怎么删。首先从子节点的`Next`获取到`Tuple`及其RID，然后我们使用`table_info_->table_->UpdateTupleMeta`，将这个RID对应的meta数据的`is_deleted_`改成`true`，然后别忘了要同步修改哈希表，这里调用`index_info->index_->DeleteEntry`。插入的数据来源于`plan_->target_expressions_`，计算其表达式组合成`Tuple`。

### Delete

删除节点就没什么好讲的了，无非是`Update`节点进行删除后不进行插入。其他都一样。

### IndexScan

显然的，当我们需要查找主键等于某个特定值的`Tuple`时，一个一个翻找`Table`就太慢了。而我们的可扩哈希表就是为了对付这种查找任务的。

在本项目中，SQL语句为``SELECT FROM <table> WHERE <index column> = <val>``时，会调用到这个节点，并且`Index`一定是`HashTableIndexForTwoIntegerColumn`类型，可以安全的进行强制类型转换。本项目也是非常简化的，不支持数据拥有相同的Key，所以只会找到一个符合条件的数据。

- `Init`，由前所述，我们首先`htable_ = dynamic_cast<HashTableIndexForTwoIntegerColumn *>(index_info_->index_.get());`，然后通过`plan_->pred_key_->val_`和`htable_->GetKeySchema()`构建一个Key，之后通过`htable_->ScanKey`来获取所有的符合条件的数据（事实上只有一个，但为了符合接口还是将其保存到`result_`的vector中）。
- `Next`，和`SeqScan`就比较像了，只不过不再遍历`Table`，而是直接遍历`result_`了，也要注意处理被删除的情况、处理谓词。

### Optimizing SeqScan to IndexScan

正如前面所说的，当`WHERE`只要求一个列等于给定值的条件，并且这一列已经被索引，就可以用`IndexScan`替代`SeqScan`，而这个“替代”是需要我们实现的。

当使用`SELECT * FROM t1 WHERE v1 = 1;`时，planner会生成两个节点，分别为`Filter`和`SeqScan`。在优化为`IndexScan`之前，操作就会被优化为带有谓词的`SeqScan`。所以我们在优化的时候是寻找包含特殊谓词的`SeqScan`。

原项目已经实现了一些优化措施可以给我们参考，优化到`IndexScan`的函数为`OptimizeSeqScanAsIndexScan`，流程如下（TODO：详解）

- 获取函数输入的`plan`的子节点，因为我们是从根节点依次往下递归优化的，所以我们首先要对子节点调用`OptimizeSeqScanAsIndexScan`，然后将优化后的节点保存下来
- 调用`plan->CloneWithChildren`，将优化后的子节点传入，获得本节点优化后的版本，即`optimized_plan`
- 前面也说过，优化是在树上递归执行的，如果本节点并不是`SeqScan`，那么直接返回即可。
- 如果`optimized_plan->GetType() == PlanType::SeqScan`，那么就要检查其谓词是否符合条件。首先通过类型转换将`optimized_plan`转换为`SeqScanPlanNode`，然后调用`.filter_predicate_.get()`得到其谓词表达式。
- 如果谓词为空或者不为`ComparisonExpression`，则直接返回。
- 获取谓词的左右操作数，确保一个是`ColumnValueExpression`，另一个是`ConstantValueExpression`，否则直接返回。
- 通过`catalog_.GetTableIndexes`获取`index_info`，当`column_expr.GetColIdx()`等于`index_info`的`key_attrs`时，就可以构造一个新的`IndexScanPlanNode`进行返回了。否则还是返回`optimized_plan`。

## Task 2



# 附录

## Tuple


# TODO

[https://github.com/kegalas/bustub/blob/project3/src/execution/insert_executor.cpp](https://github.com/kegalas/bustub/blob/project3/src/execution/insert_executor.cpp)中似乎`cnt++;`没有考虑到插入失败的情况

project4中检查clang-tidy的建议。
