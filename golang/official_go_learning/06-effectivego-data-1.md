[effective go](https://golang.google.cn/doc/effective_go.html)

[todo]
[本文的视频地址](https://www.bilibili.com/video/av71150528) 

# 数据

## 分配器 new

Go 语言有两个可以分配内存的命令，分别是内建的函数 `new` 和 `make`。他们做的事情不一样，能分配的类型也不一样，很容易搞混，不过 `new` 和 `make` 的使用规则还算容易记住。这一小节首先讨论 `new` 函数。`new` 是一个用来分配内存的内建函数，但是和其他语言中的 `new` 函数不同，Go 语言中的 `new` 函数只把内存块全部置零，除此之外不做任何其他的初始化操作。换句话说，`new(T)` 给类型为 `T` 的实体分配一块全是 0 的内存，然后返回这块内存的地址（返回值的类型为 `*T`）。在 Go 语言的术语里，`new` 返回类型 `T` 的指针，并且这个指针指向的是一块类型为 `T` 的全零值内存。

因为 `new` 返回的是全零值的内存地址，因此如果你的数据结构可以直接使用内存里的这些零值（不需要其他的初始化过程），使用 new 就很方便；这意味着通过 `new` 创建完实例马上就可以用它干活了（不需要其他的操作）。比如，`bytes.Buffer` 的文档中就注明了：`Buffer` 的零值是一个可以直接使用的空缓存。同样的，`sync.Mutex` 这个结构体并没有显式的构造器也没有相关的 `Init` 函数，因为它的零值就是一个没有上锁的互斥锁。

**“零值即可用”**的属性很有工程实践意义。考虑下面的类型：

```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

`SyncedBuffer` 类型的值一旦声明（被分配到内存）就可以使用了，不需要等其他的初始化操作。下面的代码片段里，`p` 和 `v` 都可以直接工作，

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

## 构造函数与复合字面表达式

有时候类型的零值不能直接使用，这时候就需要构造函数了，比如下面从 `os` 包里抽取出来的一个例子：

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```

上面的代码显得有点冗长，我们可以通过**复合字面表达式**来简化它。**复合字面表达式**在每次执行的时候都会创建一个新的实例，使用方式如下：
```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```

可以注意到，在 Go 语言中可以返回临时变量的地址（与 C 语言不同）；被函数返回后，临时变量的内存会被保留而不会被立即回收掉，也就是说该局部变量对应的数据在函数返回后依然有效。实际上，每当取复合字面表达式的地址时，都会为一个新对象分配内存，因此我们可以把上面例子中最后两行代码改写成:

```go
    return &File{fd, name, nil, 0}
```

复合字面表达式中的字段必须要按定义时的顺序**依次**、**全部**给出来。不过，如果通过 **“字段 : 值”** 的形式显式地指定元素及其值，初始化的时候就可以以任意顺序传入值，此时如果有字段不指定值默认会被设置为这个字段对应的零值，因此上面的代码还可以简化为（只有 `fd` 和 `name` 两个字段）：

```go
    return &File{fd: fd, name: name}
```

作为一种限定，假如符合字面表达式中不包含任何的字段，它也会创建零值的实例，这个时候 `new(File)` 和 `&File{}` 是等价的。

还可以通过复合字面表达式来创建数组、切片、映射（map），此时它的 “字段标签” 就变成了切片的索引或者映射的键。在下面的例子中，只要 `Enone`, `Eio`, 和 `Einval`值不一样，都可以进行正常的初始化。

```go
// 可以定义下面的常量，此时
// a 被初始化为长度为 31 的数组，
// s 被初始化为长度 31 的切片
// m 被初始化为长度为 3 的映射（map）
// const (
// 	Enone  = 1
// 	Eio    = 2
// 	Einval = 30
// )
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

## 分配器 make

回到内存分配。内建函数 `make(T, args)` 的作用于 `new(T)` 的不同。`make` 函数只能用来创建切片、哈希 map 和 信道，并且返回一个类型为 `T` 且被初始化的值（内存不是全零）。区别的原因是，这三个类型的数据底层引用了其他的数据结构，而这些数据结构在使用前必须先初始化才可以使用。比如，切片是一个**三组件**的描述符，包含一个指向具体数据的指针（指向数组）、切片长度、切片容量（TODO 是不是能说明切片的线程安全性？）。在这些组件被初始化之前，切片的值是 `nil`。对于切片、map 和信道来说，`make` 初始化了他们底层的数据结构让他们的值可以使用。比如：

```go
make([]int, 10, 100)
```

上面的语句初始化了一个 100 个整数的数组，同时创建了一个长度为 10、容量为 100 、指向数组的前 10 个元素的切片（当创建切片的时候，其容量是可以省略的，见切片小节查看更多内容）。作为对比，`new([]int)` 返回了一个新创建且被置零的切片的指针，指向的是值为 `nil` 的切片值。

下面的例子描述了 `new` 和 `make` 之间的区别。

```go
var p *[]int = new([]int)       // allocates slice structure; *p == nil; rarely useful
var v  []int = make([]int, 100) // the slice v now refers to a new array of 100 ints

// Unnecessarily complex:
var p *[]int = new([]int)
*p = make([]int, 100, 100)

// Idiomatic:
v := make([]int, 100)
```

记住，`make` 只可以作用于 map、切片和信道，并且返回的不是指针。如果想得到指针，可以使用 `new` 或者显式取变量的地址获得。

## 数组

在设计内存的详细布局的时候**数组**会很有用，有时候数组还可以避免内存分配；不过数组的主要作用是给切片提供底层的数据块，这将在下一节讲。为了给切片做铺垫，这里简单介绍一下数组：

数组在 Go 和 C 中的主要区别是它的工作方式。在 Go 语言中，

- 数组是值；如果把一个数组赋值给另一个数组会把所有的元素复制一份；
- 如果一个函数的参数是数组，调用函数的时候得到的是数组的完全拷贝而不是它的指针；
- 数组的大小属于它的类型的一部分，类型 `[10]int` 和 `[20]int` 不一样。


数组可能很有用，但是使用的代价也很高；如果希望得到 C 语言中的那种行为和效率，可以使用数组的指针，如下面的示例代码：

```go
func Sum(a *[3]float64) (sum float64) {
    for _, v := range *a {
        sum += v
    }
    return
}

array := [...]float64{7.0, 8.5, 9.1}
x := Sum(&array)  // Note the explicit address-of operator
```
但是在 Go 中这种风格的使用很不常见，更多情况是使用切片。


## 切片
切片封装了数组，并给出操作一个数据序列的更通用、更有效也更方便的接口。除了具有明确维数的项（比如变换矩阵），在 Go 中大部分都是使用切片来进行数组编程，不会直接使用数组。

切片保存了底层数组的引用，如果把一个切片赋值给另一个切片，两个切片会指向同一个数组。如果函数把切片作为参数，且在函数内部修改了切片的某个元素，调用者将会感知到这种变化，效果就好像是传入了底层数组的指针一样。因此，`Read` 函数可以接收一个切片类型的参数而不再需要指针和数量；切片的长度是指能够读取的元素的最大数量。下面是包 `os` 中类型 `File` 的 `Read` 方法的声明签名：

```go
func (f *File) Read(buf []byte) (n int, err error)
```

`Read` 方法返回了读取的字节数量和一个错误值（如果存在错误）。如果想读取一个很大的缓存 `buf` 的前 32 个字节的数据，可以把缓存**切**一部分出来：

```go
    n, err := f.Read(buf[0:32])
```

这种切片的方式是很常见且很有效率的。实际上，除去效率的考虑，下面的代码也可以读取缓存的前 32 个字节：

```go
    var n int
    var err error
    for i := 0; i < 32; i++ {
        nbytes, e := f.Read(buf[i:i+1])  // Read one byte.
        n += nbytes
        if nbytes == 0 || e != nil {
            err = e
            break
        }
    }
```


只要没有超出底层数组的限制，切片的长度可能会发生变化；只需把新切片赋值给同一个切片变量。可以通过内建函数 `cap` 来得到切片的**容量**，这个值表示当前切片能达到的最大长度。有一个专门的函数用于给切片追加数据（内建的 `append` 函数）。如果在追加过程中元素个数超过了切片的容量，切片会被重新进行内存分配，然后返回最终结果的切片（里面涉及元素的复制）。如果切片值为 nil，使用 `len` 和 `cap` 也是合法的，且会得到 0 值。

```go
func Append(slice, data []byte) []byte {
    l := len(slice)
    if l + len(data) > cap(slice) {  // reallocate
        // Allocate double what's needed, for future growth.
        newSlice := make([]byte, (l+len(data))*2)
        // The copy function is predeclared and works for any slice type.
        copy(newSlice, slice)
        slice = newSlice
    }
    slice = slice[0:l+len(data)]
    copy(slice[l:], data)
    return slice
}
```

上面的源码中，当使用 `Apend` 函数的时候最后必须要返回一个切片，虽然 `Append` 会修改 `slice` 的元素，但是 slice （运行时数据结构包含了指针、大小、容量）是以传值的方式到达函数的，切片内在的数组指针、大小、容量改变后必须通过返回值通知调用方。


给切片追加元素的点子非常有用，内建的函数 `append` 的实现就是同样的逻辑。如果想理解 `append` 函数的设计，我们还需要更多的知识，后面的小节再继续聊。


## 两维切片
Go 的数组和切片都是一维结构。如果想创建一个等效的二维数组或者切片，必须定义一个数组的数组或者切片的切片，就像下面的定义：

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

因为切片的长度可变，因此高维数组的切片可以有不同的长度，这种情况还是很常见的，比如下面 `LinesOfText` 的例子：其中每行有独立的长度。

```go
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

有时候分配一个二维的切片是必要的，比如当需要扫描每行的像素的时候就需要二维切片。有两种方式来完成这个目的。其中一种方式是分配独立的切片，另一种方式是分配一个独立的数组然后把每个切片映射到这个数组。使用哪种方式取决于你的需求。如果切片可能会增长或者收缩，为了避免行与行之间的数据应该分配独立的切片；否则就可以一次分配然后构造各个对象，这种方式会更高效。为了说明问题，下面给出了两种方式的用法，首先是每次一行的方式：

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

然后分配一次，然后把每行对应起来：

```go
// Allocate the top-level slice, the same as before.
picture := make([][]uint8, YSize) // One row per unit of y.
// Allocate one large slice to hold all the pixels.
pixels := make([]uint8, XSize*YSize) // Has type []uint8 even though picture is [][]uint8.
// Loop over the rows, slicing each row from the front of the remaining pixels slice.
for i := range picture {
	picture[i], pixels = pixels[:XSize], pixels[XSize:]
}
```
