[effective go](https://golang.google.cn/doc/effective_go.html)

[todo]
[本文的视频地址](https://www.bilibili.com/video/av71150528) 

# 数据

## 映射
映射（map）是一种既方便使用又功能强大的内建数据结构，它把一种类型的值（键）和另一种类型的值（值）关联起来。任何可以比较相等性的类型都可以作为键来使用，比如整数、浮点数、复数、字符串、指针、接口（只要它的动态类型可以判定相等性）、结构和数组。因为不能判定两个切片是否相等，因此不可以把切片作为映射的键。和切片类似的是，映射持有底层数据结构的引用。当给函数传递映射变量时，如果函数修改了映射里的内容，函数调用者是可以感知到这种改变的。


映射可以使用普通的复合字面语句进行创建，键值对之间通过逗号分隔，语法简单，很方便创建。


```go
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

从映射中存取键值对的语法和数组、切片的语法类似，只不过哈希的键可能不是整形数。

```go
offset := timeZone["EST"]
```

如果尝试获取映射中没有的键值对，得到的是映射值类型对应的零值。比如，如果映射的值保存的是整数（类似 `var a map[string]int`)，查找不存在的键值会直接返回 `0`，不会抛出任何错误。基于这个特性，我们可以使用值为 `bool` 类型的映射实现一个集合：把某个键的值设置为 `true` 表示对应的值添加到集合里面了，之后就可以通过键来检测它的存在了。


```go
attended := map[string]bool{
    "Ann": true,
    "Joe": true,
    ...
}

if attended[person] { // will be false if person is not in the map
    fmt.Println(person, "was at the meeting")
}
```

因为在哈希中不存在的键值的值是零值，有时候可能需要区分“键不存在”还是“对应的键的值被设定为零值”。如果 `var timeZone map[string]int` 中保存了各个时区的偏移量，其中 `UTC` 的值应该是 `0`，那么我们怎么判定映射中有 `UTC` 的键呢？当我们取 `timeZone["UTC"]` 时返回了 `0` 是因为它不存在于映射中吗？此时可以通过多值赋值的方式来判定这种情况：  

```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

上面展示的是 **“逗号与ok”** 的惯用语法。在这个例子中，如果 `tz` 存在，`seconds` 将会被正确设置，此时 `ok` 的值为 `true`；如果 `tz` 不存在，`seconds` 会被设置为 0， `ok` 会被设置为 `false`。下面的函数把 **“逗号与ok”** 的用法和错误提示结合在了一起：

```go
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

如果只是想知道映射中是否存在某个元素，不需要知道具体的值，可以用**空标识符**替代接收变量：

```go
_, present := timeZone[tz]
```


如果想删除映射中的某个键值对，可以使用内建函数 `delete` 来实现，参数是映射和将要删除的键。这个函数可以在键存在的时候调用也可以在键不存在的时候调用，多次不会报错，用起来很省心 :)

```go
delete(timeZone, "PDT")  // Now on Standard Time
```

## 打印

为了能格式化打印内容，Go 有一组和 C 中的 `printf` 类似的函数，但是功能更丰富也更通用。这些函数在 `fmt` 包中，名字的首字母都是大写的：`fmt.Printf`, `fmt.Fprintf`, `fmt.Sprintf`，等等。字符串函数（`Sprintf`等，以 `S` 开头）返回一个字符串，其他函数会把内容直接填充到某个缓存里。 

以 `f` 结尾的函数都需要传入一个格式化字符串作为参数，为了使用上的方便，也有不需要提供格式化字符串的函数。对于 `Printf`, `Fprintf` 和 `Sprintf` 等函数都有对应不需要格式化字符串的函数，比如 `Print` 和 `Println`等，后面的这些函数会给每个参数生成一个默认的格式。其中 `Println` 会给参数之间插入一个空白符，并且在最后的输出追加一个行尾符；`Print` 函数会在**非字符串参数之间**添加一个空白符。下面的例子每行打印的内容都是一样的：

```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

格式化打印函数 `fmt.Fprint` 及其他以 `F` 开头的打印函数的第一个参数是实现了 `io.Writer` 接口的任意对象，`os.Stdout` 和 `os.Stderr` 就是这样的对象。

这里有一些和 C 语言不一样的地方。首先，数字化格式符（比如 `%d`）不接受符号类型或大小；相反，打印例程通过参数的类型来决定这些属性。

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```

打印内容如下：

```bash
18446744073709551615 ffffffffffffffff; -1 -1
```

如果只是想要默认的格式转换，比如把整数作为十进制数值来打印，可以使用万能格式符 `%v`（value 的首字母）；这也是 `Print` 和 `Println` 默认的打印模式。事实上，`%v` 格式符可以用来打印任意类型的值，包含数组、切片、结构和 map 映射。下面的例子打印了之前小节定义的时区映射变量。

```go
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
```

上面的代码会输出下面的内容：

```bash
map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]
```

对于映射， `Printf` 及其他相关函数会**按照索引的字母排序**进行输出。


当打印结构体时，格式化符 `%+v` 表示同时打印结构体的字段名和值；对于任意的值，`%#v` 则会以 Go 语法的格式把值打印出来，意味着我们可以直接把打印出来的内容赋值到源码中直接作为代码片段来使用。

```go
type T struct {
    a int
    b float64
    c string
}
t := &T{ 7, -2.35, "abc\tdef" }
fmt.Printf("%v\n", t)
fmt.Printf("%+v\n", t)
fmt.Printf("%#v\n", t)
fmt.Printf("%#v\n", timeZone)
```

打印内容如下：

```go
&{7 -2.35 abc   def}
&{a:7 b:-2.35 c:abc     def}
&main.T{a:7, b:-2.35, c:"abc\tdef"}
map[string]int{"CST":-21600, "EST":-18000, "MST":-25200, "PST":-28800, "UTC":0}
```

注意 **&** 符号。当需要打印的值是 `string` 或 `[]byte` 时，双引号括起来的字符串格式可以通过 `%q` 获得；而 `%#q` 会给字符串加上反引号。`%q` 也可以格式化整形和符文类型，生成的是单引号括起来的符文常量。与此同时， `%x` 可以作用于字符串、字节数组、字节切片，如果传入的是一个整数会生成一个长长的十六进制字符串；如果加一个空格——`% x`——会在每个字节之间加一个空格。

```go
	str := "hello 世界"
	fmt.Printf("%q\n", str)
	fmt.Printf("%+q\n", str)
	fmt.Printf("%x\n", str)
	fmt.Printf("% x\n", str)
	fmt.Printf("%x\n", 12)
```

打印的结果如下：
```bash
"hello 世界"
"hello \u4e16\u754c"
68656c6c6f20e4b896e7958c
68 65 6c 6c 6f 20 e4 b8 96 e7 95 8c
c
```

另一个使用起来很方便的格式符是 `%T`，它可以把值的类型打印出来。`fmt.Printf("%T\n", timeZone)` 的打印结果是 `map[string]int`。

如果想控制自定义类型的打印格式，只需要给相应的类型定义一个签名为 `String() string` 的方法。对于简单类型 `T`，其可能的方法如下：

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

上面的代码会打印出下面的结果：

```go
7/-2.35/"abc\tdef"
```

如果想打印类型 `T` 及其指针的值，`String` 方法的接收者必须是值类型；上面的例子接受者使用了指针，是因为对于结构体来说指针更高效且惯用。可以查看《指针接收器与值接收器》一节了解更多内容）


自定义的 `String` 函数里面可以调用 `Sprintf` 函数，因为打印的例程完全可重入且可以以这种方式封装起来。但是，有一个很重要的细节需要理解：不能在 `String` 方法里面以某种方式调用 `Sprintf` 函数，然后 `Sprintf` 再触发调用 `String` 方法，如此往复形成死循环。如果 `Sprintf` 尝试以字符串的形式直接打印接受者就可能会无限调用自定义的 `String` 函数。比如下面的例子：

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

上面的问题也很容易修复：只需把参数转换为基础的字符串类型即可，如下面的代码所示：

```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

在《初始化一节》我们还会看到另一个避免这种循环调用的技术。

还有一种打印用法是把一个打印例程的参数直接传给另一个类似的例程。函数 `Printf` 的声明中使用 `...interface{}` 作为它的最后一个参数类型，因此可以在格式字符串后面传递任意数量任意类型的参数。

```go
func Printf(format string, v ...interface{}) (n int, err error) {
```

在函数 `Printf` 中，`v` 就像是类型 `[]interface{}` 的值，可以把它作为正常的参数列表传递给另一个不定参数的函数。下面的代码是 `log.Println` 的一种实现，它把自己的参数传给了`fmt.Sprintln` 进行实际的格式输出。


```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

在上面的代码示例中，调用 `Sprintln` 时在 `v` 后添加了 `...` 从而把 `v` 作为一个参数列表传入；如果没有这三个点，`v` 会作为一个切片类型的值传递给 `Sprintln`。


可以查看 `godoc` 中 `fmt` 的文档了解更多打印相关的内容。


顺便提一句，`...` 参数可以作为一个特殊的类型，比如查找最小值的函数可以传入 `...int` 类型的参数，然后选择列表中最小的整数。


```go
func Min(a ...int) int {
    min := int(^uint(0) >> 1)  // largest int
    for _, i := range a {
        if i < min {
            min = i
        }
    }
    return min
}
```

### Append（追加）

接下来我们继续聊内建函数 `append` 的相关内容。内建函数 `append` 的声明方式和我们上面自定义的 `Append` 函数不同，如下面的形式：

```go
func append(slice []T, elements ...T) []T
```

这里的 *T* 是一个占位符，用来表示任意给定的类型。实际上，在 Go 语言中我们不能写带泛型的函数，这也是 `append` 是内建函数的原因——它需要编译器层面的支持。

`append` 做的事情是在一个切片的尾部追加元素，然后返回追加以后的结果。`append` 函数必须要返回结果，原因就像我们手写的 `Append` 函数一样，在追加过程中，切片底层的数组可能会发生变化，此时必须通过返回结果来通知这种变化。

```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
```

上面的例子打印出 `[1 2 3 4 5 6]`。一定程度上可以认为 `append` 的传参方式和 `Printf` 有点像，可以传入任意多数目的参数。

如果想使用 `append` 函数把一个切片追加到另一个切片后面，必须使用 `...` 运算符，类似上面中 `Output` 展示的用法。下面的代码输出的内容和上面的代码输出的内容是一样的。

```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

在上面的例子中，如果不带 `...`，代码没有办法编译通过，因为 `y` 的类型是一个切片而不是需要的 `int`。