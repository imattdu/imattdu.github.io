







静态数据

### matt

DAG有向无环图:任务1执行输出的结果作为任务2的输入 。。。。。



```sh
cd /opt/module/hadoop-3.1.3/share/hadoop/mapreduce
    
    
    
sz hadoop-mapreduce-examples-3.1.3.jar    
```









![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/15/20211215004015.png)























FIFO调度器:job1运行完才可以运行job2





c7关闭 虚拟内存检查





```xml
<!-- 选择调度器，默认容量 -->
<property>
<description>The class to use as the resource scheduler.</description>
<name>yarn.resourcemanager.scheduler.class</name>
<value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capaci
ty.CapacityScheduler</value>
</property>
<!-- ResourceManager 处理调度器请求的线程数量,默认 50；如果提交的任务数大于 50，可以
增加该值，但是不能超过 3 台 * 4 线程 = 12 线程（去除其他应用程序实际不能超过 8） -->
<property>
<description>Number of threads to handle scheduler
interface.</description>
<name>yarn.resourcemanager.scheduler.client.thread-count</name>
<value>8</value>
</property>

<!-- 是否让 yarn 自动检测硬件进行配置，默认是 false，如果该节点有很多其他应用程序，建议
手动配置。如果该节点没有其他应用程序，可以采用自动 -->
<property>
<description>Enable auto-detection of node capabilities such as
memory and CPU.
</description>
<name>yarn.nodemanager.resource.detect-hardware-capabilities</name>
<value>false</value>
</property>
<!-- 是否将虚拟核数当作 CPU 核数，默认是 false，采用物理 CPU 核数 -->
<property>
<description>Flag to determine if logical processors(such as
hyperthreads) should be counted as cores. Only applicable on Linux
when yarn.nodemanager.resource.cpu-vcores is set to -1 and
yarn.nodemanager.resource.detect-hardware-capabilities is true.
</description>
<name>yarn.nodemanager.resource.count-logical-processors-ascores</name>
<value>false</value>
</property>
<!-- 虚拟核数和物理核数乘数，默认是 1.0 -->
<property>
<description>Multiplier to determine how to convert phyiscal cores to
vcores. This value is used if yarn.nodemanager.resource.cpu-vcores
is set to -1(which implies auto-calculate vcores) and
yarn.nodemanager.resource.detect-hardware-capabilities is set to true.
The number of vcores will be calculated as number of CPUs * multiplier.
</description>
<name>yarn.nodemanager.resource.pcores-vcores-multiplier</name>
<value>1.0</value>
</property>
<!-- NodeManager 使用内存数，默认 8G，修改为 4G 内存 -->
<property>
<description>Amount of physical memory, in MB, that can be allocated
for containers. If set to -1 and
yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
automatically calculated(in case of Windows and Linux).
In other cases, the default is 8192MB.
</description>
<name>yarn.nodemanager.resource.memory-mb</name>
<value>2048</value>
</property>
<!-- nodemanager 的 CPU 核数，不按照硬件环境自动设定时默认是 8 个，修改为 4 个 -->
<property>
<description>Number of vcores that can be allocated
for containers. This is used by the RM scheduler when allocating
resources for containers. This is not used to limit the number of
CPUs used by YARN containers. If it is set to -1 and
yarn.nodemanager.resource.detect-hardware-capabilities is true, it is
automatically determined from the hardware in case of Windows and Linux.
In other cases, number of vcores is 8 by default.</description>
<name>yarn.nodemanager.resource.cpu-vcores</name>
<value>2</value>
</property>
<!-- 容器最小内存，默认 1G -->
<property>
<description>The minimum allocation for every container request at theRM in MBs. Memory requests lower than this will be set to the value of
this property. Additionally, a node manager that is configured to have
less memory than this value will be shut down by the resource manager.
</description>
<name>yarn.scheduler.minimum-allocation-mb</name>
<value>1024</value>
</property>
<!-- 容器最大内存，默认 8G，修改为 2G -->
<property>
<description>The maximum allocation for every container request at the
RM in MBs. Memory requests higher than this will throw an
InvalidResourceRequestException.
</description>
<name>yarn.scheduler.maximum-allocation-mb</name>
<value>2048</value>
</property>
<!-- 容器最小 CPU 核数，默认 1 个 -->
<property>
<description>The minimum allocation for every container request at the
RM in terms of virtual CPU cores. Requests lower than this will be set to
the value of this property. Additionally, a node manager that is configured
to have fewer virtual cores than this value will be shut down by the
resource manager.
</description>
<name>yarn.scheduler.minimum-allocation-vcores</name>
<value>1</value>
</property>
<!-- 容器最大 CPU 核数，默认 4 个，修改为 2 个 -->
<property>
<description>The maximum allocation for every container request at the
RM in terms of virtual CPU cores. Requests higher than this will throw an
InvalidResourceRequestException.</description>
<name>yarn.scheduler.maximum-allocation-vcores</name>
<value>2</value>
</property>
<!-- 虚拟内存检查，默认打开，修改为关闭 -->
<property>
<description>Whether virtual memory limits will be enforced for
containers.</description>
<name>yarn.nodemanager.vmem-check-enabled</name>
<value>false</value>
</property>
<!-- 虚拟内存和物理内存设置比例,默认 2.1 -->
<property>
<description>Ratio between virtual memory to physical memory when
setting memory limits for containers. Container allocations are
expressed in terms of physical memory, and virtual memory usage is
allowed to exceed this allocation by this ratio.
</description>
<name>yarn.nodemanager.vmem-pmem-ratio</name>
<value>2.1</value>
</property>
```









```sh
hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount /input /o2
```













```xml
<!-- 指定多队列，增加 hive 队列 -->
<property>
 <name>yarn.scheduler.capacity.root.queues</name>
 <value>default,hive</value>
 <description>
 The queues at the this level (root is the root queue).
 </description>
</property>
<!-- 降低 default 队列资源额定容量为 40%，默认 100% -->
<property>
 <name>yarn.scheduler.capacity.root.default.capacity</name>
 <value>40</value>
</property>
<!-- 降低 default 队列资源最大容量为 60%，默认 100% -->
<property>
 <name>yarn.scheduler.capacity.root.default.maximum-capacity</name>
 <value>60</value>
</property>
```





```xml
<!-- 指定 hive 队列的资源额定容量 -->
<property>
 <name>yarn.scheduler.capacity.root.hive.capacity</name>
 <value>60</value>
</property>
<!-- 用户最多可以使用队列多少资源，1 表示 -->
<property>
 <name>yarn.scheduler.capacity.root.hive.user-limit-factor</name>
 <value>1</value>
</property>
<!-- 指定 hive 队列的资源最大容量 -->
<property>
 <name>yarn.scheduler.capacity.root.hive.maximum-capacity</name>
 <value>80</value>
</property>
<!-- 启动 hive 队列 -->
<property>
 <name>yarn.scheduler.capacity.root.hive.state</name>
 <value>RUNNING</value>
</property>


<!-- 哪些用户有权向队列提交作业 -->
<property>
 <name>yarn.scheduler.capacity.root.hive.acl_submit_applications</name>
 <value>*</value>
</property>
<!-- 哪些用户有权操作队列，管理员权限（查看/杀死） -->
<property>
 <name>yarn.scheduler.capacity.root.hive.acl_administer_queue</name>
 <value>*</value>
</property>
<!-- 哪些用户有权配置提交任务优先级 -->
<property>

<name>yarn.scheduler.capacity.root.hive.acl_application_max_priority</nam
e>
 <value>*</value>
</property>
<!-- 任务的超时时间设置：yarn application -appId appId -updateLifetime Timeout
参考资料： https://blog.cloudera.com/enforcing-application-lifetime-slasyarn/ -->
<!-- 如果 application 指定了超时时间，则提交到该队列的 application 能够指定的最大超时
时间不能超过该值。
-->
<property>
 <name>yarn.scheduler.capacity.root.hive.maximum-applicationlifetime</name>
 <value>-1</value>
</property>
<!-- 如果 application 没指定超时时间，则用 default-application-lifetime 作为默认
值 -->
<property>
 <name>yarn.scheduler.capacity.root.hive.default-applicationlifetime</name>
 <value>-1</value>
</property>
```



```sh
vim capacity-scheduler.xml
```











```sh
hadoop jar ./share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar wordcount -D mapreduce.job.queuename=hive /input /o3



```





优先级

```sh
hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar pi 5 2000000


# 优先级高
hadoop jar /opt/module/hadoop-3.1.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.1.3.jar pi -D mapreduce.job.priority=5 5 2000000

```





yarn tool 解决自定义程序无法指定yarn参数

```sh
yarn jar stu-yarn-1.0-SNAPSHOT.jar com.matt.yarn.WordCountDriver wordcount /input /o4



yarn jar stu-yarn-1.0-SNAPSHOT.jar com.matt.yarn.WordCountDriver wordcount -Dmapreduce.job.priority=5 /input /041

yarn jar stu-yarn-1.0-SNAPSHOT.jar com.matt.yarn.WordCountDriver wordcount -D mapreduce.job.priority=5 /input /042


stu-yarn-1.0-SNAPSHOT.jar
```

