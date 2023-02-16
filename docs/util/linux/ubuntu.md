





apt = apt-get、apt-cache 和 apt-config 中最常用命令选项的集合。







```sh
# 更新软件列表
apt update



# 更新已安装的软件包
apt upgrade 包名
```





```sh
apt list

# 显示已经安装的软件包
apt list --installed


# 显示可升级的软件包
apt list --upgradeable
```





```sh
apt install xxx


apt remove xxx

# 自动清理不在使用的依赖
apt autoremove

apt show xxx
```

