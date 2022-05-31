## 安装

### 安装



```sh
brew install redis
```

默认安装目录

/opt/homebrew/Cellar/redis/7.0.0



![](https://raw.githubusercontent.com/imattdu/img/main/img/202205310300731.png)







 

```sh
make PREFIX=/usr/local/redis install
```

这里多了一个关键字 **`PREFIX=`** 这个关键字的作用是编译的时候用于指定程序存放的路径。比如我们现在就是指定了redis必须存放在/usr/local/redis目录。假设不添加该关键字Linux会将可执行文件存放在/usr/local/bin目录，

库文件会存放在/usr/local/lib目录。配置文件会存放在/usr/local/etc目录。其他的资源文件会存放在usr/local/share目录。这里指定号目录也方便后续的卸载，后续直接rm -rf /usr/local/redis 即可删除redis。





会告诉你配置文件目录

### 启动

启动服务端

```sh
//方式一：使用brew帮助我们启动软件
brew services start redis
//方式二
redis-server /opt/homebrew/etc/redis.conf
```



启动客户端

```sh
redis-cli -h 127.0.0.1 -p 6379

redis-cli
```

### 关闭

```sh
redis-cli shutdown
```





### linux安装





```sh
yum install gcc

gcc --version




```









## 配置



```sh
daemonize yes
```

