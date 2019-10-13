[effective go](https://golang.google.cn/doc/effective_go.html)

[本文的视频地址](https://www.bilibili.com/video/av70367389/)

# 控制结构
Go 语言中的控制结构和 C 语言有点像，但是也有很多不一样的地方：Go 语言 ① 没有 do 和 while 循环语句，只有一个通用的 for 循环语句；② switch 的用法比 C 语言中 switch 的用法更灵活；③ if 和 switch 可以像 for 语句那样接收一个初始化语句；④ break 和 continue 语句可以通过一个可选的标签来指定 break 和 continue 的位置；⑤ Go 语言可以根据数值的类型选择执行的逻辑分支；⑥ Go 还多出来一个新的控制结构 select，可以用来做多路通信分发器。和 C 语言相比， Go 的语法也有一些不同的地方：for, if, switch, select 等控制语句后面不需要小括号，且他们的执行体都必须用大括号括起来。

## If 结构

在 Go 语言中，简单的 if 语句如下：
```go
if x > 0 {
    return y
}
```

强制添加大括号可以逼迫大家把简单的 if 语句写成多行的这种样式，毫无疑问这种多行的样式读起来更容易理解，尤其当执行体内部包含 return 或 break 的时候尤其明显，这里照顾了代码的可读性。

鉴于 if 和 switch 语句都允许有一个初始化语句，因此可以经常看到类似下面的用法（创建了一个本地临时变量）：

```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

在 Go 的源码库中，可以经常看到这种情况：如果 if 的执行体以 break, continue, goto, 或者 return 结束，逻辑上来说 else 分支就没有任何作用了，就会被省略掉。

```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

下面的代码是一个很常见的用法示例，代码考虑了一系列的错误情况。在代码中，如果去除处理错误的语句，控制流能成功地从头执行到尾，这种代码就很容易阅读。同时因为错误处理的 if 执行体都以 return 结束，因此代码中没有必要出现 else 语句。

```go
f, err := os.Open(name)
if err != nil {
    return err
}
d, err := f.Stat()
if err != nil {
    f.Close()
    return err
}
codeUsing(f, d)
```

【也就是说，如果 os.Open 调用出错，整个函数会直接 return err 返回，后面的代码也就不会执行；如果 os.Open 调用不出错，代码不会进入到 if 的执行体，后面的代码顺理成章地被执行。如果 f.Stat() 出错，也会 return err 返回，后面的代码也不会执行；如果 f.Stat() 不出错，后面的代码会顺理成章地执行。】

## 重新声明和重新赋值

上一小节的最后一个例子很好地阐释了 `:=` 短声明语句的工作方式。短声明语句 f, err := os.Open(name) 声明了两个变量：f 和 err。紧接着后面的 d, err := f.Stat() 好像又声明了两个变量： d 和 err。需要注意 err 同时出现在了两个声明语句中。在 Go 中，这种“重复声明”是允许的：err 首先由第一个语句声明，在第二个语句中被重新赋值。这意味着调用 f.Stat 时使用了上面已经被声明的 err 变量，并且给了它一个新的值。


在 := 声明中，即使变量 v 已经被声明过也可以再次出现，只需满足：

* := 语法声明和已存在的 v 在同一个作用域内（如果 v 是在外面的作用域声明的，这种情况下会创建一个新的变量）；
* 初始化语句中的值可以赋值给 v，并且
* 在声明语句中至少有一个未声明过的新变量。

§（脚注）需要注意，在 Go 语言中，虽然函数（包括方法）的参数和返回值出现在函数执行体的大括号外面，但是它们的作用域是函数的整个执行体，因此也适用上面的这几个条件。

## For 语句

Go 语言中的 for 循环和 C 语言的 for 循环有类似的用法，但是也有不一样的用法。Go 中的 for 语句统一了 C 语言中的 for 和 while 循环，因此在 Go 中没有 do-while 循环这种用法。在 Go 中有三种形式的 for 循环，其中只有一种带分号，另外两种没有分号。

```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

可以通过短声明很容易地生成 for 循环中的索引变量。

如果在数组、切片、字符串、映射（map）上进行循环，或者循环读取信道中的元素，可以使用 range 语法来控制整个循环。

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

在 range 语法中如果只需要第一个元素的值，可以省略第二个变量：

```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

在 range 语法中，如果只需要第二个元素的值，可以使用空标识符（下划线）显式地丢弃第一个元素：

```go
sum := 0
for _, value := range array {
    sum += value
}
```

对于字符串来说，range 会做更多的事情，它会检索出以 UTF-8 编码的每个 Unicode 字节码（按 rune 进行遍历，而并不是按照字节进行遍历，所以可以看到下面的 position 输出不是连续的）。错误的编码会产生一个替代符文 U+FFFD，这个时候相应的索引会前进一个字节（跳过引起错误的字节）。（rune 是 Go 内建的一个类型，用来表示 Unicode 字节码，可以到[Go语言规范](https://golang.google.cn/ref/spec#Rune_literals)查看细节）。可以查看下面的例子：

```go
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```

上面的代码打印结果为：

```go
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```

最后，Go 语言中没有逗号操作符；符号 ++ 和 -- 是语句而不是表达式。因此，如果想在 for 循环中维护多个变量应该使用平行赋值语句，不能使用 ++ 和 -- 运算符，否则会编译报错。

```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```


## switch 语句

Go 里的 switch 比 C 语言中的 switch 更通用了。它的表达式不局限在常量，甚至可以不是整数。switch 的 case 语句会从上到下依次执行直到匹配到某个 case 为止。如果 switch 后没有跟任何的表达式，则默认是一个 true 值，这个时候 case 语句只要运算得到 true 值就会执行对应的逻辑。一般情况下可以选择把 if-else-if-else 写成 switch 的语句。

```go
func unhex(c byte) byte {
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

默认情况下 switch 不会执行多个 case（不会从一个 case 执行完以后再执行下一个 case），但是可以给 case 语句设置一个逗号分隔的列表。

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

和其他类 C 的编程语言不同，在 Go 语言中，break 语句可以用来提前终止 switch 语句。不过，有时候我们需要终止的可能是一个包围起来的循环而不是 switch 的逻辑，这种情况下我们可以在循环上定义一个标签，然后就可以通过标签标注 break 终止的是循环而不是 switch 语句了。下面的例子展示了两种用法（没有标签的 break 终止的是 switch 逻辑，有 Loop 标签的 break 终止的是 for 循环）：

```go
Loop:
	for n := 0; n < len(src); n += size {
		switch {
		case src[n] < sizeOne:
			if validateOnly {
				break
			}
			size = 1
			update(src[n])

		case src[n] < sizeTwo:
			if n+1 >= len(src) {
				err = errShortInput
				break Loop
			}
			if validateOnly {
				break
			}
			size = 2
			update(src[n] + src[n+1]<<shift)
		}
    }
```

当然，continue 表达式也可以设置一个可选的标签，不过 continue 只能用在循环中，不能用在 switch 中。

作为本小节的结尾，下面使用两个 switch 来比较两个字节切片：

```go
// Compare returns an integer comparing the two byte slices,
// lexicographically.
// The result will be 0 if a == b, -1 if a < b, and +1 if a > b
func Compare(a, b []byte) int {
    for i := 0; i < len(a) && i < len(b); i++ {
        switch {
        case a[i] > b[i]:
            return 1
        case a[i] < b[i]:
            return -1
        }
    }
    switch {
    case len(a) > len(b):
        return 1
    case len(a) < len(b):
        return -1
    }
    return 0
}
```

## 类型 switch

switch 语句还可以用来探索**接口变量**的动态类型。类型 switch 使用带括号的关键字 type 来对一个接口变量的类型进行断言。如果在 switch 表达式把断言的值赋给了一个变量，这个变量会保存每个 case 对应的类型的**值**。在这些 case 中一般会复用变量的名称，实际上会为每个 case 声明一个同名不同类的新变量。

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```
