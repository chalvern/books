[effective go](https://golang.google.cn/doc/effective_go.html)

# 数据

## 分配器 new

Go 语言有两个可以分配内存的命令，分别是内建的函数 `new` 和 `make`。他们做的事情不一样，能分配的类型也不一样，很容易搞混，不过 `new` 和 `make` 的使用规则还算容易记住。这一小节首先讨论 `new` 函数。`new` 是一个用来分配内存的内建函数，但是和其他语言中的 `new` 函数不同，Go 语言中的 `new` 函数只把内存块全部置零，除此之外不做任何其他的初始化操作。换句话说，`new(T)` 给类型为 `T` 的实体分配一块全是 0 的内存，然后返回这块内存的地址（返回值的类型为 `*T`）。在 Go 语言的术语里，`new` 返回类型 `T` 的指针，并且这个指针指向的是一块类型为 `T` 的全零值内存。

因为 `new` 返回的是全零值的内存地址，因此如果你的数据结构可以直接使用内存里的这些零值（不需要其他的初始化过程），使用 new 就很方便；这意味着通过 `new` 创建完实例马上就可以用它干活了（不需要其他的操作）。比如，`bytes.Buffer` 的文档中就注明了：`Buffer` 的零值是一个可以直接使用的空缓存。同样的，`sync.Mutex` 这个结构体并没有显式的构造器也没有相关的 `Init` 函数，因为它的零值就是一个没有上锁的互斥锁。

**“零值即可用”**的属性在工程上很有实践意义。考虑下面的类型：

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

有时候类型的零值不能直接使用，这时候就需要构造函数了，比如下面从 `os` 包里抽取出来的例子：

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

上面的代码显得很冗长很啰嗦，其实我们可以通过**复合字面表达式**来简化它。**复合字面表达式**在每次执行的时候都会创建一个新的实例，使用方式如下：
```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```

可以注意到，Go 语言中的函数可以返回临时变量的地址（这一点与 C 语言不同）；临时变量被函数返回后，它对应的内存会被保留（不会被立即回收掉），也就是说该局部变量对应的数据在函数返回后依然有效。实际上，每当取一个复合字面表达式的地址的时候，都会为一个新对象分配内存，因此可以进一步把上面的代码的最后两行改写成:

```go
    return &File{fd, name, nil, 0}
```

如上面所示的样式，复合字面表达式的字段必须要按定义时的顺序**依次**、且**全部**给出来。不过，如果通过 **“字段 : 值”** 的形式显式地指定元素值，就可以以任意顺序传入值了，此时如果有字段不指定默认值就会被设置为对应的零值，因此上面的代码还可以简化为（只有 `fd` 和 `name` 两个字段，因为 `dirinfo` 和 `nepipe` 均为零值）：

```go
    return &File{fd: fd, name: name}
```

作为一种限定，如果符合字面表达式中不包含任何的字段，如何字面表达式也会创建一个零值的实例，此时 `new(File)` 和 `&File{}` 就是等价的。

除了结构体，还可以通过复合字面表达式来创建数组、切片、映射（map），这个时候 **“字段标签”** 就变成了切片的索引或者映射的键。在下面的例子中，只要 `Enone`, `Eio`, 和 `Einval`值不一样，都可以进行正常的初始化。

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

继续内存分配的话题。内建函数 `make(T, args)` 和 `new(T)` 不一样：`make` 函数**只能**用来创建切片、映射（map） 和 信道，返回的是一个被初始化、类型为 `T` 的值（内存不是全零）。造成这种区别的原因是，切片、映射和信道 这三个类型的数据底层引用了其他的数据结构，而这些底层的数据结构在使用前必须先初始化才可以工作。比如，切片是一个**包含三个组件**的描述符，包含指向具体数据的指针（指向数组）、切片长度、切片容量。在这些组件被初始化之前，切片的值是 `nil`。对于切片、映射、信道来说，`make` 初始化了他们底层的数据结构继而他们的值才可以使用。比如：

```go
make([]int, 10, 100)
```

上面的语句初始化了一个包含 100 个整数的**数组**，同时创建了一个长度为 10、容量为 100 、指向**这个数组** 的前 10 个元素的切片（当创建切片的时候，其容量是可以省略的，可到“切片小节”查看更多内容）。作为对比，`new([]int)` 返回了一个新创建且被置零的切片的指针，指针指向的是值为 `nil` 的切片值。

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

**记住**，`make` 只可以作用于 切片、映射和信道，并且返回的是不是指针。如果想得到指针，可以使用 `new` 函数，或者显式地取变量的地址获得。

## 数组

在设计内存布局细节的时候使用**数组**可能会很方便，有时候数组还可以避免多余的内存分配过程；不过在 Go 语言中数组的主要作用是给切片提供底层的数据块，别急，下一小节讲的就是切片的内容。为了给讲切片做铺垫，这里简单介绍一下数组：

对于 Go 和 C 来说，数组的主要区别表现在它的工作方式。在 Go 语言中，

- 数组是值；如果把一个数组赋值给另一个数组会把所有的元素复制一份；
- 如果一个函数的参数是数组，调用函数的时候得到的是数组的完全拷贝而不是数组的指针；
- 数组的大小属于数组的类型的一部分，比如类型 `[10]int` 和 `[20]int` 不一样。


传数值的特性可能很有用，但是应用的代价也可能很高；如果想得到 C 语言中的数组那样的用法和效率，可以在 Go 中使用数组的指针，如下面的示例代码：

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
但是在 Go 中这种风格的使用很不常见，更多情况是使用切片来模仿类似的行为。

## 切片
切片封装了数组，并为操作数据序列提供了更通用、更有效、也更方便的操作接口（方法）。除了具有明确维数的场景（比如线性代数中的变换矩阵），数组相关的操作，在 Go 中大部分情况会使用切片，而不会使用数组。

切片保存了底层数组的引用，如果把一个切片赋值给另一个切片，这两个切片会指向同一个数组。如果函数把切片作为参数传入，然后在函数内部修改了切片的某个元素，函数的调用者将会感知到这种变化，其效果就好像是传入了数组的指针一样。因此，`Read` 函数可以接收一个切片类型的参数，这样就能避免传入一个指针变量和一个表示数量的变量；对切片而言，切片的长度是指能够读取的元素的最大数量。下面是包 `os` 中类型 `File` 的 `Read` 方法的声明：

```go
func (f *File) Read(buf []byte) (n int, err error)
```

`Read` 方法返回一个读取的字节数量和一个错误值（存在错误的情况）。如果只想读取一个很大的缓存 `buf` 的前 32 个字节的数据，可以把 `buf` **切**一部分出来：

```go
    n, err := f.Read(buf[0:32])
```

这种切片的方式很常见且很高效。如果不考虑效率，下面的代码也可以读取缓存的前 32 个字节：

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

因为切片里的数据主要保存在底层数组里，而数组一旦声明后其长度是无法改变的，因此只要不超出底层数组的容量，切片的长度可以任意改变。可以通过内建函数 `cap` 来得到切片的**容量**，即当前切片能达到的最大长度。虽然切片有容量的说法，但并不意味着切片的长度不能大于这个容量；内建的 `append` 函数可以用来给切片追加元素，如果在追加元素的过程中元素的个数超过了切片的容量，切片会被重新进行内存分配，并返回一个新的切片（整个过程中涉及到了元素的复制）。如果切片值为 nil，使用 `len` 和 `cap` 也是合法的，都会得到 `0` 的结果。

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

上面的 `Apend` 函数最后返回了一个切片，这个返回是必要的；虽然 `Append` 会修改 `slice` 的元素，但是 slice （运行时的数据结构里包含了底层数组的指针、切片的大小和容量）是以传值的方式到达函数内部的，如果切片的底层数组指针、大小、容量发生了改变必须通过返回值通知调用方，否则调用方将感知不到这种变化。

给切片追加元素的场景非常普遍，Go 语言中有一个内建的函数 `append`，它的实现和上面的函数的逻辑类似。不过如果想理解 `append` 函数设计的更多细节，我们还需要更多的知识储备，后面的小节再继续聊。


## 两维切片

Go 的数组和切片都是一维结构。如果想创建二维数组或者而且切片，就必须定义数组的数组或者切片的切片，比如下面的定义：

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

因为切片的长度可变，因此内部的切片可以有不同的长度，这种情况还是很常见的，比如下面 `LinesOfText` 的例子，其中每行的长度都是不一样的。

```go
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

有时候需要声明一个二维的切片，比如当需要扫描每行像素的时候。有两种方式来实现这个目的。其中一种方式是声明独立的切片，还有另一种方式是先分配一个独立的数组然后把每个切片映射到这个数组。具体使用哪种方式取决于你的需求：假如切片的长度可能会变化，为了避免像素行与行之间的数据覆盖，此时应该选择独立声明的方式；如果切片的长度是固定的，这时候就可以先分配一个大的切片然后构造各个小的切片对象，因为只有一次内存分配的过程（构造大切片），这种方式会更高效。为了说明问题，下面给出了两种方式的用法，首先是每次一行的方式：

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

下面是一次分配，然后对应各个子切片：

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
