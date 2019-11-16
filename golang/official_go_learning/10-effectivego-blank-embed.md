[effective go](https://golang.google.cn/doc/effective_go.html)

[本文的视频地址](https://www.bilibili.com/video/av75076225)

# The blank identifier（空白标识符）

至此为止我们已经提到过很多次空白标识符，在 “for-range 循环” 和 “映射 maps” 小节都有提到。空白标识符可以被任意类型的任意值进行赋值，而且这些值会静默地被丢弃掉。它有点像 Unix 的 `/dev/null` 文件描述符：空白标识符（_）表示一个只写的值，它可以作为占位符使用，如果有一个变量的值不会被使用但是又必须要设置一个变量来接收这个值，此时就可以使用空白标识符。当然，除此外空白标识符的用处还有很多。

### The blank identifier in multiple assignment（多值赋值语句中的空白占位符）

for-range 循环中的空白占位符是一种通用情况的特例：属于多值赋值语句。

如果一个赋值语句需要在左侧有多个变量，但是其中部分变量不会被代码使用，这时候用空白标识符就可以避免创建多余的变量并且显式告诉指明丢弃对应的值。比如，当我们调用返回 值和错误 两个值的函数的时候，如果我们只关注它的错误，就可以使用空白标识符来丢弃这个函数返回的值，比如下面的例子：

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```

偶尔情况下你还可以看到忽略错误值的代码，当然这种习惯是非常不好的，在实际的编码实践中也不推荐这么做；最佳实践应该总是检查错误的返回值，毕竟错误中总包含着一些错误的原因，一般是代码应该处理的。

```go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
```

### Unused imports and variables（未使用的导入与未使用的变量）

如果导入了没有使用的包或者声明了没有使用的变量，代码编译的时候编译器都会报错。如果导入一个包却不使用它会让程序显得臃肿而且会拖慢代码的编译速度；一个变量初始化未经使用则意味着计算资源的浪费，有时候甚至意味着很大的缺陷（bug）。不过，代码在开发过程中总会出现尚未使用的导入和变量，如果仅仅为了编译通过而删掉他们后面写到对应的逻辑以后再把它们加回来，这种操作显得有点烦人，此时就可以使用空白描述符：

下面完成了一半的代码中有两个没有使用过的导入（`fmt` 和 `io`），还有一个没有使用过的变量 `fd`，明显下面的这段代码是不会被编译通过的。但是我们可能想提前知道这段代码是不是可以运行，如果能运行，至少能给开发者一点小小的自信（程序员总是希望能早点看到输出结果😆）。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```

为了能让编译器正常编译上面的代码而不报错，我们可以使用空白标识符来标记导入的包；同样的道理，可以把未使用的 `fd` 变量赋值给空白标识符。比如下面的代码就可以成功编译通过：

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

传统意义上来说，为了避免编译器报错的全局声明语句（类似上面代码中的 `var _ = fmt.Printf` 和 `var _ io.Reader`）应该紧跟在导入语句的后面并且注释一下情况，这样未来可以方便地找到这些语句，同时还能够作为一种删除提醒。

### Import for side effect（为了副作用的导入）

在前面的例子中，空白标识符的使用仅仅意味着代码还在开发状态，像 `fmt` 和 `io` 这种未经使用的包最后要么被使用要么被移除。不过真实编码过程中我们可能只需要导入包从而引起某种副作用，但是确实不会在文件中显式地使用这个包。

比如，包 `net/http/pprof` 的 `init` 函数会注册提供调试信息的 HTTP 处理器，虽然这个包有导出的 API，但是导入此包的文件需要的仅仅是调试处理器然后通过它提供的网页页面获取信息，因此导入此包的文件没有必要显式引用这些 API。为了某个副效果而导入某个包却不使用这个包的变量，这种情况就可以给这个包重命名为空白标识符。

```go
import _ "net/http/pprof"
```

因为没有其他地方使用 `net/http/pprof` 这个包，这种方式的导入可以很明显地表明目的：在这个文件中这个包没有名字，之所以导入它只是为了获得这个包的副效果。

### Interface checks（接口检查）

就像我们在“接口”那一节看到的，类型不需要显式指明自己实现了哪些接口。相反，类型仅需实现接口中包含的方法隐式地来实现接口。实际编码中，大部分的接口转换都是静态的，因此在编译时就会进行类型检查。比如，如果一个函数希望得到 `io.Reader` 类型的参数，可以给它传递一个实现了 `io.Reader` 的 `*os.File` 类型的变量，如果 `*os.File` 没有实现 `io.Reader` 接口，代码不能编译通过。

不过，有时接口的检查是在运行时发生的。比如在 `encoding/json` 这个包里面有一个 `Marshaler` 接口，当 JSON 编码器接收到一个实现了这个接口的值的时候，编码器会调用这个值的 `marshal` 方法把这个值转化成为 JSON，这个时候就不会调用标准的转换过程了。编码器会在运行时对变量进行类型断言从而检查它的属性：

```go
m, ok := val.(json.Marshaler)
```


如果仅需要知道某个类型是否实现了某个接口，可以使用空白标识符从而忽略掉类型检查得到的值，如下面代码所示：

```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

这种情况下，我们必须在实现了该类型的包内保证它实现了相应的接口。比如，如果类型 `json.RawMessage` 需要一个自定义的 `JSON` 表示，它应该实现 `json.Marshaler` 接口，但是代码中没有任何静态转换让编译器确认这种实现。如果这个类型没有实现相应的接口方法，`JSON` 编码器仍然会工作，只是这个时候不会使用自定义的实现而是调用默认的转换方法。为了保证某个类型实现了正确的方法，可以在包中写一个使用了空白标识符的声明：

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

在这个声明中，赋值语句包含一次转换，从 `*RawMessage` 转换为 `Marshaler`，这种检查会在编译时进行。如果接口 `json.Marshaler` 变化了，这个包不会编译通过，这个时候我们就知道它需要更新了。

在上面的这种情况下，空白标识符的出现标明声明语句仅仅为了类型检查，而并不会创建一个变量。但是，如果知道类型肯定满足某个接口则没有必要做这种声明。通常情况下，这种用法仅在源码中不存在静态类型转换的情况时使用，而这种情况其实很少见到。

## Embedding（嵌套）

Go 没有子类的概念，但是我们可以通过在结构体或接口中**嵌套**某个类型来实现类似的功能。

接口的嵌套非常简单，我们前面提到过的 `io.Reader` 和 `io.Writer` 接口，下面是它们的声明：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

上面列出的两个接口都只有一个方法，在 `io` 包中还导出了几个包含多个方法的接口。比如，`io.ReadWriter` 接口，它包含了 `Read` 和 `Write` 两个方法。我们可以直接把两个方法直接写在 `io.ReadWriter` 接口定义里，但是更简单更明了的方式是把上面的两个接口 `Reader` 和 `Writer` 嵌进新的接口中，就像下面的代码所示：

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

上面的代码的含义是：`ReadWriter` 接口，它既是 `Reader` 又是 `Writer`，它是嵌套起来的接口的联合体（这些联合体之间的方法不能有重名）。需要注意的是，接口只能被嵌套到接口里。

结构体也有嵌套的用法，但是含义更深远也更复杂。包 `bufio` 有两个结构体类型 `bufio.Reader` 和 `bufio.Writer`，分别实现了包 `io` 中的相关接口。同时 `bufio` 包还实现了带缓存的读写器（`reader/writer`），通过嵌套的方式在同一个结构中组合 `reader` 和 `writer`。如下代码所示（在组合结构体声明中列出子结构体，但是不给定字段名）：

```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

嵌套元素是子结构体的指针，因此初始化时必须要指向合法的结构体才能使用（大家可以回顾初始化一节的内容）。`ReadWriter` 结构体也可以写为：

```go
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

但如果指定了字段名，为了使用字段里的方法，同时为满足 `io` 包对应的接口，不得不手动定义代理方法，比如下面的 `Read` 方法：

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

通过直接嵌套结构，我们可以避免上面啰嗦的编码方式。嵌套类型的方法自动提升为组合体的方法，也就是说，`bufio.ReadWriter` 不仅实现了 `bufio.Reader` 和 `bufio.Writer`这两个接口中的方法，而且同时满足下面的三个接口：`io.Reader`, `io.Writer`, 和 `io.ReadWriter`。

嵌套与子类继承的模型还是很不一样的。如果我们嵌套了一个类型，则这个类型所有的方法都会成为外面类型的方法，但是当方法被触发调用的时候其接收者依然是里面的类型而不是外面的类型。比如上面的例子中，当 `bufio.ReadWriter` 的 `Read` 方法被触发的时候，和上面手动写的代理方法是一样的效果：接收者是 `ReadWriter` 的 `reader` 字段，而不是 `ReadWriter` 自己。

嵌套也可以很便利的应用。下面的例子中，嵌套的字段和普通的命名字段一起使用：

```go
type Job struct {
    Command string
    *log.Logger
}
```

上面定义的 `Job` 类型有 `Print`, `Printf`, `Println` 和 `*log.Logger` 的其他方法。我们可以给 `Logger` 取一个字段名，但是没必要这么做。现在，我们可以向初始化后的 `Job` 类型的对象打日志了：

```go
job.Println("starting now...")
```

`Logger` 是结构体 `Job` 的一个常规字段，因此我们可以通过常规的方式来构建 `Job`，比如下面的方式：

```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

也可以使用下面的符合语法：

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

如果我们需要直接引用嵌套的字段，字段的名称默认是**忽略包修饰符的名称**，就像 `ReadWriter` 结构体中的 `Read` 方法一样。如果我们需要访问 `Job` 变量中的 `*log.Logger`，我们可以写成 `job.Logger`，如果想重新定义 `Logger` 的方法，这种写法非常有用，如下面的例子。

```go
func (job *Job) Printf(format string, args ...interface{}) {
    job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

嵌套类型虽然好用，但同时引入了命名冲突的问题，不过 Go 语言解决这种冲突的方式也很简单。首先，字段或方法 `X` 会覆盖任何深层嵌套类型的 `X`。比如，如果 `log.Logger` 包含了一个叫 `Command` 的字段或方法，`Job` 中的 `Command` 字段会覆盖它。

其次，如果同样的名字在**同一级的嵌套**中出现（对比不同级的情况），这会导致错误。如果 `Job` 结构包含了另一个名称为 `Logger` 的字段或者方法，此时如果再嵌套 `log.Logger` 就会报错。然而有特例，如果重复的名称不会在类型定义之外使用，就不会引起问题；这可以保护结构体不会因为被嵌套的外部类型的变化而变得不可用，也就是说当添加的字段与另一个子类型的字段冲突时，如果两个字段都不会被使用，这种情况下是不会报错的。