# 包、变量与函数
包：
mcall函数的调用，表明要切换栈了。
此代码用圆括号将导入的包分成一组，这是“分组”形式的导入语句。
使用分组导入语句要更好。--分开的话，底层也会把它打包成一个组；而且写那么多import也很累。
在 Go 中，如果一个名字以大写字母开头，那么它就是已导出的。在导入一个包时，你只能引用其中  **已导出**  的名字。 任何 **「未导出」** 的名字在该包外均无法访问。

函数：
```go
package main
import "fmt"
func add(x int, y int) int {
	return x + y
}
func main() {
	fmt.Println(add(42, 13))
}
```
当连续两个或多个函数的已命名形参类型相同时，除最后一个类型以外，其它都可以省略。
```go
package main
import "fmt"
func add(x, y int) int {
	return x + y
}
func main() {
	fmt.Println(add(42, 13))
}
```
函数可以返回任意数量的返回值。
Go 的返回值可被命名，它们会被视作定义在函数顶部的变量。返回值的命名应当能反应其含义，它可以作为文档使用。

```go
func swap(x, y string) (string, string) {
	return y, x
}

func main() {
	a, b := swap("hello", "world")
	fmt.Println(a, b)
}
```
函数可以返回任意数量的结果。

没有参数的 `return` 语句会直接返回已命名的返回值，也就是「裸」返回值。
裸返回语句应当仅用在下面这样的短函数中。在长的函数中它们会影响代码的可读性。
```go
package main
import "fmt"
func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}
func main() {
	fmt.Println(split(17))
}
```

`var` 语句用于声明一系列变量。和函数的参数列表一样，类型在最后。
```go
var c, python, java bool

func main() {
	var i int
	fmt.Println(i, c, python, java)
}
>>> 0 false false false
```
**如果提供了初始值，则类型可以省略** ,变量会从初始值中推断出类型。
```go
package main
import "fmt"
func main() {
	var i, j int = 1, 2
	k := 3
	c, python, java := true, false, "no!"
	fmt.Println(i, j, k, c, python, java)
}
```
在函数中，短赋值语句 `:=` 可在隐式确定类型的 `var` 声明中使用。
函数外的每个语句都 **必须** 以关键字开始（`var`、`func` 等），用处：
- **`:=` 结构不能在函数外使用**
- 可以对**已有同名变量**再次赋值

和导入语句一样，变量声明也可以「分组」成一个代码块。
```go
package main
import (
	"fmt"
	"math/cmplx"
)
var (
	ToBe   bool       = false
	MaxInt uint64     = 1<<64 - 1
	z      complex128 = cmplx.Sqrt(-5 + 12i)
)
func main() {
	fmt.Printf("类型：%T 值：%v\n", ToBe, ToBe)
	fmt.Printf("类型：%T 值：%v\n", MaxInt, MaxInt)
	fmt.Printf("类型：%T 值：%v\n", z, z)
}
```
`int`、`uint` 和 `uintptr` 类型在 32-位系统上通常为 32-位宽，在 64-位系统上则为 64-位宽。当你需要一个整数值时应使用 `int` 类型， 除非你有特殊的理由使用固定大小或无符号的整数类型。
```go
bool
string

int  int8  int16  int32  int64
uint uint8 uint16 uint32 uint64 uintptr

byte // uint8 的别名
rune // int32 的别名
     // 表示一个 Unicode 码位

float32 float64
complex64 complex128
```
没有明确初始化的变量声明会被赋予对应类型的 **零值**。
零值是：
- 数值类型为 `0`
- 布尔类型为 `false`
- 字符串为 `""`（空字符串）

类型转换（强转）：
表达式 `T(v)` 将值 `v` 转换为类型 `T`，Go 在不同类型的项之间赋值时需要显式转换。
```go
package main
import (
	"fmt"
	"math"
)
func main() {
	var x, y int = 3, 4
	var f float64 = math.Sqrt(float64(x*x + y*y))
	var z uint = uint(f)
	fmt.Println(x, y, z)
}
```
类型推断:
在声明一个变量而不指定其类型时（即使用不带类型的 `:=` 语法 `var =` 表达式语法），变量的类型会通过右值推断出来。
```go
i := 42           // int
f := 3.142        // float64
g := 0.867 + 0.5i // complex128
```
不过当右边包含未指明类型的数值常量时，新变量的类型就可能是 `int`、`float64` 或 `complex128` 了，这取决于常量的精度.

常量：
- 常量的声明与变量类似，只不过使用 `const` 关键字。
- 常量可以是字符、字符串、布尔值或数值。
- 常量不能用 `:=` 语法声明。
数值常量是高精度的 **值**。一个未指定类型的常量由上下文来决定其类型。
# 流程控制语句
Go 的 `for` 语句后面的三个构成部分外没有小括号， 大括号 `{ }` 则是必须的。
初始化语句和后置语句是可选的。
```go
func main() {
	sum := 1
	for ; sum < 1000; {
		sum += sum
	}
	fmt.Println(sum)
}
```
C 的 while 在 Go 中叫做 for。如下：
```go
func main() {
	sum := 1
	for sum < 1000 {
		sum += sum
	}
	fmt.Println(sum)
}
```
如果省略循环条件，该循环就不会结束，因此无限循环可以写得很紧凑。
```go
func main() {
	for {
	}
}
```

Go 的 `if` 语句与 `for` 循环类似，表达式外无需小括号 `( )`，而大括号 `{ }` 则是必须的。
和 `for` 一样，`if` 语句可以在条件表达式前执行一个简短语句。该语句声明的变量作用域仅在 `if`之内。
在 `if` 的简短语句中声明的变量同样可以在对应的任何 `else` 块中使用。
```go
func pow(x, n, lim float64) float64 {
	if v := math.Pow(x, n); v < lim {
		return v
	} else {
		fmt.Printf("%g >= %g\n", v, lim)
	}
	// can't use v here, though
	return lim
}
func main() {
	fmt.Println(
		pow(3, 2, 10),
		pow(3, 3, 20),
	)
}
>>> 27 >= 20
>>> 9 20
```

Go 的 `switch` 语句类似于 `C、Java `中的，不过 Go 只会运行选定的 `case`，而非之后所有的 `case`。
- 在效果上，Go 的做法相当于这些语言中为每个 `case` 后面自动添加了所需的 `break` 语句。在 Go 中，除非以 `fallthrough` 语句结束，否则分支会自动终止。
- `switch` 的 `case` 无需为常量，且取值不限于整数。
```go
func main() {
	fmt.Print("Go runs on ")
	switch os := runtime.GOOS; os {
	case "darwin":
		fmt.Println("macOS.")
	case "linux":
		fmt.Println("Linux.")
	default:
		// freebsd, openbsd,
		// plan9, windows...
		fmt.Printf("%s.\n", os)
	}
}

// go的枚举
switch i {
case 0:
case f():
}
```
##  defer 推迟
defer 语句会将函数推迟到外层函数返回之后执行。
- 推迟调用的函数其参数会立即求值，但直到外层函数返回前该函数都不会被调用。

**defer 栈**
推迟调用的函数调用会被压入一个栈中。 当外层函数返回时，被推迟的调用会按照后进先出的顺序调用。
```go

fmt.Println("counting")          // 1. 立即执行，输出 counting
for i := 0; i < 10; i++ {
    defer fmt.Println(i)         // 2. 循环 10 次，每次把“fmt.Println(i)”连带着当时 i 的值
}                                //    压进 defer 栈；先压 0，再压 1 … 最后压 9
fmt.Println("done")              // 3. 立即执行，输出 done
// 4. main 即将返回，按“后进先出”顺序把刚才压的 10 个函数全部执行：先执行 fmt.Println(9)，再 fmt.Println(8) … 最后 fmt.Println(0)
```
# 更多类型：结构体、切片和映射
## 指针
go有指针，一个指向内存地址的值的指针。
- **`&` 是取地址运算符**：`&v` 表示“获取变量 `v` 的内存地址”，结果的类型是 **指向 `Vertex` 的指针**，即 `*Vertex`。
- **`*` 是解引用运算符**：`*p` 表示“获取指针 `p` 指向的值”。
它的零值是nil。
```go
var p *int

i := 42
p = &i
```
`&`运算符生成一个指向其操作数的指针。

Go 指针最重要的特点：**没有指针算术** ; 而C 允许指针算术
	不能通过移动指针来遍历数组、不能计算 `p+offset`。这杜绝了 **缓冲区溢出、越界访问、野指针** 等一大类内存安全漏洞。

重点解释“安全返回局部变量地址”：
```c
int* bad() {
    int x = 42;
    return &x;  //  返回栈上地址，函数返回后该地址无效
}
```

```go
func good() *int {
    x := 42
    return &x   //  编译器会判断 x 逃逸到堆上，指针仍然有效
}
```
Go 中“没有指针算术”如何遍历数组进行工作？使用 **切片（slice）** 代替原始指针算术。
**Go 的指针就是 C 指针去掉算术运算，再加上自动垃圾回收和安全返回局部变量地址的能力。**
## 结构体
结构体是一个字段的集合。
结构体域用`.`进行访问可以通过结构指针访问结构域。
当有结构指针p时，要访问结构体的字段X，可以写`（*p）.X`。然而，这种表示法很麻烦，所以语言允许我们只写p.X，而不需要显式的解引用。
```go
type Vertex struct {
	X int
	Y int
}

func main() {
	v := Vertex{1, 2}
	p := &v
	p.X = 1e9
	fmt.Println(v)
}
```
结构体常量：
```go
type Vertex struct {
	X, Y int
}

var (
	v1 = Vertex{1, 2}  // has type Vertex
	v2 = Vertex{X: 1}  // Y:0 is implicit
	v3 = Vertex{}      // X:0 and Y:0
	p  = &Vertex{1, 2} // has type *Vertex
)
```
## 数组
类型`[n]T`是一个包含n个类型为T的值的数组。
Go提供了一种方便的处理数组的方法。
```go
func main() {
	var a [2]string 
	a[0] = "Hello"
	a[1] = "World"
	fmt.Println(a[0], a[1])
	fmt.Println(a)

	primes := [6]int{2, 3, 5, 7, 11, 13}
	fmt.Println(primes)
}
```
类型`[]T`是包含类型为T的元素的切片。  
通过指定两个下界和上界来形成切片，下界和上界由冒号分隔：`a[low : high]`
包含第一个元素但不包括最后一个元素的半开放范围。

---
**切片不存储任何数据**，它只是描述底层数组的一段。
	更改切片的元素会修改其底层数组的相应元素。  共享相同底层数组的其他片将看到这些更改。
下面两个等价：
```go
[3]bool{true, true, false}
[]bool{true, true, false}
```
切片时，可以省略高或低边界，而使用其默认值。  
- 对于下界，默认值为零，对于上界，默认值为底层切片或数组的长度。

---
切片的长度和容量可以用表达式 `len(s)` 和 `cap(s)` 求得。
- **`len(s)`**：切片的 **长度**，表示**当前切片中元素的个数**
- **`cap(s)`**：切片的 **容量**，表示从切片第一个元素开始算起，**底层数组最多还能容纳多少个元素**（即在不重新分配内存的前提下，切片最多能扩展到的长度）。
您可以通过重新切片来延长切片的长度，只要它有足够的容量。

切片的零值为nil。nil切片的长度和容量为0，并且没有底层数组。

---
切片可以用内置的**make函数创建**；这就是如何**创建动态大小的数组**。
make函数分配一个归零数组，并返回一个指向该数组的切片：
```go
a := make([]int, 5)  // len(a)=5
```
要指定容量，请传递第三个参数：
```go
b := make([]int, 0, 5) // len(b)=0, cap(b)=5

b = b[:cap(b)] // len(b)=5, cap(b)=5
b = b[1:]      // len(b)=4, cap(b)=4
```
片可以包含任何类型，包括其他片:
```go
// Create a tic-tac-toe board.
board := [][]string{
	[]string{"_", "_", "_"},
	[]string{"_", "_", "_"},
	[]string{"_", "_", "_"},
}
```

如果后备数组太小，无法容纳所有给定的值，将分配一个更大的数组。返回的切片将指向新分配的数组:
```go
func append(s []T, vs ...T) []T
```
## Range
- `range` 可以遍历**切片**（slice）、**数组**、**映射**（map）、**字符串**（string）或**通道**（channel）
- - 当遍历切片 `pow` 时，`range` 会返回两个值：**索引**（`i`）和**该索引位置的元素值**（`v`）
```go
var pow = []int{1, 2, 4, 8, 16, 32, 64, 128}

func main() {
	for i, v := range pow {
		fmt.Printf("2**%d = %d\n", i, v)
	}
}

```
可以通过给_赋值来跳过索引或值。
```go
for i, _ := range pow
for _, value := range pow
// 如果只需要索引，可以省略第二个变量。
for i := range pow
```
## maps
映射的零值为nil。nil映射没有键，也不能添加键。  
map 字面量和 struct 字面量（结构体字面量）都使用花括号 `{}` 包含一系列初始化值。
- **区别**：在 struct 字面量中，**字段名可以省略**（按声明顺序赋值）；但在 map 字面量中，**每个元素必须显式指定键（key）**，不能省略。

当你写一个复合字面量（比如切片、数组、map、结构体）时，如果**最外层的类型**只是一个**类型名**（例如 `[]int`、`Point`），那么对于**内部的元素**（也是复合字面量）可以**省略掉重复的类型名**，因为外层已经指明了元素类型。
```go
var grid = [][]int{    |     var grid = [][]int{
    []int{1, 2, 3},    |         {1, 2, 3},
    []int{4, 5, 6},    |         {4, 5, 6},
}                      |     }
``` 

---
在映射 `m` 中插入或更新某个元素：
```go
m[key] = elem
```
获取某个元素：
```go
elem = m[key]
```
删除某个元素：
```go
delete(m, key)
```
通过双值赋值来检测某个键是否存在：
```go
elem, ok = m[key]
```
- 如果 `key` 存在于 `m` 中，那么 `ok` 就等于 `true` 。否则， `ok` 就等于 `false` 。
- 如果 `key` 不在该映射中，那么 `elem` 就相当于该映射中该元素类型的默认值/零值。
注意：如果 `elem` 或 `ok` 尚未被声明，可以使用简短的声明方式来代替：
```go
elem, ok := m[key]
```
## Function values
函数也是一种值。它们可以像其他值一样被传递给其他地方。
函数值可以用作函数的参数和返回值。
```go
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}

func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))

	fmt.Println(compute(hypot))
	fmt.Println(compute(math.Pow))
}
```
Go 语言中的函数可以是**闭包**。
所谓闭包，指的是那些能够引用其定义范围之外变量的函数。该函数可以访问这些变量，并对它们进行赋值操作。从这个意义上说，该函数与这些变量是“绑定”在一起的。
- **一个函数“记住”了它外面的一些变量，即使外面函数已经执行完了，这个函数还能用、还能改那些变量。**
# Methods and interfaces
## Methods
Go 语言没有类这一概念。不过，可以在各种类型上定义方法。
方法是一种带有特殊接收者参数的函数。该接收者出现在其自己的参数列表中，位于 `func` 关键字与方法名之间。
```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(v.Abs())
}
```
在这个例子中， `Abs` 方法的作用对象是类型为 `Vertex` 的实体，该实体的名称为 `v` 。

请记住：方法只不过是一个带有接收参数的函数而已。
以下是 `Abs` 以**普通函数**的形式表示出来的版本，其功能完全不变。
```go
type Vertex struct {
	X, Y float64
}

func Abs(v Vertex) float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func main() {
	v := Vertex{3, 4}
	fmt.Println(Abs(v))
}
```

函数和方法的核心区别在于：
- **函数**：独立存在，不依附于任何类型。
- **方法**：绑定到某个**自定义类型**（通过**接收者**参数），可以看作是“属于”该类型的行为。
使用指针作为接收参数的方法可以修改该指针所指向的值的值。由于方法通常需要修改其接收到的参数，因此指针作为接收参数的情况比使用值作为接收参数的情况更为常见。

---
**方法的接收者类型限制**:
1. 可以在非结构体类型上定义方法
2. 接收者的类型必须和该方法定义在同一个包内
**只能为自己所在包（package）中的类型添加方法，不能为其他包（包括标准库中的内置类型，如 `int`、`string`）添加方法。**
```go
type MyFloat float64   // 在当前包中定义新类型
func (f MyFloat) Abs() float64 {   // 合法：接收者类型 MyFloat 与 main 包在同一包内
    if f < 0 {
        return float64(-f)
    }
    return float64(f)
}
func main() {
    f := MyFloat(-3.14)
    fmt.Println(f.Abs())
}
```

---
指针接收器：
尝试去掉`Scale` 函数声明中的 `*` ，然后观察程序的行为有何变化。
```go
type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

// 去掉该函数中的 * 会拿到，原来结构体的副本
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

// 去掉该函数中的 * 会报错
func Scale(v *Vertex, f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}
```
Go 语言中**方法接收者**的类型决定了方法是否能修改原始数据。去掉 `*` 后，接收者从**指针类型** `*Vertex` 变成了**值类型** `Vertex`，导致 `Scale` 内部修改的是 `v` 的一个**副本**，而不是 `main` 函数中的原始变量。

**带有指针参数的函数，必须接受指针作为参数**
```go
var v Vertex
ScaleFunc(v, 5)  // Compile error!
ScaleFunc(&v, 5) // OK
```
以指针作为接收参数的方法，在被调用时，可以接收数值或指针作为参数
```go
var v Vertex
v.Scale(5)  // OK
p := &v
p.Scale(10) // OK
```
为了方便使用，Go 语言会将语句 `v.Scale(5)` 解释为 `(&v).Scale(5)` ，因为 `Scale` 方法要求以指针作为参数。

---
**函数（参数）对类型要求严格，而方法（接收者）则更灵活**。
- 如果一个函数定义时参数是 **值类型**（比如 `func AbsFunc(v Vertex)`），那么调用时**必须传入一个相同类型的值**（即 `Vertex` 类型）。
- 如果你试图传入一个**指针** `&v`（类型是 `*Vertex`），编译器会报错，因为类型不匹配。
```go
var v Vertex
fmt.Println(AbsFunc(v))   //  正确，参数类型是 Vertex
fmt.Println(AbsFunc(&v))  //  编译错误：cannot use &v (type *Vertex) as type Vertex
```
- 如果一个方法的接收者是**值类型**（`func (v Vertex) Abs() float64`），那么调用该方法时，**既可以用值调用，也可以用指针调用**。
- 当你用指针 `p` 调用 `p.Abs()` 时，Go 会自动将其解引用，相当于 `(*p).Abs()`。这得益于 Go 提供的语法糖。
```go
var v Vertex
fmt.Println(v.Abs())   //  值调用
p := &v
fmt.Println(p.Abs())   //  指针调用，等价于 (*p).Abs()
```

使用指针接收器有两个原因：
1. 该方法能够修改其接收者所指向的值的数值。
2. 避免在每次方法调用时都复制该值。如果接收方是一个较大的结构体，这样做会更高效。
## Interfaces
接口类型被定义为一组方法签名。
接口类型的变量可以包含任何实现了该接口中所有方法的值。
一个类通过实现接口中的各个方法来满足该接口的要求。
隐式接口将接口的定义与其实现分离，这样一来，实现部分可以出现在任何包中，而无需事先进行安排。
go与java等语言的接口功能几乎一样，只是形式有所不同。
```go
package main

import "fmt"

// 1. 定义接口：Speaker 包含一个 Speak 方法
type Speaker interface {
    Speak() string
}

// 2. 定义两个不同的结构体类型
type Dog struct {
    Name string
}

type Cat struct {
    Name string
}

// 3. Dog 类型实现 Speak 方法（隐式实现接口）
func (d Dog) Speak() string {
    return d.Name + " says: 汪汪！"
}

// 4. Cat 类型实现 Speak 方法
func (c Cat) Speak() string {
    return c.Name + " says: 喵喵！"
}

// 5. 一个接受 Speaker 接口的函数，可以处理任何实现了 Speaker 的类型
func MakeSound(s Speaker) {
    fmt.Println(s.Speak())
}

func main() {
    dog := Dog{Name: "旺财"}
    cat := Cat{Name: "咪咪"}

    // dog 和 cat 都是 Speaker 类型，可以传递给 MakeSound
    MakeSound(dog)
    MakeSound(cat)

    // 也可以直接调用方法
    fmt.Println(dog.Speak())
    fmt.Println(cat.Speak())

    // 接口变量可以存储任何实现了该接口的值
    var s Speaker
    s = dog
    fmt.Println(s.Speak()) // 旺财 says: 汪汪！
    s = cat
    fmt.Println(s.Speak()) // 咪咪 says: 喵喵！
}
```

从内部机制来看，接口值可以被视作一个由数值和具体类型构成的元组。
```go
(value, type)
```
在接口值上调用某个方法时，实际上是在该接口所对应的实际类型上执行同名的方法。

---
**一个接口值即使内部存储的具体值是 `nil`，接口值本身也不等于 `nil`。**
```go
func returnError() error {
    var p *MyError = nil
    return p   // 返回了一个 error 接口，但 p 是 nil 指针
}

func main() {
    err := returnError()
    if err != nil {   // 这里 err != nil 为 true！因为接口包含类型信息
        fmt.Println("error happened")  // 会执行，即使实际值 nil
    }
}
```
**接口的“空值”有两个层面**：
- 接口本身 `nil`（无类型、无值）。
- 接口包含一个类型但值为 `nil`（此时接口非 `nil`，但方法调用时接收者为 `nil`）。
Go 鼓励在方法中优雅处理 `nil` 接收者，以避免空指针异常。

- **`nil` 接口** → 无类型信息 → 无法定位方法 → 调用方法时直接运行时 panic。
- **非 `nil` 接口但内部值为 `nil`** → 有类型信息 → 可以正常调用方法（只是接收者可能为 `nil`）。
如果你试图在一个 `nil` 接口上调用方法，运行时系统没有足够信息（没有类型）去决定调用哪个实现，因此会报错。

---
不包含任何方法的接口类型被称为“空接口”。
```go
interface{}
```
空接口可以包含任何类型的值。（每种类型至少实现零个方法。）
空接口被那些需要处理未知类型值的代码所使用。例如， `fmt.Print` 可以接受任意数量的 `interface{}` 类型的参数。
### 类型断言
类型断言使得我们可以获取接口值所对应的实际具体值。
```go
t := i.(T)
```
该声明指出：接口值 `i` 具有具体的类型 `T` ，同时将其背后的 `T` 值赋给变量 `t` 。
如果 `i` 不包含 `T` ，则该语句将引发程序崩溃。

```go
t, ok := i.(T)
```
为了判断某个接口值是否属于某种特定类型，类型断言可以返回两个值：该值的实际数值，以及一个布尔值，用来指示断言是否成功。

---
类型转换器
类型转换是一种允许将多个类型断言依次进行的结构。
```go
switch v := i.(type) {
case T:
    // here v has type T
case S:
    // here v has type S
default:
    // no match; here v has the same type as i
}
```
类型switch与普通的switch语句类似，不过在类型开关中，各个“case”选项对应的是具体的类型（而非具体的值）。系统会将该值与某个接口所对应的值的类型进行比较。

### Stringers
最普遍使用的接口之一，就是由 `fmt` 包所定义的 `Stringer` 接口。
```go
type Stringer interface {
    String() string
}
```
`Stringer` 是一种能够将自己描述为字符串的数据类型。 `fmt` 包（以及许多其他包）都利用这一接口来输出数值。
## Error
Go 语言中，错误状态通过 `error` 值来表示，`error` 类型是一种内置接口。
```go
type error interface {
    Error() string
}
```
函数通常会返回一个 `error` 值，调用代码应通过检查该错误值是否等于 `nil` 来处理错误情况。
在 `Error()` 方法内部，**不要直接使用会间接调用 `Error()` 的函数**（如 `fmt.Sprint(e)`）来格式化接收者本身，否则会形成无限递归。
```go
type ErrNegative float64
func (e ErrNegative) Error() string {
    // 错误示范：直接使用 fmt.Sprint(e)
    return fmt.Sprint(e)   // 这里会再次调用 e.Error()！
}
```
- `fmt.Sprint(e)` 需要获取 `e` 的字符串表示
- 因为 `e` 实现了 `error` 接口（有 `Error()` 方法），`fmt` 包会优先调用 `e.Error()` 来生成字符串
- 于是 `Error()` → `fmt.Sprint(e)` → 又调用 `e.Error()` → `fmt.Sprint(e)` → ... 形成无限循环，最终导致程序崩溃（栈溢出）
 在 `Error()` 方法中，永远不要对接收者本身使用 `fmt.Sprint`（或任何会尝试获取字符串表示的常用函数），除非你先将其转换为非 `error` 的底层类型。

## Reader
Go 标准库中包含了许多实现该接口的组件，包括用于处理文件、网络连接、压缩数据、加密数据等功能的相关模块。
`io.Reader` 接口拥有一个 `Read` 方法：
`Read` 用数据填充给定的字节切片，同时返回已填充的字节数以及错误信息。当数据流结束时，它会返回 `io.EOF` 错误。
使用：
```go
func main() {
	r := strings.NewReader("Hello, Reader!")

	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```
## Images
包图像定义了 `Image` 接口：
```go
package image

type Image interface {
    ColorModel() color.Model
    Bounds() Rectangle
    At(x, y int) color.Color
}
```
注意： `Bounds` 方法的 `Rectangle` 返回值实际上是一个 `image.Rectangle` ，因为该值的声明位于 `image` 包内部。
# Generics-泛型
Go 语言支持使用类型参数来进行泛型编程。
```go
func Index[T comparable](s []T, x T) int
```
- `[T comparable]` 表示声明一个类型参数 `T`，并且 `T` 必须满足内置的 `comparable` 约束（即可以用 `==` 和 `!=` 比较）。
- `s []T` 表示切片元素类型就是 `T`，`x T` 表示要查找的值也是类型 `T`。
- 函数内部可以用 `==` 来比较 `x` 和切片中的每个元素。
使用：
```go
// Index returns the index of x in s, or -1 if not found.
func Index[T comparable](s []T, x T) int {
	for i, v := range s {
		// v and x are type T, which has the comparable
		// constraint, so we can use == here.
		if v == x {
			return i
		}
	}
	return -1
}

func main() {
	// Index works on a slice of ints
	si := []int{10, 20, 15, -10}
	fmt.Println(Index(si, 15))

	// Index also works on a slice of strings
	ss := []string{"foo", "bar", "baz"}
	fmt.Println(Index(ss, "hello"))
}
```

---
Go 语言中的**泛型类型（Generic Types）**，即**带有类型参数的自定义类型**。
```go
type List[T any] struct {
    head *Node[T]
    // ... other attributes
}

type Node[T any] struct {
    val  T
    next *Node[T]
}
```
这里的 `[T any]` 表示类型参数 `T`，`any` 是约束（允许任何类型）。然后你可以创建 `List[int]`、`List[string]` 等不同类型的链表。
# Concurrency-并发
Goroutine 是由 Go 运行时所管理的一种轻量级线程。
```go
go f(x, y, z)
```
`f` 、 `x` 、 `y` 和 `z` 的演化在当前的 goroutine 中完成，而 `f` 的执行则在新创建的 goroutine 中完成。
evaluation（评估/演化）指：**对函数名和参数表达式的求值过程**。
## Channels
Channels（通道）是一种有序的传输路径，通过通道操作符 `<-` ，你可以在这条路径上发送和接收各种数值。
```go
ch <- v    // Send v to channel ch.
v := <-ch  // Receive from ch, and
           // assign value to v.
```
与 maps 和 slices 类似，通道也必须在使用之前先创建出来：
```go
ch := make(chan int)
```
默认情况下，数据的发送和接收会一直被阻塞，直到对方准备好。这样一来，goroutines 就能在无需使用显式的锁或条件变量的情况下实现同步。

使用：
```go
func sum(s []int, c chan int) {
	sum := 0
	for _, v := range s {
		sum += v
	}
	c <- sum // send sum to c
}

func main() {
	s := []int{7, 2, 8, -9, 4, 0}

	c := make(chan int)
	go sum(s[:len(s)/2], c)
	go sum(s[len(s)/2:], c)
	x, y := <-c, <-c // receive from c

	fmt.Println(x, y, x+y)
}
```

---
Channels可以被缓冲处理。要将Channels设置为缓冲模式，请将缓冲长度作为第二个参数传递给 `make` ：
```go
ch := make(chan int, 100)
```
只有当缓冲区已满时，才会将数据发送到缓冲通道块中。当缓冲区为空时，才会接收数据块。
当带缓冲的 channel **已满**时，继续向它发送数据会发生：
- **阻塞当前 goroutine**，直到有其他goroutine协程接收者来接收数据
- 如果永远没有接收者，并且主 goroutine 被阻塞，就会引发 **死锁**（`fatal error: all goroutines are asleep - deadlock!`）。
## Range and Close
发送方可以通过 `close` 某个通道来表示不会再向该通道发送任何数据了。接收方则可以通过在接收操作中添加第二个参数来检测该通道是否已被关闭：在……之后。
```go
v, ok := <-ch
```
当没有更多的数据需要接收且通道已关闭时， `ok` 就等于 `false` 。
```go
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}

func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
```
**注意**：只有发送方才能关闭通道，接收方绝不能关闭。在已关闭的通道上继续发送数据会导致系统崩溃。
**另外需要注意的是**：通道与文件不同，通常无需关闭它们。只有当需要让接收方知道不再有数据传输时，才需要关闭通道。
## Select
`select` 语句允许一个 goroutine **同时等待多个通信操作** 的结果。
`select` 会一直处于阻塞状态，直到其中的某个条件满足后才会开始执行该条件对应的操作。如果有多个条件同时满足，它会随机选择其中一个来执行。

如果没有任何其他条件满足，那么就会执行 `select` 中的 `default` 情况。
使用 `default` 模式来尝试发送或接收数据，而不会造成阻塞：
```go
select {
case i := <-c:
    // use i
default:
    // receiving from c would block
}
```

---
为什么要有channel和select？
- **如果没有 channel**，我们只能使用锁 + 条件变量 + 共享变量来实现 goroutine 间的交互，代码复杂且容易出错。
- **没有 select**，我们可能需要为每个 channel 单独启动一个 goroutine，然后通过复杂的同步机制汇集结果，这会浪费资源且难以管理。
## `sync.Mutex`-同步.互斥
Go 语言的标准库通过 `sync.Mutex` 及其两个相关方法来实现互斥机制。
- 可以用 `Lock` 和 `Unlock` 来包围某段代码，从而实现该代码的互斥执行。
- 也可以使用 `defer`(推迟unlock到函数之外再进行解锁) 来确保互斥锁能够被解锁

# 小结
- Go 没有对象、没有类、没有继承
- Go 通过组合匿名字段来达到类似继承的效果
- 通过以上手段去掉了面向对象中复杂而冗余的部分
