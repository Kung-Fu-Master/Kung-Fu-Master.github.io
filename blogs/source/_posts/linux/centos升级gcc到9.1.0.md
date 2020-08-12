---
title: centos升级gcc到9.1.0
tags: 
categories:
- linux
---
centos 升级GCC到9.1.0
下载gcc-9.1.0源代码
cd gcc-9.1.0
vim contrib/download_prerequisites   #查看需要安装的依赖库
 30 gmp='gmp-6.1.0.tar.bz2'
 31 mpfr='mpfr-3.1.4.tar.bz2'
 32 mpc='mpc-1.0.3.tar.gz'
 33 isl='isl-0.18.tar.bz2'
直接搜索相应库下载对应版本或下面链接
1.gmp http://ftp.gnu.org/gnu/gmp/
2.mpfr http://ftp.gnu.org/gnu/mpfr/
3.mpc http://ftp.gnu.org/gnu/mpc/
4.isl http://isl.gforge.inria.fr/

cd gmp-6.1.0
./configure --prefix=/usr/local/gmp-6.1.0 && make
make install

cd mpfr-3.1.4
./configure --prefix=/usr/local/mpfr-3.1.4 --with-gmp=/usr/local/gmp-6.1.0 && make
make install

cd mpc-1.0.3
./configure --prefix=/usr/local/mpc-1.0.3 --with-gmp=/usr/local/gmp-6.1.0 --with-mpfr=/usr/local/mpfr-3.1.4 && make
make install

cd isl-0.18
$./configure --prefix=/usr/local/isl-0.18 --with-gmp-prefix=/usr/local/gmp-6.1.0
$make
$make check
$make install

$export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/isl-0.18/lib
或设置到所有tty
#vi ~/.bashrc
添加export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/isl-0.18/lib
再source bashrc

如果报检测不到gmp,mpfr,mpc相关文件，则把下面对应的也加入进来
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/mpc-1.0.3/lib:/usr/local/gmp-6.1.0/lib:/usr/local/mpfr-3.1.4/lib:/usr/local/isl-0.18/lib

cd gcc-9.1.0
$./configure --prefix=/usr/local/gcc-9.1.0 --enable-threads=posix --disable-checking --disable-multilib --enable-languages=c,c++ --with-gmp=/usr/local/gmp-6.1.0 --with-mpfr=/usr/local/mpfr-3.1.4 --with-mpc=/usr/local/mpc-1.0.3 --with-isl=/usr/local/isl-0.18
$make -j4 #启用4个job，需要大约25分钟时间
$make install
查看安装路径：
$whereis gcc
如果有旧的Gcc,替换版本：
$cd /usr/bin
$mv gcc gcc.bak
$mv g++ g++.bak
$ln -s /usr/local/gcc-9.1.0/bin/gcc /usr/bin/gcc
$ln -s /usr/local/gcc-9.1.0/bin/g++ /usr/bin/g++
查看版本：
$gcc -v
$g++ -v

运行以下命令检查动态库：
strings /usr/lib64/libstdc++.so.6 | grep GLIBC

cp /usr/local/gcc-9.1.0/lib64/libstdc++.so.6.0.26 /usr/lib64/
mv libstdc++.so.6 libstdc++.so.6.bak
ln libstdc++.so.6.0.26 libstdc++.so.6
