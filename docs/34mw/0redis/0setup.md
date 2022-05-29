## 安装

### 安装



```sh
brew install redis
```

默认安装目录

/opt/homebrew/Cellar/redis/7.0.0



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

