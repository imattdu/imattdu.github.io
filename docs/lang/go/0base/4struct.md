





# 结构体

## 概述

go 语言并没有类，而采用结构体，仍然有面向对象中封装继承多态等特性。



## 结构体使用



### 声明结构体



```go
type 结构体名 struct {
    字段名 类型
}
```





```go
type Person struct {
	Name string
	Age  int
    // 不是一个包访问不到
	p *int
	slice []int
	map1 map[string]string
}
```



结构体是值类型

**创建一个结构体变量后，如果没有给字段赋值，都会有零值，如果是引用类型，在没有进行分配内存则它的值为 nil 。**



### 结构体初始化



```go
func main() {
	// 1
	var person Person
	person.Age = 11
	person.Name = "matt"
	fmt.Println(person)
	//2
	var p1 = Person{}
	fmt.Println(p1)

	// 3
	var p3 *Person = new(Person)
	(*p3).Name = "33"
	p3.Name = "44优化等价于上面的"
	fmt.Println(*p3)

	// 4
	var p4 *Person = &Person{}
	p4.Age = 111
	fmt.Println(*p4)
}
```



创建结构体变量也可以指定字段值

```go
func main() {
	var c Computer = Computer{
		Age: 1,
		Name: "matt",
	}
	fmt.Println(c)
}
```

可以只写部分字段，这样指定字段可以不按结构体中的顺序，否则就要按结构体中的顺序。



### 结构体使用注意

1.结构体所有字段在内存中连续的

2.struct 的每个字段上，可以写上一个 tag, 该 tag 可以通过反射机制获取，常见的使用场景就是序列化和反序列化。



```go
import "fmt"
import "encoding/json"

type A struct {
	age int
	num int
}

type B struct {
	age int
	num int
}

type C struct {
	Name string `json:"name"`
}
func main() {
	var a A
	var b B
	a.age = 1
	// 要求字段一致
	b = B(a)
	var c C
	c.Name = "hello ma"
	str1, _ := json.Marshal(c)
	fmt.Println(string(str1))
	fmt.Println(b)
}

```

![](http://raw.githubusercontent.com/imattdu/img/main/img/20210701154924.png)

3.俩个结构体进行转换的时候，要求这俩个结构体需要有相同的字段



```go
a = B(b)
```

4.对结构体进行重新定义， golang 认为是不同的数据类型

```go
type P Person
```





# 方法



## 使用



```go
import "fmt"

type Person struct {
	Age int
}

func (p Person) getAge() int {
	return p.Age
}

func main() {
	var p Person
	p.Age = 18
	fmt.Println(p.getAge())

}
```

该方法和 Person 结构体进行绑定

方法进行调用会把调用该方法的变量赋值给该方法



## **方法使用注意**



1.**结构体类型是值类型，在方法调用中，遵守值类型的传递机制，是值拷贝传递方式**



**方法也可以接收结构体指针,引用传递**

方法接收类型是什么就是什么传递



```go
func (circle *Circle) test()  {
	// (*circle).R
    // 进行了优化
	fmt.Println(circle.R)
}
```



2.方法不仅可以作用于结构体，还可以作用于 int,string等



```go
import "fmt"

func (i integer) testInt() {
	fmt.Println(i)
}

type integer int

func main() {
	var j integer = 10
	j.testInt()
}
```

3.方法的访问范围控制的规则，和函数一样。方法名首字母小写，只能在本包访问，方法首字母
大写，可以在本包和其它包访问。



4.如果一个类型实现了 String()这个方法，那么 fmt.Println 默认会调用这个变量的 String()进行输出，类似于 java 中的 toString() 方法



```go
func (c Computer) String() string {
	str := fmt.Sprintf("Name=%s Age=%d", c.Name, c.Age)
	return str
}
```



# 面向对象特性

## 封装

### 概念

将对象属性隐藏起来，属性的操作只可以通过被授权的方法。



### 使用

结构体属性、方法首字母小写，其他包就不可以访问，当前包仍然可以访问。



*为结构体提供一个创建的函数，相当于java中的构造函数。*



*编写set,get 方法。*



```go
package main

type Person struct {
	name string
	age  int
}

func NewPerson() Person {
	var person = Person{}
	return person
}

func (p *Person) SetName(name string) {
	p.name = name
}

func (p *Person) GetName() string {
	return p.name
}

```

## 继承



### 概述

子类具有父类的属性和方法



### 使用



```go
type S struct {
	Name string
	Age int
}

type X struct {
	S
}

func (s S) showInfo() {
	fmt.Println(s.Name)
}

func (x *X) showInfo() {
	fmt.Printf("x---name=%v age=%v\n",x.Name, x.Age)
}

func main() {
	var x  = &X{}
	x.S.Name = "matt"
	x.S.Name = "aa"
	x.Name = "ma"

	x.S.showInfo()

	x1 := X{
		S{
			Name: "aa",
			Age: 11,
		},
	}
	fmt.Println(x1)


}
```



### 继承使用注意

结构体可以使用嵌套匿名结构体所有的字段和方法(首字母大小写都可以获得)



结构体访问可以简化



```go
func main() {
	var x  = &X{}
	x.S.Name = "matt"
	// 对上面进行简化
	x.Name = "ma"
	x.S.showInfo()

}
```

1.上述Name字段，编译器首先会从 X 中查找该属性，找不到在从不S中找，最后找不到则报错。

当结构体和匿名结构体有相同的字段或者方法时，编译器采用就近访问原则访问，如希望访问
匿名结构体的字段和方法，可以通过匿名结构体名（x.S.Name）来区分



2.结构体嵌入两个(或多个)匿名结构体，如两个匿名结构体有相同的字段和方法(同时结构体本身 没有同名的字段和方法)，在访问时，就必须明确指定匿名结构体名字，否则编译报错。



3.如果一个 struct 嵌套了一个有名结构体，这种模式就是组合，如果是组合关系，那么在访问组合 的结构体的字段或方法时，必须带上结构体的名字 **字段需要指定 方法不需要指定**



```go
type A struct {
	Name string
}

type B struct {
	a A
}

func main() {
	var b B = B{}
	b.a.Name = "matt"
	fmt.Println(b)
}
```

可以在创建结构体变量时指定匿名结构体值



```go
var a A = B{
    B{
        Name: "matt sir"
    }
}
```



```go
package main

import "fmt"

type F1 struct {
	Name string
}

type S1 struct {
	F1
	Age int
}

func main() {
	s1 := S1{
		Age: 11,
		F1: F1{
			Name:"matt",
		},
	}

	fmt.Println(s1)
}

```







## 接口



### 使用

```go
type I interface {
	start()
}

type A struct {
}

func (a *A) start() {
	fmt.Println("A 开始...")
}
```

结构体 A 实现接口 I

### 使用接口注意



1.接口中不可以有变量,只有方法，接口中所有的方法都没有实现



2.一个自定义类型（包括结构体）实现了一个接口的所有方法，就认为实现了该接口





3.接口本身不能创建实例,但是可以指向一个实现了该接口的自定义类型的变量(实例)

interface 类型默认是一个指针(引用类型)，如果没有对 interface 初始化就使用，那么会输出 nil





```go
// I是接口 s 是S的实例，S实现I
var s I
```





4.一个接口可以继承多个接口，比如A接口继承B，C, 如果实现A接口，那么就要把A,B,C中所有方法实现。





5.空接口是没有方法，即任何变量实现了空接口



### 实现和继承



继承的价值主要在于：解决代码的复用性和可维护性。 

接口的价值主要在于：设计，设计好各种规范(方法)，让其它自定义类型去实现这些方法





## 多态



多态参数：

一个方法中的一个参数是I类型，那么就可以接收实现I类型的所有类



多态数组

一个数组中的I类型，那么就可以接收实现I类型的所有类





## 类型断言



### 概述

一个变量知道他是某一个接口类型，但是不知道它是哪一个具体类型，所以使用断言。





### 使用



```go
func main() {
	var a interface{}
	var b B = B{}
	a = b
	c, ok := a.(B)
	fmt.Println(ok)
	fmt.Println(c)
}
```















## 其他





### 反引字段



```go
type User struct {
	Id   int64  `json:"id"`
	Name string `json:"name"`
	Age  int64  `json:"age"`
	// omitempty 如果不设置这个字段就不会输出
	// 不加omitempty，如果不设置这个字段也会输出默认值
	H string `json:"h,omitempty"`
}
```









get: 得到，获得
select：选择
find：强调找的结果。
search：强调找的过程。

我用一句话让你明白区别。先SEARCH，再FIND(OUT），再SELECT，最后GET

