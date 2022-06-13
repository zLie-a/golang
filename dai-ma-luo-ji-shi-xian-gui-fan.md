# 代码逻辑实现规范

## 变量、常量定义规范

* 函数内使用短变量声明(:=)
* 函数外使用长变量声明(var)，var关键字一般用于包级别变量什么，或者函数内的零值情况
* 变量、常量的分组声明一般需要按照功能来区分，而不是将所有的类型都分在一组
* 如有有可能，尽量缩小变量的作用范围；如果需要在if之外使用函数调用的结果，则不应尝试缩小变量的作用范围
* 如果是枚举变量，需要先创建相应类型

```go
type StatusCode int
const (
    StatusOK StatusCode  = 201
    StatusContentNotFound = 404
)
```

* 自定义的枚举类型应该从1开始，除非从0开始是有意义的

```go
type Operation int
const (
    Add Operation = iota + 1
    Subtract
    Multipy
)
```

## string 类型定义规范

* 声明 Printf-style String 时，将其设置为 const 常量，这有助于 go vet 对 String 类型实例执行静态分析
* 优先使用 strconv 而不是 fmt，将原语转换为字符串或从字符串转换时，strconv 速度比 fmt 快
* 避免字符串到字节的转换，不要反复从固定字符串创建字节 Slice，执行一次性完成转换

## slice、map 类型定义规范

* 尽可能指定容器的容量，以便为容器预先分配内存，向 make() 传入容量参数会在初始化时尝试调整 Slice、Map 类型实例的大小，这将减少在将元素添加到 Slice、Map 类型实例时的重新分配内存造成的损耗
* 如果 Map 类型实例包含固定的元素列表，则使用 map literals（map 初始化列表）的方式进行初始化
* 在追加 Slice 类型变量时优先指定切片容量，在初始化要追加的切片时为 make() 提供一个容量值
* Map 或 Slice 类型实例是引用类型，所以在函数调用传递时，要注意在函数内外保证实例数据的安全性，除非你知道自己在做什么。这是一个深拷贝和浅拷贝的问题
* 返回 Map 或 Slice 类型实例时，同样要注意用户对暴露了内部状态的实例的数值进行修改

## 结构体定义规范

* 嵌入结构体中作为成员的结构体，应位于结构体内的成员列表的顶部，并且必须有一个空行将嵌入式成员与常规成员分隔开
* 在初始化 Struct 类型的指针实例时，使用 `&T{}` 代替 `new(T)`，使其与初始化 Struct 类型实例一致

## 接口定义规范

* 特别的，如果希望通过接口的方法修改接口实例的实际数据，则必须传递接口实例的指针（将实例指针赋值给接口变量），因为指针指向真正的内存数据

## 函数、方法定义规范

* 函数、方法的参数排列顺序遵循以下几点原则（从左到右）:
  1. 参数的重要程度与逻辑顺序
  2. 简单类型优先于复杂类型
  3. 尽可能将同种类型的参数放在相邻位置，则只需写一次类型
* 函数、方法的顺序一般需要按照依赖关系由浅入深由上至下排序，即最底层的函数出现在最前面。例如，函数 ExecCmdDirBytes 属于最底层的函数，它被 ExecCmdDir 函数调用，而 ExecCmdDir 又被 ExecCmd 调用
* 避免实参传递时的语义不明确（Avoid Naked Parameters），当参数名称的含义不明显时，使用块注释语法
* 上述例子中，更好的做法是将 bool 类型换成自定义类型。将来，该参数可以支持不仅仅是两个状态（true/false）
*   避免使用 init() 函数，否则 init() 中的代码应该保证:

    1. 函数定义的内容不对环境或调用方式有任何依赖，具有完全确定性
    2. 避免依赖于其他init()函数的顺序或副作用。虽然顺序是明确的，但代码可以更改， 因此 init() 函数之间的关系可能会使代码变得脆弱和容易出错
    3. 避免访问或操作全局或环境状态，如：机器信息、环境变量、工作目录、程序参数/输入等
    4. 避免 I/O 操作，包括：文件系统、网络和系统调用

    不能满足上述要求的代码应该被定义在 main 中（或程序生命周期中的其他地方）

## Named Result Parameters

给函数返回值命名。尤其对于当你需要在函数结束的 defer 中对返回值做一些事情，返回值名字是必要的

## Receiver Names

结构体方法中，接受者的命名（Receiver Names）不应该采用 me，this，self 等通用的名字，而应该采用简短的（1 或 2 个字符）并且能反映出结构体名的命名风格，它不必像参数命名那么具体，因为我们几乎不关心接受者的名字。

例如：Struct Client，接受者可以命名为 c 或者 cl。这样做的好处是，当生成了 go doc 后，过长或者过于具体的命名，会影响搜索体验。

## Receiver Type

编写结构体方法时，接受者的类型（Receiver Type）到底是选择值还是指针通常难以决定。一条万能的建议：如果你不知道要使用哪种传递时，请选择指针传递吧！

建议：

* 当接受者是 map、chan、func，不要使用指针传递，因为它们本身就是引用类型
* 当接受者是 slice，而函数内部不会对 slice 进行切片或者重新分配空间，不要使用指针传递
* 当函数内部需要修改接受者，必须使用指针传递
* 当接受者是一个 struct，并且包含了 sync.Mutex 或者类似的用于同步的成员。必须使用指针传递，避免成员拷贝
* 当接受者类型是一个 struct 并且很庞大，或者是一个大的 array，建议使用指针传递来提高性能
* 当接受者是 struct、array、slice，并且其中的元素是指针，并且函数内部可能修改这些元素，那么使用指针传递是个不错的选择，这能使得函数的语义更加明确
* 当接受者是小型 struct，小 array，并且不需要修改里面的元素，里面的元素又是一些基础类型，使用值传递是个不错的选择

## 错误处理规范

* err 总是作为函数返回值列表的最后一个
* 如果一个函数返回error，一定要检查是否为nil，判断函数是否调用成功，如果err不为nil，一定要处理
* 尽量不要使用 \_ 丢弃任何return的err。若不进行错误处理，那么再次向上游返回或者使用log记录
* 错误提示（Error Strings）不需要大写字母开头的单词，即使是句子的首字母也不需要。除非是专有名词或者缩写。同时，错误提示也不要以句号结尾，因为通常在打印错误提示后还需要跟随别的提示信息
* 采用独立的错误流进行处理。尽可能减少正常逻辑代码的缩进，这有利于提高代码的可读性，便于快速分辨出哪些是正常的业务逻辑代码

```go
// 错误写法
if err != nil {
    ...
} else {
    ...
}

// 推荐写法
if err != nil {
    ...
}
```

* 如果我们需要用函数的返回值来初始化某个变量，应该把这个函数调用单独写在一行

```go
// 错误写法
if x, err := f(); err != nil {
    ...
} else {
    ... // use x
}

// 推荐写法
x, err := f()
if err != nil {
    ...
}
... // use x
```

* 尽量不要使用 panic，除非你知道你在做什么。只有当实在不可运行的情况下采用 panic，例如：文件无法打开，数据库无法连接导致程序无法正常运行。但是对于可导出的接口不能有 panic，不要抛出 panic 只能在包内采用。建议使用 log.Fatal 来记录错误，这样就可以由 log 来结束程序

##