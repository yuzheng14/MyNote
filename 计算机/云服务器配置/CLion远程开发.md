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

