# 编码规范

## 大小约定

* 单个文件的长度尽量不超过500行
* 单个函数的长度尽量不超过50行
* 单个函数圈复杂度尽量不超过10，禁止超过15
* 单个函数中嵌套不超过3层
* 单行注释尽量不超过80个字符
* 单行语句尽量不超过80个字符

## 缩进、括号和空格约定

缩进、括号和空格都使用gofmt工具处理

* 强制使用tab缩进
* 强制左大括号不换行
* 强制所有的运算符和操作数之间要留空格

## 命名规范

所有命令遵循“意图”原则

### 包、目录命名规范

* 包名和目录名保持一致，一个目录尽量维护一个包下的所有文件
* 包名为全小写单词，不使用复数，不使用下划线
* 包名应该尽可能简短

### 文件命名规范

文件名为全小写单词，使用`_`分词。golang通常具有以下几种代码文件类型：

* 业务代码文件
* 模型代码文件
* 测试代码文件
* 工具代码文件

### 标识符命名规范

短名优先，作用域越大命名越长且越有意义

#### 变量、常量名

* 统一遵循驼峰法
* 首字母根据访问控制原则使用大写或小写
* 对于常规缩略词，一旦选择了大写或小写的风格，就应当在整份代码中保持这种风格，不要首字母大写和缩写两种风格混用。以URL为例，如果选择了URL这种风格，则应在整份代码中保持。错误：UrlArray，正确：urlArray或URLArray。再以ID为例，如果选择缩写ID这种风格，错误：appleId，正确：appleID
* 对于只在本文件中有效的顶级变量、常量，应该使用`_`前缀，避免在同一个包中的其他文件以外使用错误的值，例如

```go
var (
    _defaultPort = 8080
    _defaultUser = "user"
)
```

* 若变量、常量为`bool`类型，则名称应该以`Has`、`Is`、`Can`或`Allow`等判断性动词开头

```go
var isExist bool
var hasConflict bool
var canManage bool
var allowGitHook bool
```

* 如果模块的功能较为复杂、常量名称容易混淆的情况下，为了更好的区分枚举类型，可以使用完整的前缀

```go
type PullRequestStatus int
const (
    PullRequestStatusConflict PullRequestStatus = iota
    PullRequestStatusMergeable
)
```

#### 函数、方法名

* 函数、方法命名规则：动词+名词
* 若函数、方法为判断类型（返回值主要是bool类型），则名称应以`Has`、`Is`、`Can`或`Allow`等判断性动词开头

```go
func HasPrefix(name string, prefixes []string) bool { ... }

func IsEntry(name string, entries[]string) boo { ... }

func CanManage(name string) bool { ... }

func AllowGitHook() bool { ... }
```

#### 结构体、接口名

* 结构体命名规则：名词或名词短语
* 接口命名规则：以"er"作为后缀，例如：Reader、Writer。接口实现的方法则去掉"er"，例如：Read，Write

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type WriteFlusher interface {
    Write([]byte) (int, error)
    Flush() error
}
```

## 空行、注释、文档规范

### 空行

* 空行需要体现代码逻辑的关联，所以空行不能随意，否则会影响代码的可读性
* 保持函数内部实现的组织粒度是相近的，用空行分隔

### 注释与文档

Golang的`go doc`工具可以根据注释生成代码文档，所以注释的质量决定了代码文档的质量

#### 注释风格

* 统一使用英文注释，严格使用英文标点符号
* 注释应当是一个完整的句子，以句号结尾
* 句子类型的注释首字母需大写，短语类型的注释首字母小写
* 注释的单行长度不要超过80个字符。

#### 包注释

* 每个包都应该有一个包注释。包注释会首先出现在`go doc`网页上，包注释应该包含：包名，简介；创建者；创建时间
* 对于main包，通常只有一行简短的注释用以说明包的用途，且以项目名开头
* 对于简单的非 main 包，也可用一行注释概括
* 对于一个复杂项目的子包，一般情况下不需要包级别注释，除非是代表某个特定功能的模块
* 对于相对功能复杂的非 main 包，一般都会增加一些使用示例或基本说明，且以 Package 开头
* 对于特别复杂的包说明，一般使用 doc.go 文件用于编写包的描述，并提供与整个包相关的信息

#### 函数、方法注释

每个函数、方法都应该有注释说明，包括三个方面

#### 结构体、接口注释

## 导入规范

使用`goimports`工具，在保存文件时自动检查`import`规范

* 如果使用的包没有导入，则自动导入；如果导入的包没有被使用，则自动删除
* 强制使用分行导入，即使仅导入一个包
* 导入多个包注释按照类别顺序并使用空行区分：标准库包、程序内部包、第三方包
* 禁止使用相对路径导入
* 禁止使用 Import Dot(.) 简化导入
* 在所有其它情况下，除非导入之间有直接冲突，否则应避免导入别名

```go
import (
    "fmt"
    nettrace "golang.net/x/trace" // not recommand  
)
```

* 如果包名与导入路径的最后一个元素不匹配，则必须使用别名

```go
import (
    client "example.com/client-go"
    trace "example.com/trace/v2"
)
```
