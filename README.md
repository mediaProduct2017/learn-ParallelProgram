# learn-ParallelProgram

## GPU

与parallel programming相对的是serial programming，前者是并行程序，后者是串行程序。
 
在服务器前端的serial programming中，可以采取异步的编程方式，也就是程序各部分的结果可以相互独立展示，其中一部分的结果出错不影响整个页面的展示。
 
编程中最重要的两部分是data和code，或者说是data store和data process，涉及到的最重要的两个系统是数据库系统和计算系统。

数据库系统本质上是两个部分，一部分是硬盘文件，另一部分是硬盘与内存数据交互的程序。对于单机（单个控制系统、单个硬盘）的数据库系统，最常用的是mysql，最常用的语言是SQL。为了解决要存储的数据极大的问题，目前多采取把多台计算机、多个硬盘并行的方式，也就是分布式的方式，目前最常见的分布式数据库系统是hadoop。分布式数据库系统所存在的物理介质是a collect of machines known as a cluster （more machines can be added to the cluster when data grow）. 对于hadoop，所使用的硬盘系统是HDFS，硬盘与内存数据交互的方法是MapReduce。

Map就是在储存数据的单机上完成单机的处理，也就是one to one的并行方式，Reduce就是把所有单机的处理结果汇总到一台单机上来，也就是all to one的并行方式。硬盘与内存数据交互的MapReduce可以用C语言来写，但写起来很费事。有现成的工具可以帮助用简单的编程完成MapReduce，比如Hive和Pig。在Hive中，可以使用SQL语言来完成硬盘与内存数据交互（底层还是在做MapReduce. Hive interpreter turns the SQL into MapReduce code which then runs on the cluster）。Pig用的不是SQL，但也是一种很简单的脚本语言，比直接用C语言来写MapReduce要简单的多。Hive和Pig本质上用的仍然是MapReduce的方法，但多了一步把其他语言转换为C语言的过程，很多时候速度是很慢的。Impala用的也是SQL语言，但思路上并不是转换为C语言的MapReduce code，而是直接作用于HDFS文件（optimized for low latency queries, 比较注重查询的速度而不是通量）。Hive注重的是通量（optimized for long batch processing jobs）。还有一些工具是帮助加速或方便把数据load into the HDFS file system。

sqoop就是帮助把数据从SQL数据库中导入到HDFS file中。flume做的事把其他应用产生的数据导入到HDFS file中。HBase is a real time database，和impala是类似的。Hue is a graphical front end to the questor. Oozie is a workflow management tool. Mahout is a machine learning library. 

![Hadoop ecosystem](images/CDH.png)

对于计算系统，有三种办法来增加计算力，一是增加单个processor的计算力，二是增加单个chip上processor的数量，三是把多个chip并行起来。单个processor的计算力目前已经进入瓶颈期，增加单个chip上processor的数量是目前的主流，速度还是要比把多个chip并行起来快很多。多核电脑本质上就是在一个cpu chip上配有多个processor. 当然，为了在最大程度上增加计算力，可以把二和三结合起来，既在chip上放置很多的processor，又把多个chip并行起来。

在分布式系统中，可以这样处理：首先是cluster集群的Map。在每台单机上，把HDFS数据库中的数据读入到cpu内存中，然后转入到gpu内存中，经由gpu计算后，再转入到cpu内存中，存入到单机HDFS数据库中。然后是数据交互的MapReduce。把单机HDFS数据库中的数据先处理再归一到一台机器上，呈现最后的结果。

CPU和GPU都是把多个processor放置在chip上，完成计算，但二者的设计目标是不同的。CPU的控制系统更加复杂，目标是高速度、低延迟，相当于跑车。GPU的控制系统更加简单，适合计算简单的重复任务，目标是高通量，相当于大客车。对于机器学习分析数据，追求的是高通量，使用GPU相当合适。由于控制系统简单，GPU的chip上可以放置比CPU多得多的processor（CPU如果放置这么多的processor的话，控制系统根本没法合理安排）。

GPU最初的设计是用来计算处理图像的，图像处理具有这样高通量的特性。当然，除了图像处理，还有机器学习等多种应用需要高通量的特性。GPU之外，还可以设计其他高通量的处理器，比如专门为机器学习优化过的高通量处理器（GPU是专门为图像处理优化的高通量处理器）。

CUDA是GPU的操作系统，就好像hadoop是分布式系统的操作系统一样。可以按CUDA的方式用C语言编写程序，用CUDA compiler编译后，控制GPU的计算。在CUDA编程中，首先设好kernal，是单个thread所要完成的任务，是GPU所要完成的任务。然后设置CPU的host memory，然后把CPU的数据转入GPU中（一般转入global memory，由CPU控制）。之后，由CPU启动kernal。kernal完成计算后，再由CPU把GPU中的数据转到CPU的host memory中。

Parallel programming有多种communication patterns, 前面提到的map和reduce是其中两种，map is one to one, reduce is all to one. Besides, gather is many to one, scatter is one to many. Stencil can be finished by gather or scatter. Transpose is like matrix transpose for an array. 

GPU计算的基本物理单位是SM（processor）, block, thread。kernal是功能单位。GPU memory分为local，shared，global。速度上local > shared >> global。为了使不同的thread相互配合，需要synchronization和barriers。还可以使用atomic operation来使得不同的thread逐次执行。

Effieicent GPU programming:

1. Access memory faster
    * move data to local or shared memory
    * coalescing global memory, that is, make adjacent threads access continuous memory
2. Avoid thread divergence such as branches and loops
    * if different thread execute different loops, we shouldn't let some threads wait for others


## CPU

[concurrency](https://github.com/arfu2016/nlp/tree/master/nlp_models/concurrent)

线程比进程功能更强大，适合进行内存等层面的数据交流，进程比线程更安全，对于没有数据交流的线程，把他们放到不同的进程中能保证互不干扰。Queue.Queue的话，适合用于不同线程的数据交流，进行过优化，避免了使用lock机制(其实是Queue.Queue帮我们做了lock的事情)，数据产生者、Queue.Queue、数据消费者用的都是同一块内存。redis的话（也可以是其他中间件，比如celery中可用的一系列中间件，如卡夫卡），数据产生者可以把数据存入redis，不同的数据产生者相互之间没有干扰（假设存入redis不需要有规定的先后），可以用多进程实现。数据消费者从redis中消费数据，和数据产生者也没有直接的数据交流，可以用单独的进程实现？实际上涉及到三块内存：数据产生者用的内存（分成了很多小块），redis用的内存（从数据产生者用的内存copy而来，要copy到数据消费者用的内存中去），数据消费者用的内存。可以看出来，用Queue.Queue实现，更省内存，效率更高，用redis实现，更费内存，效率更低（多了copy操作），但更加安全。

网络上的两台电脑：http交互

一个cluster中的多个node：某种通讯方式

多路cpu：是不同的node，不在一块主板上，独特的通讯方式，不能共用一块内存？

多核cpu：一块主板上，主板上的总线通讯，可以共用一块内存

多进程：三块内存，生产者，消费者，中间商

多线程：一块内存，用Queue.Queue来管理内存，或者用代码来主动管理内存


在不同进程中的多个线程

在同一进程中的多个线程

在同一个cpu核上运行的多个线程

在不同cpu核上运行的多个线程

两个处于不同进程中的线程肯定是不同的线程，而且互不干扰，不共享内存；从这个意义上讲，多进程肯定是多线程

多线程有两层意思，一是共享内存的多个线程，二是广义上的多个线程，不管是否共享内存，有可能存在于多个进程中

一个线程肯定只能运行在一个cpu核上，不能运行在多个核上；一个进程是一组线程，虽然多个线程可以运行在多个核上，但一个进程一般只运行在一个核上，不会运行在多个核上


Note the distinction between a program and a thread; the program con- tains instructions, whereas the thread consists of the execution of those instructions. Even for single-threaded programs, this distinction matters. If a program contains a loop, then a very short program could give rise to a very long thread of execution. Also, running the same program ten times will give rise to ten threads, all executing one program.

Each thread has a lifetime, extending from the time its first instruc- tion execution occurs until the time of its last instruction execution. If two threads have overlapping lifetimes, we say they are concurrent. 


Sequential threads

Concurrent threads running simultaneously on two processors

Concurrent threads (with gaps in their executions) interleaved on one processor（一个thread执行时有I/O，忙着读写，cpu就空下了，这时候就可以运行另一个thread，但过一会儿前一个thread是会回来的）

If the computer hardware includes multiple processors, then it will naturally be possible to run threads concurrently, one per pro- cessor. However, the operating system’s users will often want to run more concurrent threads than the hardware has processors. Therefore, the operating system will need to divide each pro- cessor’s attention between multiple threads. In this introductory textbook I will mostly limit myself to the case of all the threads needing to be run on a single processor. I will explicitly indicate those places where I do address the more general multi-processor case.

Whenever a program initially starts running, the computer carries out the program’s instructions in a single thread. Therefore, if the program is intended to run in multiple threads, the original thread needs at some point to spawn off a child thread that does some actions, while the parent thread continues to do others. (For more than two threads, the program can repeat the thread-creation step.) Most programming languages have an application programming interface (or API) for threads that includes a way to create a child thread. 

Fundamentally, most uses for concurrent threads serve one of two goals:
Responsiveness: allowingthecomputersystemtorespondquicklytosome- thing external to the system, such as a human user or another com- puter system. Even if one thread is in the midst of a long computation, another thread can respond to the external agent. Our example pro- grams in Section 2.2 illustrated responsiveness: both the parent and the child thread responded to a timer.
Resource utilization: keeping most of the hardware resources busy most of the time. If one thread has no need for a particular piece of hard- ware, another may be able to make productive use of it.

A third reason why programmers sometimes use concurrent threads is as a tool for modularization. With this, a complex system may be decomposed into a group of interacting threads.

下面这种情况其实适合多进程，不需要共享内存：

In the first case, when you work on a spreadsheet, the two concurrent threads have almost nothing to do with one another, and the op- erating system’s job, beyond allowing them to run concurrently, will mostly consist of isolating each from the other, so that a bug in the web browser doesn’t overwrite part of your spreadsheet, for example. This is generally done by encapsulating the threads in separate protection environments known as processes. (Some systems call processes tasks, while others use task as a synonym for thread.)

下面这种情况适合共享内存、需要调节的多线程，也许也可以用多进程实现，但效率肯定不如多线程高

If, on the other hand, you continue using the browser’s user interface while the download continues, the concurrent threads are closely related parts of a single application, and the operating system need not isolate the threads from one another. However, it may still need to provide mechanisms for regulating their interaction. For example, some coordination between the downloading thread and the user-interface thread is needed to ensure that you can scroll through as much of the page as has been downloaded, but no further. This coordination between threads is known as synchronization.

一个是I/O密集，一个是计算密集

The explanation for the half-hour savings in elapsed time is that the virus scanning program spends most of its time using the disk drive to read files, with only modest bursts of processor activity each time the disk completes a read request, whereas the rendering program spends most of its time doing processing, with very little disk activity. 

Of course, this assumes the operating system’s scheduler is smart enough to let the virus scanner have the processor’s attention (briefly) whenever a disk request completes, rather than making it wait for the rendering program.


SP：stack pointer，储存代码栈在内存中的位置

IP：instruction pointer，储存代码执行到什么位置了

The i386 architecture is also known as the x86 or IA-32; it is a popular processor architecture used in standard personal computer processors such as Intel’s Core, Xeon, and Atom families and AMD’s FX and Opteron families.

线程的同步与协调是通过scheduler进行的，有多种协调的方式，不同的方式所追求的目标也是不一样的。多线程相互切换的时间代价比多进程启动与切换要小，但是，对于多线程来说，缓存是共用的，当切换线程时，之前线程的缓存很可能被覆盖掉了，频繁切换的话，基本相当于没有缓存。多个进程之间相互之间内存是独立的，缓存也是相互独立的。

操作系统涉及到不同的线程通信及协调，不同进程的管理。分布式系统涉及到不同IP主机的管理，以及网络通信。

tensorflow中，variable和placeholder一个是变量（可以通过训练而改变，当然也可以设为不变），一个是系数，但是从储存的角度是类似的，只储存一次，其他都是引用。但是对于constant，是实实在在的存储，有可能有多份。

### concurrent.futures.ThreadPoolExecutor (ProcessPoolExecutor)

[concurrent](https://github.com/arfu2016/nlp/tree/master/nlp_models/concurrent)

对于ThreadPoolExecutor(2)，是新开2个子线程，不管主线程是否被阻塞，不管有几个独立任务等待执行，2个子线程是不变的。如果主线程被阻塞（阻塞调用，比如executor.map(), future.result()等，比如time.sleep(1)，比如调用I/O程序或函数），cpu交替执行两个线程（在必要的时候释放GIL锁）。如果主线程没有被阻塞（使用executor.submit()，在future.result()之前，或者使用的是回调函数来返回结果），没有挂起，主线程继续，两个子线程也在跑，cpu交替执行三个线程。

对于ProcessPoolExecutor(2)，是总共有2个进程（可能跑在两个核上，其中一个核上跑着主进程）。假设进程1是主进程，跑着主线程，该进程中往往还有一个新开任务线程1，一个进程下有两个线程，另一个进程是新开进程，一般只跑着一个线程，新开的任务线程2。如果主线程没有被阻塞，没有挂起，不管有几个独立任务在等待执行，都是只有2个新的任务线程在执行，算上主线程，是两个进程、三个线程在执行。如果主线程被阻塞，此时主线程挂起，但是同时，会在主进程中新开一个任务线程3来代替主线程，这时，就有3个新的任务线程在执行，总的来说，还是两个进程、三个线程在执行。

    executor.map()
    executor.submit()
    future.as_completed()
    future.result()
    future.done()
    future.add_done_callback()
    
