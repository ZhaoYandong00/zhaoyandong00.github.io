---
title: 树莓派上HomeBridge 的安装
categories: 树莓派 HomeBridge 
tags: 树莓派 HomeBridge Raspberrypi
description: 树莓派上安装HomeBridge
---
[参考连接](https://github.com/homebridge/homebridge/wiki/Install-Homebridge-on-Raspbian)
#  安装
## 安装Node.js

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo bash -
sudo apt-get install -y nodejs gcc g++ make python
node -v
sudo npm install -g npm
```

## 安装Homebridge
```
sudo npm install -g --unsafe-perm homebridge homebridge-config-ui-x
```

# 设置
## 设置开机启动
```
sudo hb-service install --user homebridge
```

