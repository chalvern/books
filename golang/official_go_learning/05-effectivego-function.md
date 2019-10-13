[effective go](https://golang.google.cn/doc/effective_go.html)

# 函数

## 多返回值
Go 语言非同寻常的一个特性是它的函数和方法可以返回多个值。和 C 语言比，Go 语言的这个特性可以避免 C 程序中不优雅的使用习惯：比如通过返回 -1 表示 EOF 错误，比如为达到某个目的通过地址传值的方式传入某个参数然后修改这个参数（参考后面 `nextInt` 的例子）。

在 C 语言中，写错误由负数表示，错误码与错误类型的对应关系则保存在内存里，必要的时候手动去查找这个对应关系。在 Go 语言中， Write（写） 可以同时返回一个数值和一个错误，能很自然地表明这种含义：“虽然你成功写了一些字节，但是因为设备满了导致剩了一些字节没有写成功。” Go 标准库中包 `os` 中的 Write 方法的定义是 `func (file *File) Write(b []byte) (n int, err error)`，它返回写成功的数量，而且当 n != len(b) 的时候还会返回一个非空的错误。这是一种很常见的形式，可以在“错误处理”的章节查看更多的例子。

同样的，多返回值的特性还可以避免传入一个指针类型的参数来承接返回值。下面给出了一个简单明了的函数，其功能是在一个字节数组的某个位置开始获取一个数字，然后返回这个数字以及下一个位置。

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

可以用上面的函数来扫描切片 b 中的数字：

```go
for i := 0; i < len(b); {
    x, i = nextInt(b, i)
    fmt.Println(x)
}
```

如果函数只能返回一个值，那么 `nextInt` 将只能返回**目标数字**，但是没有办法返回找到的这个目标数字的位置，为了获取这个位置，就必须给 `nextInt` 函数传入一个指针类型的参数，然后在 `nextInt` 函数里面修改这个值，从而让它的调用方获取这个值。

## 命名的返回变量

Go 语言中可以给函数的返回值（或者结果值）指定名称，这样就可以在函数体中像使用入参那样使用返回值变量了。返回值一旦被命名成为变量，它们就会在函数执行之初被初始化为对应类型的零值；如果函数执行了 return 命令且 return 后面没有显式指定任何参数，那么 return 语句执行时返回值变量的值就是实际意义上的返回值。

函数返回值的命名不是必要的，但是他们可以让代码更简洁明了——他们属于文档的一部分。比如我们把 nextInt 的返回值分别命名为 `value` 和 `nextPos`，就能很清楚地表明返回的是**值**和**位置**，比如下面的定义：

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

因为命名的返回值在函数原型之初就已经被初始化好了，因此可以直接 return ，这种应用方式能让代码变得更简洁。下面给出了 io.ReadFull 的一个版本，很好地应用了这个特性：

```go
func ReadFull(r Reader, buf []byte) (n int, err error) {
    for len(buf) > 0 && err == nil {
        var nr int
        nr, err = r.Read(buf)
        n += nr
        buf = buf[nr:]
    }
    return
}
```

## defer 推迟函数

Go 语言的 defer 语句可以指定某个函数（被 deferred 的函数）在当前函数返回给调用者之前再运行。defer 的这种用法有点不寻常，但是非常有用：当代码中引用了某个资源（某个 Mutex 锁、打开了某个文件），不管程序执行到哪里都必须释放这个资源，这种情况使用 defer 就很方便处理。defer 典型的应用就是**解锁互斥锁**和**关闭文件句柄**。

```go
// Contents returns the file's contents as a string.
func Contents(filename string) (string, error) {
    f, err := os.Open(filename)
    if err != nil {
        return "", err
    }
    defer f.Close()  // f.Close will run when we're finished.

    var result []byte
    buf := make([]byte, 100)
    for {
        n, err := f.Read(buf[0:])
        result = append(result, buf[0:n]...) // append is discussed later.
        if err != nil {
            if err == io.EOF {
                break
            }
            return "", err  // f will be closed if we return here.
        }
    }
    return string(result), nil // f will be closed if we return here.
}
```

通过 defer 的方式推迟执行类似上例中 `f.Close()` 函数的用法有两个好处。第一，它能保证调用必然会被执行，防止以后修改函数逻辑（比如加了另一个 return 的分支）忘记关闭文件句柄。第二，这种用法意味着打开文件与关闭文件的代码可以放在一起写，比起分开写的方式（打开文件句柄的代码在函数开始的地方，关闭的代码在函数末尾）更直观也更容易理解。

当 defer 语句执行的时候，传递给 defer 的函数（也可能是一个方法）的参数表达式马上就会运算，不会推迟到与被 defer 的函数一起执行。比如 `defer fmt.Printf("%d ", i+i)`，在这条语句执行的时候，`i+i` 立马会被执行得到一个值，而`fmt.Printf` 会被推迟到最后再执行。上面的特性意味着我们需要避免函数执行过程中被 defer 的参数值被修改，同时意味着我们可以通过一条推迟语句推迟多个函数执行体。下面是一个简单的示例（仅仅是示例，实际开发中不会这么写）：

```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

被延迟的函数会按照后进先出（LIFO）的顺序依次执行，因此上面的代码在函数返回后会打印出 `4 3 2 1 0`。

defer 的应用的另一个具有象征意义的例子是跟踪整个程序中函数的执行顺序。我们可以简单的写一段跟踪代码如下：

```go
func trace(s string)   { fmt.Println("entering:", s) }
func untrace(s string) { fmt.Println("leaving:", s) }

// Use them like this:
func a() {
    trace("a")
    defer untrace("a")
    // do something....
}
```

上面我们提到过，当 `defer` 表达式开始执行时，被 defer 的函数的参数表达式马上就会被执行，利用这个特性我们可以写出更优雅的代码。上面的 `trace` 函数可以作为 `untrace` 函数的参数使用，修改后的代码如下：

```go
func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func un(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer un(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer un(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```
运行上面的代码输出的结果如下：
```bash

entering: b
in b
entering: a
in a
leaving: a
leaving: b

```

对于习惯了通过不同的代码块来管理资源（比如 `python` 中的 `try...except...else...finally` 这种用法，通过 `finally` 来控制必然需要执行的一些逻辑）的其他语言的开发者来说，，可能 defer 的使用有点奇特，但是 defer 最有意思也最具应用价值的地方恰恰是因为 defer 并不基于块而是基于函数来起作用，让代码变得更简洁，让编码更容易。在 panic 和 recover 小节我们会看到一些其他应用 defer 的例子。