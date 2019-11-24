
[effective go](https://golang.google.cn/doc/effective_go.html)

[本文的视频地址](https://www.bilibili.com/video/av75898542)

# Concurrency（并发）

## Share by communicating（以通信的方式共享变量）

并发编程是个很大的话题，本小节只考虑 Go 相关的特性。

在很多环境中，并发编程都不容易；为了确保正确访问共享变量，里面需要注意很多的实现细节。Go 鼓励一种不同的编程方式：避免直接在不同的线程间共享变量，而是通过信道来共享变量的值。在任何时间点只有一个协程有权访问变量的值；在设计上就杜绝了数据竞争的情况。为了鼓励这种共享变量的方式，它还有一句口号：**不要通过共享内存进行通信，相反，通过通信来共享内存**。

当然，这种“以通信的方式共享变量”的方式也不是万能的。比如，引用计数的最佳实现方式依然是给一个整数添加一个互斥锁的方式。但是作为一种高级别方式，使用信道来控制共享变量的访问能够让开发者更容易写出简明正确的代码。

那么该怎么思考这个模型呢？假设我们有一个独立线程的程序，它只运行在一个 CPU 核上，这种情况下显然没有必要进行任何同步。现在思考有另一个类似的实例，它也是独立线程的，而且也只允许在一个 CPU 核上，这个实例也不需要任何的同步。然后，我们让这两个线程进行通信；如果通信方式是同步的，那么这两个独立的线程之间就不需要其他的同步了。Unix 里的 Pipline（管道） 模式就属于类似的模型。虽然 Go 解决并发的解决方案源自 Hoare 的CSP（通信序列过程）模型，但是它也可以看做是 Unix 管道的带类型安全限定的通用版本。

## Goroutines（Go 协程）

之所以叫 **goroutines（Go协程）** ，是因为已有的术语——线程、协程、进程等——和要传达的含义并不完全匹配。每个 Go协程 都包含同一个简单的模型：首先它是一个函数，并且它和其他的 Go协程 在同一个地址空间同时运行。Go协程 是轻量级的，消耗的资源几乎只包含分配堆栈空间花费的那一点资源；同时因为它的堆栈初始化时很小，且总是依照实际需要来增加（或释放）堆存储，它也是很廉价的（意味着我们可以很容易地创建成千上万的 goroutines）。

Go协程 多路复用多个操作系统线程。也就是说，假设有 m 个 Go协程 n 个系统线程，这 m 个 Go协程 里的任意一个都可能在 n 个系统线程的任意一个上面执行。因此如果某个 Go协程 因为 I/O 调用阻塞了，其他的协程可以继续运行。Go 运行时在这里的设计隐藏了很多线程创建和管理的复杂度。

Go协程的创建也很简单，只需要在函数/方法的前面加一个关键词 `go` 就可以让这个函数/方法在一个新的 goroutine 中运行。当函数/方法调用结束时，goroutine 会静默退出。（这里有点类似于 Unix Shell 中的 `&` 符号）

```go
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

函数声明语法可以配合 goroutine 的声明使用。

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

在 Go 中，**函数即闭包：在实现上，如果某个变量被函数引用了，那么它就会与函数一起存活**。

因为上面的例子里函数都没有标识自己什么时候退出，我们也就无法知道它们是不是把任务完成了，因此没有太大的实践意义。为此需要引入信道。

### Channels（信道）

和映射一样，信道通过 `make` 来分配内存，返回的结果等效于是对底层数据结构的引用。在使用 `make` 时如果传入了一个可选的整数，会创建带缓存的信道；如果不传默认就是 0，此时创建的是一个没有缓存的信道或者称为同步信道。

```go
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

如果两个 Go协程 通过没有缓存的信道进行通信，这个没有缓存的信道同时可以做一些同步的事情，从而保证两个 Go协程 运行在已知的状态。

有很多应用信道的惯例，我们可以从下面的例子开始。前面小节我们在后台启用了一个排序例程，但是无法知道这个例程什么时候执行完毕；信道可以允许我们启动一个 Goroutine 并显式地等待相应的例程结束。

```go
c := make(chan int)  // Allocate a channel.
// Start the sort in a goroutine; when it completes, signal on the channel.
go func() {
    list.Sort()
    c <- 1  // Send a signal; value does not matter.
}()
doSomethingForAWhile()
<-c   // Wait for sort to finish; discard sent value.
```

在接收到数据前接收器（比如上例中 `<-c` 语句）会一直阻塞在那里。对于一个没有缓存的信道来说，在接收器**接收完成**之前发送器也会一直阻塞在那里（属于内存模型的一部分）。如果信道有空闲的缓存空间，发送器只在值拷贝到缓存区之前是阻塞的；如果缓冲区满了，发送器会一直等待，直到有接收器消费缓存中的值。

带缓存的信道可以作为信号量来使用，比如可以用它来控制吞吐量。在下面的例子里，过来的请求被传到 `handle`，然后 `handle` 会首先给一个信道塞一个值（相当于占坑），接着处理这个请求，最后再从信道中消费一个值释放“信号量”从而给下一个消费者腾出资源。这里信道的缓冲区容量限定了 `process` 的并发数量。

```go
var sem = make(chan int, MaxOutstanding)

func handle(r *Request) {
    sem <- 1    // Wait for active queue to drain.
    process(r)  // May take a long time.
    <-sem       // Done; enable next request to run.
}

func Serve(queue chan *Request) {
    for {
        req := <-queue
        go handle(req)  // Don't wait for handle to finish.
    }
}
```


一旦 `MaxOutstanding` 个处理器运行 `process`，任何其他的 Go协程 尝试发送数据到信道的行为都会被阻塞，直到某个 `handle` 结束并消费了信道中的数据。

虽然很优雅，不过上面的设计是有问题的：虽然控制了最多只能有 `MaxOutstanding` 个 Go协程 同时执行，但是 `Serve` 会给每个进来的请求都创建一个新的 Go协程。也就是说，只要请求进来的够快，程序会无限消耗资源。我们可以通过修改 `Serve` 函数限定 goroutine 的创建数量来解决这个缺陷。下面是一个显而易见的解决方案（不过隐藏了一个 bug 需要修复）：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func() {
            process(req) // Buggy; see explanation below.
            <-sem
        }()
    }
}
```

上面代码的 bug 发生在 Go 的 `for` 循环：**循环变量在每次迭代时会被复用**，因此 `req` 变量在各个 Go协程 之间是共享的。由于上面的例子里我们不希望 `req` 被共享，因此需要给每个 Go协程 传一个唯一的 `req`。下面的代码给每个 Go协程 的闭包传递了 `req` 作为其参数，因为函数参数是传值的（大家可以回顾函数那一节的内容），因此避免了上面的 bug：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        sem <- 1
        go func(req *Request) {
            process(req)
            <-sem
        }(req)
    }
}
```

可以把上面的代码和前面带 `handle` 函数的代码比较，看看闭包是如何声明并运行的。其实上面的 bug 还有一个解决方案，就是每次循环时创建一个同名的临时变量，如下面的代码所示：

```go
func Serve(queue chan *Request) {
    for req := range queue {
        req := req // Create new instance of req for the goroutine.
        sem <- 1
        go func() {
            process(req)
            <-sem
        }()
    }
}
```

或许 `req := req` 的写法很怪异，但是在 Go 中这是合法的惯用写法。通过这种方法我们可以得到一个同名的新变量，通过刻意覆盖本地的循环变量的方式给每个 Go协程 提供了唯一的变量（而不是共享的 `req` 变量）。

接下来让我们回到写服务器常见的资源管理的问题。上面讨论的例子里，另一种资源管理的好方法是初始化固定数目的 `handle` Go协程 ，所有这些 Go协程 从信道中读取请求的数据；这种情况下 Go协程 的数目限定了同时执行 `process` 的并发数量。我们也可以给 `Serve` 函数接收一个信道，并在这个信道上接收退出信号。下面的代码示例展示了上面提到的两个技术点：

```go
func handle(queue chan *Request) {
    for r := range queue {
        process(r)
    }
}

func Serve(clientRequests chan *Request, quit chan bool) {
    // Start handlers
    for i := 0; i < MaxOutstanding; i++ {
        go handle(clientRequests)
    }
    <-quit  // Wait to be told to exit.
}
```

### Channels of channels（信道中的信道）

Go 最重要的特性之一就是：**信道属于第一级类型**。信道可以像其他的类型那样初始化和传递，这个特性的一种应用是实现并行安全的多路复用模式。

在前面的例子中，`handle` 对请求来说是一个理想的处理器，但是我们并没有提及它处理的 `Request` 类型的细节。如果 `Request` 类型包含一个用来专门接收响应的信道，那么每个客户端就都可以提供自己的响应信道从而获取响应。下面给出类型 `Request` 的一种定义：

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

在 `Request` 中定义了一个函数一个参数列表，同时还定义了一个信道来接收响应。

```go
func sum(a []int) (s int) {
    for _, v := range a {
        s += v
    }
    return
}

request := &Request{[]int{3, 4, 5}, sum, make(chan int)}
// Send request
clientRequests <- request
// Wait for response.
fmt.Printf("answer: %d\n", <-request.resultChan)
```

在服务器端，只需要修改处理器函数 `handle`，代码如下所示：

```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

为了让功能更丰满，上面的代码显然还有很多东西可以做，但是作为一个无锁的限流、并发、非阻塞的 RPC 系统的框架的原型，已经能说明一些问题了。

### Parallelization（并行）

Go协程和信道的另一个应用场景是在多 CPU 核心上进行并发运算。如果某个运算可以被分解成为同步运行的独立子过程，它就可以进行并行计算，其中当某个子过程（Go协程）结束后通过信道来通知它的结束。

假设我们有一个耗时的向量运算，向量中每个值的运算都是独立的，如下所示：

```go
type Vector []float64

// Apply the operation to v[i], v[i+1] ... up to v[n-1].
func (v Vector) DoSome(i, n int, u Vector, c chan int) {
    for ; i < n; i++ {
        v[i] += u.Op(v[i])
    }
    c <- 1    // signal that this piece is done
}
```

我们可以把循环中能够独立出来的逻辑拆分开，让每个 CPU 各自去执行。这些拆分出来的逻辑可以以任意的次序执行，在启动所有的 Go协程 后我们只需要对完成信号的数量进行计数（读取所有信道中的值），代码如下所示：

```go
const numCPU = 4 // number of CPU cores

func (v Vector) DoAll(u Vector) {
    c := make(chan int, numCPU)  // Buffering optional but sensible.
    for i := 0; i < numCPU; i++ {
        go v.DoSome(i*len(v)/numCPU, (i+1)*len(v)/numCPU, u, c)
    }
    // Drain the channel.
    for i := 0; i < numCPU; i++ {
        <-c    // wait for one task to complete
    }
    // All done.
}
```

相对于为 numCPU 创建常量的值，我们可以通过运行时来准确获取 CPU 的数量。函数 `runtime.NumCPU` 返回硬件 CPU 的核心数量，因此我们可以写为 `var numCPU = runtime.NumCPU()`。

```go
var numCPU = runtime.NumCPU()
```

Go 中还有一个函数 `runtime.GOMAXPROCS`，可以用它来报告 Go 程序可以并发运行的核心数量。它的默认值是 `runtime.NumCPU`，开发者可以通过设置相同名称的环境变量，或者通过显式调用这个函数（传一个正整数）来覆盖这个值。如果调用这个函数时传入的是 0，会得到这个值当前的大小。所以，我们还可以向下面这样写：

```go
var numCPU = runtime.GOMAXPROCS(0)
```

这里需要注意，我们不要混淆并发和并行的概念。并发是指把代码构造为独立的可执行模块；并行是指高效地在多 CPU 核心上同时执行。尽管 Go 的并发特性可以让构建并行计算变得简单，但是 Go 毕竟是一个并发的语言，它并不是一个并行语言，而且并不是所有的并行问题都契合 Go 的模型。可以查看 [这里](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html)查看更多相关的讨论。

### A leaky buffer（缓存漏斗）

有时候并发编程模型也可以让非并发的想法更容易表达。下面是从 RPC 包抽象出来的一个例子。客户端对应的 Go协程 循环从某些源中（可能是网络）接收数据，为了避免重复分配和回收缓存，它保存了一个空闲列表，并用一个带缓存的信道承载它。如果信道是空的，就分配一个新的缓存，否则从信道中获取一个已有的缓存。消息的缓存一旦就绪，就会被发送到 `serverChan` 从而发给服务端。

```go
var freeList = make(chan *Buffer, 100)
var serverChan = make(chan *Buffer)

func client() {
    for {
        var b *Buffer
        // Grab a buffer if available; allocate if not.
        select {
        case b = <-freeList:
            // Got one; nothing more to do.
        default:
            // None free, so allocate a new one.
            b = new(Buffer)
        }
        load(b)              // Read next message from the net.
        serverChan <- b      // Send to server.
    }
}
```

服务端对应的 Go协程 循环获取从客户端传递过来的每个消息，处理完成后把对应的缓存释放并反还给空闲列表。

```go
func server() {
    for {
        b := <-serverChan    // Wait for work.
        process(b)
        // Reuse buffer if there's room.
        select {
        case freeList <- b:
            // Buffer on free list; nothing more to do.
        default:
            // Free list full, just carry on.
        }
    }
}
```

上面的代码通过几行代码构建了一个漏斗桶的空闲列表。整体来看就是，客户端试图从 `freeList` 获取缓存，如果没有可用的缓存它就分配一个新的缓存。服务端用完缓存后把 `b` 返回给 `freeList`，如果空闲列表满了，缓存会被丢弃并最终被垃圾回收器回收掉（在 `select` 中 `default` 的分支会在没有其他的 case 匹配的时候执行，也就意味着上面代码中 `select` 语句不会阻塞）。