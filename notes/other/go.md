# Go 语言相关

##### 1、Go 语言环境安装

https://golang.org/dl/

```
wget -O /usr/local/go1.8.3.linux-amd64.tar.gz https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz

tar -C /usr/local -zxvf /usr/local/go1.8.3.linux-amd64.tar.gz

echo 'export PATH=$PATH:/usr/local/go/bin' |tee >> /etc/profile

source /etc/profile
```

##### 2、编写第一个程序

```
vim test.go

#hello world

package main

import "fmt" //必须双引号

func main() {
  fmt.Println("Hello World!")
}

go run test.go
```

> package main 定义了包名。你必须在源文件中非注释的第一行指明这个文件属于哪个包，如：package main。package main表示一个可独立执行的程序，每个 Go 应用程序都包含一个名为 main 的包。

> fmt 包实现了格式化 IO（输入/输出）的函数。

> func main() 是程序开始执行的函数。main 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有 init() 函数则会先执行该函数）。

> /*...*/ 是注释

当标识符（包括常量、变量、类型、函数名、结构字段等等）以一个大写字母开头，如：Group1，那么使用这种形式的标识符的对象就可以被外部包的代码所使用（客户端程序需要先导入这个包），这被称为导出（像面向对象语言中的 public）；标识符如果以小写字母开头，则对包外是不可见的，但是他们在整个包的内部是可见并且可用的（像面向对象语言中的 protected ）。

##### 变量声明

* 第一种，指定变量类型，声明后若不赋值，使用默认值。

  ```
  var v_name v_type
  v_name = value
  ```

* 第二种，根据值自行判定变量类型。

  ```
  var v_name = value
  ```

* 第三种，省略var, 注意 :=左侧的变量不应该是已经声明过的，否则会导致编译错误。

  ```
  v_name := value
  
  // 例如
  var a int = 10
  var b = 10
  c : = 10
  ```

##### TODO

```
//类型相同多个变量, 非全局变量
var vname1, vname2, vname3 type
vname1, vname2, vname3 = v1, v2, v3

var vname1, vname2, vname3 = v1, v2, v3 //和python很像,不需要显示声明类型，自动推断

vname1, vname2, vname3 := v1, v2, v3 //出现在:=左侧的变量不应该是已经被声明过的，否则会导致编译错误

// 这种因式分解关键字的写法一般用于声明全局变量
var (
    vname1 v_type1
    vname2 v_type2
)

//这种不带声明格式的只能在函数体中出现
//g, h := 123, "hello"

func main(){
    g, h := 123, "hello"
    println(x, y, a, b, c, d, e, f, g, h)
}

空白标识符 _ 也被用于抛弃值，如值 5 在：_, b = 5, 7 中被抛弃。
_ 实际上是一个只写变量，你不能得到它的值。这样做是因为 Go 语言中你必须使用所有被声明的变量，但有时你并不需要使用从一个函数得到的所有返回值。

常量的定义格式：
const identifier [type] = value

1<<i是把1左移i位，每次左移以为就是乘以2，所以1<<i的结果是1乘以2的i次方
i<<1就是把i左移一位，即i乘以2，假如i=5,最后结果就是5*2=10
至于为什么左移一位是乘以2，这是运算器内部机理，说起来就更多了，计算机做乘法运算的时候不是一个个的相加，而是用移位来实现的。>>这个符号是右移，与左移相反，右移是除以2.
这里还有一点容易搞错的，就是移位符号的左边是需要计算的数，右边是需要移动的位数。

const (
    i=1<<iota
    j=3<<iota
    k
    l
)

i= 1
j= 6
k= 12
l= 24

&	返回变量存储地址	&a; 将给出变量的实际地址。

- 指针变量。	*a; 是一个指针变量

当a是一个指针的时候，*a就是这个指针指向的内存的值
在定义的时候加了*的都是指针变量，都是一个地址。
在赋值的时候加了*的都是表示这个指针指向内存的值，在等号前面就是给这个值赋值，后面就是取这个值

#if
if a < 20 {
   /* 如果条件为 true 则执行以下语句 */
   fmt.Printf("a 小于 20\n" )
}else {
   /* 如果条件为 false 则执行以下语句 */
   fmt.Printf("a 不小于 20\n" );
}

#switch
func main() {
   /* 定义局部变量 */
   var grade string = "B"
   var marks int = 90

   switch marks {
      case 90: grade = "A"
      case 80: grade = "B"
      case 50,60,70 : grade = "C"
      default: grade = "D"  
   }

   switch {
      case grade == "A" :
         fmt.Printf("优秀!\n" )     
      case grade == "B", grade == "C" :
         fmt.Printf("良好\n" )      
      case grade == "D" :
         fmt.Printf("及格\n" )
      case grade == "F":
         fmt.Printf("不及格\n" )
      default:
         fmt.Printf("差\n" );
   }
   fmt.Printf("你的等级是 %s\n", grade );      
}

func main() {
   var x interface{}
     
   switch i := x.(type) {
      case nil:	  
         fmt.Printf(" x 的类型 :%T",i)                
      case int:	  
         fmt.Printf("x 是 int 型")                       
      case float64:
         fmt.Printf("x 是 float64 型")           
      case func(int) float64:
         fmt.Printf("x 是 func(int) 型")                      
      case bool, string:
         fmt.Printf("x 是 bool 或 string 型" )       
      default:
         fmt.Printf("未知型")     
   }   
}

<- 接收操作符，只作用于通道类型的值。使用时，需要注意两点： 
(1). 从一个通道类型的空值（即nil）接收值的表达式将会永远被阻塞。 
(2). 从一个已被关闭的通道类型值接收值会永远成功并立即返回一个其元素类型的零值。

一个由接收操作符和通道类型的操作数所组成的表达式可以直接被用于变量赋值或初始化，如下所示(在赋值语句讲解时，再细说)

v1 := <-ch
v2 = <-ch

特殊标记 = 用于将一个值赋给一个已被声明的变量或常量。 
特殊标记 := 则用于在声明一个变量的同时对这个变量进行赋值，且只能在函数体内使用。

又如下：

v, ok = <-ch
v, ok := <-ch

当同时对两个变量进行赋值或初始化时，第二个变量将会是一个布尔类型的值。这个值代表了接收操作的成功与否。如果这个值为false，就说明这个通道已经被关闭了。（之后讲解通道类型会详细介绍）。

由上至下代表优先级由高到低：
优先级	运算符
7	^ !
6	* / % << >> & &^
5	+ - | ^
4	== != < <= >= >
3	<-
2	&&
1	||



/* 函数返回两个数的最大值 */
func max(num1, num2 int) int {
   /* 定义局部变量 */
   var result int

   if (num1 > num2) {
      result = num1
   } else {
      result = num2
   }
   return result 
}

func swap(x, y string) (string, string) {
   return y, x
}

下面定义一个结构体类型和该类型的一个方法：
package main

import (
   "fmt"  
)

/* 定义函数 */
type Circle struct {
  radius float64
}

func main() {
  var c1 Circle
  c1.radius = 10.00
  fmt.Println("Area of Circle(c1) = ", c1.getArea())
}

//该 method 属于 Circle 类型对象中的方法
func (c Circle) getArea() float64 {
  //c.radius 即为 Circle 类型对象中的属性
  return 3.14 * c.radius * c.radius
}
以上代码执行结果为：
Area of Circle(c1) =  314

声明数组
var variable_name [SIZE] variable_type

var balance [10] float32

初始化数组

var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}

var balance = []float32{1000.0, 2.0, 3.4, 7.0, 50.0}

如果忽略 [] 中的数字不设置数组大小，Go 语言会根据元素的个数来设置数组的大小：

在指针类型前面加上 * 号（前缀）来获取指针所指向的内容。
package main

import "fmt"

func main() {
   var a int= 20   /* 声明实际变量 */
   var ip *int        /* 声明指针变量 */

   ip = &a  /* 指针变量的存储地址 */

   fmt.Printf("a 变量的地址是: %x\n", &a  )

   /* 指针变量的存储地址 */
   fmt.Printf("ip 变量储存的指针地址: %x\n", ip )

   /* 使用指针访问值 */
   fmt.Printf("*ip 变量的值: %d\n", *ip )
}
以上实例执行输出结果为：
a 变量的地址是: 20818a220
ip 变量储存的指针地址: 20818a220
*ip 变量的值: 20

Go 空指针
当一个指针被定义后没有分配到任何变量时，它的值为 nil。
nil 指针也称为空指针。
nil在概念上和其它语言的null、None、nil、NULL一样，都指代零值或空值。
一个指针变量通常缩写为 ptr。
```

