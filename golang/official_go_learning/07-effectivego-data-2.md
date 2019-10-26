[effective go](https://golang.google.cn/doc/effective_go.html)

[todo]
[本文的视频地址](https://www.bilibili.com/video/av71150528) 

# 数据

## 映射
映射（map）是一种方便且强大的内建数据结构，它把一种类型的值（键）和另一种类型的值（元素或值）关联起来。任意一种可以比较是否相等的类型都可以作为键类型，比如整数、浮点数、复数、字符串、指针、接口（只要它的动态类型可以判定相等性）、结构和数组。切片不可以作为映射的键，因为不能判定两个切片是否相等。和切片类似，映射持有底层数据结构的引用。当给函数传递了一个映射，如果函数修改了映射里的内容，函数调用者是可以感知到其改变的。


映射可以使用普通的复合文本语法进行创建，其中键值对之间通过逗号分隔；因此在初始化的时候创建哈希是很容易的。


```go
var timeZone = map[string]int{
    "UTC":  0*60*60,
    "EST": -5*60*60,
    "CST": -6*60*60,
    "MST": -7*60*60,
    "PST": -8*60*60,
}
```

给哈希的值赋值和从哈希中拿数据的语法和数组以及切片的语法类似，只不过哈希的键可能不是整形。

```go
offset := timeZone["EST"]
```

如果尝试获取哈希中没有的键值对，会得到哈希的元素类型对应的零值。比如，如果哈希保存的是整数，查找一个不存在的键会返回 `0`。可以使用值为 `bool` 类型的哈希来实现一个集合；把某个键的值设置为 `true` 从而把对应的值放进集合，然后就可以通过键来测试它的存在了。


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

因为在哈希中不存在的键值的值是零值，而有时候可能需要区分是键不存在还是对应的键的值确实是零值。哈希中有 `UTC` 的键吗？返回 `0` 是以为它不存在与哈希吗？其实可以通过一种多值赋值的方式来区分这种情况。 

```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

显而易见地，这是 **“逗号与ok”** 的惯用法。在这个例子中，如果 `tz` 存在，`seconds` 将会被正确设置，此时 `ok` 将会被设置为 `true`；如果 `tz` 不存在，`seconds` 将会被设置为 0， 此时 `ok` 将会是 `false`。下面的函数把这种用法和错误提示结合在了一起：

```go
func offset(tz string) int {
    if seconds, ok := timeZone[tz]; ok {
        return seconds
    }
    log.Println("unknown time zone:", tz)
    return 0
}
```

如果只是想知道哈希中是否存在某个元素而不需要知道其具体的值，可以用**空标识符**替代接收元素值的变量的位置。

```go
_, present := timeZone[tz]
```


如果想删除哈希中的某个元素，可以使用内建函数 `delete`；它的参数是哈希和对应要删除的键。这个函数可以在键存在的时候调用也可以在键不存在的时候调用，不会报错 :)

```go
delete(timeZone, "PDT")  // Now on Standard Time
```

## 打印

在 Go 中格式化的输出使用类似 C 中的 `printf` 家族函数，但是功能更丰富和通用。这些函数在包 `fmt` 包中，且有首字母大写的名字：`fmt.Printf`, `fmt.Fprintf`, `fmt.Sprintf`，等等。字符串函数（`Sprintf`等）会返回一个字符串而不是填充某个缓存。 

你不需要提供格式化字符串。对于 `Printf`, `Fprintf` 和 `Sprintf` 来说，每个都有对应类似的函数，比如 `Print` 和 `Println`。后面的这些函数不需要格式化字符串，而是给每个参数生成一个默认的格式。函数 `Println` 会给参数之间插入一个空白符，并且在最后向输出追加一个行尾符；函数 `Print` 则会在非字符串之间添加一个空白符。下面的例子总每行都会产生相同的输出。

```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

格式化打印函数 `fmt.Fprint` 和它的友函数的第一个参数是实现了 `io.Writer` 接口的任意对象；变量 `os.Stdout` 和 `os.Stderr` 是相似的实例。

这里开始有一些和 C 不同的事情。首先，数字化格式符，比如 `%d` ，不接受符号类型或大小；相反，打印例程通过类型来决定这些属性。

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```

打印内容如下：

```bash
18446744073709551615 ffffffffffffffff; -1 -1
```

如果只是想要默认的转换，比如小数到整数的转换，你可以使用万能格式符 `%v`（value 的首字母）；结果就是调用 `Print` 和 `Println` 会产生的值。另外，`%v` 葛师傅可以用来打印任意类型的值，甚至包含数组、切片、结构和哈希 map。下面的离子打印了之前小节定义的时区哈希值。

```go
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
```

上面的代码会输出下面的内容：

```bash
map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]
```

对于哈希， `Printf` 及其友函数会按照索引的字母排序进行输出。


当打印结构体时，格式化符 `%+v` 表示同时打印结构体的字段名和值；对于任意的值，`%#v` 会以 Go 语法的格式把值打印出来。

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

注意 **&** 符号。当需要打印的值是 `string` 或 `[]byte` 时，双引号括起来的字符串格式可以通过 `%q` 获得；而 `%#q` 会给字符串加上反引号。（`%q` 也可以格式化整形和符文类型，生成单引号括起来的符文常量。）与此同时， `%x` 可以作用于字符串、字节数组、字节切片，如果传入的是一个整数会生成一个长十六进制字符串；如果加一个空格，`% x` 会在每个字节之间加一个空格。


另一个便利的格式符是 `%T`，它会把值的类型打印出来。`fmt.Printf("%T\n", timeZone)` 的打印结果是 `map[string]int`。

```go
fmt.Printf("%T\n", timeZone)
```

打印

```go
map[string]int
```

如果想控制自定义类型的打印格式，只需要给相应的类型定义一个签名是 `String() string` 的方法。对于简单类型 `T`，其可能的方法如下：

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

如果你想打印类型 `T` 的值以及 `T` 类型的指针，`String` 方法的接收者必须是值类型（什么鬼？如果是值的话其指针岂不是临时变量的了？）；上面的例子使用了指针，因为对于结构体来说指针更高效且惯用。可以查看《[指针接收器与值接收器](#pointers_vs_values)》一节了解更多内容）


自定义的 `String` 函数可以调用 `Sprintf` 函数，因为打印的例程完全可重入且可以以这种方式封装起来。但是，这里有一个很重要的细节需要理解：不能构造这样的 `String` 方法——里面以某种方式调用 `Sprintf` 函数，然后 `Sprintf` 函数在调用定义的 `String` 方法，如此往复形成无限循环。如果 `Sprintf` 调用尝试以字符串的形式直接打印接受者就可能会触发这个问题，无限触发自定义的 `String` 函数。这个错误很容易犯，就像下面的例子：

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

上面的问题也很容易修复：只需要把参数转换为基础的字符串类型即可，这样就不会循环调用 `String` 方法了，如下面的代码所示：

```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

在“初始化一节”我们还会看到另一个避免这种循环调用的技术。


另一种打印技术是把一个打印例程的参数直接传给另一个类似的例程。函数 `Printf` 的声明中使用了类型 `...interface{}` 作为它的最后一个参数，因此可以在格式字符串后面传递任意数量的任意类型的参数。

```go
func Printf(format string, v ...interface{}) (n int, err error) {
```

在函数 `Printf` 中，`v` 就像是类型 `[]interface{}` 的值，但是如果它被传递给另一个不定参数的函数，它会表现得像一个正常的参数列表。下面的代码是 `log.Println` 的一种实现，它把自己的参数直接传给了`fmt.Sprintln` 进行实际的格式输出。


```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

上面的代码示例中，当在内部调用 `Sprintln` 时，我们在 `v` 后添加了 `...` 从而把 `v` 视为一个参数的列表；如果没有这三个点，会把 `v` 作为单独的一个切片参数传递给 `Sprintln`。


其实除了这里讨论的内容外还有很多打印相关的内容。可以查看 `godoc` 中 `fmt` 的文档了解更多的内容。


顺便提一句，`...` 参数可以作为一个特殊的类型，比如查找最小值的函数可以传入 `...int` 类型的参数，从而选择列表中最小的整数。


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

接下来我们继续聊内建函数 `append`的相关内容。内建函数 `append` 的声明方式和我们上面自定义的 `Append` 函数不同，如下面的形式：

```go
func append(slice []T, elements ...T) []T
```

这里的 *T* 是一个占位符，用来表示任意给定的类型。实际上，在 Go 语言中我们不能写一个带泛型的函数，这也是 `append` 是内建函数的原因——它需要编译器层面的支持。

`append` 做的事情就是在一个切片的尾部追加元素，然后返回追加以后的结果。`append` 函数必须要返回结果，原因就像我们手写的 `Append` 函数一样，在追加过程中，切片底层的数组可能会发生变化，此时必须通过返回结果来通知这种变化。

```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
```

上面的例子打印出 `[1 2 3 4 5 6]`。一定程度上可以认为 `append` 的传参方式和 `Printf` 有点像，可以传入任意多数目的参数。

如果想使用 `append` 函数把一个切片追加到另一个切片上面，就必须使用 `...` 运算符，类似上面中 `Output` 展示的用法。下面的代码输出的内容和上面的代码输出的内容是一样的。

```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

在上面的例子中，如果不带 `...`，代码时没有办法编译通过的，因为 `y` 的类型是一个切片而不是需要的 `int`。