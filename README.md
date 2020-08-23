Libco阅读笔记
===========
个人阅读笔记，可能有理解错误的地方，望指出
进度 20%

### 前置知识点
基本了解epoll、poll区别以及接口使用
hook系统调用


### 基本用法
| 函数名              | 作用|
| :----------------- | :----|
| co_create          | 创建一个协程，用法和pthread_create差不多 |
| co_resume          | 恢复指定协程 |
| co_eventloop       | 进入协程的epoll循环 |
| co_yield           | 从当前协程切出去 |
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


### 疑问
Q1：协程和epoll的关系
A1：感觉关系不大，协程的实现本身和epoll无关，协程核心在维护函数栈和切换上下文。但是libco库hook系统read和write函数依赖epoll来判断什么时候可读可写，或者超时
Q2：为什么libco既要用epoll又要用poll
A2：最终都是用epoll，hook了poll函数里面，在co_poll_inner的实现里将poll转为epoll。用poll感觉是为了统一对外接口