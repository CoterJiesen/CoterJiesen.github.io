---
layout: post
title: 也谈C++中char*与wchar_t*之间的转换 
category: 技术
tags: CPP
keywords: 
description: 
---

##也谈C++中char*与wchar_t*之间的转换  

关于C++中的char*与wchar_t*这两种类型的相互转换，网上说的大多很繁琐，可行性也不高。下面这个方法是在MSDN里面找到的，个人认为还比较不错：
 
####把char*转换为wchar_t*
用stdlib.h中的mbstowcs_s函数，可以通过下面的例子了解其用法：
```
char *CStr = "string to convert";
size_t len = strlen(CStr) + 1;
size_t converted = 0;
wchar_t *WStr;
WStr=(wchar_t*)malloc(len*sizeof(wchar_t));
mbstowcs_s(&converted, WStr, len, CStr, _TRUNCATE);
```
其结果是WStr中储存了CStr的wchar_t版本。
 
####把wchar_t*转换为char*
和上面的方法类似，用stdlib.h中的wcstombs_s函数，例子：
```
wchar_t *WStr = L"string to convert";
size_t len = wcslen(WStr) + 1;
size_t converted = 0;
char *CStr;
CStr=(char*)malloc(len*sizeof(char));
wcstombs_s(&converted, CStr, len, WStr, _TRUNCATE);
```
这时WStr中的内容将被转化为char版本储存在CStr中。

另外还可以通过流的方法来char*类型转换为wchar_t*类型，但这样的转换得到的结果将是const类型，而类似的方法不能将wchar_t*类型转换为char*类型。

把（const）char*转换为const wchar_t*
需要用到 sstream 头文件：
```
char *cstr="string to convert";
wstringstream wss;
wss<<cstr;
```
再调用wss.str().c_str(); 即可得到 const wchar_t* 类型的返回值。
虽然stringstream流不能将wchar_t*转换成char*，但可以用来进行数值类型和字符串之间的转换，例如：
```
double d=2734792.934f;
stringstream ss;
ss<<d;
```
调用ss.str()可得到string类型字符串 ”273479e+006”，又如：
```
string str("299792458");
stringstream ss;
long i=0;
ss<<str;
ss>>i;
```
此时i=299792458。