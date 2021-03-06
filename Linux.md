1. 僵尸进程和孤儿进程是什么，有什么危害
    
    1) 僵尸进程：一个进程使用fork创建子进程，如果子进程退出，而父进程并没有获取子进程的状态，那么子进程的相关信息仍然保留在进程表中。这种进程称之为僵尸进程。
    僵尸进程会一直占用系统资源，造成资源浪费。大量的僵尸进程可能造成无法创建新进程。
    2) 孤儿进程：父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)领养，所以孤儿进程没有实质性的危害。

2. 什么是守护进程

    1) 守护进程也称精灵进程(Daemon),是运行在后台的一种特殊进程，它独立于控制终端，在后台运行，不能直接和用户交互。与会话无关、会话关闭仍然可以运行。在系统启动时经常以 root 权限运行。
    2) Linux 下查看守护进程
        ps axj, TPGID == -1 的进程
    3) 创建守护进程
        守护进程需要摆脱原会话，摆脱原进程组，摆脱终端控制。创建守护进程一般步骤:
        - 调用 fork 创建子进程，父进程直接退出，使 shell 认为程序已经运行完毕，达到脱离终端控制，同时保证子进程不是进程组的组长，防止调用 setsid 时出错。
        - 在子进程中调用 setsid 创建新的会话
        - 再次 fork 一个子进程，父进程退出，保证子进程不是会话首进程，从而保证子进程不能与终端关联。
        - 调用 chdir 使根目录成为子进程的工作目录
        - 调用 umask 更改进程的文件掩码
        - 忽略 SIGCHLD 信号，避免僵尸进程
        - 在子进程中关闭不需要的文件描述符

        或者调用 daemon 函数，使用 nohup 命令也可以达到类似守护进程的效果

3. 进程和线程的区别

    进程是资源分配基本单位，线程是进程的一个实体，是 CPU 调度的基本单位。进程的创建、销毁、切换开销较大。每个进程都有自己的虚拟地址空间，线程共用虚拟地址空间。进程更加稳定，一个进程崩溃后，其他进程无影响。一个线程崩溃后，整个线程组都会崩溃。进程间通信需要使用额外的手段，线程间通信可以使用全局变量、信号，但是需要同步互斥机制。

4. 进程间通信

    1) 管道 (内核提供的一块内存)
        - 半双工，全双工要开两个管道
        - 只能在有亲缘关系的进程间使用
        - 面向字节流
        - 生命周期随进程
    2) 命名管道(FIFO)
        - 可以用于任意进程,速度慢
    3) 消息队列 (内核提供的链表)
        - 面向数据报，需要考虑上一次没读完的问题
        - 生命周期随内核，进程结束，消息队列仍存在
    4) 共享内存
        - 速度最快，但需要同步
    5) 信号量
        - 不能传递复杂数据，主要用来同步
    6) socket
        - 可以跨主机通信

5. 多线程怎么实现同步与互斥

    1) 互斥锁
    2) 条件变量
    3) 信号量

6. 多线程间共享的数据和私有的数据

    1) 共享
        - 同一虚拟地址空间
        - 文件描述符表
        - 信号的处理方式
        - 当前工作目录
        - 用户 id 和组 id
    2) 私有
        - 线程 id
        - 寄存器
        - 栈
        - errno
        - 信号屏蔽字
        - 线程优先级

7. 测试大小端

    大端模式，是指数据的高字节保存在内存的低地址中。小端模式，是指数据的高字节保存在内存的高地址中，而数据的低字节保存在内存的低地址中。
    1) 方法一
        ``` c
        int main(int argc, char *argv[]) {
            int i = 0x12345678;
            char *c = (char *)&i;
            printf("%x \n", *c);
            return 0;
        }

        ```
    2) 方法二
        ``` c
        typedef union {
            int i;
            char c;
        } Node;
 
        int main(int argc, char *argv[]) {
            Node node;
            node.i = 0x12345678;
            printf("%x\n", node.c);
            return 0;
        }
        ``` 
        
8. Linux 开机过程

    1) 开机 BIOS 自检，加载硬盘
    2) 读取 MBR， 进行 MBR 引导
    3) grub 引导菜单(Boot Loader)
    4) 加载内核 kernel
    5) 启动 init 进程，设定运行级别
    6) 执行 rc.sysinit
    7) 启动内核模块，执行不同级别的脚本文件
    8) 执行 /ect/rc.d/rc.local
    9) 进入系统登录界面

9. ps aux 和 ps -ef 的区别

    ps aux 是 BSD 风格的，ps -ef 是 System V 风格的，ps aux 会发生截断

10. 内核态和用户态的区别

    通过处理器状态和特权级别，操作系统可以做一般程序无法做的事，使操作系统和普通程序隔离，帮助操作系统实现控制。
    - 内核态: CPU可以访问内存所有数据, 包括外围设备, 例如硬盘, 网卡. CPU也可以将自己从一个程序切换到另一个程序
    - 用户态: 只能受限的访问内存, 且不允许访问外围设备. 占用CPU的能力被剥夺, CPU资源可以被其他程序获取
    用户态向内核态切换
        a. 系统调用
        b. 异常
        c. 外围设备中断 

11. select poll epoll

    - select 
        1) select 是跨平台的
        2) 使用位图表示 fd_set
        3) 监听的文件描述符数量有限，取决于 sizeof(fdset)
        4) 每次需要把 fd 集合从用户态拷贝到内核态
        5) 每次需要遍历所有的 fd,采用轮询的方式
        6) 需要手动设置 fd 集合，使用起来不方便

    - poll
        1) 使用 pollfd 指针来表示 fd 状态
        2) poll 没有监听的文件描述符上限
        3) poll 也是采用轮询的方式，监听的描述符较多时，效率会降低

    - epoll
        1) 监听的文件描述符数量没有上限，所有的文件描述符通过红黑树管理
        2) 以事件驱动，描述符就绪时，采用回调机制，把就绪的文件描述符加入就绪队列(双向链表)中
        3) 采用内存映射机制，内核直接调用 mmap 把就绪队列映射到用户态，避免了内存的拷贝

12. epoll 的工作方式

    1) 水平触发 LT
        - 当文件描述符就绪时可以不立即处理，或只处理一部分
        - 当缓冲区中有数据时，epoll_wait 仍会返回
        - 支持阻塞 io, 和非阻塞 io
    2) 边沿触发 ET
        - 文件描述符就绪时，必须立即处理
        - 缓冲区中有数据，epoll_wait 也不会返回，也就是只有一次处理机会
        - 只支持非阻塞 io，就绪时需要一直读，直到出错
        - ET 模式更加高效，不会阻塞在 io 中，并且减少了 epoll_wait 的返回次数

13. Linux 常用指令

    ls mkdir cd chmod chown cp ps netstat ss grep 
    
14. 软硬链接的区别

    为了解决文件的共享问题，引入了软硬链接。
    - 硬链接是有着相同 inode,文件名不同的文件，硬链接使用引用计数机制，每增加一个硬链接引用计数加一，硬链接只能对已存在的文件使用，也不能对目录使用，不能跨越文件系统。
    - 软链接有自己的 inode 和数据块，只是数据块存储的内容是其他文件的路径。软链接可以对文件和目录使用，可以对不存在的文件和目录使用，可以跨越文件系统，有自己的 inode 和 数据块，有自己的文件权限和属性
    软链接可以用来当作快捷方式，硬链接可以作为备份
    
15. 进程的虚拟地址空间分布

    从低地址到高地址依次是代码段、已初始化数据段、未初始化数据段、堆、内存映射区、栈、命令行参数和环境变量
    
16. 什么是死锁，死锁产生的条件，如何避免

    死锁 (deallocks) 是指两个或两个以上的进程(线程)在执行过程中，因争夺资源而造成的一种互相等待的现象。
    死锁产生的原因
        - 系统资源的竞争
        - 进程运行推进顺序不当
    死锁产生的条件
        - 互斥
        - 请求和保持
        - 不可抢占
        - 循环等待
    死锁的预防
        - 打破请求和保持，如果没有获得所有资源，就释放当前资源。
        - 打破不可剥夺，资源可以被其他进程抢占
        - 打破循环等待，撤销一些进程 
    死锁的避免
        银行家算法: 每一个线程进入系统时，它必须声明在运行过程中，所需的每种资源类型最大数目，其数目不应超过系统所拥有每种资源总量，当线程请求一组资源系统必须确定有足够资源分配给该进程，若有在进一步计算这些资源分配给进程后，是否会使系统处于不安全状态，不会（即若能在分配资源时找到一个安全序列），则将资源分配给它，否则等待

17. 静态链接和动态链接的区别
    在链接时将目标文件与库一起链接打包到可执行文件中，这种方式就静态链接,动态链接不需要打包到源文件，在运行时加入。