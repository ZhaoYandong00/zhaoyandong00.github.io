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
### 错误处理
- 若安装出现错误，就清理缓存解决

```
npm cache verify
npm cache clean --force
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
# 树莓派插件
## 安装摄像机插件
### 摄像机配置
- 系统启动摄像机
- 将摄像机加到modules里

```
sudo nano /etc/modules
```
- 添加`bcm2835-v4l2`到文件，保存退出
- 系统重启

```
sudo reboot
```
- 安装ffmpeg
```
sudo apt install ffmpeg
```

### 安装插件
- 安装插件
```
sudo npm install -g homebridge-camera-rpi
```
- 编辑homebridge的config.json，在platforms里添加以下内容
```
sudo nano /var/lib/homebridge/config.json
```
```json
    {
      "platform": "rpi-camera",
      "cameras": [{"name": "Pi Camera"}]
    }
```

- 重启
```
sudo reboot
```

## 安装温湿度插件
- 安装插件
```
sudo npm install -g homebridge-dht
```
- 编辑homebridge的config.json
```
sudo nano /var/lib/homebridge/config.json
```
```json
{
    "bridge": {
        "name": "Homebridge",
        "username": "0E:25:C7:A5:27:FA",
        "port": 51864,
        "pin": "509-96-738"
    },
    "accessories": [
        {
            "accessory": "Dht",
            "name": "Outside"
             "service":"dht11"
        }
    ],
    "platforms": [
        {
            "name": "Config",
            "port": 8581,
            "auth": "form",
            "theme": "auto",
            "tempUnits": "c",
            "lang": "zh-CN",
            "platform": "config"
        },
        {
            "platform": "rpi-camera",
            "cameras": [
                {
                    "name": "Pi Camera"
                }
            ]
        }
    ]
}   
```

# 小米插件
## 安装miio
- 安装
```
sudo npm install -g miio
```
- 配置
```
miio discover
```
- millo读取不到token,暂时放弃

# 自定义插件
## BH1750
### package.json
```json
{
    "name": "homebridge-rasspberypi-BH1750",
    "version": "1.0.0",
    "description": "This is an Homebridge accessory plugin for illuminance sensor connected via i2c.",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "engines": {
        "node": ">=0.12.0",
        "homebridge": ">=0.2.0"
      }
}
```

### index.js
```js
var Service, Characteristic;
var BH1750 = require('bh1750');

module.exports = function (homebridge) {
    Service = homebridge.hap.Service;
    Characteristic = homebridge.hap.Characteristic;
    homebridge.registerAccessory('homebridge-rasspberypi-BH1750', 'illuminance', illuminance);
};

function illuminance(log, config) {
    this.log = log;
    this.log("Adding Accessory");
    this.name = config.name;
    this.sensor = new BH1750();
    this.refresh = config.refresh || "60"; // Every minute
}
illuminance.prototype = {
    identity: function (callback) {
        this.log("Identify requested!");
        callback(); // success
    },
    getServices: function () {
        this.log("INIT: %s", this.name);
        this.services.informationService = new Service.AccessoryInformation();
        this.services.informationService
            .setCharacteristic(Characteristic.Manufacturer, 'ZYD')
            .setCharacteristic(Characteristic.Model, 'Illumiance Sensor')
            .setCharacteristic(Characteristic.SerialNumber, this.name);
        this.log('creating Illumiance Sensor');
        this.services.illumianceService = new Service.IllumianceSensor(this.name);
        this.services.illumianceService
            .getCharacteristic(Characteristic.CurrentIllumiance)
            .on('get', this.getIllumiance.bind(this));
        setInterval(function () {
            this.getIllumiance(function (err, illuminance) {
                if (err) {
                    illuminance = err;
                }
                this.illumianceService
                    .getCharacteristic(Characteristic.CurrentIllumiance).updateValue(illuminance);
            }.bind(this));
        }.bind(this), this.refresh * 1000);

        return [this.informationService, this.illumianceService];
    },
    getIllumiance: function (callback) {
        this.sensor.readLight(function (error, value) {
            if (err) {
                this.log("light error: " + err);
                callback(err);
            } else {
                this.log("light value is: ", value, "lx");
                callback(null, value);
            }
        }.bind(this));
    }
};
```