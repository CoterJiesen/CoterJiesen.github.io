---
layout: post
title: CString到const char *的转换 
category: 技术
tags: CPP
keywords: 
description: 
---

#CString到const char *的转换 

起因是因为要实现一个UdpAddress实例，照示例程序的写法：
UdpAddress address((LPCTSTR)ipadd); 
提示没有匹配的构造函数，好吧看看构造函数都有神马，能以IP地址+端口号构造的只有：
UdpAddress(const char *inaddr);
百度了一下有网友说LPCTSTR就是const char *类型，但在这里看来显然是不一样了，看了定义：
typedef __nullterminated CONST WCHAR *LPCWSTR, *PCWSTR;
……菜鸟头疼CString到const <wbr>char <wbr>*的转换

直接
UdpAddress address((const char *)ipadd);
提示无参数匹配的构造函数； 
使用getBuffer()转换，
UdpAddress address(ipadd.getBuffer());
同样提示不匹配，getBuffer()返回的是LPWSTR类型；
于是两者结合:
UdpAddress address((const char *)ipadd.GetBuffer());
总算消停CString到const <wbr>char <wbr>*的转换

==============================================================
也就是const char * 到CString可以进行直接转换，反之不可。CString到const <wbr>char <wbr>*的转换
```
       const char * a;
       CString b;
       //a=b;//不存在从CSring到const char *的适当转换函数
       a=(const char *)b.GetBuffer();
       b=a;
```
一篇比较全的介绍【from http://blog.csdn.net/laiconglin/article/details/6641753】
####1.CString 转化为string

```
#include <string>
#include <cstdlib>
using namespace std;

//以上是需要在MFC工程中添加的头文件
string CStringToString(CString cstr)
{
    string str;
    setlocale(LC_ALL,"chs");
    wchar_t wch[255];
    char temp[255];
    wcscpy(wch,cstr.GetString());
    wcstombs(temp,wch,254);    
    str.append(temp);        
    return str;
}
 ```
 
###32.如果是string 转化为CString其实就比较简单了。
```
string str=“hello中文”;
CString cstr=str.c_str();
```
####3.如果要将CString或者string都可以利用string中很好用的string.c_str()将字符变为char * 的类型。
 
####4. CString 中比较好用的是
```
CString cstr,cstr1;
cstr.Format(_T("%s"),sctr1);这个函数还是很好用的。
 ```
####5.几个比较给力的函数
```
strlen(char *)
wcslen(wchar_t *)
strcpy(char *,char *)
wcscpy((wchar_t *,wchar_t *)
```
另外如果想将char * 加到string里面，那么可以很方便的使用string.append(char *)
里面的STL的string由于其好用并且强大，往往可以用来做中间的桥梁，连接char * 到达CString ，CString由于在MFC中有时候确实是方便，所以如果要使用MFC，不得不面对这个又爱又恨的数据结构吧。
