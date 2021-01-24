Libco阅读笔记
===========
个人阅读笔记，可能有理解错误的地方，望指出  
进度 40%  

### 前置知识点
基本了解epoll、poll区别以及接口使用  
hook系统调用  


### 基本用法
| 函数名              | 作用|
| :----------------- | :----|
| co_create          | 创建一个协程，用法和pthread_create差不多 |
| co_resume          | 恢复指定协程 |
| co_eventloop       | 进入协程的epoll循环 |
| co_yield           | 从当前协程切出去，由co_yield_env实现 |
| co_yield_env       | 在当前协程环境做切换 |
| co_poll            | 用法类似poll接口，本质是epoll实现 |
| co_enable_hook_sys | hook系统函数的开关，开启后使用read、write等系统调用会使用libco的实现 |


### 代码简要
| 文件名               | 作用|
| :-------------------- | :----|
| co_closure.h          | 定义了一些宏 |
| co_epoll.h/cpp        | 简单封装了一层epoll |
| co_hook_sys_call.cpp  | hook各种系统调用的实现 |
| co_routine_inner.h    |  |
| co_routine_specific.h |  |
| co_routine.h/cpp      | 协程的主要实现 |
| coctx_swap.S          | 协程的灵魂，汇编交换寄存器 |

###流程描述
最核心的步骤就是切换两个执行中的函数的调度。
保存当前运行的函数的步骤是先获取当前函数的栈地址，由于函数运行是栈分配的空间，直接在co_swap开头分配一个变量，就能获取到函数栈顶指针。
如果是使用了shareStack，知道了栈顶和栈底后，由于内存是连续的，可以直接把整个函数栈内存保存起来，注意这里其实开销可能比较大，所以尽量避免使用栈生成大内存。
关闭shareStack的情况下，每个协程栈都是自己独立的内存，也不需要做拷贝。
libco用了一个巧妙的方法，在堆上分配一块连续的内存，直接塞给寄存器，也就是说协程的函数是运行在堆上的。



### 疑问
Q1：协程和epoll的关系  
A1：感觉关系不大，协程的实现本身和epoll无关，协程核心在维护函数栈和切换上下文。但是libco库hook系统read和write函数依赖epoll来判断什么时候可读可写，或者超时  
Q2：为什么libco既要用epoll又要用poll  
A2：最终都是用epoll，hook了poll函数里面，在co_poll_inner的实现里将poll转为epoll。用poll感觉是为了统一对外接口  
Q3：交换寄存器信息的时候，我们知道寄存器是一个CPU一套，一个CPU可以跑多个进程/线程。在交换的过程中，如果CPU把当前进程/线程切走了，这时候会不会有异常
A3：不会，切换进程/线程再切换回来的时候，还是保持和切换前一样的寄存器信息，切换到其他CPU也不会有异常
Q4：shareStack的作用是什么，为什么不用动态扩展内存池来保证每个协程的栈内存是独立的。开启shareStack后，还得保存栈内存到save_buffer里，开销不会很大吗