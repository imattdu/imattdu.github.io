

## 发布订阅





### 概述

发布者：发送消息

订阅者:接收信息





### redis发布订阅





订阅管道c1

```sh
subscribe c1
```

在管道c1发布消息

```sh
publish c1 1
```



## 新数据类型





### bitmaps

存储01组成的





#### 命令

##### setbit



![](https://raw.githubusercontent.com/imattdu/img/main/img/202206282336522.png)



offset从0开始

```sh
setbit b1 0 0
```





##### getbit

getbit key offset

```sh
getbit b1 0
```



##### bitcount



bitcount key start end



start,end 均包含 start->end之间1的个数

-1：最后一位

-2：倒数第二位



```sh
bitcount b1 0 3
```



##### bitop



bitop and(or/not/xor)  destkey k1 k2 k3 ....





and：&&

or：或者

not：非

xor:异或



```sh
bitop and b3 b1 b2
```



### HyperLogLog



#### 概述

统计某个key 不重复元素的个数



#### 优势

并不会存储输入的元素，而是根据输入元素计算基数





HyperLogLog只需要12KB 就可以计算 2e64不同元素的基数







{1, 2, 3, 3, 4, 5} 

基数集

{1, 2, 3, 4, 5}

基数

5





#### 命令

##### pfadd

```sh
pfadd key v1 v2 ....


pfadd p1 1 2 3

```



##### pfcount

计算多个key的基数

```sh
pfcount k1 k2 k3 ....


pfcount p1


pfcount p1 p2
```





##### pfmerge

将多个key进行合并

```sh
pfmerge destkey k1 k2 k3




pfmerge p3 p2 p1
```



### Geospatial 



#### 概述

经纬度





#### 命令



##### geoadd 添加

有效的经度从 -180 度到 180 度。有效的纬度从 -85.05112878 度到 85.05112878 度。

```sh

geoadd key 经度 纬度 成员 [经度 纬度 成员]


geoadd g1 1 1 a1
```



##### geopos 定位



获取指定成员的经度 纬度

```sh

geopos key member [member ...]


127.0.0.1:6379> geopos g1 a1
1) 1) "0.99999994039535522"
   2) "0.99999945914297683"
```



##### geodist 获取俩个位置的直线距离

m 表示单位为米[默认值]。

km 表示单位为千米。

mi 表示单位为英里。

ft 表示单位为英尺。

```sh
geodist key member1 member2 [m | km | ft | mi]


geodist g1 a1 a2 m
```

##### georadius 给定经纬度为中心 找到一定半径内的元素





```sh
georadius key 经度 纬度 radius m | km | ft | mi



georadius g1 1 1 1000 km
```











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

