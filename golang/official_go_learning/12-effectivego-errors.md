# Errors（错误）

原则上，第三方库在报错时能给调用者返回有指示含义的错误信息。像之前提到的 Go 的**多返回值**特性让**同时返回普通返回值和详尽的错误描述**变得很简单。通过这个特性来提供详细的错误信息是一种很好的代码风格。比如，系统库函数 `os.Open` 在发生错误的时候不仅返回一个 `nil` 的指针，同时返回一个错误变量来描述出问题的原因。

为了方便，错误有一个专门的类型 `error`，它是一个内建的接口。

```go
type error interface {
    Error() string
}
```

包的创建者可以为自定义类型实现 `error` 接口，为调用者提供一些错误的上下文从而便于排查错误。就像刚刚提到的，系统库函数 `os.Open` 除了返回 `*os.File` 类型的值外，还返回了一个错误值；如果文件打开成功，错误值是 `nil`，如果文件打开失败，会返回 `os.PathError` 错误：

```go
// PathError records an error and the operation and
// file path that caused it.
type PathError struct {
    Op string    // "open", "unlink", etc.
    Path string  // The associated file.
    Err error    // Returned by the system call.
}

func (e *PathError) Error() string {
    return e.Op + " " + e.Path + ": " + e.Err.Error()
}
```

上面 `PathError` 的 `Error` 方法会产生下面类似的字符串提示：

```bash
open /etc/passwx: no such file or directory
```

上面的错误信息里面**包含了**出问题的①文件名、②操作以及③触发的操作系统错误，这个信息对找出问题是很有帮助的，即使函数被层层调用报出这个错误也很容易定位问题。相比较而言，这种示意的输出比 “找不到相关的文件或目录” 这种提示要有用的多。

大多数情况下，错误字符串最好能输出他们的源信息，比如什么操作导致的什么错误，比如哪个包的什么错误，等等。以包 `image` 为例，由未知格式导致的编码错误的字符串提示统一都是：“图片：未知的格式”。

如果调用者关注详细的错误类型，那就可以用 **类型 switch** 或者**类型断言**来查找特定的错误并抽取它的详细信息。对于下面代码中的 `PathErrors`，为了从操作失败中恢复，它可能包含了一个内部的 `Err` 字段：

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        // 没有空间了，删除一些临时文件获取空间，然后重试
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

第二个 `if` 语句是一个类型断言，如果断言失败了，`ok` 值就是 false，此时 `e` 的值是 `nil`；如果断言成功，`ok` 值是 true，此时意味着错误值的类型是 `*os.PathError`，接着 `e` 就可以做进一步的检查并执行对应的操作了。

## Panic（Panic）

报错最常见的方式是给调用者返回一个额外的 `error` 类型的值；典型的 `Read` 方法是一个很好的例子，它返回字节数的同时还返回一个 `error`类型的值。那么我们思考，如果错误没有被覆盖怎么办呢？这时候程序可能不能继续运行下去。

Go 的内建函数 `panic` 会创建一个运行时错误，并且终止程序的执行。这个函数可以接收一个任意类型的参数（通常是一个字符串），程序终止后会把这个参数打印出来。除此之外，还可以使用 `panic` 来表明代码里存在一些不应该发生的逻辑，比如存在一个无限循环逻辑，如下面的代码所示：

```go
// A toy implementation of cube root using Newton's method.
func CubeRoot(x float64) float64 {
    z := x/3   // Arbitrary initial value
    for i := 0; i < 1e6; i++ {
        prevz := z
        z -= (z*z*z-x) / (3*z*z)
        if veryClose(z, prevz) {
            return z
        }
    }
    // A million iterations has not converged; something is wrong.
    panic(fmt.Sprintf("CubeRoot(%g) did not converge", x))
}
```

上面的代码只是一个示例，真实的**库**代码逻辑中应该避免使用 `panic` 函数。假如问题可以被预知并被处理掉，比起终止整个程序更好的方式是让代码继续运行下去。在少有的一些情况下，使用 panic 能简化编码逻辑，比在计数例子的初始化过程中可以使用 `panic`：如果包不能成功启用，意味着可能发生了什么重大的错误，此时可以抛出 `panic`。下面代码示例中如果没有设置 `USER` 系统变量，就抛出 `Panic`：

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

## Recover（恢复）

在发生运行时错误时，比如越界访问切片，或者类型断言失败，`panic` 会被调用，此时当前函数的执行立即停止，并且开始展开 Goroutine 的堆栈，在这个过程中被推迟执行（defer）的函数会被依次执行。如果堆栈一直展开到达了 goroutine 的栈顶，代码就退出了。我们可以通过内建的 `recover` 函数来重新获取 goroutine 的控制权并且恢复其正常的执行。

调用 `recover` 可以停止堆栈的展开，并获取得到传递给 `panic` 的参数。因为在堆栈展开的过程中只有被 defer 的函数会被执行，因此 `recover` 只有放在被 defer 的函数中才有意义。

`recover` 的一个应用场景是：当服务器中有失败的 Goroutine 的时候，直接关闭对应的 Goroutine，避免影响其他正在执行的 Goroutine。

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)
    }
}

func safelyDo(work *Work) {
    defer func() {
        if err := recover(); err != nil {
            log.Println("work failed:", err)
        }
    }()
    do(work)
}
```

在上面的例子中，如果 `do(work)` 运行时出错（`panic`），结果会被记录下来，并且 Goroutine 会干净利落地退出而不会影响到其他的 Goroutine。对于上面的情况，在**推迟**函数中直接调用 `recover` 就可以完全处理相应的状况，不需要做其他的事情。

如果不是在被 defer 的函数中调用，`recover` 总是返回 `nil` 值；因此，被推迟执行的函数逻辑里可以调用使用了 `panic` 和 `recover` 的第三方库。比如，在 `safelyDo` 的推迟函数里可以在调用 `recover` 之前调用一个日志函数，日志相关的逻辑不会受 panicking 的状态的影响。

使用上面例子中的 `recovery` 的用法，`do` 函数（包含它调用的任何东西）可以通过调用 `panic` 干净利落地处理任何坏状况。我们可以把这种模式应用在复杂的软件中从而简化错误处理。让我们看一下理想状态下的 `regexp` 包，可以通过调用 `panic` 并传入一个本地的错误类型来解析错误信息（这里指的是**准确地解析**，继续看下面的解释）。下面的代码定义了 `Error` 类型， `error` 和 `Compile` 方法：

```go
// Error is the type of a parse error; it satisfies the error interface.
type Error string
func (e Error) Error() string {
    return string(e)
}

// error is a method of *Regexp that reports parsing errors by
// panicking with an Error.
func (regexp *Regexp) error(err string) {
    panic(Error(err))
}

// Compile returns a parsed representation of the regular expression.
func Compile(str string) (regexp *Regexp, err error) {
    regexp = new(Regexp)
    // doParse will panic if there is a parse error.
    defer func() {
        if e := recover(); e != nil {
            regexp = nil    // Clear return value.
            err = e.(Error) // Will re-panic if not a parse error.
        }
    }()
    return regexp.doParse(str), nil
}
```

如果 `doParse` 报错，恢复的代码块会把返回值 `regexp` 设置为 `nil`——被 defer 的函数可以修改命名的返回值。然后在 `err` 的赋值语句断言 `e` 是不是一个本地类型的 `Error`。如果它不是本地的 `Error`，类型断言会失败，从而造成一个新的 `panic`，因此堆栈会就像没有被中断过一样继续展开。这种检查意味着如果有未预知的事情发生，比如索引溢出，即使我们使用了 `panic` 和 `recover` 处理了解析错误的报错，代码依然会报错。

有了上面错误处理的逻辑以后，配合定义的 `error` 方法，同时简化了报错的方式，不用再想着手动处理堆栈的事情（因为 `error` 方法绑定到了类型上，而且与内建的 `error` 类型有相同的名字，因此使用起来也很优雅自然，）。

```go
if pos == 0 {
    re.error("'*' illegal at start of expression")
}
```

虽然 `panic/recover` 的这种模式很有用，但是应该只限定在一个包里使用。`Parse` 没有把 `panic` 暴露给自己的调用者，而是把内部的 `panic` 转换成了 `error` 值返回，这是一个可供参考的最佳实践。

需要注意，如果 `recover` 后又发生了 `panic`，那么这种写法会改变 `panic` 的值。不过幸运的是，在崩溃报告中原始报错和新的报错都会打印出来，因此导致错误的根源依然是可见的。如果想展示引发 `panic` 的原始值，可以写一点代码来过滤未知问题然后用原生的错误重新发起 `panic`，大家可以自己来实践一下。



## 【effective go 全篇翻译】