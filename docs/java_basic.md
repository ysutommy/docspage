# 知识点整理

## 一、操作系统与计算机网络

### 1、Linux常用命令

* **文件**
  * **chmod** 修改指定目录或文件权限。chmod 777 1.txt
  * **mkdir** 创建目录
  * **rmdir** 删除空目录
  * **rm -f ** 删除不为空的目录
  * **touch** 创建新文件
  * **vi** 编辑（不存在则创建）文件
  * **mv** **cp** 
  * **scp** 将本地的文件或目录复制到远程服务器
  * **wget** 下载ftp或http文件到本地
  * **cat** **more** **less** **head** **tail**
  * **tail -f** 自动刷新显示文件后n行数据
  * **find** **whereis** **which** **grep**
* **关机和重启**
  * **shutdown** -r 立即重启；-k 发出警告；-h 关机不重启
  * **poweroff** **init** **reboot** **halt** 

> 重点awk工具、top、netstat、grep等命令

### 2、进程和线程

* 进程是系统资源分配的最小单位，线程是程序执行的最小单位
* 进程使用独立的数据空间，线程共享进程的数据空间
* Java程序至少包含2个线程（Main、GC）

#### 2.1 进程调度

进程有5种状态

* **创建**（NEW）
* **就绪**（READY）获得了除处理器之外的一切资源
* **运行**（RUNNING）
* **阻塞**（WAITING）进程正在等待某一事件而暂停运行如等待某资源为可用或等待 IO 操作完成
* **结束**（TERMINATED）

![132501_6PVU_1863332](C:\Users\tommy\Desktop\md\132501_6PVU_1863332.jpg)

进程的调度算法

* **先到先服务（FCFS）** 从就绪队列中选择一个最先进入该队列的进程为之分配资源，使它立即执行并一直执行到完成或发生某事件而被阻塞放弃占用 CPU 时再重新调度

* **短作业优先（SJF）** 从就绪队列中选出一个估计运行时间最短的进程为之分配资源，使它立即执行并一直执行到完成或发生某事件而被阻塞放弃占用 CPU 时再重新调度

* **时间片轮转** 又称 RR(Round robin)调度，每个进程被分配一个时间段

* **多级反馈队列调度（MFQ）**  既能使高优先级的作业得到响应又能使短作业（进程）迅速完成，规则如下：

  * 如果A优先级高于B，则运行A
  * 如果A、B优先级相同，则A、B以RR调度
  * 当任务进入系统时，它被放置在最高优先级队列
  * 一旦工作占用了给定级别的时间分配(不管它放弃了多少次CPU)，它的优先级就降低了
  * 在一段时间之后，将系统中的所有作业移到最顶层队列（防止饥饿现象）

  ![8435e5dde71190ef5858f621c51b9d16fcfa60a8](C:\Users\tommy\Desktop\md\8435e5dde71190ef5858f621c51b9d16fcfa60a8.png)

* **优先级调度** 具有相同优先级的进程以 FCFS 方式执行。可以根据内存要求，时间要求或任何其他资源要求来确定优先级

#### 2.2 进程间通信

进程间通信有三种类型

* **基于共享存储器的通信**
* **基于消息传递系统的通信**
* **基于管理文件的通信**

进程间通信方式包括

* **管道（pipe）** 半双工的通信，数据仅仅能单向流动；仅仅能在具有亲缘关系的进程间使用
* **流管道（s_pipe）** 能够双向传输
* **有名管道（name_pipe）** 有名管道严格遵循**先进先出(FIFO)**，以磁盘文件的方式存在，可以实现本机任意两个进程通信
* **信号（Singal）** 一种比较复杂的通信方式，用于通知接收进程某个事件已经发生
* **消息队列** 
  * 具有特定格式，存放在内核中（匿名管道只存在于内存中；有名管道存在于实际的磁盘介质或文件系统），并由消息队列标识符标识
  * 可实现消息随机查询，不一定要以先进先出的次序读取，也可以按消息的类型读取
  * 克服了信号承载信息量少，管道只能承载无格式字节流集缓冲区受限等缺点

* **信号量（Semaphore）** 是一个计数器，能够控制多个进程对共享资源的访问，常作为一种锁机制
* **共享内存** 映射一段能被其它进程访问的内存，是最快的IPC方式；往往与其它通信机制，如信号量、互斥锁等配合使用，来实现进程间的同步和通信
* **套接字** 不同机器间通信

#### 2.3 线程调度及同步

线程调度包括

* **抢占式调度** 指的是每条线程执行的时间、线程的切换都由系统控制（JVM采用此种方式）
* **协同式调度** 指某一线程执行完后主动通知系统切换到另一线程上执行

线程间同步有3中方式

* **互斥量（Mutex）** 采用互斥对象机制，只有拥有互斥对象的线程才有访问公共资源的权限。因为互斥对象只有一个，所以可以保证公共资源不会被多个线程同时访问。比如 Java 中的 synchronized 关键词和各种 Lock 都是这种机制
* **信号量（Semaphore）** 它允许同一时刻多个线程访问同一资源，但是需要控制同一时刻访问此资源的最大线程数量
* **事件（Event）** Wait/Notify：通过通知操作的方式来保持多线程同步

#### 2.4 协程



### 3、内存管理

操作系统的内存管理主要负责内存的分配与回收（申请：malloc，释放：free），另外地址转换也就是将逻辑地址转换成相应的物理地址等功能也是内存管理做的事情

#### 3.1 内存管理机制

* **连续分配管理方式** 
  * **块式管理** 将内存分为几个固定大小的块，每个块中只包含一个进程。内存利用率小

* **非连续分配管理方式**
  * **页式管理** 把主存分为大小相等且固定的一页一页的形式，页较小，相对相比于块式管理的划分力度更大，提高了内存利用率，减少了碎片。页式管理通过页表对应逻辑地址和物理地址
  * **段式管理** 页式管理虽然提高了内存利用率，但是页式管理其中的页实际并无任何实际意义。 段式管理把主存分为一段段的，每一段的空间又要比一页的空间小很多  。但是，最重要的是段是有实际意义的，每个段定义了一组逻辑信息，例如,有主程序段 MAIN、子程序段 X、数据段 D 及栈段 S 等。段式管理通过段表对应逻辑地址和物理地址
  * **段页式管理** 段页式管理机制结合了段式管理和页式管理的优点。简单来说段页式管理机制就是把主存先分成若干段，每个段又分成若干页，也就是说段页式管理机制中段与段之间以及段的内部的都是离散的

### 3、扩展知识

https://snailclimb.gitee.io/javaguide-interview/#/./docs/c-4%E6%93%8D%E4%BD%9C%E7%B3%BB%E7%BB%9F



### 4、计算机网络

#### 4.1 4/7层网络模型

![1589284663(1)](C:\Users\tommy\Desktop\md\1589284663(1).png)

#### 4.2 TCP协议

TCP是传输层协议，具有以下特点

* 基于链接，双向通信，可靠传输
* 传输基于字节流而不是报文。将数据按字节大小进行编号，接收端通过ACK来确认收到的数据编号，通过这种机制，TCP协议能够保证接收数据的有序性和完整性
* 提供流量控制能力，通过滑动窗口来控制数据的发送速率。滑动窗口的本质是动态缓冲区，接收端根据自己的处理能力，在TCP的Header中动态调整窗口大小，通过ACK应答包通知给发送端，发送端根据窗口大小调整发送的速度
* 提供拥塞机制，以应对网络问题。处理拥塞机制主要用到了慢启动、拥塞避免、拥塞发生、快速恢复4个算法

> 扩展：TCP协议的报文状态；滑动窗口的工作流程；Nagel算法的规则；Nagel和ACK延迟机制配合使用可能会出现delay 40ms超时后才回复ACK的问题

**3次握手建连**

![1589286211(1)](C:\Users\tommy\Desktop\md\1589286211(1).png)

* 三次握手是为了建立**双向**的连接
* 建连时，可能发生SYN洪水攻击。即Client端建立连接后，不回复ACK，导致Server端大量链接处于SYN_RCVD状态，进而影响其他正常请求的建连。可设置 tcp_synack_retries=0加快半连接的回收速度，或者调大tcp_max_syn_backlog来应对少量的SYN洪水攻击

**4次挥手断连**

![1589286626(1)](C:\Users\tommy\Desktop\md\1589286626(1).png)

* 4次挥手是为了双向关闭连接
* 为什么需要等待2倍最大报文段生存时间？
  1. 保证TCP协议的全双工连接能够可靠关闭
  2. 保证这次连接的重复数据段从网络中消失，防止端口被重用时可能产生数据混淆
* 开启tcp_tw_reuse和tcp_tw_recycle能够加快TIME_WAIT的socket的回收；大量CLOSE_WAIT可能是被动关闭一方存在代码bug

####4.3 UDP协议

特点

* **无连接**，不存在连接时延。空间上，无需维护连接状态
* 分组首部开销小，TCP首部20字节，UDP首部8字节
* UDP没有拥塞控制
* 不保证可靠交付
* 面向报文

#### 4.4 HTTP协议



#### 4.5 QUIC





## 二、JVM





## 三、Java基础





## 四、Java高级

### 1、IO





###2、反射



