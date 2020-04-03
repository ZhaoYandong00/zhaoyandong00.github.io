---
title:  Jekyll 在windows下的安裝
categories:  Jekyll githubpages
tags: Jekyll  blog  githubpages
highlight:  true
---
#   安装Ruby 环境

## 从 [rubyinstaller](https://rubyinstaller.org/downloads/)下载Ruby+Devkit的安装包；

## 双击开始安装，选择全部安装；

## 在安装结束时，去除ridk install的选项，因为从默认的原去下载几百兆会非常缓慢；

## 查找Ruby安装目录下的msys64\etc\pacman.d，编辑更新源：

Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/i686 加入mirrorlist.mingw32首位

Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/mingw/x86_64 加入mirrorlist.mingw64首位

Server = https://mirrors.tuna.tsinghua.edu.cn/msys2/msys/$arch 加入mirrorlist.msys首位

## 执行在命令行执行ridk install（如果安装时选择了不加入系统环境变量的，去Ruby安装目录的bin之下执行），选择3 msys2+MINGW 一路回车至结束；

## 更新gem源：
`
gem sources --add https://mirrors.aliyun.com/rubygems/ --remove https://rubygems.org/
`

## 安装Bundler和Jekyll
`
sudo gem install bundler jekyll
`

## 测试安装
```
ruby -v
gem -v
bundler -v
jelyll -v
 ```

# 设置jekyll 网站
## 新建站
`jekyll new site`
## 测试
### 启动网站
```
cd site
bundle install
bundle exec jekyll s
```

然后在浏览器输入 [localhost:4000](http://localhost:4000/)即可看到网站内容
### 退出测试
按下`Ctrl+C`即可
### 删除网站
```
cd ..
rm -r site
```

#  GitHub Pages
先按网上教程在github建立github.io库

然后从网上复制一个模板下来
```git
git clone https://github.com/ZhaoYandong00/zhaoyandong00.github.io.git
``` 
修改后，传到自己的库里
```git
git  remote add origin git@github.com:ZhaoYandong00/zhaoyandong00.github.io.git
git push
```
