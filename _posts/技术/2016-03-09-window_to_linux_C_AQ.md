---
layout: post
title: windows迁移linux问题集锦
category: 技术
tags: CPP;Linux
keywords: 
description: 
---

#windows迁移linux问题集锦

1）‘_wcsicmp’在此作用域中尚未声明
```
#ifdef WIN32
#define _tcsicmp        _wcsicmp
#else
#define _tcsicmp        wcscasecmp
#endif
```
2）_stricmp 在此作用域中尚未声明
```
#include <string.h>
```
将_stricmp改成strcasecmp

3）atoi的wchar版本不存在，
```
#define _ttoi       _wtoi
改成使用
#define _tcstol     wcstol
```
4）_atoi64
改成
```
atoll
```

5）‘itoa’在此作用域中尚未声明 或者 ‘_itoa’在此作用域中尚未声明
改成
```
sprintf(buf,"%d",n);

```
6）‘ultoa’在此作用域中尚未声明 或者 ‘_ultoa’在此作用域中尚未声明
改成
```
sprintf(buf,"%ul",n);
```
7）‘ltoa’在此作用域中尚未声明 或者 ‘_ltoa’在此作用域中尚未声明
改成
```
sprintf(buf,"%l",n);
```
8）‘_i64toa’在此作用域中尚未声明
改成
```
sprintf(buf,"%lld",n);
```
9）‘_ui64toa’在此作用域中尚未声明
改成
sprintf(buf,"%llu",n);

10）htonl,htons,ntohl,ntohs 在此作用域中尚未声明
包含
```
#include <arpa/inet.h>
```
11）‘__int64’没有命名一个类型

将__int64 改为 long long
32位为ILP32(int,long,pointer为32位),64位采用LP64(long,pointer为64),其他不变

12)‘struct in_addr’没有名为‘S_un’的成员
```
in_addr ip;
ip.S_un.S_addr = "127.0.0.1";
将ip.S_un.S_addr改为ip.s_addr，如下：
ip.s_addr = "127.0.0.1";
```
13)
```
std::vector<InternalObj<T>* >::iterator iter = used_obj_.begin();
 错误：expected ‘;’ before ‘iter’
```
std::vector<int> testv;
std::vector<int>::iterator iter1 = testv.begin();
以上两句语法正确。

C++规定，引用嵌套模版类的内部类型（如std::vector<T>::iterator），必须显示告诉编译器这是个type而不是variable。
例如
typename std::vector<InternalObj<T>* >::iterator iter = used_obj_.begin();

在VC或Intel Compiler中不会出现这样的问题。
但GCC编译器则会严格按照C++规定认为是个变量。
http://blog.csdn.net/tedious/article/details/6063910

14)
```
boost::timer
elapsed函数windows返回正确，linux下返回0
```
15)
```
‘memset’在此作用域中尚未声明
‘strlen’在此作用域中尚未声明
‘memcpy’在此作用域中尚未声明
 #include <string.h>
```
16)
```
 ‘free’在此作用域中尚未声明
 ‘malloc’在此作用域中尚未声明
#include <stdlib.h>
```
17)‘_vsnprintf’在此作用域中尚未声明
```
#include <stdarg.h>
将_vsnprintf改成vsnprintf
```
18）‘_snprintf’在此作用域中尚未声明
```
#include <stdarg.h>
将_snprintf改成snprintf
```
19）‘_access’在此作用域中尚未声明
```
#include <unistd.h>
将_access改成access 
```
20）
```
typedef ACE_Hash_Map_Manager< CQQ_USERSERIAL, SUserAppDACItem*,ACE_Thread_Mutex> MAP_USER_APP_DAC;
ACE_Thread_Mutex在windows中和ACE_SYNCH_RECURSIVE_MUTEX效果相同，都为递归锁。
但在linux下
ACE_Thread_Mutex为非递归锁，同一个线程第二次进入则会死锁
```
21）
```
char p;
windwos下为：
typeid(p).name() 等于 "char"
linux下为：
typeid(p).name() 等于 "c"

unsigned long p
windwos下为：
typeid(p).name() 等于 "unsigned long"
linux下为：
typeid(p).name() 等于 "m"

unsigned short p
windwos下为：
typeid(p).name() 等于 "unsigned short"
linux下为：
typeid(p).name() 等于 "t"
```
22)
```
ssp_ep.cpp:65: 错误：跳转至标号‘redispatch_lab’
ssp_ep.cpp:37: 错误：从这里
ssp_ep.cpp:61: 错误：跳过‘CServerRegEvent* e’的初始化
```
gcc规定goto语句后不能定义变量，要定义需要用{}括起来