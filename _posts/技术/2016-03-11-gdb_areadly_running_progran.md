---
layout: post
title: 使用gdb调试当前运行的程序
category: 技术
tags: CPP;Linux
keywords: 
description: 
---

##使用gdb调试当前运行的程序



用gdb可以调试当前的程序的使用情况，读出他的参数。
以下用一个简单的程序做为例子：来说明gdb的调试。

第一步  编译一个死循环程序。 

    /* File name malloc.c*/

    #include   <stdio.h>                                                           #include   <stdlib.h>
    #include   <string.h>

    void getmem(void **p, int num){
        *p = (void *)malloc(num);
    }

    void test(void){
        char *str = NULL;
        getmem((void **)&str, 100);
            strcpy(str, "Hello");
        printf("%s\n", str);
    }

    int main(void){
        int i = 0;
        while(1){
            if (i == 1){ 
                test();
                return 1;
            }   
        }
        return 0;
    }                 

我们可以看出，这个程序就是malloc一段内存空间，用来供strcpy使用，由于只是调试一下，就没有在test程序中加上一些关于strcpy的正确性判断语句。
函数的正常退出的情况是i==1，但是程序运行过程中根本无法使i==1成立。i的变量的值将会在使用gdb时用到。

开始编译

	$gcc -g malloc.c 

得用gdb，加上-g还是需要的。生成的可执行文件为a.out

第二步  让gdb连接到正在执行的进程上去 
首先运行程序。

	$./a.out 
明显的，是一个死循环。
重新开一个shell

	$ps -u 
我的机器的运行情况如下所示：

    Warning: bad ps syntax, perhaps a bogus '-'? See http://procps.sf.net/faq.html
    USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    wyc       7712  0.0  0.1   6092  3644 pts/8    Ss   10:24   0:00 bash
    wyc       7880  0.0  0.1   6092  3608 pts/9    Ss   10:27   0:00 bash
    wyc       7929  0.0  0.3  10848  6468 pts/9    S+   10:28   0:00 gdb
    wyc       8347 93.0  0.0   1652   284 pts/8    R+   10:42   0:13 ./a.out
    ...

看到没有？ ./a.out的进程号是8347。

现在启动gdb

	$gdb 

由于是调试运行的进程，不是可执行文件，后面不需要跟任何参数。在用 gdb调试运行状态下的程序时，最核心的就是gdb内部的attach命令
用法为
 
 	(gdb) attach 

这是我的机器上的例子：

(gdb) attach 8347
Attaching to process 8347


在运行到第20行命令的时候，可以看一下到运行./a.out的那个shell，应该hello字符串在标准输出上了。当gdb中显示进程退出时，./a.out的shell应该结束了当前进程了。
在gdb中用set var i=1 来修改变量i的值(用set i=1不能识别命令)，使程序能够正常退出。

在调试时，当前程序调用的所有库也全部都出来了。这个例子中的
Reading symbols from /home/wyc/desktop/my_program/review/a.out...done.
Reading symbols from /lib/tls/i686/cmov/libc.so.6...(no debugging symbols found)...done.
Loaded symbols for /lib/tls/i686/cmov/libc.so.6
Reading symbols from /lib/ld-linux.so.2...(no debugging symbols found)...done.
Loaded symbols for /lib/ld-linux.so.2
是a.out程序所调用的全部库。可以用这种办法分析当前运行的程序的库的调用情况。

千万不要关掉gdb，以下调试更精彩：

第三步 在gdb中重启程序 
在上面已经知道了程序正常退出了，但是gdb还没有退出，这时在gdb中运行run效果如何？

    (gdb) run 
    Starting program: /home/wyc/desktop/my_program/review/a.out 

可以看出，用gdb连接进程后，他会找到运行这个进程所需的全部文件，当前进程关闭后，仍然可以在gdb中启动这个程序。

不得不佩服GDB的调试功能的强大

    gdb中的其它命令，就看你分析程序时是否用到了，例如下面的一些简单的命令：
    常用的bt, p , p/x , setp, info registers, break , jump ...... 