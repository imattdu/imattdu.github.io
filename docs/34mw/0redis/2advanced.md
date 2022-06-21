







查看集群信息

 ```sh
 cluster nodes
 ```

![](https://raw.githubusercontent.com/imattdu/img/main/img/202206191121531.png)







```sh
127.0.0.1:6379> set s1 1
-> Redirected to slot [15224] located at 192.168.199.222:6381
OK
```









```sh
192.168.199.222:6381> mset name{user} matt age{user} 12
-> Redirected to slot [5474] located at 192.168.199.222:6380
```





只能看自己槽位 别的看不到





```sh
127.0.0.1:6380> cluster nodes
2fcb1f4118c3ba1d0f00cc0e859e812fae552942 192.168.199.222:6379@16379 master,fail - 1655622346443 1655622343385 1 disconnected
73d2f2aa22351cb220cb4d70da9ddcbcf0066e40 192.168.199.222:6381@16381 master - 0 1655622363871 3 connected 10923-16383
2a7c5c0eb6a9d43298245584f1c1fad4420b3aaf 192.168.199.222:6390@16390 slave 73d2f2aa22351cb220cb4d70da9ddcbcf0066e40 0 1655622364000 3 connected
23b19e64cf536fdffc11b7748e01d9b232f30045 192.168.199.222:6391@16391 master - 0 1655622364903 7 connected 0-5460
282398ec24fbcf9743a50d8389dcc536c2dee6a0 192.168.199.222:6389@16389 slave 6511a15d81a488b8e776f73e655d27bd36004e5c 0 1655622365933 2 connected
6511a15d81a488b8e776f73e655d27bd36004e5c 192.168.199.222:6380@16380 myself,master - 0 1655622362000 2 connected 5461-10922
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202206191509291.png)









分布式锁



a

1 获取锁

3 释放锁

2 卡注



2执行 释放锁 会把别的线程的锁释放















```sh
127.0.0.1:6379> acl list
1) "user default on nopass ~* &* +@all"
```







```sh
 acl setuser matt
```

