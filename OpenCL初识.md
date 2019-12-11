

## 愿景

写一次，可以在各个平台上运行。

支持

- Server GPU (nvidia , AMD) ; 

- 移动端的GPU （adreno, mali , powervr  etc）
- CPU , DSP , FPGA ,只要有一个驱动，就可以跑在opencl上



## 概念

![image-20191206114258115](/home/linjinhao/.config/Typora/typora-user-images/image-20191206114258115.png)

- 平台：选择一个平台在上面跑，可以是cpu的集群,gpu集群
- Device : 选择一个平台之后 ，再选择一台设备，在上面跑
- Context :  操作CPU ，GPU需要的资源都在Context 上 ， 比如CPU，GPU共享的一些数据
- Program : 一段代码 ， 编译到特定的targit上，比如CPU的 binary  ， GPU的binary
-  Kernel :  一段Program可能有很多个kernel ,比如 add ,理解成外界的入口函数。Kernel里有两个很重要的概念：
  - LWS：把很多小的task打包成一个一个小的块，然后丢到一个PE(process element)里，LWS描述多少任务放到PE里 （依赖于平台 ，不同平台 的LWS参数不一样）
  - GWS：到底有多少个任务 （比如一个m*n矩阵的conv，进行分块计算 ，分块的大小确定GWS的大小，而LWS的大小可能和设备性能相关）
- Memobj:opencl有两种Memobj-->
  - Buffer : 有一个LR cache ，
  - Image : 有一个LE cache , 如果数据重用率高的话，尽量放在Image里
- CommandQueue : 执行队列 ， 有两种Queue -->
  - IO : inorder , 更常用 
  - OoO : out of order



## Kernel是怎么编译的

![image-20191206130422956](/home/linjinhao/.config/Typora/typora-user-images/image-20191206130422956.png)

1. 打包 。将CL的代码进行打包 ， Program只是进行打包， .cl文件还是没有编译过的。Program也可以通过预编译好的binary来创建 

2.  ClBuildProgram。生成Program(IR) ,对于高通的IR来说，这就是一个可执行代码；

   但在mali上，需要到ClCreateKernel时才会生成可执行文件 。

   {**为什么ClCreateKernel会非常非常慢**？因为build program时出来的不是一个真正的binary，解决方法是在Program模块里，先对所有的Kernel进行编译，序列化 }

3. Enqueue。把编译好的kernel发过去



## OpenCL的优化

- Memory
  - 向量化（和CMD是一样的）和 coalescing（比如两个task，task1处理第一个元素 ，task2处理第二个元素，coalescing将相临两个task items的内存访问进行合并 ）
  -  Image(相对于buffe有好处，Buffer比较灵活，Image需要对齐）
    - 有L1 cache, 
    - 有bounary checking , 对边界外的自动补零 ，
    - 有内嵌的bilinear filtering ，对实现一些work操作有优势
    - 有两个不同的内存访问通道，TP
  - Global memory , Local memory , Private memory , Constant memory(和设备相关)
- Zero Copy
  - 移动端的GPU的device内存和Host的内存是共享的, 如果要实现内存隔离，cuda是做不到的，opencl mali上是可以的
  - 方法 map/unmap (创建buffer时，将内存分配到Host可访问到的地方，然后执行map，就可以直接读取了，不会触发一次copy) （但有一个问题：需要考虑到flush 和 cpu cache 的实现，就是它有可能把数据破坏）
  - ION ， Android特有的 ， 实现不同设备的共享内存（比如一个程序执行在GPU上，一个程序执行在DSP上），使用ION的接口分配一段内存，可以直接给GPU和DSP用
- WorkGroup size
  - GWS 和LWS对性能影响特别特别大 ， 所以怎么进行调优
- Data type and bit width 
  - 尽量用short ，不要用int  
- Math functions
  - 尽量用naive函数 or fast-math ，会影响精度 ，但是快
  - 尽量少用整数除法 和 取模 （高通上浮点性能是整型的10倍）

## 例子

![image-20191206141831642](/home/linjinhao/.config/Typora/typora-user-images/image-20191206141831642.png)

![image-20191206141846427](/home/linjinhao/.config/Typora/typora-user-images/image-20191206141846427.png)

![image-20191206142320976](/home/linjinhao/.config/Typora/typora-user-images/image-20191206142320976.png)

矩阵乘法的三种优化：

- 分块
- Image
- interleave（优势：内存连续 ；劣势：增加了一个额外的kernel开销）



## OpenCL in megbrain

-  内存管理 ：
  - 要求内存的指针访问是可连续的（opencl通过buffer 和image封装了内存管理，两个buffer是不可以相加的，megbrain里加了一层hashmap，通过映射，使之能完成连续的功能 ）
-  CompNode Sequence Recorder
  - 发一个kernel 大概需要 200ns ， 当网络比较小，kernel数比较多时，开销太大
  - 方法 ：第一次跑的时候，直接纪录这个kernel在哪里，是哪个kernel，访问啸块内存，第二次执行时，直接把这个kernel放到任务队列里，只是input变了，其它都不变
  - 连续发Kernel会更快
- opencl 有多个阶段的编译，所以设计了不同阶段的cache ， 
  -  program cache 
  - kernel Cache 
  - Image Cache(这部分内存，megbrain没办法知道用了多少)



## Algo Searcher

- 同一算法在不同设备上性能不一样，在某个 device上先进行算法选择 ， 选择这个模型在这个设备上的最优算法。

- 怎么去选择：
  - brute force 
  - Searching Mode , 有两种模式
    - FAST  ， 预先训练好的一棵决策树 ， 在一些特定平台上定好一些规则 ，比如在高通上不跑image , 在mali上interleave比delacte算法更慢
    - FULL
- 使用：
  - 离线搜参
  - 在线应用这些算法

​	

## NHWCD4 Layout

- 当有一个Layout , 需要把buffer中的数据拷到image，为什么不在创建buffer时对w进行对齐。

- NHWCD4在 WC这维是对齐的，比如128对齐的话，我们在后面补0对齐

- image是rgba的格式  ， 所以C一维应该 是4的倍数，如果不是则需要对齐

![image-20191206160854074](/home/linjinhao/.config/Typora/typora-user-images/image-20191206160854074.png)

- 我们一般分块的时候 是这么分的：N和H 是某一个的分块，W 是某一个分块 ，C是某一个分块
- （OC/4,FH，FW，IC，4）VS（0C/4,IC，FH，FW，4）,Filter的访问是连续的