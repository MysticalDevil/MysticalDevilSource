---
title: "Go Generic"
date: 2023-01-23T19:50:59+08:00
categories : [
"Go",
"Programming",
]
tags : [
"Go",
"Generic"
]
---

## 关于 Go 语言泛型的一些学习和理解

### 序言

2022年3月15日，争议非常大但同时也备受期待的泛型终于伴随着 Go 1.18 发布了。~~设置者之前还说为了 Go 语言的简洁，不会加入泛型~~

### 为什么要引入泛型

在 1.18 版本之前，如果想在 Go 语言中让一个函数/方法支持多种类型，那么开发者只能为每个类型挨个实现相关的函数/方法。比如我有一个 Add 函数，定义如下

```go
func Add(a, b int) int {
    return a + b
}
```

但是上面的函数只支持 int 类型，如果我想支持其他类型那么需要在定义其他类型的函数，如下

```go
func AddFloat32(a, b float32) float32 {
    return a + b
}

func AddFloat64(a, b float64) float64 {
    return a + b
}
```

这也太麻烦了吧，🌿

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

### Go 的泛型

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

#### 类型形参、类型实参、类型约束和泛型类型

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
type NewMap [K int | string, V float32 | falot64] map[KEY]VALUE

var m NewMap[int, float64] = map[int]float64 {
    1: 5.0,
    2: 4.0,
}
```

其他的泛型类型

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

类型形参也可以进行嵌套，例如

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

