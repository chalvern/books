[effective go](https://golang.google.cn/doc/effective_go.html)


# Initialization（初始化）


表面上看 Go 的初始化方式与 C/C++ 的初始化方法没有太大区别，但事实上 Go 的初始化方式更强大。Go 语言可以在初始化过程中直接创建复合结构类型（比如 结构体、映射、切片等），能够正确处理初始化对象之间的顺序问题（包括不同的包之间对象的初始化顺序）。

## Constants（常量）

在 Go 中，常量仅仅指常量。常量（包括包裹在函数内的常量）在编译时就会被创建，它只能是数字、字符（UTF8符文）、字符串或布尔值。由于常量在编译时就要确定下来，因此定义常量的表达式必须是**编译器可以执行**的常量表达式。比如 `1<<3` 是一个常量表达式，`math.Sin(math.Pi/4)` 不是常量表达式，因为 `math.Sin` 只能在运行时调用，不能在编译时调用。

在 Go 中，枚举型常量经常通过使用 `iota` 枚举器来创建。鉴于 `iota` 可以作为表达式的一部分来使用，同时表达式可以隐式地重复，因此很方便创建复杂枚举值集合。

```
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```


可以给自定义的任何类型添加方法（比如 `String` 方法），这种能力使得任意类型的值被打印时都能定义自己的输出格式。最常见的是给结构体添加方法，不过我们也可以给标量类型（比如 浮点类型 `ByteSize`）添加方法。

```
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```

表达式 `YB` 会打印 `1.00YB`， 而 `ByteSize(1e13)` 会打印 `9.09TB`。

上面的代码通过使用 `Sprintf` 来实现 `ByteSize` 的 `String` 方法是安全的（不存在上一小节提到的循环调用问题），因为使用了 `%f` 格式符，而它不是一个字符串格式化符：`Sprintf` 只在需要字符串类型的时候才会调用 `String` 方法，`%f` 指定的是浮点值，也就自然而然避免了上面代码中 `String` 方法调用 `fmt.Sprintf` 方法后者再进一步触发 `String` 方法的循环调用链。


## Variables（变量）


可以像常量那样初始化变量，但是变量的初始化表达式可以是在运行时进行计算的通用表达式。

```
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

## The init function（init 函数）


**每个源文件**都可以定义 `init` 函数用来初始化它需要的状态。（事实上每个文件可以有多个 `init` 函数，而且这些 `init` 函数会按序执行，比如 [gosmile](https://github.com/chalvern/gosmile/tree/template/db/migration) 数据库迁移的设计就可以利用这个特性）。`init` 会在包中所有的变量声明被执行后再调用，而且只在所有被导入的包初始化以后才会被调用（被导入的包先初始化，然后常量和变量被初始化，接着才是 `init` 函数的执行）。

`init` 函数大都是为了在真实运行开始前校验或者修复程序的状态，除此外不应该用作其他的用途。（根据开发经验，在 `init` 中添加过多的逻辑会让测试很难写）

```
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

# Methods（方法）

## Pointers vs. Values（指针与值）

就像看到的 `ByteSize` 那样，我们可以给任何有名称的类型（除了指针和接口）定义方法，并不限定必须是结构体类型。

在切片一节的讨论中，我们编写了一个 `Append` 函数。其实我们可以把它定义为切片的一个方法。为了达成这个目的，首先我们需要基于切片声明一个新的类型从而方便把函数绑定到这个类型上，然后就可以很自然地在切片的变量上调用这个方法。

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as the Append function defined above.
}
```


由于上一小节提到的切片的特点，上面的方法仍然需要返回更新后的切片，否则方法的调用方是感知不到这种更新的。不过我们可以把接受者改为`ByteSlice`的**指针**从而改进上面笨拙的写法，从而避免显式地返回切片，如下所示：

```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

实际上我们可以做的更好，比如模仿标准的 `Write` 方法继续改进 `Append` 函数，如下：

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

如此，类型 `*ByteSlice` 实现了便利的标准接口 `io.Writer`。比如下面的代码：

```go
    var b ByteSlice
    fmt.Fprintf(&b, "This hour has %d days\n", 7)
```


上面的代码中我们传递了 `ByteSlice` 的地址，因为只有 `*ByteSlice` 实现了 `io.Writer` 接口。关于**指针接受者和值接受者**的规则是：**值方法可以在指针接受者上面和值接受者上面触发调用，但是指针方法只能被指针接受者触发调用**。


之所以有上面的规则，是因为指针方法可以修改接受者；如果在值接受者上面触发指针方法会导致方法接收一个值的拷贝，任何对该值的修改都会被丢弃，因此 Go 语言层面上不允许这种错误。不过，这里有一个特例。如果值是可取址的，当在一个值上调用指针方法的时候，Go 语言会照顾到这种常见的用法在值上自动添加一个取址运算符。比如上面的例子中，`b` 是可取址的，因此我们可以通过 `b.Write` 调用它的 `Write` 方法，编译器会为我们把它重写为 `(&b).Write`。

```go
package main

type User struct {
	Name string
	Age  int
}

func (u *User) SetName(newName string) {
	u.Name = newName
}

func newUser() User {
	return User{"angel", 18}
}

func main() {
	user := newUser()
	user.SetName("darling")
	// 等效于
	(&user).SetName("darling")

	// 下面的代码编译报错
	// cannot take the address of newUser()go
	newUser().SetName("darling")
}
```


顺便提一下，`bytes.Buffer` 就是通过在字节的切片上定义 `Write` 方法来实现的。