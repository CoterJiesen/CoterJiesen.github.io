---
layout: post
title: Cmake使用指南
category: 技术
tags: CPP Linux
keywords: 
description: 
---

#Cmake使用指南


###1.Cmake简介
    CMake是一个跨平台的安装(编译)工具，可以用简单的语句来描述所有平台的安装(编译过程)。他能够输出各种各样的Makefile或者Project文件，能测试编译器所支持的C++特性，类似UNIX下的automake。只是CMake的组态档取名为CmakeLists.txt。Cmake并不直接建构出最终的软件，而是产生标准的建构档（如 Unix 的Makefile 或 Windows Visual C++ 的 projects/workspaces），然后再依一般的建构方式使用。这使得熟悉某个集成开发环境（IDE）的开发者可以用标准的方式建构他的软件，这种可以使用各平台的原生建构系统的能力是 CMake 和 SCons 等其他类似系统的区别之处。　　
    CMake可以编译源代码、制作程式库、产生适配器（wrapper）、还可以用任意的顺序建构执行档。CMake 支援 in-place 建构（二进档和源代码在同一个目录树中）和 out-of-place 建构（二进档在别的目录里），因此可以很容易从同一个源代码目录树中建构出多个二进档。CMake 也支持静态与动态程式库的建构。　　
		“CMake”这个名字是“crossplatform make”的缩写。虽然名字中含有“make”，但是CMake和Unix上常见的“make”系统是分开的，而且更为高阶。
###2.Cmake安装
   Cmake的下载地址：http://www.cmake.org/cmake/resources/software.html
   根据自己的需要下载相应的安装包即可，Windows下可以下载Zip压缩文件的绿色版本，也可以下载.exe的安装文件，由于Cmake是开源软件所以可以下载到源代码。

###3.Cmake使用
####3.1.单个文件示例
   下面是一个简单的C程序代码（打印“helloworld!”字符串）：
		#include <stdio.h>
		int main(void)
        {
           printf("hello world!\n");
           return 0;
        }
   在H:\CmakeTest目录下创建一个Exp1目录，在该目录下创建一个main.c文件，内容为上面的C程序代码，然后再创建一个CMakeLists.txt文件，内容如下：
   
        PROJECT (HELLO)
        SET (SRC_LIST main.c)
        ADD_EXECUTABLE (hello ${SRC_LIST})
   最后，在Exp1目录中创建一个build子目录，进入build目录，调用cmake命令自动生成项目文件，然后执行nmake进行编译生成可执行文件：
H:\CmakeTest\Exp1> mkdir build & cdbuild
H:\CmakeTest\Exp1\build> cmake ..-G"NMake Makefiles”
H:\CmakeTest\Exp1\build> nmake
   注1：为了简单起见，上面采用cmake的Out-Of-Source方式构建，即生成的中间产物和代码相分离。
   注2：必须使用“VisualStudio 2008 Command Line”快捷方式打开命令行窗口，然后执行上面的命令，否则不能创建“NMake Makefiles”的项目。
   注3：如果使用MinGW开发环境进行编译，执行下面的cmake命令：
   
        #> cmake .. -G”MinGW Makefiles”
        #> make
   nmake编译完成后，在当前目录下会生成一个hello.exe，运行它会在屏幕上打印“hello world!”字符串。
H:\CmakeTest\Exp1\build>hello.exe
hello world!
   此时，目录结构如下：
   
        H:\CMAKETEST\EXP1
        │  CMakeLists.txt
        │  main.c
        │
        └─build
            │  CMakeCache.txt
            │  cmake_install.cmake
            │  hello.exe   
            │  ……
   CMakeLists.txt文件说明：
   第一行PROJECT不是强制性的，推荐使用，这一行会引入两个变量HELLO_BINARY_DIR和HELLO_SOURCE_DIR。同时cmake自动定义了两个等价的变量PROJECT_BINARY_DIR和PROJECT_SOURCE_DIR。由于基于Out-Of-Source方式构建，务必注意这两个变量对应的目录，可以通过MESSAGE来输出变量的值，例如：MESSAGE(${PROJECT_SOURCE_DIR})。
   SET命令用于设置变量。
   ADD_EXECUTABLE命令用于告诉工程生成一个可执行文件。
   ADD_LIBRARY命令用于告诉工程生成一个库文件。
   注意：CMakeLists.txt文件中，命令的名称是不区分大小写的，而参数和变量名是大小写敏感的。
   Cmake命令说明：
   
    Cmake命令后面跟一个路径参数，它用来指定CMakeLists文件所在的位置。
    由于系统中可能有多套构建工程的开发环境，我们可以通过-G命令行参数来指定生成那种工程，你可以通过cmake --help命令获得-G参数的详细信息。
    如果要显示执行构建工程过程中详细信息，可以在CMakeLists.txt文件中加入：
    SET (CMAKE_VERBOSE_MAKEFILEon)
      或者执行make时:
    $ make VERBOSE=1
      或者
    $ exportVERBOSE=1
    $ make
####3.2.多个文件示例
    下面针对多个文件构建工程，假设有hello.h/hello.c/main.c三个源代码文件，如下所示：
Hello.h代码：

        #ifndef __HELLO_H__
        #define __HELLO_H__
        void hello(const char *name);
        #endif
        Hello.c代码：
        #include "hello.h"
        #include <stdio.h>

        void hello(const char *name)
        {
            printf("hello%s!\n", name);
        }
Main.c代码：
        #include "hello.h"

        int main(void)
        {
            hello("world");
            return 0;
        }
  在CMakeTest目录下创建Exp2目录，并且在Exp2目录中创建上面的三个源代码文件，然后再创建CMakeLists.txt文件，内容如下：
  
        PROJECT(HELLO)
        SET(SRC_LIST main.c hello.c)
        ADD_EXECUTABLE (hello ${SRC_LIST})


创建build目录，进入到该目录进行构建，如下所示：

        H:\CmakeTest\Exp2>mkdirbuild & cd build
        H:\CmakeTest\Exp2\build>cmake.. -G"NMake Makefiles"
构建完成后使用nmake进行编译生成hello程序，然后运行程序如下所示：

        H:\CmakeTest\Exp2\build>nmake
        H:\CmakeTest\Exp2\build>hello
        hello world!
            Exp2的目录结构如下所示：
        H:\CMAKETEST\EXP2
        │  CMakeLists.txt
        │  hello.c
        │  hello.h
        │  main.c
        │
        └─build
            │  CMakeCache.txt
            │  cmake_install.cmake
            │  hello.exe
####3.3.生成库文件示例
   还是使用上面例子中多个源代码，这里我们将hello.c生成一个库，然后在main.c中调用。修改CMakeLists.txt文件如下：
  
        PROJECT (HELLO)
        SET(SRC_LIBhello.c)
        SET(SRC_APPmain.c)
        ADD_LIBRARY(libhello ${SRC_LIB})
        ADD_EXECUTABLE(hello ${SRC_APP})
        TARGET_LINK_LIBRARIES(hellolibhello)
   在上面的文件中，我们添加一个新的目标库libhello，并且把它链接到hello程序中。
    注：在CMakeTest目录下创建一个Exp3目录，然后把Exp2目录中除build目录以外的4个文件拷贝到该目录下，最后修改CMakeLists.txt文件内容。
    创建一个build目录进行构建工程：
    
        H:\CmakeTest\Exp3>mkdirbuild & cd build
        H:\CmakeTest\Exp3\build>cmake.. -G"NMake Makefiles"
        H:\CmakeTest\Exp3\build>hello.exe
   由于可执行程序使用hello这个名称，此时添加类时就不能使用这个名称，而是使用libhello，此时编译生成的库名为libhello.lib，如果希望生成hello.lib库，添加下面一行：
   
	SET_TARGET_PROPERTIES(libhelloPROPERTIES OUTPUT_NAME “hello”)
   注意：推荐添加设置选项为cmake_minimum_required(VERSION2.8)。
####3.4.多个目录示例
   源文件位于多个目录情况下构建工程，源代码放置于不同路径下的结构如下：
   
    H:\CMAKETEST\EXP4
    │  CMakeLists.txt
    │
    ├─build
    ├─libhello
    │      CMakeLists.txt
    │      hello.c
    │      hello.h
    │
    └─src
           CMakeLists.txt
           main.c
    Exp4目录下的CMakeLists.txt文件，内容如下：
    
    CMAKE_MINIMUM_REQUIRED (VERSION 2.8)
    PROJECT(HELLO)
    ADD_SUBDIRECTORY(src)
    ADD_SUBDIRECTORY(libhello)
        Src目录下的CMakeLists.txt文件，内容如下：
    CMAKE_MINIMUM_REQUIRED (VERSION 2.8)
    INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/libhello)
    SET(SRC_APP main.c)
    ADD_EXECUTABLE(hello${SRC_APP})
    TARGET_LINK_LIBRARIES(hellolibhello)
        Libhello目录下CMakeLists.txt文件，内容如下：
    CMAKE_MINIMUM_REQUIRED (VERSION 2.8)
    SET(SRC_LIB hello.c)
    ADD_LIBRARY(libhello ${SRC_LIB})
    SET_TARGET_PROPERTIES(libhello PROPERTIES OUTPUT_NAME"hello")
        建立build目录构建工程，在build目录中执行下面命令：
    H:\CmakeTest\Exp4\build>cmake .. -G"NMake Makefiles"
    H:\CmakeTest\Exp4\build>nmake
    H:\CmakeTest\Exp4\build>src\hello
        生成的执行文件位于build\src目录中，库文件位于build\libhello目录中。
       在Exp4目录中的CMakeLists.txt文件中：
       ADD_SUBDIRECTORY命令用来告诉cmake去子目录中查找可用的CMakeLists.txt文件。
       在Src目录中的CMakeLists.txt文件中：
       INCLUDE_DIRECTORIES命令用来指明头文件所在的路径。
###3.5.指定输出目录示例
   要求把生成的可执行文件和库文件分别输出到bin目录和lib目录下，目录结构如下：
   
    H:\CMAKETEST\EXP5
    │  CMakeLists.txt
    │
    ├─build
    │  ├─bin
    │  └─lib
    ├─libhello
    │      CMakeLists.txt
    │      hello.c
    │      hello.h
    │
    └─src
            CMakeLists.txt
            main.c
   第一种方法，修改Exp5目录下的CMakeLists.txt文件，如下所示：
   
    CMAKE_MINIMUM_REQUIRED (VERSION 2.8)
    PROJECT(HELLO)
    ADD_SUBDIRECTORY(src bin)
    ADD_SUBDIRECTORY(libhello lib)
   这种方法在build目录下生成bin和lib目录替换上例中src和libhello目录，不满足要求。
   第二种方法，不修改顶级文件，修改其它两个文件，如下：
   src/CMakeLists.txt文件：
   
        CMAKE_MINIMUM_REQUIRED (VERSION2.8)
        INCLUDE_DIRECTORIES(${PROJECT_SOURCE_DIR}/libhello)
        #LINK_DIRECTORIES(${PROJECT_BINARY_DIR}/lib)
        SET(SRC_APP main.c)
        SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
        ADD_EXECUTABLE(hello ${SRC_APP})
        TARGET_LINK_LIBRARIES(hello libhello)
            libhello/CMakeLists.txt文件：
        CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
        SET(SRC_LIB hello.c)
        ADD_LIBRARY(libhello ${SRC_LIB})
        SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
        SET_TARGET_PROPERTIES(libhello PROPERTIES OUTPUT_NAME"hello")
   上面主要是设置LIBRARY_OUTPUT_PATH和EXECUTABLE_OUTPUT_PATH两个环境变量。
   在build目录下使用cmake构建工程，nmake进行编译，命令如下：
   
    H:\CmakeTest\Exp4\build>cmake.. -G"NMake Makefiles"
    H:\CmakeTest\Exp4\build>nmake
####3.6.编译动态库示例
   上述示例中都是使用静态库，下面考虑下如何编译成动态库。
   如果不是Windows操作系统，Linux/Unix下生成动态库很容易，只要修改下libhello/CMakeLists.txt文件中的ADD_LIBRARY命令（添加SHARED参数）即可，如下所示：
   
	ADD_LIBRARY(librarySHARED ${SRC_LIB})
    
   如果要做到Windows和Linux下可移植性，需要修改hello.h的头文件，如下所示：

    #ifndef__HELLO_H__
    #define__HELLO_H__
    #ifdef WIN32
    #ifLIBHELLO_BUILD
    #defineLIBHELLO_API __declspec(dllexport)
    #else
    #define LIBHELLO_API__declspec(dllimport)
    #endif
    #else
    #defineLIBHELLO_API
    #endif
    LIBHELLO_APIvoid hello(const char *name);
    #endif
        修改libhello/CMakeLists.txt文件
    CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
    SET(SRC_LIBhello.c)
    ADD_DEFINITIONS("-DLIBHELLO_BUILD")
    ADD_LIBRARY(libhelloSHARED${SRC_LIB})
    SET(LIBRARY_OUTPUT_PATH${PROJECT_BINARY_DIR}/lib)
    SET_TARGET_PROPERTIES(libhelloPROPERTIES OUTPUT_NAME "hello")
   接下来可以在build目录下使用cmake进行构建，构建完成后就可以编译和执行。
   
    H:\CmakeTest\Exp6\build>cmake.. -G"NMake Makefiles"
    H:\CmakeTest\Exp6\build>nmake
   注：在bin目录下的hello.exe无法运行，必须先把lib目录下的hello.dll拷贝到bin目录才能正常运行。