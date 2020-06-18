---
title: 缓存:内存模型
date: 2018-03-15
tags:
- Cache
---

<!-- TOC -->

- [内存模型](#内存模型)
- [原理](#原理)
- [存在的问题](#存在的问题)
- [MESI协议](#mesi协议)

<!-- /TOC -->

# 内存模型

![](https://gitee.com/LuVx/img/raw/master/cpu_cache.png)

高速缓存:即Cache, 介于CPU与内存之间的高速存储器, 通常由SRAM(Static Ram, 静态存储器)构成,
容量比内存小的多但是交换速度却比内存要快得多, 但不如CPU快.

SRAM:static random access memory,一种具有静态存储功能的存储器,不需要刷新电路就能保存内部存储的数据,但其集成度较低,和DRAM相比,同样的容量需要更大的体积, 这也是限制缓存大小的一个因素.

具体细分还包括一级缓存(L1 Cache), 二级缓存(L2 Cache), 在一些高端CPU上还存在三级缓存, 每级缓存的数据命中率大约在80%.

每一级缓存中所储存的全部数据都是下一级缓存的一部分，这三种缓存的技术难度和制造成本是相对递减的，所以其容量也是相对递增的.

> 上图中可看出:一级缓存是由一级数据缓存(Data cache,D-Cache)和一级指令缓存(Instruction cache,I-Cache)组成
>
> I-Cache:用于暂时存储并向CPU递送各类运算指令
>
> D-Cache:暂时存储并向CPU递送运算所需数据

# 原理

系统会将CPU在近几个时间段经常访问的内容存入高速缓冲, 在处理数据时, 首先从高速缓存中找, 找不到才会去处理速度较慢的内存中去找,同时将这个数据所在的数据块加入缓存.

这样的读取机制使得CPU需要的数据中的80%都能够存在于缓存中,减少了去主内存中读取数据的频率.

# 存在的问题

现代多核CPU中, 每核各有独自的L1, 共有L2的情况也很常见, 因此也就可能出现多线程中的每个线程由不同的处理器处理, 不同的处理器处理各自的cache, 而各自cache中存在同一个数据时, 有可能缓存不一致的情况, 这种情况下就需要使缓存保持一致(缓存一致性).

缓存一致方案通常有2个:
1. 在总线加锁的方式
2. 缓存一致性协议

CPU和内存通信是通过总线, 在总线上加锁, 可以保证某一块内存同时只能被一个处理器使用, 但这样会阻塞其他处理器访问内存, 影响性能.

缓存一致性协议是在硬件层面实现的, 如Intel 的MESI协议, 处理器写数据时, 如果遇到共享数据(多处理器共用)时, 会写入内存并将其他处理器缓存中的该数据的拷贝置为无效, 那么其他处理器在使用数据时, 将不得不重新从内存中读取.

所以在Java等程序中, 设计多线程的变量多用`volatile`关键字修饰, 该关键字表示被修饰的变量被修改后, 立即被写入内存, 其他处理器在处理此变量时直接从内存中读取.

# MESI协议

Modified, Exclusive, Shared, Invalid的首字母缩写,同时也是缓存的4种状态:

* Modified:已修改,已经被所属处理器修改,同时对应数据在其他处理器中的缓存变为失效状态.
* Exclusive:独占,和主内存的数据一致,当一个处理器占有某个内存地址的缓存,其他处理器就不能获得同一地址的缓存
* Shared:共享,和主内存的数据一致,但只能读不能写,因此多处理可以拥有内存地址的缓存
* Invalid:失效,内容过时,会被忽略这种状态的缓存

可以看出只有缓存处于M或E的状态下,处理器才能进行写操作.

MESI大致的意思是：若干个CPU核心通过ringbus连到一起。每个核心都维护自己的Cache的状态。
如果对于同一份内存数据在多个核里都有cache，则状态都为S（shared）。
一旦有一核心改了这个数据（状态变成了M），其他核心就能瞬间通过ringbus感知到这个修改，从而把自己的cache状态变成I（Invalid），并且从标记为M的cache中读过来。
同时，这个数据会被原子的写回到主存。最终，cache的状态又会变为S。

作者：大宽宽
链接：
https://www.zhihu.com/question/65372648/answer/415311977
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

MESI是保持一致性的协议。它的方法是在CPU缓存中保存一个标记位，这个标记位有四种状态:
* M: Modify，修改缓存，当前CPU的缓存已经被修改了，即与内存中数据已经不一致了；
* E: Exclusive，独占缓存，当前CPU的缓存和内存中数据保持一致，而且其他处理器并没有可使用的缓存数据；这个状态跟modified很类似，只是该状态下，cache的数据已经同步到主存了，所以即使丢弃也无所谓。
* S: Share，共享缓存，和内存保持一致的一份拷贝，多组缓存可以同时拥有针对同一内存地址的共享缓存段；
* I: Invalid，失效缓存，这个说明CPU中的缓存已经不能使用了。

CPU的读取遵循下面几点：
* 如果缓存状态是I，那么就从内存中读取，否则就从缓存中直接读取。
* 如果缓存处于M或E的CPU读取到其他CPU有读操作，就把自己的缓存写入到内存中，并将自己的状态设置为S。
* 只有缓存状态是M或E的时候，CPU才可以修改缓存中的数据，修改后，缓存状态变为M。

这样，每个CPU都遵循上面的方式则CPU的效率就提高上来了。