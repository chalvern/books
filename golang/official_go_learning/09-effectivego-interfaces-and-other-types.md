[effective go](https://golang.google.cn/doc/effective_go.html)

[本文的视频地址](https://www.bilibili.com/video/av74451767)


# Interfaces and other types（接口与其他类型）

## Interfaces（接口）

在 Go 中，可以用接口来指定一个对象的行为：如果一个对象可以做**某件事情**，那么它就可以用在**某个指定的地方**。之前已经看到过几个简单的例子：① 实现 `String` 方法可以自定义类型的打印格式，② `Fprintf` 可以向任何实现了 `Write` 方法的类型打印内容。在 Go 源码中，有很多只包含一两个方法的接口，一般情况下这些接口的名称直接衍生自他们所包含的方法，比如只有一个 `Write` 方法的 `io.Writer` 接口。

一个类型可以实现多个接口。比如包 `sort` 可以给实现了 `sort.Interface` 接口的集合进行排序，也就是只要集合类型实现 `Len()`, `Less(i, j int) bool`, 和 `Swap(i, j int)` 方法，就可以使用 `sort.Sort` 进行排序了；在实现了 `sort.Interface` 接口后，集合还可以通过实现 `fmt.Stringer` 接口来自定义输出格式。下面的 `Sequence` 同时实现了 `sort.Interface` 和 `fmt.Stringer` 两个接口：

```go
type Sequence []int

// Methods required by sort.Interface.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Copy returns a copy of the Sequence.
func (s Sequence) Copy() Sequence {
    copy := make(Sequence, 0, len(s))
    return append(copy, s...)
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    s = s.Copy() // Make a copy; don't overwrite argument.
    sort.Sort(s)
    str := "["
    for i, elem := range s { // Loop is O(N²); will fix that in next example.
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```


## Conversions（类型转换）


上面 `Sequence` 的 `String` 方法（复杂度为 O(N²)）做了很多 `Sprint` 已经做完的事情——`Sprint` 函数内置了打印集合类型值的功能。因此如果调用 `Sprint` 前我们把 `Sequence` 转换为普通的 `[]int`，就可以避免重复的工作同时还会加快执行速度。

```go
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```


这个方法是另一个在 `String` 方法中安全调用 `Sprintf` 函数的例子（不会触发循环调用）。因为 `Sequence` 和 `[]int` 这两个类型只是类型名不同，其底层的数据结构是一模一样的，因此这两个类型的变量可以相互进行转换。这里的类型转换并不会创建新的值，它只是临时把一个存在的值作为另一种类型的值来看待（有特例，比如 go 允许整数转换成为浮点数，而这种转换会产生一个新的值。）

在 Go 中，我们可以通过类型转换的方式获取新类型上的方法的调用权限，这种方式算是一种惯用手法吧。比如，我们可以使用已经存在的类型 `sort.IntSlice` 来减少代码量：

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

上面的代码把 `Sequence` 转换为不同的类型从而使用每种类型包含的功能，每种转换完成一部分的工作，一起达成最后期望的效果；语句 `sort.IntSlice(s)` 把 `s` 转换为 `sort.IntSlice` 类型，这种类型实现了 `sort.Interface` 接口，因此可以直接调用 `Sort()` 方法，就避免我们自己在定义 `Len/Less/Swap` 方法了。这种类型变来变去的用法看起来不同寻常，但是写起来很高效。


## Interface conversions and type assertions（接口转换与类型断言）

**类型 switch** 语法是类型转换的一种方式：他首先接收某个**接口变量**，然后在每个 case 里把接口变量转换为对应的类型。下面的代码是简化版的 `fmt.Printf` 方法，用来展示如何通过类型 switch 把一个接口变量转换成字符串。如果接口变量已经是一个字符串，只需要把这个字符串值返回就可以了；否则假如它实现了 `String` 方法（实现了 `Stringer` 接口），我们可以通过调用这个方法并返回其结果。

```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

上面的代码里，第一个 `case` 直接使用了接口变量的底层数据；第二个 `case` 把一个接口转换成另一个接口（`interface{}` 的值转换为 `Stringer` 类型的值），在 Go 中这种把类型杂糅在一起使用的场景也是比较多见的。


为了转换类型上面大动干戈地使用了 `switch...case...`语句，假如我们知道某个值保存的是 `string` 类型的值，该怎么直接把这个值解析出来？当然可以使用只有一个 `case` 的 `switch` 语句来实现，但是可以更简单地通过**类型断言**来实现。类型断言可以在接口类型的值上面抽取特定类型的值。它的语法是从 **类型 switch** 中借鉴的，但是 switch 中使用的是 `value.(type)`，类型断言的括号中直接显式指定类型 `value.(typeName)`。

```go
value.(typeName)
```

`value.(typeName)` 的结果是静态类型 `typeName` 对应的新值。这个静态类型要么是接口变量的底层数据类型，要么是接口变量的底层数据类型实现的另一个接口。如果我们已经知道某个值存储的是字符串，直接通过 `str := value.(string)` 就可以获取到。

```go
str := value.(string)
```

类似 `str := value.(string)` 的写法很简单很易用，但是如果在 `value` 中保存的不是一个字符串，会产生运行时错误导致代码崩溃。为了避免这种情况，可以使用 “逗号与ok” 的惯用语法来检测相应的值是否是字符串：

```go
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```

如果类型断言失败了，`str` 依然存在且是字符串类型，只不过它的值会是空字符串。

为了说明这种 “逗号与ok” 形式的类型断言的功能，下面的 `if - else` 表达式与上面的类型 switch 示例实现了相同的功能。

```go
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
```

## Generality（泛型）


如果类型的存在只是为了实现某个接口，除了接口中限定必须要实现的方法没有其他需要导出的方法，此时就没有必要导出这个类型，只需导出对应接口即可，表明这个类型的值除了接口里定义的方法外没有其他的方法。假如对同一个接口有不同的实现，这种做法还能避免重复地给方法写文档（只需要在接口里注明每种方法的作用、用法，各个类型实现这些方法的时候只需要简单注明是哪个接口的方法就可以了）。

在这种模式下，类型的构造器应该返回接口的值而不是相应的类型的值。比如，在哈希库 `crc32.NewIEEE` 和 `adler32.New` 两个构造器返回的是 `hash.Hash32` 接口类型的值。如果要把 CRC-32 的算法替代为 Adler-32 的算法，只需要改变一下调用的构造器，其他的不需要任何改变。

同样的，可以让不同 `crypto` 包中的流密码算法与块加密算法分开。在 `crypto/cipher` 包中的 `Block` 接口限定了块加密算法对某个独立的数据块进行加密的行为；为了通用性，使用实现 `Block` 接口的加密包来创建流加密器，从而封装块密码算法的细节，把使用者的注意力解放出来；最后为了通用性，返回 `Stream` 接口类型的数据。

接口 `crypto/cipher` 的源码如下：

```go
type Block interface {
    BlockSize() int
    Encrypt(dst, src []byte)
    Decrypt(dst, src []byte)
}

type Stream interface {
    XORKeyStream(dst, src []byte)
}
```


下面是 CTR（计数器模式）流的定义，它可以把块密码转换成为流密码；可以看到块密码相关的细节被抽象了：

```go
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```

`NewCTR` 不局限应用到某种特定的加密算法和数据源，它能够应用在所有实现了 `Block` 接口和 `Stream` 接口的加密算法上面。同时因为 `NewCTR` 返回的是 `Stream` 接口类型，因此用其他的加密算法替代 `CTR` 算法也很简单，只需要把调用了 `NewCTR` 的地方替换成为其他加密算法的构造器就可以了。

## Interfaces and methods（接口与方法）


在 Go 中几乎所有东西都可以关联方法，都可以满足某个接口。其中一个例子是定义了 `Handler` 接口的 `http` 包：任何实现了 `Handler` 接口的对象都可以处理 HTTP 请求。

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```


其中 `ResponseWriter` 本身就是一个接口，它提供了响应客户端的几个方法，包含标准的 `Write` 方法，因此 `http.ResponseWriter` 可以被使用在任何应用 `io.Writer` 的地方。`Request` 是一个结构体，包含了从客户端解析出来的请求体。


为了简化，让我们忽略 POSTs 并且假设 HTTP 请求只有 GETs。下面是一个短小但是完整的 HTTP 请求处理器（实现了 ServerHTTP 方法），它能够用于计数页面被查看的次数。

```go
// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```


（感兴趣的话可以探索一下 `Fprintf` 是如何把内容打印到 `http.ResponseWriter` 的）

作为参考，下面的代码展示了如果把这样的服务绑定到 URL 树上的：

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

在这个例子里其实整数类型也可以满足需求，那为什么还要把 `Counter` 声明成一个结构体呢？（这里需要注意接收者必须是指针才可以让调用者感知到它的增加）

```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

如果页面被访问时需要进行通知，代码该怎么写？此时可以考虑给网页绑一个信道：

```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

最后，如果我们想展示服务器启动时在 `/args` 路径里传递的参数，可以很容易地写一个函数来打印这些参数。

```go
func ArgServer() {
    fmt.Println(os.Args)
}
```

那我们该怎么把它变成一个 HTTP 服务呢？我们可以定义任意的类型并给这个类型写一个 `ArgServer` 方法，然后不使用接收者的值就可以了，但是有一个更好的方法：因为我们可以给任何类型（除了指针和接口）定义方法，因此我们可以给函数定义方法。在 `http` 中就包含这样的代码：

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

`HandlerFunc` 类型定义了 `ServeHTTP` 方法，因此这个类型的值可以处理 HTTP 请求。可以琢磨一下上面代码中 `ServeHTTP` 这个方法的使用：接受者是一个函数 `f`，在 `ServeHTTP` 方法的内部调用了 `f`。可能这看起来有一点奇怪，但是这和 “接收者是信道并且在对应的方法中向信道中发送内容” 是一样的设计模式。

为了让 `ArgServer` 注册为 HTTP 服务器，我们可以先修改它让它有正确的签名：

```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

上面的代码中 `ArgServer` 和 `HandlerFunc` 有相同的签名，因此我们可以把它转换为`HandlerFunc` 类型从而可以使用这个类型的方法，就好像我们把 `Sequence` 转换为 `IntSlice` 使用 `IntSlice.Sort` 一样。下面的代码简洁明了：

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

这样当有人访问了页面 `/args`，处理器在这个页面安装了类型为 `HandlerFunc` 的 `ArgServer`，因此 HTTP 服务器会触发 `HandlerFunc` 类型的方法 `ServeHTTP`，上面的代码里对应的接收器是 `ArgServer`，并且会通过 `HandlerFunc.ServeHTTP` 内的 `f(w, req)`）调用 `ArgServer` 方法，如此参数就被展示出来。

在这一小节中我们在结构体、整数、信道和函数上面分别创建了 HTTP 服务器，可以做到这一切都是因为接口的纯粹性：① 接口只关注方法，②可以给几乎所有的类型（除了指针和接口）关联方法从而实现接口。
