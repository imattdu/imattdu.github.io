### vim



### 命令模式

```
:set nu


```

### 退出模式

```
esc

:wq
:q!
```





### 编辑模式

进入编辑模式

- i：光标位置前
- I：当前行首个非空字符前
- a：光标位置后
- A：当前行最后一个非空字符后
- o:下一行行首
- O:上一行行首



#### 跳到指定行

```
normal模式

# 跳到第n行
:n
:4

# 当前位置上下跳n行
:+/-n
+1

#查找 /abc -> n从上往下 N从下往上
/abc


#跳到文件末尾
a.shift + g
b.:$


```

#### 检索

```

# 前往后找, n找下一个
/abc


# 后往前找，N找上一个
?abc


# f从前往后 F从后往前
fx
Fx



```







### awk



#### 基础用法

```
# 输出每一行第一列，第二列
awk '{print $1,$2}' a.txt
# 格式化输出
awk '{printf "%s,%s",$1,$2}' a.txt

# 指定分隔符
awk -F, '{print $1}' a.txt 
# 指定多个分隔符
awk -F '[@,]' '{print $1,$2}' a.txt
```



```
格式化输出
%9s 前面补空格
$-9s 后面补空格
%08s 前面补0
```





#### 运算符

| op           | desc         |      |
| ------------ | ------------ | ---- |
| +- */ &      | 加减乘除取余 |      |
| ==,>=,<=,>,< | 逻辑运算     |      |
| &&,\|\|      | 关系运算     |      |



```
awk '$1>2 && $2s==3 {print $0}' a.txt
```



#### 内建变量

| var        | desc                    |
| ---------- | ----------------------- |
| $1         | 第几个字段              |
| $0         | 一行数据                |
| NR, FNR    | 输出序号，文件行数      |
| IGNORECASE | =1,忽略大小写,不适用mac |
|            |                         |



```
awk '{print NR,FNR,$1}'
```



#### 正则

```
awk '$1 ~ /abc/ {print $0}' a.txt

awk '/abc/ {print 0}' a.txt


#取反
awk '$1 !~ /abc/ {print $0}' a.txt
awk '!/abc/' a.txt

#忽略大小写
awk 'BEGIN {IGNORECASE=1} {print $0}' a.txt
```



#### awk脚本



代码结构

```
# 执行前
BEFORE {

}

#执行
{

}

执行后
END {

}

```



引用脚本文件

```
awk -f a.awk a.txt
```



```
awk '{count += gsub("abc", "")} END {print count}' 文件名


awk '{count += gsub("abc", "")} END {print count}' a.txt
```







### wc

统计文件的行数、单词数、字节

```
wc a.txt
#行数 单词数 字节
      2       2      10 a.txt
      
      
# c字节 l行数 w单词数
wc -l a.txt      
```





### sed

#### 基础用法



```

# 添加
# 行后添加
nl a.txt | sed '1a abc'
# 行前添加
nl a.txt | sed '4i abc'


# 删除指定行
nl a.txt | sed '4d'

nl a.txt | sed '4,5d'
nl a.txt | sed '4,$d'

# 替换
nl a.txt | sed '3,4c abc'


# 输出
nl a.txt | sed -n '3,4p'



```



#### 搜寻删除

```
nl a.txt | sed '/abc/d'
```



#### 搜寻替换

```
nl a.txt | sed 's/abc/ff/g'
```



#### 搜寻执行

```
nl a.txt | sed -n '/abc {s/gg/hh/g;p}'
```



#### 多次编辑

```
nl a.txt | sed -e '3,4d' -e '1a ff'
```



#### 执行文件

```
sed -i '1,2d' a.txt
```





### 磁盘

```
# 查看分区
df -h

# 查看磁盘占比
du -h a.txt
```





cpu, mem

```
top

mac: o cpu,mem

linux: P M


ps -ef


lsof -i:8000

lsof -nP -p 9000 | grep LISTEN
```



