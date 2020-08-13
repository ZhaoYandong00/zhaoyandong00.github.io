---
title: 树莓派上HomeBridge 的安装
categories: 树莓派 HomeBridge 
tags: 树莓派 HomeBridge Raspberrypi
description: 树莓派上安装HomeBridge
edit: 2020-04-15
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
- 在[homebridge-config-ui-x](http://raspberrypi:8581)（默认账号：admin 密码：admin)插件中，搜索`rpi-camera`
- 选择`Homebridge Camera Rpi`安装

- 编辑配置`config.json`，在platforms里添加以下内容
```json
    {
      "platform": "rpi-camera",
      "cameras": [{"name": "Pi Camera"}]
    }
```

- 重启homebridge


## 安装温湿度插件
- 在[homebridge-config-ui-x](http://raspberrypi:8581),搜索`dht`
- 选择`Homebridge Dht`安装
- 编辑`config.json`
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
      },
    "keywords":[
    "hombridge-plugin"
    "illuminance"
    ],
    "dependnices":{
    "i2c-bus":">=3.1.0"
    }
}
```
### BH1750.js
```js
var i2c = require('i2c-bus');

var BH1750 = function (opts) {
    this.options = opts || {
        address: 0x23,
        bus: 1,
        device: '/dev/i2c-1',
        command: 0x10,
        length: 2
    };
    this.verbose = this.options.verbose || false;
    if (i2c.openSync) {
        this.wire = i2c.openSync(this.options);
     } else {
        this.wire = new i2c(this.options.address, {device: this.options.device});
     }
    if (!this.wire.readBytes) {
        this.wire.readBytes = function(offset, len, callback) {
            this.writeSync([offset]);
            this.read(len, function(err, res) {
                callback(err, res);
            });
        }
    }
};

BH1750.prototype.readLight = function (cb) {
    var self = this;
    if (!cb) {
        throw new Error("Invalid param");
    }
    self.wire.readBytes(self.options.command, self.options.length, function (err, res) {

        if (err) {
            if (self.verbose)          
            return cb(err, null);
        }
        var hi = res[0];
        var lo = res[1];
        if (Buffer.isBuffer(res)) {
           hi = res.readUInt8(0);
           lo = res.readUInt8(1);
        }

        var lux = ((hi << 8) + lo)/1.2;
        if (self.options.command === 0x11) {
            lux = lux/2;
        }
        cb(null, lux);
    });
};

module.exports = BH1750;
```

### index.js
```js
var Service, Characteristic;
var BH1750 = require('./bh1750.js');

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
