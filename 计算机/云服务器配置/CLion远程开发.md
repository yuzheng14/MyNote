# CLion远程开发

## Linux配置

如果cmake版本为2

```bash
[root@VM-8-12-centos ~ ]# cmake -version
cmake version 2.8.12.2
```

卸载自带的cmake2

```bash
yum remove cmake -y
```

安装必要的环境

```bash
yum install -y gcc g++ gcc-c++ make automake texinfo wget
```

## 安装CMake

去这里找到需要的cmake版本下载链接[Index of /files (cmake.org)](https://cmake.org/files/)，尽量3.7以上

```bash
wget https://cmake.org/files/v3.23/cmake-3.23.0-rc3-linux-x86_64.tar.gz
```

解压构建

```bash
tar -xf cmake-3.23.0-rc3-linux-x86_64.tar.gz
cd cmake-3.23.0-rc3-linux-x86_64.tar.gz/
./boostrap
```

编译链接

```bash
make
```

安装

```bash
make install
```

构建软连接

```bash
ln -s /usr/local/bin/cmake /usr/bin/cmake
```

## 安装gdb

在镜像中找到所需版本[Index of /gnu/gdb/ (ustc.edu.cn)](http://mirrors.ustc.edu.cn/gnu/gdb/)，下载安装

步骤基本同上

```bash
wget http://mirrors.ustc.edu.cn/gnu/gdb/gdb-<version>.tar.xz
tar -xf gdb-<version>.tar.xz
cd gdb-<version>
./configure
make
make install
```

> 如果安装提示configure: error: no termcap library found
>
> 请在[这里](https://ftp.gnu.org/gnu/termcap/)找到所需版本下载安装termcap库，安装步骤同gdb及cmake

构建软连接

```bash
ln -s /usr/local/bin/gdb /usr/bin/gdb 
```

## 安装gmp

若安装gdb过程中出现`make`找不到gmp的情况，则需安装gmp

```
configure: error: GMP is missing or unusable
```

首先查询m4是否安装

```bash
yum list installed |grep m4
```

若未安装，则需先安装m4

```bash
yum -y install m4
```

去[gmp官网](https://gmplib.org/)下载安装包

解压

```bash
yum -y install lzip //如果安装包为tar.lz格式则需安装lzip
lzip <packagename>
tar -xf <packagename>
cd <directoryname>
./configure -enable-cxx
make
make install
```

