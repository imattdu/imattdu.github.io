



## 安装

### mac 

```
brew install nginx
```









## 使用

```
# 启动
nginx

# 关闭
nginx -s quit

# 重载配置文件
nginx -s reload
```

./nginx -s quit:此方式停止步骤是待nginx进程处理任务完毕进行停止。

 ./nginx -s stop:此方式相当于先查出nginx进程id再使用kill命令强制杀掉进程。





查看使用的配置文件

```sh
❯ nginx -t
nginx: the configuration file /opt/homebrew/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /opt/homebrew/etc/nginx/nginx.conf test is successful
```







