# 控制结构
Go 中的控制结构和 C 语言有点像但是也有很不一样的地方：在 Go 语言中没有 do 和 while 循环，只有一个通用的 for 循环；switch 相对更灵活；if 和 switch 可以像 for 语句那样接收一个可选的初始化语句；break 和 continue 语句可以通过一个可选的标签来指定 break 或 continue 的位置；可以根据类型选择执行的逻辑分支；多出来一个新的控制结构 select，可以用来做多路通信分发器。和 C 语言相比， Go 的语法也有一些不同的地方：Go 语言中没有括号，且他们的执行体都必须以大括号进行分割。

## If 结构

在 Go 语言中，简单的 if 语句如下：
```go
if x > 0 {
    return y
}
```

强制添加大括号可以促使大家把简单的 if 语句写成多行的样式，无疑这种多行的样式会更好看，尤其当执行体内部包含 return 或 break 这种控制语句的时候尤其明显。

鉴于 if 和 switch 中允许有一个初始化语句，因此可以经常看到类似下面的用法（创建了一个本地临时变量）：

```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

在 Go 的库源码中，可以很明显看到这种情况，如果 if 的执行体以 break, continue, goto, 或者 return 结束，逻辑上没任何作用的 else 就会被省略掉。

```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

上面是一个很常见的例子，代码必须要考虑一系列的错误情况。如果去除那些错误处理的情况，控制流成功地贯穿页面，代码很容易阅读。同时因为错误处理的那个 if 执行体以 return 结束，因此代码中没有必要出现 else 语句。【也就是说，如果 os.Open 调用出错，整个函数会直接 return err 返回，后面的代码（codeUsing）也就不会执行；如果 os.Open 调用不出错，代码不会进入到 if 的执行体，后面的代码顺理成章地被执行】

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

## 重声明和重赋值

旁白：上一小节的最后一个例子很好地阐释了 := 短声明语句的工作方式。声明语句 f, err := os.Open(name) 声明了两个变量：f 和 err。紧接着后面的 d, err := f.Stat() 貌似又声明了变量 d 和 err。但是需要注意 err 出现在了两个声明语句中。在 Go 中，这种重复是允许的：err 首先由第一个语句声明，在第二个语句中被重新赋值。这意味着调用 f.Stat 时使用了上面已经被声明的 err 变量，然后又给了它一个新的值。


在 := 声明中，即使变量 v 已经被声明过也可以再次出现，只需满足：

* := 语法声明和已存在的 v 在同一个作用域内（如果 v 是在外面的作用域声明的，这种情况下会创建一个新的变量）；
* 初始化语句中的值可以赋值给 v，并且
* 在声明语句中至少有一个未声明过的新变量。

§（脚注）需要注意，在 Go 语言中，虽然函数参数及其返回值出现在函数执行体的花括号外面，但是它们的作用域是函数的整个执行体。

## For 语句

Go 中的 for 循环和 C 语言的类似，但是也不完全一样。for 统一了 for 和 while 循环，因此在 Go 中没有 do-while 循环。在 Go 中有三种形式的 for 循环，其中只有一种带分号。

```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

可以通过短声明很容易地生成 for 循环中的索引变量。

如果在数组、切片、字符串、哈希 map 上进行循环，或者循环读取信道中的元素，可以使用 range 语法来控制整个循环。

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

在 range 语法中，如果只需要第二个元素的值，可以使用空标识符（下划线）丢弃第一个元素：

```go
sum := 0
for _, value := range array {
    sum += value
}
```

对于字符串来说，range 会做更多的事情，它会检索出以 UTF-8 编码的每个 Unicode 字节码。错误的编码会产生一个替代符文 U+FFFD，此时相应的索引会前进一个字节（跳过引起错误的字节）。（rune 是 Go 内建的一个类型，用来表示 Unicode 字节码，可以到相关的章节查看细节）。具体的可以查看下面的例子：

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

Go 没有逗号操作符；符号 ++ 和 -- 是状态表达式而不是值表达式。因此，如果想在 for 循环中维护多个变量应该使用平行赋值语句（此时不能使用 ++ 和 -- 运算符）。

```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```


## switch 语句

Go 里的 switch 比 C 语言中的更通用。它的表达式不必是常量，甚至可以不是整数，它的 case 会从上到下依次执行直到匹配到某个 case 为止。如果 switch 后没有跟任何的表达式，则默认是一个 true 值。一般情况下可以选择把 if-else-if-else 写成 switch 的语句。

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

默认情况下 switch 不会执行多个 case（不会从一个 case 执行完以后再执行下一个 case），但是可以在一个 case 中一个由多个逗号分隔的列表。

```go
func shouldEscape(c byte) bool {
    switch c {
    case ' ', '?', '&', '=', '#', '+', '%':
        return true
    }
    return false
}
```

不像其他类 C 的编程语言那样常见，在 Go 语言中，break 语句可以用来提前终止 switch 语句。但是，有时候我们需要终止的是一个包围起来的循环而不是 switch 逻辑，这种情况下我们可以在循环上定义一个标签，然后就可以通过标签标注 break 终止那个循环了。下面的例子展示了两种应用：

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

当然，continue 表达式也可以设置一个可选的标签，不过 continue 只能用在循环中。

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

switch 语句还可以用来探索接口变量的动态类型。类型 switch使用带括号的关键字type来对一个变量的类型进行断言。如果在 switch 表达式中声明了变量，这个变量会保存每个 case 对应的类型的值。在这些 case 中一般会复用名称，实际上会为每个 case 声明一个同名不同类的新变量。

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
