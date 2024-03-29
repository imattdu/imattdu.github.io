

# 111



## mac



### 安装



```sh
brew install go
```



### 配置



```sh
vim .zshrc
```





```sh title="配置"
GOROOT=/opt/homebrew/Cellar/go/1.17.6/libexec
export GOROOT
export GOPATH=/Users/matt/workspace/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN:$GOROOT/bin # (1)!


```
1.  :man_raising_hand: I'm a code annotation! I can contain `code`, __formatted
    text__, images, ... basically anything that can be written in Markdown.






``` yaml
this is good # (1)!
```

1.  If you ++cmd++ :material-plus::material-cursor-default-outline: me, I'm
    rendered open in a new tab. You can also right-click me to __copy link
    address__ to share me with others.
















## win 

### 安装

[下载地址](https://golang.org/dl/)

win10/64-》选择x86-64



![](https://raw.githubusercontent.com/imattdu/img/main/img/202111190035153.png)







![](https://raw.githubusercontent.com/imattdu/img/main/img/202111190036104.png)



### 配置



GOROOT:指定go的安装目录

path:指定go的安装目录下的bin目录

GOPATH:工作目录，go项目的工作目录



系统环境变量中添加



GOROOT

```
D:\develop\env\go
```

path

```
%GOROOT%\bin
```

GOPATH

```
D:\matt\workspace\go
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202111190039095.png)





![](https://raw.githubusercontent.com/imattdu/img/main/img/202111190040183.png)







## 设置

### 设置国内镜像

```go
go env -w GOPROXY=https://goproxy.cn,direct
```

### go的依赖管理，开启gomod

```go
go env -w GO111MODULE=on
```

### 验证配置,可能验证的默认网站无法访问 sum.golang.org

```bash
go env -w GOSUMDB="sum.golang.google.cn"
```

**或者关闭验证**

```bash
go env -w GOSUMDB=off
```



### 验证

验证go环境是否安装成功

```bash
go version
```

格式化代码

```
go fmt -w main.go
```



## 资料

[中文API](https://studygolang.com/pkgdoc)

[官网](https://golang.org/)





## 使用





### 项目放在目录下



项目都放在src下面

下载的依赖都会进入到pkg文件夹

可执行文件进入bin







![](https://raw.githubusercontent.com/imattdu/img/main/img/202111190043410.png)





### 包的管理



GO111MODULE可以设置为：off、on、auto（默认值），从GO111MODULE变量名可以看出，是Go1.11版本之后才出来有依赖包管理办法。

- 为`off`时，则不使用go mod，查找依赖包的顺序是：当前项目根目录*/vendor*，其次是*$GOPATH/src*  *$GOROOT/src*（这是Golang1.11版本之前的用法）。
- 为`on`时，则开启go mod，查找依赖包是以当前项目根目录的`go.mod`文件为基准，会忽略 `$GOPATH` 和 `vendor` 文件夹，只根据`go.mod`下载依赖。
- 为`auto`或未设置，则go命令根据当前目录启用或禁用模块支持。根据当前文件夹是否有go.mod来决定是否开启gomod









![](https://raw.githubusercontent.com/imattdu/img/main/img/202111190118080.png)







### **goland 注意事项**



![](https://raw.githubusercontent.com/imattdu/img/main/img/202111190145646.png)





#### 1.Global GOPATH

选则你在环境变量中配置的GOPATH路径

#### 2.Project GOPATH

项目的GOPATH,最好不好设置Global GOPATH,因为那你的项目将会使用到所用配置到GOPATH的文件

#### 3.Use GOPATH that`s defined in system environment

如果选中这个，他将使用系统定义的环境变量，并设置到Global GOPATH

#### 4.Index entire GOPATH:

会使用系统中配置的gopath,一般不要勾选这个



## 常用命令





### 编译



```bash
go build -o ./bin/main main.go
```





### 运行

```sh
go run main.go -config app.cfg
```









### go get

1. download
2. compile
3. install



#### 参数



```sh
go get -u xxx
```

|      |                            |
| ---- | -------------------------- |
| -d   | 只下载不安装               |
| -u   | 更新下载，默认不会更新下载 |
|      |                            |



#### 指定版本

```sh
go get k8s.io/klog@v1.0.0

```







### go  install

1. compile
2. install







```sh
go mod init

go mod tidy
```







go mod download 下载模块到本地缓存，缓存路径是 $GOPATH/pkg/mod/cache
go mod edit 是提供了命令版编辑 go.mod 的功能，例如 go mod edit -fmt go.mod 会格式化 go.mod
go mod graph 把模块之间的依赖图显示出来
go mod init 初始化模块（例如把原本dep管理的依赖关系转换过来）
go mod tidy 增加缺失的包，移除没用的包
go mod vendor 把依赖拷贝到 vendor/ 目录下
go mod verify 确认依赖关系
go mod why 解释为什么需要包和模块
