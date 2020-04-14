---
title: 树莓派设置
categories: 树莓派
tags: 树莓派 Raspberrypi 
description: 树莓派上一些基本设置
---
# 开启远程桌面
```
sudo apt install xrdp
```

# 更新
## raspbian源
```
sudo nano /etc/apt/sources.list
```
```
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/ buster main contrib non-free rpi
```

## debian源
```
sudo nano /etc/apt/sources.list.d/raspi.list
```
```
deb http://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian/ buster main ui
```

## 更新
```
sudo apt update
sudo apt upgrade
sudo apt dist-upgrade
```
