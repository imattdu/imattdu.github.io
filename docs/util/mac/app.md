

### brew

#### 安装

方法一 

原生

第一次使用443 ，后面配置git 代理就解决了 会帮你下载commad line tool

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```





安装完成会有如下提示，根据提示输入运行即可

```sh
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/matt/.zprofile

eval "$(/opt/homebrew/bin/brew shellenv)"
```





方法二

使用国内镜像 没测试过

```sh
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

#### 常用命令



```sh
# 安装
brew install git


# 卸载
brew uninstall git


#更新
brew update git
# 更新全部
brew update


# 一般默认会执行，环境变量生效
brew link git
# 失效
brew unlink git
```



```sh
# 搜索软件 对号表示已安装
brew search git

# 显示已安装的软件
brew list

# 显示软件内容信息
brew info git
```





















## dev

### typora



图片无法识别

![](https://raw.githubusercontent.com/imattdu/img/main/img/202206051723849.png)













### dbeaver



1.先安装jdk 

2.安装dbeaver

3.输入key 



解决xxx已损坏,无法打开,你应该将它移到废纸篓

1.

sudo spctl --master-disable

先打开 系统偏好设置 -> 安全与隐私 -> 通用 选项卡，检查是否已经启用了 任何来源 选项。如果没有启用，先点击左下角的小黄锁图标解锁，然后选中任何来源。



现在打开软件 如果不行看2



2.

sudo xattr -rd com.apple.quarantine /Applications/DBeaverUltimate.app

再打开软件看一下





### mysql-navicat 16.5

```
sudo xattr -rd com.apple.quarantine /Applications/Navicat\ Premium.app
```



### redis-redis desktop manager

安装即可

## life



### paste

old

[下载地址](https://www.macat.vip/4164.html)

剪切板



sudo xattr -rd com.apple.quarantine /Applications/Paste.app





### office

先安装app 在安装破解包





### xmind

```
sudo xattr -rd com.apple.quarantine /Applications/XMind.app
```

### tencent lemon 

[下载地址](https://lemon.qq.com/)

清理工具，官网下载默认安装即可, free

### Rectangle

[下载地址](https://rectangleapp.com/)

分屏软件， 官网下载默认安装即可, free





### The Unarchiver

free





### clearview X

阅读器

Version 3.0.4



### Downie 4  

安装即可

V4.6.8



### omi录屏专家





### pdfreaderpro

版本2.8.22.1 (2.8.22.1)

安装即可







todo

