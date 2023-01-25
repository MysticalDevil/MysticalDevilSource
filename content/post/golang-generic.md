---
title: "Go 新特性 -- 泛型"
date: 2023-01-23T19:50:59+08:00
categories : [
"Go",
"Programming",
“Notes"
]
tags : [
"Go",
"Generic"
]
---

## 序言

2022年3月15日，争议非常大但同时也备受期待的泛型终于伴随着 Go 1.18 发布了。~~设计者之前还说为了 Go 语言的简洁，不会加入泛型🙄~~

## 为什么要引入泛型

在 1.18 版本之前，如果想在 Go 语言中让一个函数/方法支持多种类型，那么开发者只能为每个类型挨个实现相关的函数/方法。比如我有一个 Add 函数，定义如下

```go
func Add(a, b int) int {
    return a + b
}
```

但是上面的函数只支持 int 类型，如果我想支持其他类型那么需要为每个类型都写一个对应函数（比如著名的cast包，用于类型转换），如下

```go
func AddFloat32(a, b float32) float32 {
    return a + b
}

func AddFloat64(a, b float64) float64 {
    return a + b
}
```

这也太麻烦了吧，:herb:

不过我们也可以使用 空接口 `interface{}` 加反射的方式实现（现在空接口被包装为 `any` 关键字），使用如下方式来实现。~~这不是更麻烦吗~~

```go
func Process(x any) {
    t := reflect.TypeOf(x)
    v := reflect.ValueOf(x)
    switch t.Kind() {
    case reflect.Int:
        fmt.Println("x is an int with value", v.Int())
    case reflect.String:
        fmt.Println("x is a string with value", v.String())
    case reflect.Float64:
        fmt.Println("x is a float64 with value", v.Float())
    default:
        fmt.Println("x is of unknown type")
    }
}
```

但是这种实现方法有很多问题：
    1. 用起来很麻烦，反正 go 语言的反射我是没怎么写明白
    2. 缺少了编译时的类型检查，很容易出错，这对于我们实现的应用来说是很致命的
    3. 性能不理想，因为反射是运行时的东西

而在泛型适用的时候，它能解决上面这些问题。但这也不意味着泛型是万金油，泛型有着自己的适用场景，当你疑惑是不是该用泛型的话，请记住下面这条经验：

> 如果你经常要分别为不同的类型写完全相同逻辑的代码，那么使用泛型将是最合适的选择

## Go 的泛型

Go 语言引入了如下的新概念来实现泛型

- 类型形参（Type parameter）
- 类型实参（Type argument）
- 类型形参列表（Type parameter list）
- 类型约束（Type constraint）
- 实例化（Instantiations）
- 泛型类型（Generic type）
- 泛型接收器（Generic receiver）
- 泛型函数（Generic function）

等等等等. . .这 TM 也太多了吧，下面来一个个介绍一下

### 类型形参、类型实参、类型约束和泛型类型

在 1.18 之前，NewType 一般只允许有一个类型，例如

```go
type IntSlice []int

var a IntSlice = []int{1, 2, 3}
var a IntSlice = []float64(1.o, 2.0, 3.0) // error 因为 IntSlice 底层为 []int
```

那我想要一个可以容纳其他类型的切片怎么办？简单，对应的每个类型写一个就行~~🌿~~

```go
type Float32Slice []float32
type Float64Slice []float64
type StringSlice []string
```

泛型的引入就很好的解决了这一问题，即

```go
type Slice[T int | float32 | float64 | string] []T
```

不同于其他语言的泛型使用尖括号`<>`，Go语言为了编译器的简单（不是我说的）使用了方括号`[]` ~~也挺反人类的~~

对于以上个各部分对应的概念如下

- `T` 为**类型形参**，在定义 Slice 的类型时不代表具体的类型，只作为一个占位符
- `int | float32 | float64 | string` 为**类型约束**，中间的`|`就是告诉编译器我只接受这四种类型的实参
- 中括号里的`[T int | float32 | float64 | string]`一整串因为定义了所有的类型形参，所以称其为**类型形参列表**
- 这里定义了一个 NewType 为 Slice[T]

这种类型定义中带类型形参的类型，称之为**泛型类型**

泛型类型不能直接使用，需要传入**类型实参**，即类型形参列表中的类型，而传入类型的操作被称为**实例化**，例如

```go
// 这里传入了类型实参 int，泛型类型 Slice[T] 被实例化了具体的类型为 Slice[int]
var a Slice[int] = []int{1, 2, 3}
fmt.Printf("Type Name: %T\n", a)

// 这里被实例化为了 Slice[string]
var s Slice[string] = []int{"a", "b", "c"}
fmt.Printf("Type Name: %t\n", s)

// error，byte 不在类型约束中
var b Slice[byte] = []byte{'a', 'b', 'c'}
```

类型形参的数量不止有一个  ~~这个泛型map就开始反人类了，这么多`[]`，有点眼花~~

```go
// NewMap 定义两个类型形参 K， V
// 这个类型的名字就叫做: NewMap[K, V]
type NewMap [K int | string, V float32 | float64] map[KEY]VALUE

var m NewMap[int, float64] = map[int]float64 {
    1: 5.0,
    2: 4.0,
}
```

### 其他的泛型类型

```go
// 泛型结构体
type NewStruct[T int | string] struct {
    Name string
    Data T
}

// 泛型接口
type IPrintData[T int | float64 | string] {
    Print(data T)
}

// 泛型Channel
type NewChan[T int | string] chan T
```

### 类型形参的嵌套

```go
type WowStruct[T int | float32, S []T] struct {
    Data S
    MinValue T
    MaxValue T
}

var s WowStruct[int, []int] = {
    Data: []int{1, 2, 3},
    MinValue: 1,
    MaxValue: 100,
}
```

### 常见语法错误

1. 定义泛型时，**基础类型不能只有类型形参，如下：

   ```go
   // error, 类型形参不能单独使用
   type CommonType[T int | string | float64] T
   ```

2. 当类型约束的一些写法会被编译器误认为时表达式时会报错，如下：

   ```go
   // error, T *int 会被编译器误认为是表达式 T 乘以 int，而不是 int 指针
   type NewType[T *int] []T
   
   // error, 这里的 * 会被认为是乘法符号，| 会被认为时按位或
   type NewType[T *int | float64] []T
   
   // error
   type NewType[T (int)] []T
   ```

   如何避免这种错误呢？可以给类型约束用 `interface{}` 包上 或者加上逗号消除歧义，如下：

   ```go
   type NewType[T interface{*int}] []T
   type NewType[T interface{*int|*float64}] []T
   
   // 类型约束只有一个类型，可以添加一个逗号消除歧义
   type NewType[T *int,] []T
   // error，类型约束有多个时加逗号无法消除歧义
   type NewType[T *int | *float64,] []T
   ```

### 特殊的泛型类型

这里讨论一下比较特殊的泛型类型，如下

```go
type Foo[T int | string] int

var a Foo[int] = 123 // right
var a Foo[string] = 123 // right
var a Foo[int] = "bar" // error，因为"hello"不能赋值给底层类型int
```

这里虽然使用了类型形参，但是它的底层类型还是 int 类型，无论传入什么类型实参，实例化之后都是 int 类型

### 泛型类型的套娃

泛型类型可以进行套娃，如下：

```go
// 先定义一个 Slice[T]
type Slice[T int | string | float32 | float64] []T

// 再定义一个基于 Slice[T] 的泛型类型 FloatSlice[T]
type FloatSlice[T float32 | float64] Slice[T]

// error, 泛型类型 Slice[T] 的类型约束不包含 uint, uint8
type UintSlice[T uint | uint8] Slice[T]

// 定义一个基于 Slice[T] 的 IntAndStringSlice[T]
type IntAndStringSlice[T int | string] Slice[T]

// 在 map 中嵌套泛型类型 Slice[T]
type FooMap[T int | string] map[string]Slice[T]
type FooMap[T Slice[int] | Slice[string]] map[string]T
```

### 类型约束的两种写法

```go
type FooStruct[T int | string] struct {
    Name string
    Data []T
}

type BarStruct[T []int | []string] struct {
    Name string
    Data T
}
```

> 匿名结构体不支持泛型
>
> 这使得在进行单元测试时，为泛型做单元测试变得很麻烦

## 泛型 Receiver

Go 泛型支持给泛型类型添加方法，如下：

```go
type Slice[T int | string] []T

func (s Slice[T]) Sum() T {
    var sum T
    for _, value := range s {
        sum += value
    }
    return sum
}
```

实例化

```go
var s Slice[int] = []int{1, 2, 3}
fmt.Printf("%v\n", s.Sum())
```

通过泛型 Receiver，泛型的实用性得到了很大的扩展，我们可以很简单的创建通用的数据结构，在这之前我们如果想要实现通用的数据结构（堆、栈、链表、队列）只能使用两种方法

- 每种类型挨个实现（啰嗦、麻烦）
- 接口 + 反射（麻烦、难理解）

### 基于泛型的队列

我们不必像之前那样为每个类型都实现一个队列，直接使用泛型 Receiver 便可简单的实现，如下：

```go
type Queue[T any] struct {
	elements []T
}

// 将数据加入队列
func (q *Queue[T]) Put(value T) {
	q.elements = append(q.elements, value)
}

// 将数据从队列取出
func (q *Queue[T]) Pop() (T, bool) {
	var value T
	if len(q.elements) == 0 {
		return value, true
	}

	value = q.elements[0]
	q.elements = q.elements[1:]
	return value, len(q.elements) == 0
}

// 队列大小
func (q *Queue[T]) Size() int {
	return len(q.elements)
}

var q1 Queue[int]
q1.Put(1)
q1.Put(2)
q1.Pop()

var q1 Queue[string]
q2.Put("A")
q2.Pur("B")
q2.Pop()
q2.Pop()
```

> 泛型不能像空接口一样进行类型的动态判断，但是我们可以使用反射进行类型的动态判断~~这很恶心🤢~~

## 泛型函数

泛型函数的写法如下：

```go
func Add[T int | float64](a, b T) T {
    return a + b
}
```

它和普通函数的区别就是在函数名后带了类型形参，泛型函数可以显式的传入类型实参也可以不传入类型实参，编译器可以自动推断；一般情况下编译器会告诉你传入类型实参是没必要的

```go
var value = Add(1, 2) // 编译器推断出类型实参为 int
var value = Add[int](1, 2) // 显式的传入类型实参 int
var value = Add[float32](1.0, 2.0) // 显式的传入类型实参 float32
```

**匿名函数**不支持泛型，但是匿名函数可以使用别处定义好的类型实参，如下：

```go
func FooFunc[T int | float32 | float64](a, b T) {
    fn2 := func(i, j T) T {
        return i*2 + j *2
    }
    
    fn2(a, b)
}
```

### 泛型方法？

函数都支持泛型了，而且含有泛型 Receiver，那方法就一定支持泛型吧？很遗憾，目前Go官方并不支持泛型方法，如下

```go
type A struct {}

// error, 不支持泛型方法
func(receiver A) Add[T int | float32 | float64](a, b T) T {
    return a + b
}
```

但是因为 receiver 支持泛型，所以可以曲线救国一下，通过 receiver 使用类型形参

```go
type A [T int | float32 | float64] struct {}

func (receiver A[T]) Add(a, b T) T {
    return a + b
}

var a A[int]
a.Add(1, 2)

var b A[float64]
b.Add(1.0, 2.0)
```

## 小结

Go 的泛型目前可以使用在如下三个地方：

- 泛型类型 -- 类型定义中带类型形参的类型
- 泛型 Receiver -- 泛型类型的 Receiver
- 泛型函数 -- 带类型形参的函数

## 变复杂的接口

有时候使用泛型进行编程的时候，可能会书写一个很长长长的类型约束

```go
type Slice[T int | int8| int16 | int32 | int64 | uint | uint8 | uint16| uint32 | uint64]
```

使得可读性大大的下降，这可是无法忍受的。而 Go 支持将这些类型进行抽象，将其抽象为一个接口

```go
type IntUint interface {
	int | int8 | int16 | int32 | int64 | uint | uint8 | uint16 | uint32 | uint64
}

type Slice[T IntUint] []T
```

这段代码吧类型约束单独拿出来，写入了接口类型 IntUint中，但是可维护性还是有点低，这是我们可以将接口和接口、接口和普同类型之间通过 `|` 进行组合，如下

```go
type Int interface {
    int | int8 | int16 | int32 | int64
}

type Uint interface {
    uint | uint8 | uint16 | uint32 | uint64
}

type Slice[T Int | Uint] []T	
```

在接口中也能直接组合其他接口，如下

```go
type SliceElement interface {
    Int | Uint | string
}

type Slice[T SliceElement] []T
```

### 指定底层类型： ~

上面定义的 `Slice[T]` 虽然可以增加可维护性，但是有一个问题，就是它不能使用 NewType 来作为类型实参

```go
var s1 Slice[int] // right

type NewInt int
var s2 Slice[NewInt] // error, NewInt 的底层类型是 int 但是他不是 int 类型，不符合 Slice[T] 的类型约束
```

这种问题该如何解决呢，贴心的官方给了我们一个新的符号 `~`，在类型约束中使用类型 `~int`这种写法，就代表任何底层类型为 int 类型的 NewType 都可以作为 Slice[T] 的类型实参，如下

```go
type Int interface {
    ～int | ～int8 | ～int16 | ～int32 | ～int64
}

type Uint interface {
    ～uint | ～uint8 | ～uint16 | ～uint32 | ～uint64
}

type Slice[T Int | Uint]

type FooInt int
var s1 Slice[FooInt] // right

type BarInt FooInt
var s2 Slice[BarInt] // right
```

**限制**：使用 `~` 有一定的限制

- `~` 后面不能为接口
- `~` 后面的类型必须为基本类型，不能为 NewType

```go
type NewInt int

type Int interface {
    ~[]byte // right
    ~NewInt // error，~ 后类型必须为接口
    ~error // error，~ 后的类型不能为接口
}
```

### 从方法集（Method Type）到类型集（Type Set）

上面的写法在1.18版本之前是没有的，因为在1.18之前官方对`接口（interface）`的定义为：接口是一个方法集（method set）

> An interface type specifies a **method set** called it interface

就如同下面这段代码一样，`ReadWrite`接口定义了一个接口（方法集），这个集合包含了`Read()`和`Write()`两个方法。所有同时实现了这两种方法的类型就会被视为实现了这一接口（鸭子型别）

```go
type ReadWrite interface {
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```

但是如果换个角度来重新思考这个接口的话，会发现接口的定义实际上还能理解为：

> 可以把`ReadWrite` 接口看成代表了一个类型的集合，所有实现了`Read()` 和 `Write()` 这两个方法的类型都在接口代表的类型集合中

换个角度后，接口的定义就从**方法集**变为了**类型集**。而 Go 1.18 也将接口的定义更改为了：接口是一个类型集合（type set）

> An interface type defines a **type set**

### 接口定义的变化

从 Go 1.18 开始，当满足如下条件时，我们便可以说**类型 T 实现了接口 I（type T implements interface I）**

- T 不是接口时：类型 T 是接口 I 代表的类型集中的一个成员（T is an element of the type set of I）
- T 是接口时：T 接口代表的类型集是 I 代表的类型集的子集（Type set of T is subset of the type of I）

### 类型的并集

之前一直是用的 `|` 符号就是求类型的并集（union）

```go
type IntUint interface {
    int | int8| int16 | int32 | int64 | uint | uint8 | uint16| uint32 | uint64 
}
```

### 类型的交集

接口（类型集）可以写多行，当为多行时，那么会取他们的交集

```go
type AllInt interface {
     int | int8| int16 | int32 | int64 | uint | uint8 | uint16| uint32 | uint64 
}

type Uint interface {
    ~uint | ~uint8 | ~uint16| ~uint32 | ~uint64
}

type A interface {
    AllInt
    Uint
}

type B interface { // 接口 B 即为 int
    AllInt
    ~int
}
```

- 接口 A 代表的就是 `AllInt` 和 `Uint` 的交集，即 `uint | uint8 | uint16 | uint32 | uint64`

- 接口 B 代表的就是 `AllInt` 和 `~int` 的交集，即 `int`

也可以进行一些放屁的写法，如下

```go
type C interface {
    ~int
    int
}
```

接口`C` 代表的就是 `int`

```go
type D interface {
    int
    float32
}
```

接口 `D` 代表的就是一个空集，不同于空接口 `any`，它是无法传递类型实参的

### 空接口和 `any`

在 1.18 之后，空接口 `interface{}` 的定义为：

> 空接口代表了所有类型的集合

所以对于 1.18 后的空接口应理解为：

- 虽然空接口没有写入任何类型，即没有类型约束，但是它代表了所有类型的集合，而非空集
- 类型约束中指定了空接口的意思是指定了一个包含所有类型的类型集，并不是类型约束限定了只能使用**空接口**来做类型

因为空接口包含了所有的类型，所有 1.18 提供了一个新关键字 `any` ，它是 `interface{}` 的封装，能使代码更简单易懂

```go
type Slice[T any] []T // 等价于 type Slice[T interface{}] []T
```

如果想把  1.18 版本之前的项目迁移到 1.18 中，可以使用如下命令（这是可选的）

```shell
gofmt -w -r 'interface{} -> any' ./...
```

> Go 语言项目中就有人曾提出过将所有的 interface{} 替换为 any 的 issue，但是因为影响过大且无法确定会发生什么奇奇怪怪的 :bug:，理所当然的被驳回了。我觉得有一部分是理由是为了兼容性

### 可比较（Comparable）和可排序（Ordered）

在写泛型的时候我们需要在类型约束中限制只能接受能 != 和 == 对比的类型，类似于 Rust 中的 `PartialEq Eq`trait，如 map：

```go
// error，因为 map 中键的类型必须是可比较的
type NewMap[K any, V any] map[k]V
```

这时贴心的官方给你内置了一个接口，叫做 `comparable`，它代表了所有的可比较类型

```go
type NewMap[k comparable, V any] map[K]V // right
```

但是这里的可比较指的是可以执行 `!=` 和 `==` 操作的类型，并不确保这个类型可以执行 `> < >= <=`，如下

```go
type FooStruct struct {
    a int
}

var a, b FooStruct

a == b // right，结构体可以使用 == 和 != 进行比较
a != b // right

a > b // error，结构体不支持比较大小
```

如果想比较大小，实现类似于 Rust 中的 `PartialOrd Ord` trait，官方并没有提供关键字，我们只能自己来实现。

比如我们可以参考官方的`golang.org/x/exp/constraints`如何定义

```go
// Ordered 代表所有可以进行比较的类型
type Ordered interface {
    Integer | Float | ~string
}

type Integer interface {
    Signed | Unsigned
}

type Singed interface {
    ~int | ~int8 | ~int16| ~int32 | ~int64
}

type Unsigned interface {
    ~uint | ~uint8 | ~uint16| ~uint32 | ~uint64
}

type Float interface {
    float32 | float64
}
```

### 接口的两种类型

如下的例子

```go
type ReadWriter interface {
    ~string | ~[]rune
    
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```

该如何理解这个例子？用类型集的概念就能轻松的理解这个接口的意思

> 接口类型 ReadWriter 代表了一个类型集合，所有以 string 或 []rune 为底层类型，并实现了 Read() 和 Write() 这两个方法的类型都在 ReadWriter 代表的类型集中

下面的例子中，`StringReadwriter`存在于`ReadWriter`代表的类型集中，`BytesReadWriter`并不属于`ReadWriter`代表的类型集合，因为它的底层类型为`[]byte`

```go
type StringReadWriter string

func (s StringReadWriter) Write(p []byte) (n int, err error) {
	return 0, nil
}

func (s StringReadWriter) Read(p []byte) (n int, err error) {
	return 0, nil
}

type BytesReadWriter []byte

func (b BytesReadWriter) Write(p []byte) (n int, err error) {
	return 0, nil
}

func (b BytesReadWriter) Read(p []byte) (n int, err error) {
	return 0, nil
}
```

停！！！这接口变得太复杂了吧，我定义一个 ReadWriter类型的接口变量，然后接口变量赋值的时候不仅要考虑到方法的实现，还有考虑底层类型，这也太:herb:了吧。这时贴心的官方为了保持 Go 的兼容性，1.18 之后将接口分为两种类型：

- **基本接口（Basic interface）**
- **一般接口（General interface）**

#### 基本接口

接口的定义只有方法的话，那么这种接口被称为**基本接口（Basic interface）**。这种接口就是 1.18 之前的接口，用法也和之前一致。基本接口一般用于如下几个地方：

- 最常用的，，定义接口变量赋值

  ```go
  type NewError interface { // 基本接口，只有方法
      Error() string
  }
  
  var err NewError = fmt.Errorf("new error")
  ```

- 基本接口也代表了一个类型集，所有也可以用在类型约束中

  ```go
  // io.Reader 和 io.Writer 都是基本接口，也可用于类型约束中
  type NewSlice[T io.Reader | io.Writer] []Slice
  ```

#### 一般接口

如果接口中不仅只有方法，还有类型的话或者只有类型，这种接口一般被称为**一般接口（General interface）**，如下都是一般接口：

```go
type Uint interface {
    ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uint
}

type ReadWriter interface {
    ~string | ~[]rune
    
    Read(p []byte) (n int, err error)
    Write(p []byte) (n int, err error)
}
```

**一般接口类型不能用来定义变量，只能用于泛型的类型约束中**，所以如下用法是错误的：

```go
type Uint interface {
    ~uint8 | ~uint16 | ~uint32 | ~uint64 | ~uint
}

var uintInf Uint // error，Uint是接口，只能用于类型约束，不得用于变量定义
```

这一限制保证了一般接口的使用被限定在了泛型中，不会影响到 1.18 之前的代码（兼容性），同时也减少了书写时的心智负担

### 泛型接口

所有类型的定义都可以使用类型形参，所以接口定义自然也可以使用类型形参

```go
type DataProcessor[T any] interface {
    Process(oriData T) (newData T)
    Save(data T) error
}

type DataProcessor2[T any] interface {
    int | ~struct{ Data any }
    
    Process(data T) (newData T)
    Save(data T) error
}
```

因为引入了类型形参，这两个接口都是泛型接口，**而泛型类型要使用的话必须传入类型实参实例化才有意义**。

```go
DataProcessor[string]

// 实例化后的接口定义相当于
type DataProcessor[string] interface {
    Process(oriData string) (newData string)
    Save(data T) error
}
```

`DataProcessor[string]`只有方法，所以它是就是一个**基本接口**，这个接口包含了两个能处理string类型的方法。如下实现了这两个能处理string类型的方法就算实现了这个接口

```go
type CSVProcessor struct{}

func (c CSVProcessor) Process(oriData string) (newData string) {
	fmt.Printf("I get data %v", oriData)
	return "I am a CSV"
}

func (c CSVProcessor) Save(data string) error {
	return nil
}

var processor DataProcessor[string] = CSVProcessor{}
processor.Process("name,age\nbob,12\njack,30")
_ = processor.Save("name,age\nbob,12\njack,30")

var processor DataProcessor[int] = CSVProcessor{} // error，CSVProcessor没有实现接口 DataProcessor[int]
```

再用同样的方法实例化一下 DataProcessor2[T]

```go
DataProcessor2[string]

type DataProcessor2[string] interface {
    int | ~struct{ Data any }
    
    Process(data string) (newData string)
    Save(data string) error
}
```

`DataProcessor2[string]`因为带有类型并集，所以它是**一般接口**，所以实例化后代表的意思为：

- 只有实现了 `Process( data string) (newData string)` 和 `Save(data string) error` 这两个方法，并且以 `int` 或 `struct { Data any }`为底层类型的类型才算实现了这个接口
- **一般接口**不能用于定义变量只能用于类型约束，所以接口`DataProcessor2[string]`只是定义了一个用于类型约束的类型集

```go
// XMLProcessor 实现了两个方法，但是它的底层类型是 []byte，所以它并没有实现接口 DataProcessor2[string]
type XMLProcessor []byte

func (x XMLProcessor) Process(oriData string) (newData string) {
	return "I am XML"
}

func (x XMLProcessor) Save(data string) error {
	return nil
}

// JsonProcessor 实现了两个方法，底层类型也是 struct { Data any }，所以实现了 DataProcessor2[string]
type JsonProcessor struct {
	Data any
}

func (j JsonProcessor) Process(oriData string) (newData string) {
	return "I am Json"
}

func (j JsonProcessor) Save(data string) error {
	return nil
}

// error，一般接口不能用于创建变量
var processor DataProcessor2[string]

// right，实例化后的DataProcessor2可以用于泛型的类型约束
type stringProcessor interface {
    DataProcessor2[string]
    
    PrintString()
}

// error，带方法的一般接口不能作为类型并集的成员
type StringProcessor interface {
    DataProcessor2[string] | DataProcessor2[[]byte]
    
    PrintString()
}
```

### 接口定义的种种限制

在 1.18 引入泛型后，定义类型集（接口）的时候增加了非常多的十分琐碎的限制规则，有一些规则在之前已经写过了，下面还有一些规则统一的罗列一下

1. 用 `|` 连接多个类型时，类型直接不能有相交的部分（即类型直接必须为无交集）

   ```go
   type NewInt int
   
   // error，NewInt 和 ~int 有相交的部分
   type _ interface {
       ~int | NewInt
   }
   ```

   但是相交的类型是接口的话，则不受这一限制

   ```go
   type NewInt int
   
   // right
   type _ interface {
       ~int | interface { NewInt }
   }
   
   // right
   type _ interface {
       interface{ ~int } | NewInt
   }
   
   // right
   type _ interface {
       interface{ ~int } | interface { Newint }
   }
   ```

2. 类型并集中不能有类型形参

   ```go
   type NewInf[T ~int | ~string] interface {
       ~float32 | T // error，T 是类型形参
   }
   
   type NewInf[T ~int | ~string] interface {
       T // error
   }
   ```

3. 接口不能直接或间接的并入自身

   ```go
   type Bad interface {
       Bad // error，接口不能并入自身
   }
   
   type Bad2 interface {
       Bad1
   }
   
   type Bad1 interface {
       Bad2 // error，接口 Bad1 通过 Bad2 间接并入了自身
   }
   
   type Bad3 interface {
       ~int | ~string | Bad3 // error
   }
   ```

4. 接口成员个数大于一的时候不能直接或间接并入 `comparable` 接口

   ```go
   type Ok interface {
       comparable // right，只有一个类型约束时可以使用 comparable
   }
   
   type Bad1 interface {
       []int | comparabel //error，直接并入 comparabel
   }
   
   type CmpInf interface {
       comparable
   }
   
   type Bad2 interface {
       chan int | CmpInf // error，间接并入 comparable
   }
   type Bad3 interface {
       chan  int | interface{CmpInt} // error
   }
   ```

5. 带方法的接口（无论是基本接口还是一般接口），都不能写入接口的并集中

   ```go
   type _ interface {
       ~int | ~string | error // error，error 是带方法的接口(基本接口)
   }
   
   type DataProcessor[T any] interface {
       ~string | ~[]byte
       
       Process(data T) (newData T)
       Save(data T) error
   }
   
   // error，实例化后的 DataProcessor[string] 是一般接口，不能写入类型并集
   type _ interface {
       ~int | ~string | Dataprocessor[string]
   }
   ```

## 总结

以上就是 Go 语言官方在 1.18 引入的新特性泛型。

> 泛型并不取代1.18之前利用接口+反射的方式实现的动态类型，泛型的适用场景是：当你需要对不同类型书写相同逻辑时，适用泛型是简化代码的最好办法（比如写一些数据结构，堆、栈、链表等）
