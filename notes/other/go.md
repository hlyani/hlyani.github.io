# Go 语言相关

# 一、常用

## 1、调试显示代码位置

```
log.SetFlags(log.Llongfile)
log.Print("") 
```

```
  where := func() {
    _, file, line, _ := runtime.Caller(1)
    log.Printf("%s:%d", file, line)
  }
  where()
```

## 2、go mod

```
go mod download  下载依赖的module到本地cache（默认为$GOPATH/pkg/mod目录）
go mod edit      编辑go.mod文件
go mod graph     打印模块依赖图
go mod init      初始化当前文件夹, 创建go.mod文件
go mod tidy      增加缺少的module，删除无用的module
go mod vendor    将依赖复制到vendor下
go mod verify    校验依赖
go mod why       解释为什么需要依赖
```

## 3、清理缓存

```
go clean -modcache
rm -rf mod.sum
go mod tidy
```

## 4、go.mod replace 本地源

```
//go env -w GOPRIVATE=192.168.0.90 GOINSECURE=192.168.0.90

github.com/containerd/containerd => 192.168.0.90/delta/deltacontainer v1.7.1-delta1.0.0
```

## 5、编译

```
go tool compile -d help
```

```
go build -ldflags="-s -w" -o main main.go
```

> -s 忽略符号表和调试信息
> -w 忽略DWARFv3调试信息，使用该选项后将无法使用gdb进行调试
>
> -gcflags="-l -N"可以关闭代码优化，从而缩短编译时间

## 6、压缩

```
upx
1-9，9代表最高压缩率  
go build -ldflags="-s -w" -o main main.go && upx -9 main
```

## 7、go mod

```
go mod download
```

> 下载依赖的module到本地cache（默认为$GOPATH/pkg/mod目录）

```
go mod edit
```

> 编辑go.mod文件

```
go mod graph
```

> 打印模块依赖图

```
go mod init
```

> 初始化当前文件夹, 创建go.mod文件

```
go mod tidy 
```

> 增加缺少的module，删除无用的module

```
go mod vendor
```

> 将依赖复制到vendor下

```
go mod verify
```

> 校验依赖

```
go mod why 
```

> 解释为什么需要依赖

# 二、Go 语言环境安装

https://golang.org/dl/

```
wget -O /usr/local/go1.20.3.linux-amd64.tar.gz https://golang.org/dl/go1.20.3.linux-amd64.tar.gz
tar -C /usr/local -zxvf /usr/local/go1.20.3.linux-amd64.tar.gz
echo "export PATH=$PATH:/usr/local/go/bin" |tee >> /etc/profile
source /etc/profile
```

# 三、基础语法

##### 1、第一个程序

```
vim hello.go

package main

import "fmt"

func main() {
  fmt.Println("Hello World!")
}

go run hello.go
```

> func main() 是程序开始执行的函数。main 函数是每一个可执行程序所必须包含的，一般来说都是在启动后第一个执行的函数（如果有 init() 函数则会先执行该函数）。

##### 2、声明

```
var a int = 10
var b = 10
c := 10
k1, k2, k3 = 1, 2, 3
_, b = 5, 7
var balance = [5]float32{1000.0, 2.0, 3.4, 7.0, 50.0}
a := [3][4]int{  
 {0, 1, 2, 3},
 {4, 5, 6, 7},
 {8, 9, 10, 11},
}

// 这种不带声明格式的只能在函数体中出现
vname1, vname2, vname3 := v1, v2, v3

// 这种因式分解关键字的写法一般用于声明全局变量
var (
    vname1 v_type1
    vname2 v_type2
)

空白标识符 _

常量的定义格式：
const identifier [type] = value
```

##### 3、字符串相关操作

```
// 拼接
fmt.Println("hello" + "world")

// %d 表示整型数字，%s 表示字符串
var stockcode=123
var enddate="2020-12-31"
var url="Code=%d & endDate=%s"
var target_url=fmt.Sprintf(url,stockcode,enddate)
fmt.Println(target_url)
```

##### 4、运算符

| 运算符                | 描述               |
| --------------------- | ------------------ |
| ++，--                |                    |
| ==，!=，>=，<=        |                    |
| &&，\|\|，!           |                    |
| =，+=，-=，*=，/=，%= |                    |
| <<=，>>=，&=，^=，\|= |                    |
| &                     | &a，变量的实际地址 |
| *                     | *a，指针变量       |

```
A=60 B=13
A = 0011 1100
B = 0000 1101
```

| 位运算符 | 描述                                                         | 示例                |
| -------- | ------------------------------------------------------------ | ------------------- |
| &        | 按位与运算符                                                 | A&B=0000 1100       |
| \|       | 按位或运算符                                                 | A\|B=0011 1101      |
| ^        | 按位异或运算符，两对应的二进位相异时，结果为1。              | A^B=0011 0001       |
| <<       | 左移运算符，左移n位就是乘以2的n次方。把<<左边的运算数的各二进位全部左移若干位。 | A<<2=240，1111 0000 |
| >>       | 右移运算符，右移n位就是除以2的n次方。把>>左边的运算数的各二进位全部右移若干位。 | A>>2=15，0000 1111  |

##### 5、条件语句、循环语句

```
if a < 20 {
	fmt.Printf("a 小于 20\n" );
} else if a == 20 {
	fmt.Printf("a 等于 20\n" );
} else {
	fmt.Printf("a 大于 20\n" );
}
```

```
switch {
    case grade == "A" :
    	fmt.Printf("优秀!\n" )    
    case grade == "B", grade == "C" :
    	fmt.Printf("良好\n" )      
    default:
    	fmt.Printf("差\n" );
}

fallthrough ?
```

```
func main() {
	var c1, c2, c3 chan int
	var i1, i2 int
	select {
	case i1 = <-c1:
		fmt.Printf("recived", i1, "from c1\n")
	case c2 <- i2:
		fmt.Printf("send", i2, "to c2\n")
	case i3, ok := (<-c3):
		if ok {
			fmt.Printf("recived", i3, "from c3\n")
		} else {
			fmt.Printf("c3 is closed\n")
		}
	default:
		fmt.Printf("no\n")
	}
}
```

```
sum := 0
for i := 0; i <= 10; i++ {
	sum += 1
}
```

```
for sum <= 10 {
	sum += sum
}
```

```
numbers := [6]int{1, 2, 3, 5}
    for i,x:= range numbers {
    	fmt.Printf("第 %d 位 x 的值 = %d\n", i,x)
    }
```

```
var a int = 10
LOOP: for a < 20 {
    if a == 15 {
        /* 跳过迭代 */
        a = a + 1
        goto LOOP
    }
    fmt.Printf("a的值为 : %d\n", a)
    a++    
}  
```

```
for true  {
	fmt.Printf("这是无限循环。\n");
}

for {
	fmt.Printf("这是无限循环。\n");
}
```

##### 6、指针

> 一个指针变量指向一个值的内存地址

```
var ip *int /*指向整型*/
var fp *float32 /*指向浮点型*/
if(ptr != nil)     /* ptr 不是空指针 */
if(ptr == nil)    /* ptr 是空指针 */
```

```
var a int = 20    /*声明实际变量*/
var ip *int       /*声明指针参数*/
ip = &a           /*指针变量赋值*/

fmt.Printf("a 变量的内存地址：%x\n", &a)
// 0xc0000160b0

fmt.Printf("ip 变量存储的指针地址：%x\n", ip)
// 0xc0000160b0

fmt.Printf("*ip 变量的值：%d\n", *ip)
// 20
```

##### 7、函数

```
func max(num1, num2 int) int {
   var result int

   if (num1 > num2) {
      result = num1
   } else {
      result = num2
   }
   return result
}
```

> 语言函数作为实参

```
package main

import (
   "fmt"
   "math"
)

func main(){
   /* 声明函数变量 */
   getSquareRoot := func(x float64) float64 {
      return math.Sqrt(x)
   }

   /* 使用函数 */
   fmt.Println(getSquareRoot(9))
}
```

> 闭包，可以直接使用函数内的变量，不必申明。

```
func getSequence() func() int {
	i := 0
	return func() int {
		i += 1
		return i
	}
}
```

##### 8、结构体

```
type Books struct {
   title string
   author string
   subject string
   book_id int
}

func main(){
	var Book1 Books
	Book1.title = "test"
	
	var struct_pointer *Books
	struct_pointer = &Book1
	struct_pointer.title
}
```

##### 9、切片 Slice

```
var slice1 []type = make([]type, length, capacity)
slice1 := make([]type, len)

var numbers = make([]int,3,5)
if(numbers == nil){
	fmt.Printf("切片是空的")
}
fmt.Printf("len=%d cap=%d slice=%v\n",len(numbers),cap(numbers),numbers)
// len()获取长度，cap()切片最长可以达多少
```

```
numbers := []int{0,1,2,3,4,5,6,7,8}  
printSlice(numbers)
fmt.Println("numbers[1:4] ==", numbers[1:4])

// 末尾插入0
numbers = append(numbers, 0)

// 保存到一个临时的切片
tmp := append([]int{}, slice[2:]...)

// 拷贝 numbers 的内容到 numbers1
numbers1 := make([]int, len(numbers), (cap(numbers))*2)
copy(numbers1, numbers)

// 删除下标为index的元素
slice = append(slice[:index], slice[index+1:]...)

// 删除index1~index2之间的元素
slice = append(slice[:index1], slice[index2:]...)
```

##### 10、范围 Range

```
nums := []int{2, 3, 4}
for k, v := range nums {
	fmt.Println(k, v)
}
```

##### 11、集合 Map

```
var map_variable map[key_data_type]value_data_type
或
map_variable := make(map[key_data_type]value_data_type)

countryCapitalMap := map[string]string{"France": "Paris", "Italy": "Rome"}

countryCapitalMap [ "France" ] = "巴黎"
countryCapitalMap [ "Italy" ] = "罗马"

delete(countryCapitalMap, "France")
```

##### 12、类型转换

```
var sum int = 17
var count int = 5
var mean float32
mean = float32(sum)/float32(count)

var a int64 = 3
var b int32
b = int32(a)
```

##### 13、错误处理

```
type error interface {
    Error() string
}
```

```
type DivideError struct {
    dividee int
    divider int
}

func (de *DivideError) Error() string {
    strFormat := `
    Cannot proceed, the divider is zero.
    dividee: %d
    divider: 0
`
    return fmt.Sprintf(strFormat, de.dividee)
}

func Divide(varDividee int, varDivider int) (result int, errorMsg string) {
    if varDivider == 0 {
            dData := DivideError{
                    dividee: varDividee,
                    divider: varDivider,
            }
            errorMsg = dData.Error()
            return
    } else {
            return varDividee / varDivider, ""
    }

}

func main() {
    // 正常情况
    if result, errorMsg := Divide(100, 10); errorMsg == "" {
            fmt.Println("100/10 = ", result)
    }
    // 当除数为零的时候会返回错误信息
    if _, errorMsg := Divide(100, 0); errorMsg != "" {
            fmt.Println("errorMsg is: ", errorMsg)
    }

}
```

##### 14、并发

> goroutine 是轻量级线程，goroutine 的调度是由 Golang 运行时进行管理的。同一个程序中的所有 goroutine 共享同一个地址空间。

```
package main

import (
	"fmt"
	"time"
)

func say(s string) {
	for i := 0; i < 5; i++ {
		time.Sleep(100 * time.Millisecond)
		fmt.Println(s)
	}
}

func main() {
	go say("world")
	say("hello")
}
```

##### 15、通道（channel）

> 通道是用来传递数据的一个数据结构，通道可用于两个goroutine之间通过传递一个指定类型的值来同步运行和通讯。操作符<-用于指定通道的方向，发送或接受。如果未指定方向，则为双向通道。

> 带缓冲区的通道允许发送端的数据发送和接收端的数据获取处于异步状态，不过缓冲区的大小是有限的。

> 如果通道不带缓冲区，发送方会阻塞直到接收方从通道中接受到了值。如果通道带缓冲，发送方则会阻塞直到发送的值被拷贝到缓冲区内；如果缓冲区已满，则意味着需要等待直到某个接收方获取到一个值，接收方在有值可以接受之前一直阻塞。

```
ch <- v   // 把 v 发送到通道 ch
v := <-ch // 从 ch 接受数据，并把值赋值给 v

ch := make(chan int, 100) // 通过 make 的第二个参数指定缓冲区大小
```

> 如果通道接受不到数据后 ok 就为 false，这时通道就可以使用 close() 函数来关闭。

```
v, ok := <-ch

func fibonacci(n int, c chan int) {
        x, y := 0, 1
        for i := 0; i < n; i++ {
                c <- x
                x, y = y, x+y
        }
        close(c)
}
```

# 四、实例

```
type MinStack struct {
    stack []int
    minStack []int
}


func Constructor() MinStack {
    return MinStack{
        stack: []int{},
        minStack: []int{math.MaxInt64},
    }
}


func (this *MinStack) Push(val int)  {
    this.stack = append(this.stack, val)
    this.minStack = append(this.minStack, Min(val, this.minStack[len(this.minStack) - 1]))
}
```

```
nums []int
sort.Ints(nums)

math.inf

math.MaxInt64
```
