





网络层： 机器到机器

传输层：不同主机的应用进程



数据链路层：点到点

物理层：数据信号-》物理信号

ip 层：路由器







路由层：查路由表，查到转发，查不到转走





sdn:

数据平面 交换机

控制平面： 网络操作系统







协议：对等层的实体 通信遵守的规则集合























可靠性不高？需要维护很多对主机关系的维护





路由表不维护 host到 host 关系





虚电路：依靠信令



isp:一个网络中的设备









 r = 1mbps

1 秒钟可以打 100 0000 个 bit









上次知道下层位置





type=ns 子域的地址





有缓存













Gnutella 随机抽取收到 pong 命令 作为自己的对等方

向自己的领居 汇报自己退出





集中式目录

完全分布式







混合式







BitTorrent：混合狮





检索一个文件，获取到这个文件关联的 peer 列表













ip 地址 hash 值 唯一 id





按照大小构成收尾相连的环



内容按照约定 存储在哪里









``` 
struct sockaddr_in {
short sin_family; //AF_INET 地址簿
u_short sin_port; // port struct 端口
in_addr sin_addr ; // IP address, ip地址
unsigned long char sin_zero[8]; // align 地址对齐
};
```







```
struct hostent // 域名解析
{
  char *h_name; // 域名
  char **h_aliases; // 别名
  int h_addrtype; 
  int h_length; /*地址长度*/ 
  char **h_addr_list; // 地址的链表
	#define h_addr h_addr_list[0]; 
}
```





