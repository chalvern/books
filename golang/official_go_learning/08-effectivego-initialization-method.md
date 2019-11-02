[effective go](https://golang.google.cn/doc/effective_go.html)


# Initialization（初始化）


虽然表面上看 Go 的初始化与 C/C++ 的初始化没有太大区别，但是 Go 的初始化方式更强大。在初始化过程中可以直接创建复合结构，且正确地处理了初始化对象之间的排序问题，即使在不同的包之间也是如此。

## Constants（常量）

在 Go 中，常量仅仅指常量。常量在编译时就被创建，甚至函数内的本地常量也是如此；常量只能是数字、字符列表（符文列表）、字符串或布尔值。受到编译时的限制，定义常量的表达式必须是编译器可以执行的常量表达式。比如 `1<<3` 是一个常量表达式，`math.Sin(math.Pi/4)` 不是常量表达式，因为 `math.Sin` 需要在运行时才可以调用。

在 Go 中，枚举型常量通过使用 `iota` 枚举器来创建。鉴于 `iota` 可以作为表达式的一部分，同时表达式可以隐式地重复，因此很方便创建复杂的值集合。

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


可以给任何用户定义的类型添加方法（比如 `String` 方法），这种能力使得任意值被打印时都能定义自己的输出格式。最常见的是给结构体添加方法，但是这项技术也可以用来给标量类型（比如 浮点类型 `ByteSize`）添加方法。

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

上面的代码通过使用 `Sprintf` 来实现 `ByteSize` 的 `String` 方法是安全（避免循环调用）的，倒不是因为类型转换，而是因为使用了 `%f` 格式符，它不是一个字符串格式化符：`Sprintf` 只在需要字符串类型的时候才会调用 `String` 方法，上面代码中 `%f` 希望的是浮点值。


## Variables（变量）


变量会向常量那样被初始化，但是初始化表达式可以是在运行时进行计算的通用表达式。

```
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

## The init function（init 函数）


每个源文件可以定义它自己的 `init` 函数用来初始化需要的状态。（事实上每个文件可以有多个 `init` 函数）。最后真的意味着最后：`init` 会在包中所有的变量声明被执行后再调用，而且只在所有被导入的包初始化以后才会调用。

在初始化时不能声明新东西，`init` 函数大都是为了在真实运行开始前校验或者修复程序的状态。

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

就像我们看到的 `ByteSize` 那样，可以给任何命名的类型（除了指针和接口）定义方法；接收器不必一定是结构。


在切片一节的讨论中，我们编写了一个 `Append` 函数。其实我们可以把它定义为切片的一个方法。为了达成目的，我们首先需要声明一个命名的类型从而方便把函数绑定到这个类型上，然后就可以给相应的方法传递一个相应类型的接收者。

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as the Append function defined above.
}
```


上面的函数仍然需要方法返回更新后的切片。我们可以把接受者改为`ByteSlice`的**指针**来修改上面笨拙的写法，这样就可以复写调用者里的切片了，如下所示：

```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```


实际上我们可以做的更好。如果我们模仿标准的 `Write` 方法继续修改我们的函数，如下：

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


上面的代码中我们传递了 `ByteSlice` 的地址，因为只有 `*ByteSlice` 实现了 `io.Writer` 接口。上面关于**指针接受者和值接受者**的规则是：**值方法可以在指针和值上面触发调用，但是指针方法只能被指针触发调用**。



之所以有上面的规则，是因为指针方法可以修改接受者；如果在值接受者上面触发调用，会导致方法接收一个值的拷贝，因此任何的修改都会被丢弃；正因为这个原因 Go 语言层面上不允许这种错误。不过，这里有一个特例。如果值是可取址的，当在一个值上调用指针方法的时候，Go 语言会照顾到这种常见的用法在值上自动添加一个取址运算符。比如上面的例子中，`b` 是可取址的，因此我们可以通过 `b.Write` 调用它的 `Write` 方法，编译器会为我们把它重写为 `(&b).Write`。

顺便一提，在字节的切片上使用 `Write` 是 `bytes.Buffer` 的核心实现方式。