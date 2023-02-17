




## dev





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



### dbeaver

**版本: 22.3.4.202301312050 ue**

1.打开软件包先安装jdk 

2.安装dbeaver, 不要关闭

3.输入key 





解决xxx已损坏,无法打开,你应该将它移到废纸篓

方式1

```sh
sudo spctl --master-disable
```



先打开 系统偏好设置 -> 安全与隐私 -> 通用 选项卡，检查是否已经启用了 任何来源 选项。如果没有启用，先点击左下角的小黄锁图标解锁，然后选中任何来源。

现在打开软件 如果不行看2



方式2

```sh
sudo xattr -rd com.apple.quarantine /Applications/DBeaverUltimate.app
```

再打开软件看一下





### mysql-navicat 16.5

```
sudo xattr -rd com.apple.quarantine /Applications/Navicat\ Premium.app
```



### redis desktop manager

安装即可

## os





### office

先安装app 在安装破解包



1. 下载完成后打开Microsoft office 2021镜像包，双击【Microsoft Office for mac.pkg】进行安装
2. 安装完成后回到office2021破解版镜像包，请双击打开office2021激活工具





### paste

old

[下载地址](https://www.macat.vip/4164.html)

剪切板



sudo xattr -rd com.apple.quarantine /Applications/Paste.app







### Rectangle

[下载地址](https://rectangleapp.com/)

分屏软件， 官网下载默认安装即可, free























## word





### typora









## img





### omi录屏专家

### 视屏下载-Downie 4  

安装即可

V4.6.8








## file





### xmind

```
sudo xattr -rd com.apple.quarantine /Applications/XMind.app
```





### 压缩-The Unarchiver

free



### tencent lemon 

[下载地址](https://lemon.qq.com/)

清理工具，官网下载默认安装即可, free




### pdfreaderpro

pdf编辑阅读 版本2.8.22.1 (2.8.22.1)

安装即可

打开下载好的安装包，拖动【pdf reader pro】到应用程序中安装即可

### clearview X

阅读器

**Version 3.0.4**

一步一步安装即可





![](https://raw.githubusercontent.com/imattdu/img/main/img/202302172043146.png)



拖到右侧安装即可

