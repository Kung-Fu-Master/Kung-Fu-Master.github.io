---
title: cmake
categories:
- linux
---

## **cmake 安装**

```shell
	$ wget https://github.com/Kitware/CMake/releases/download/v3.17.0/cmake-3.17.0-Linux-x86_64.tar.gz
	$ tar -zxvf cmake-3.17.0-Linux-x86_64.tar.gz
	$ ln -s /<PATH>/cmake-3.17.0-Linux-x86_64/bin/cmake /usr/bin/cmake
	$ cmake -version
```

## **函数使用说明**
参考链接:https://www.jianshu.com/p/aaa19816f7ad

## 编写 CMakeLists.txt
### 1. 源文件只有一两个时候

```shell
	$ touch CMakeLists.txt
	# CMake 最低版本号要求, 指定运行此配置文件所需的 CMake 的最低版本
	cmake_minimum_required(VERSION 2.8)
	# 项目信息, 参数值是 Demo1，该命令表示项目的名称是 Demo1
	project(Demo1)
	# 指定生成目标, 将名为 main.cc 的源文件编译成一个名称为 Demo 的可执行文件
	add_executable(Demo main.cc MathFunctions.cc)
```
### 2. 如果源文件很多
把所有源文件的名字都加进add_executable将是一件烦人的工作, 更省事的方法是使用 aux_source_directory 命令，该命令会查找指定目录下的所有源文件，然后将结果存进指定变量名, 其语法如下:  

```
	aux_source_directory(<dir> <variable>)
```

可以修改 CMakeLists.txt 如下：

```
	cmake_minimum_required(VERSION 2.8)
	project(Demo2)
	# 查找当前目录下的所有源文件
	# 并将名称保存到 DIR_SRCS 变量
	aux_source_directory(. DIR_SRCS)
	# 指定生成目标
	add_executable(Demo ${DIR_SRCS})
```

CMake 会将当前目录所有源文件的文件名赋值给变量 DIR_SRCS ，再指示变量 DIR_SRCS 中的源文件需要编译成一个名称为 Demo 的可执行文件。  
### 3. 多个目录，多个源文件

```
	./Demo3
	    |
	    +--- main.cc
	    |
	    +--- math/
	          |
	          +--- MathFunctions.cc
	          |
	          +--- MathFunctions.h
```

需要分别在项目根目录 Demo3 和 math 目录里各编写一个 CMakeLists.txt 文件。为了方便，我们可以先将 math 目录里的文件编译成静态库再由 main 函数调用.  
根目录中的 CMakeLists.tx

```
	# CMake 最低版本号要求
	cmake_minimum_required (VERSION 2.8)
	# 项目信息
	project (Demo3)
	# 查找当前目录下的所有源文件
	# 并将名称保存到 DIR_SRCS 变量
	aux_source_directory(. DIR_SRCS)
	# 添加 math 子目录, 这样 math 目录下的 CMakeLists.txt 文件和源代码也会被处理
	add_subdirectory(math)
	# 指定生成目标
	add_executable(Demo main.cc)
	# 添加链接库, 指明可执行文件 Demo 需要连接一个名为 MathFunctions 的链接库
	target_link_libraries(Demo MathFunctions)
```

子目录math中的 CMakeLists.txt：

```
	# 查找当前目录下的所有源文件
	# 并将名称保存到 DIR_LIB_SRCS 变量
	aux_source_directory(. DIR_LIB_SRCS)
	# 生成链接库, 将math 目录中的源文件编译为静态链接库
	add_library (MathFunctions ${DIR_LIB_SRCS})
```

## **编译**

```shell
	cmake .
```

## **安装和测试**
这两个功能分别可以通过在产生 Makefile 后使用 make install 和 make test 来执行.  
首先先在 math/CMakeLists.txt 文件里添加下面两行：

```
	# 指定 MathFunctions 库的安装路径
	install (TARGETS MathFunctions DESTINATION bin)
	install (FILES MathFunctions.h DESTINATION include)
```
指明 MathFunctions 库的安装路径。之后同样修改根目录的 CMakeLists 文件，在末尾添加下面几行：

```
	# 指定安装路径
	install (TARGETS Demo DESTINATION bin)
	install (FILES "${PROJECT_BINARY_DIR}/config.h"
	         DESTINATION include)
```
通过上面的定制，生成的 Demo 文件和 MathFunctions 函数库 libMathFunctions.o 文件将会被复制到 /usr/local/bin 中，而 MathFunctions.h 和生成的 config.h 文件则会被复制到 /usr/local/include 中。我们可以验证一下（顺带一提的是，这里的 /usr/local/ 是默认安装到的根目录，可以通过修改 CMAKE_INSTALL_PREFIX 变量的值来指定这些文件应该拷贝到哪个根目录）.  


## **Cmake常用命令**
### 1. add_library
该指令的主要作用就是将指定的源文件生成链接文件，然后添加到工程中去。该指令常用的语法如下：

```
	add_library(<name> [STATIC | SHARED | MODULE]
	            [EXCLUDE_FROM_ALL]
	            [source1] [source2] [...])
```
<name>表示库文件的名字，该库文件会根据命令里列出的源文件来创建。
STATIC、SHARED和MODULE的作用是指定生成的库文件的类型。
STATIC库是目标文件的归档文件，在链接其它目标的时候使用。
SHARED库会被动态链接（动态链接库），在运行时会被加载。
MODULE库是一种不会被链接到其它目标中的插件，但是可能会在运行时使用dlopen-系列的函数。
默认状态下，库文件将会在于源文件目录树的构建目录树的位置被创建，该命令也会在这里被调用。
而语法中的source1 source2分别表示各个源文件。
#### **使用案例**

```
	add_subdirectory(sub_dir)
	file(GLOB HDRS "*.h")
	file(GLOB SRCS "*.c")
	set(TARGET_NAME pnp_module)
	add_library(${TARGET_NAME} SHARED ${HDRS} ${SRCS})
	target_link_libraries(${TARGET_NAME} PRIVATE
	        ${CMAKE_THREAD_LIBS_INIT}				// 库引用了需要使用-lpthread的.c源码文件，需要加上这句
	        pnp_utils_static
	        m)
	
	// CMAKE_CURRENT_SOURCE_DIR：这是当前处理的CMakeLists.txt所在的目录。当前正在处理的源目录的路径。这是当前正由cmake处理的源目录的完整路径。
	// CMAKE_CURRENT_LIST_DIR：这是当前正在处理的listfile的目录。当前正在处理的列表文件的完整目录。
	// 在处理sub_dir/CMakeLists.txt时，CMAKE_CURRENT_LIST_DIR将引用项目/ src，而CMAKE_CURRENT_SOURCE_DIR仍指向外部目录项目。
	target_include_directories(${TARGET_NAME} PUBLIC ${CMAKE_CURRENT_LIST_DIR})
```

### 2. target_link_libraries
该指令的作用为将目标文件与库文件进行链接。该指令的语法如下：

```
target_link_libraries(<target> [item1] [item2] [...]
                      [[debug|optimized|general] <item>] ...)
```
<target>是指通过add_executable()和add_library()指令生成已经创建的目标文件。
[item]表示库文件没有后缀的名字。
默认情况下，库依赖项是传递的。当这个目标链接到另一个目标时，链接到这个目标的库也会出现在另一个目标的连接线上。
这个传递的接口存储在interface_link_libraries的目标属性中，可以通过设置该属性直接重写传递接口。

<dl>
<dt>cmake target_link_libraries() 中<PUBLIC|PRIVATE|INTERFACE> 的区别<dt>
<dd>如果目标的头文件中包含了依赖的头文件(源文件间接包含)，那么这里就是PUBLIC</dd>
<dd>如果目标仅源文件中包含了依赖的头文件，那么这里就是PRIVATE</dd>
<dd>如果目标的头文件包含依赖，但源文件未包含，那么这里就是INTERFACE</dd>
</dl>

#### **使用案例：**

 * 案例1：

```
	TARGET_LINK_LIBRARIES(myProject hello)			// 连接libhello.so库
	TARGET_LINK_LIBRARIES(myProject libhello.a)
	TARGET_LINK_LIBRARIES(myProject libhello.so)
	// 库引用了需要使用-lpthread的.c源码文件，需要加上${CMAKE_THREAD_LIBS_INIT}
	target_link_libraries (${PROJECT_NAME} ${CMAKE_THREAD_LIBS_INIT})
```

 * 案例2：

```
	add_executable(myProject main.cpp)
	target_link_libraries(myProject eng mx)     
	#equals to below 
	target_link_libraries(myProject -leng -lmx) `
	target_link_libraries(myProject libeng.so libmx.so)`
```

### 3. target_include_directories
为add_executable() 或者add_library() 中定义的输出目标指定编译选项
Include的头文件的查找目录，也就是Gcc的[-Idir...]选项

```
	target_include_directories(<target> [SYSTEM] [BEFORE]
								<INTERFACE|PUBLIC|PRIVATE> [items1...]
								[<INTERFACE|PUBLIC|PRIVATE> [items2...]
								...])
```

#### **使用案例:**

```
	# gcc头文件查找目录，相当于-I选项，e.g -I/foo/bar
	#CMAKE_SOURCE_DIR是cmake内置变量表示当前项目根目录
	target_include_directories(test_elf
	    PRIVATE
	    ${CMAKE_SOURCE_DIR}
	    ${CMAKE_SOURCE_DIR}/common
	    ${CMAKE_SOURCE_DIR}/syscalls
	)
	# 其他编译选项定义，e.g -fPIC
	target_compile_options(test_elf
	    PRIVATE
	    -std=c99 
	    -Wall 
	    -Wextra 
	    -Werror
	)
```

## **-DCMAKE_BUILD_TYPE**

```shell
	cmake -DCMAKE_BUILD_TYPE=Debug ..
```
通过以上编译能够使用 **`gdb`** 进行调试, 否则无法gdb来调试生成的可执行文件.

## **遇到的问题**

```shell
	$ mkdir build
	$ cd build
	$ cmake ..
	出现如下错误
	......
	CMake Error at /usr/local/cmake-3.17.0-Linux-x86_64/share/cmake-3.17/Modules/FindPackageHandleStandardArgs.cmake:164 (message):
	  Could NOT find PythonLibs (missing: PYTHON_LIBRARIES PYTHON_INCLUDE_DIRS)
	Call Stack (most recent call first):
	  /usr/local/cmake-3.17.0-Linux-x86_64/share/cmake-3.17/Modules/FindPackageHandleStandardArgs.cmake:445 (_FPHSA_FAILURE_MESSAGE)
	  /usr/local/cmake-3.17.0-Linux-x86_64/share/cmake-3.17/Modules/FindPythonLibs.cmake:310 (FIND_PACKAGE_HANDLE_STANDARD_ARGS)
	  python/CMakeLists.txt:2 (find_package)
```
原因是没有安装python的开发版

```shell
	$ yum install python-devel 
```
网上还有这种操作的, 可以pass掉`cmake -DPYTHON_INCLUDE_DIR=/usr/include/python2.7 -DPYTHON_LIBRARY=/usr/lib/python2.7/config/libpython2.7.so ..`

