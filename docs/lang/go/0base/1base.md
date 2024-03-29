







## 基础

### 程序

新建项目

项目下创建src文件夹，创建bin文件夹，创建pkg文件夹

```go
package main

import "fmt"

func main() {
	fmt.Println("hello word")
}
```

### 编译运行

#### 编译运行分开

编译

```bash
go build hello.go
```

运行

```bash
hello.exe
```

#### 编译运行不分开

```bash
go run hello.go
```

#### 分开和不分开区别

1) 如果我们先编译生成了可执行文件，那么我们可以将该可执行文件拷贝到没有 go 开发环境的机器上，仍然可以运行 

2) 如果我们是直接 go run go 源代码，那么如果要在另外一个机器上这么运行，也需要 go 开发环境，否则无法执行。 

3) 在编译时，编译器会将程序运行依赖的库文件包含在可执行文件中，所以，可执行文件变大了很多。



### 注意事项

go源文件以 go 为扩展名

go 的入口函数是 main() 函数

go 严格区分大小写

go 语句不需要加分号，go语言自动会加分号

**go语句是一行一行编译，所以一行不可以写多条语句**

**引用的包或者定义的变量必须使用，不使用会报错**



### 注释



```go
package main

import "fmt"
import "time"

func main() {
	// 行注释，推荐使用行注释
	/*
	  块注释
	  你好块注释
	*/
	fmt.Println(time.Now())
}

```

## 变量



### 基础

#### 转义字符

一些特殊的字符：比如用户想要我们打出换行

```bash
1) \t : 表示一个制表符，通常使用它可以排版。

2) \n ：换行符 

3) \\ ：一个\ 

4) \\" ：一个"

5) \r ：一个回车
	fmt.Println("aaa\r bb");
```

#### 变量声明

指定变量类型，

整数：0

浮点数：0

bool:false

```go
// 如果不赋值，则使用它的默认值
var j int
```

不指定类型，自己判断

```go
var i = 100
var j
```

第三种,省略var

```go
// i没有声明过
i := 100
```

多变量声明

```go
var a, b, c int
fmt.Println(a, b, c)

var d, e, f = 1, 2, 3
fmt.Println(d, e, f)

g, h := "1", "2"
fmt.Println(g, h)
```

全局变量

```go

// 全局变量
var a1 = 100
var a2 = 200
// 推荐使用
var (
	a3 = 300
	a4 = 400
)
```



常量

```go
// 常量
const i = 10
```





### 数据类型

#### 基本数据类型



##### 整数

有符号

![](https://raw.githubusercontent.com/imattdu/img/main/img/20210521005142.png)



无符号

![](https://raw.githubusercontent.com/imattdu/img/main/img/20210521005217.png)



![](https://raw.githubusercontent.com/imattdu/img/main/img/20210521005400.png)

```go
package main

// import "fmt"
// import "unsafe"
import (
	"fmt"
	"unsafe"
)

func main() {
	var i int = 100
	fmt.Println(i, unsafe.Sizeof(i))
}
```



##### 浮点数

系统默认使用float64,推荐使用float64

| 类型    | 占用存储大小 | 范围              |
| ------- | ------------ | ----------------- |
| float32 | 4            | -3.4E38～3.4E38   |
| float64 | 8            | -1.7E308～1.7E308 |



```go
package main

import (
	"fmt"
)

func main() {
	var i float64 = 1.1
	fmt.Println("浮点数", i)
}
```



##### 字符

使用'',字符有一个码值

```go
package main

import "fmt"

func main() {
	var i byte = '1'
	fmt.Println(i)
	fmt.Printf("%c\n", i)
	// %d %s %c
	// 如果一个字符超过byte表示的类型，使用int类型即可
	var j int = '中'
	fmt.Printf("%c", j)
}
```

字符

存储到 计算机中，需要将字符对应的码值（整数）找出来 

存储：字符--->对应码值---->二进制-->存储 

读取：二进制----> 码值 ----> 字符 --> 读取

​	





##### bool



```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	var i bool = false
	fmt.Println(i, unsafe.Sizeof(i))
}
```



##### string



```go
package main

import "fmt"

func main() {
	var name string = "matt"
	var address = `
		hahahninrerer
	`
	fmt.Println(name)
	fmt.Println(address)
   
}
```

1一般使用"",也可以输出反引号``

2反引号可以赋值多行





3字符串拼接

+要写在每一行后面

```go
var address = "hello" +
	"word"
```

4

```go
 // 多行输出
	fmt.Println("hello word",
		"你好") 
```



#### 类型转换



##### 基本数据类型转换



**注意**

```go
var i int8 = 10
// i + 8 是int8类型
```

一般都是强转

```go
var i int8 = 1
var j = int16(i)
```

##### 基本类型转字符串

使用fmt下的Sprintf



```go
package main

import (
    "fmt"
)

func main() {
    var i int8 = 100
    var address string = fmt.Sprintf("%d hello3434", i)
   
    fmt.Println(address)
}
```

使用strconv包





##### 字符串转其他基本数据类型

使用strconv包

![](https://raw.githubusercontent.com/imattdu/img/main/img/20210521011754.png)



```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
   
	var i string = "100"
	var j int64
    // 因为这个函数会有俩个返回值
	j, _ = strconv.ParseInt(i, 10, 8)
	fmt.Println(j)

}
```



**注意**

如果是"hello"转int，那么直接转为0，并不会报错

高精度到低精度只会精度丢失，并不会报错



#### 指针类型



##### 基础



```go
package main

import "fmt"

func main() {
	var num int = 10
	var p *int = &num
	fmt.Println(p)
	*p = 20
	fmt.Println(num)
}
```

&num:获取num地址

*int:int类型的指针

*p:获取p指针指向变量的地址

##### 值类型和引用类型

(1) 值类型，都有对应的指针类型， 形式为 *数据类型，比如 int 的对应的指针就是 *int, float32对应的指针类型就是 *float32, 依次类推。 

(2) 值类型包括：基本数据类型 int 系列, float 系列, bool, string 、数组和结构体 struct



(1) 值类型：基本数据类型 int 系列, float 系列, bool, string 、数组和结构体 struct
(2) 引用类型：指针、slice 切片、map、管道 chan、interface 等都是引用类型





**值类型：变量直接存储值，内存通常在栈中分配**

**引用类型：变量存储的是一个地址，这个地址对应的空间才真正存储数据(值)，内存通常在堆**
**上分配，当没有任何变量引用这个地址时，该地址对应的数据空间就成为一个垃圾，由GC 来回收**



### 标识符

#### 基础

Golang 对各种变量、方法、函数等命名时使用的字符序列称为标识符

凡是自己可以起名字的地方都叫标识符



#### 命名规则



(1) 由 26 个英文字母大小写，0-9 ，_ 组成

(2) 数字不可以开头。var num int //ok var 3num int //error

(3) Golang 中严格区分大小写

(4) 下划线"_"本身在Go 中是一个特殊的标识符，称为空标识符。可以代表任何其它的标识符，但 是它对应的值会被忽略(比如：忽略某个返回值)。所以仅能被作为占位符使用，不能作为标识符使用





#### 系统保留关键字

![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522201843.png)



#### 系统的预定义标识符



![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522201953.png)





**当然，这是在 go 中，go 的关键字多是语句语法，而预定义标识符多是类型**

## 运算符

### 算术运算符





![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522170845.png)



#### 注意事项

##### 1.取模%

**a % b = a - a / b * b**



与a的正负一致

##### 2.除法

```go
10/3   = 3
```

##### 3.++ --

go 语言只支持 i++, i-- 这样的，没有 --i , ++i 同时 j = i++这样也是错误的

### 关系运算符





![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522171257.png)







### 逻辑运算符





![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522171414.png)







短路

(1) &&也叫短路与：如果第一个条件为 false，则第二个条件不会判断，最终结果为 false 

(2) ||也叫短路或：如果第一个条件为 true，则第二个条件不会判断，最终结果为 true



### 赋值运算符





![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522172532.png)







### 原码反码补码

正数和0：原码反码补码相同

负数的反码：符号位不变，其他位取反

负数的补码：反码+1



**计算机都是以补码进行运算**





### 位运算符



![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522172931.png)





![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522172551.png)



因此所说的 有符号、无符号 看的就是二进制的符号位， 无符号:3就不管符号位，右移只填充0；有符号，就是符号位是啥，我就填充啥，Java中也是同理。





### 运算符优先级



![](http://raw.githubusercontent.com/imattdu/img/main/img/20210522172800.png)







1：括号，++, --

2: 单目运算 

3：算术运算符 





4：移位运算 

5：关系运算符 

6：位运算符 





7：逻辑运算符 

8：赋值运算符 

9：逗号



### **go没有三木运算符**



## 流程控制



### if else



```GO
package main

import (
	"fmt"
)

func main() {
	var age int
	fmt.Scanln(&age)
	if age < 0 {
		fmt.Println("错误的数据")
	} else if age < -1 {
		fmt.Println("未成年")
	} else {
		fmt.Println("成年人")
	}
}
```

**注意：即使if语句中只有以及仍然要加{}**



### switch



```go

package main

import (
	"fmt"
)

func main() {
	var ch byte
	// 不要使用 Scanln

	fmt.Scanf("%c", &ch)
	// switch 'a'
	// switch test('a')
	switch ch {
		case 'a', 'b':
			fmt.Println("星期一或者星期二")
		case 'c':
			fmt.Println("星期三")
		default:
			fmt.Println("其他")
	}

	

	
}
```

switch：不需要写break，go已经帮我们添加了

golang 的 case 后的表达式可以有多个，使用 逗号 间隔,表示或的意思

case 后的各个表达式的值的数据类型，必须和 switch 的表达式数据类型一致

case/switch 后是一个表达式( 即：常量值、变量、一个有返回值的函数等都可以)

```go
// switch 'a'
// switch test('a')
```

default:不是必须的

case 后面的表达式如果是常量值(字面量)，则要求不能重复



switch:可以声明一个变量，不推荐

```go
switch age := 12; {
	
	case age == 18:
		fmt.Println("18")
		// 穿透
		fallthrough
	case age < 18:
		fmt.Println("未成年")
		fallthrough
	default:
		fmt.Println("成年")
}
```

switch：模拟ifel

```go
switch {
case 1 == 1 && 2 == 3:
	fmt.Println("error")
default:
	fmt.Println("true")
}
```



switch 穿透-fallthrough ，如果在 case 语句块后增加 fallthrough ,则会继续执行下一个 case，也 叫 switch 穿透,但是只可以穿透一次



### 6.3for//**go没有while**



```go
package main

import "fmt"

func main() {

	
	for i := 0; i < 10; i++{
		fmt.Println("matt", i)
	}
	
}

```



```go
i := 0
for i < 10 {
	fmt.Println("matt", i)
    i++
}
```

遍历字符串

根据字符遍历的

```go
var str = "hello wordz中文"
// 如果使用传统的字符串遍历就会出错，因为3个字节
for index, val := range str {
	//fmt.Println(index, val)
	fmt.Printf("%d %c\n", index, val)
}
```

### 6.4break

break:跳出for循环



### 6.5continue

continue:跳出当前循环



### 6.6label

label:使用break跳出指定的for循环

同样continue也可以使用

```go
package main

import "fmt"

func main() {

	label1:
		for i := 0; i < 10; i++ {
			for j := 0; j < 10; j++ {
				if j == 3 {
					break label1
				}
				fmt.Println(j)
			}
			fmt.Println(i)
		}
}

```

### 6.7goto

goto:跳转到指定的行，不推荐使用



```go
package main

import (
	"fmt"
)

func main() {

	goto label1
	fmt.Print("不执行")
	label1:
		fmt.Print("执行")
}
```

```
输出：执行
```













# **问题**

**在函数外使用如下就会出错**

```go
i := 1

// 等价于
var i int
// i = 1 赋值语句不可以在函数外使用，是执行语句
i = 1
```













