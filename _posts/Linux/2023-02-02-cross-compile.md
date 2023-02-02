---
title: 交叉编译
categories: cross-compile 交叉编译
tags: cross-compile c linux
description: 交叉编译各种库
---
# 一般库交叉编译
- 配置
```
./configure --prefix=$(pwd)/_install --host=aarch64-linux
```
- 编译
```
make -j4 install
```
# 交叉编译FFMPEG
- 配置
```
./configure --prefix=/usr/ --target-os=linux --arch=aarch64 --enable-cross-compile --cross-prefix=aarch64-linux- --enable-shared --disable-static
```
- 编译
```
make -j4 install DESTDIR=$PWD/_install
```
# 交叉编译zlog
```
make CC=aarch64-linux-gcc;make PREFIX=$PWD/_install install
```
