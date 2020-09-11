---
title: cmake
categories:
- linux
---

## **cmake 安装**

	$ wget https://github.com/Kitware/CMake/releases/download/v3.17.0/cmake-3.17.0-Linux-x86_64.tar.gz
	$ tar -zxvf cmake-3.17.0-Linux-x86_64.tar.gz
	$ ln -s /<PATH>/cmake-3.17.0-Linux-x86_64/bin/cmake /usr/bin/cmake
	$ cmake -version


## **函数使用说明**
参考链接:https://www.jianshu.com/p/aaa19816f7ad
※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
1. add_library
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
使用案例
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

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
2. target_link_libraries
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

使用案例：
```
案例1：
TARGET_LINK_LIBRARIES(myProject hello)			// 连接libhello.so库
TARGET_LINK_LIBRARIES(myProject libhello.a)
TARGET_LINK_LIBRARIES(myProject libhello.so)
// 库引用了需要使用-lpthread的.c源码文件，需要加上${CMAKE_THREAD_LIBS_INIT}
target_link_libraries (${PROJECT_NAME} ${CMAKE_THREAD_LIBS_INIT})

案例2：
add_executable(myProject main.cpp)
target_link_libraries(myProject eng mx)     
#equals to below 
target_link_libraries(myProject -leng -lmx) `
target_link_libraries(myProject libeng.so libmx.so)`
```

※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※
3. target_include_directories
为add_executable() 或者add_library() 中定义的输出目标指定编译选项
Include的头文件的查找目录，也就是Gcc的[-Idir...]选项
```
target_include_directories(<target> [SYSTEM] [BEFORE]
							<INTERFACE|PUBLIC|PRIVATE> [items1...]
							[<INTERFACE|PUBLIC|PRIVATE> [items2...]
							...])
```
使用案例:
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


## **遇到的问题**

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
原因是没有安装python的开发版

	$ yum install python-devel 
网上还有这种操作的, 可以pass掉`cmake -DPYTHON_INCLUDE_DIR=/usr/include/python2.7 -DPYTHON_LIBRARY=/usr/lib/python2.7/config/libpython2.7.so ..`


※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※※