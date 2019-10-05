[effective go](https://golang.google.cn/doc/effective_go.html)

# 简介
Go 是一个新的编程语言。虽然它从既有的编程语言中借鉴了很多理念，但是它也有很多与众不同的特性，这使得 Go 和其他的编程语言区别开来。如果一个 Java（C++） 写的应用应用直接按照 Java（C++） 的代码逻辑转译成 Go 语言的应用程序，最后很可能得不到一个满意的效果——毕竟 Java 应用程序是用 Java 写的，里面包含了太多Java的东西，而 Java 跟 Go 比 少了很多特性，因此直接转译肯定是不合适的。换个角度来思考，从 Go 的视角分析问题并据此设计软件，最后也会得到一个独特的架构方案，此时如果把这个 Go 应用程序转译成 Java 语言等其他语言的应用程序肯定也不容易。换句话说，为了写好 Go 语言的程序，学习它的语言特性和语言规范是很重要的。而除了必须要遵循或参考的语言特性和语言规范，了解 Go 语言编程的一些人为的约定也很重要，比如命名、格式、程序构建等相关的内容，这样你写出来的程序就更容易被其他开发者理解，从而便于多人协作进行开发。

本文档给出了一些 Go 语言的编码技巧。遵从这些技巧可以编写出清晰、规范的 Go 语言程序。这篇文档是“[语言特性](https://golang.google.cn/ref/spec)”、“[Go学习(需要科学上网)](https://tour.golang.org/)”和“[如何写 Go 程序](https://golang.google.cn/doc/code.html)”的补充内容，因此在看本文前先看看这些文档再学习本文档会更好。

## 示例代码
Go 源码包不仅仅是 Go 的核心功能库，里面同时提供了语言特性的各种实际用法。除此之外，许多的 package 包都涵盖了可直接运行的例子，这些例子大都不包含其他的依赖库，因此可以直接在 golang.google.cn 这个网页上运行，比如 [这个](https://golang.google.cn/pkg/strings/#example_Map) 例子。如果你在实际开发过程中遇到一个语言特性相关或者库的应用设计甚至实现相关的问题，都可以到核心代码库搜寻一下，应该会有所收获的。

[Go 源码包](https://golang.google.cn/pkg/) 列表。

# 格式化

代码的格式一般不影响代码的正常运行，但是因为影响到人的阅读体验，所以不同的代码风格往往导致一些争议。当然可以让任何开发者从一个风格迁移到另一个代码风格，并据此进行编码，但是最好不让开发者有这种选择的机会，试想当每个人都使用同一种编码风格进行开发时，格式化问题就不会占用大家那么多时间去适应各种风格了，相对也就节省了时间。当然理想很丰满，问题是如何在短期内快速且自然地实现这个理想。

在 Go 语言中，我们可以让机器帮我们处理大部分的格式问题。程序 gofmt （也可以通过 go fmt 子命令进行调用，它会在包级别上做格式化处理，不能指定文件进行格式化，即 `usage: go fmt [-n] [-x] [packages]`，不过在 go1.13 版本是可以制定某个特定的文件的，不知道是不是哪里理解错了）读取一个 Go 代码，然后按照标准风格对代码进行缩进、垂直对齐等修改从而产生规范化的代码；这个修改的过程只限于代码，不包括注释，注释一般我们写的什么样子就是什么样子，据说也会在必要时适当重新格式化注释（如果大家遇到类似的情况欢迎分享）。如果你的代码中存在格式上的布局问题，执行 gofmt 就好了；如果使用 gofmt 的效果和预期的不一样，重新调整一下你的代码再运行一次就可以了，如果实在得不到自己想要的结果，也可以考虑给 gofmt 提交一个修改意见，总之不要在格式问题上花太多时间就对了。

我们可以以结构体为例，在声明结构体的时候，对于结构体里面的字段，我们没有必要手动对齐字段名、字段类型、注释等，gofmt 可以帮我们自动实现这种对齐。比如对于有下面的声明：

```go
type T struct {
    name string // name of the object
    value int // its value
}
```

gofmt 会对齐每一行的内容，结果如下：

```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

值得关注的是，标准库中所有的 Go 源码都被 gofmt 格式化过，因此推荐大家去看看标准库中的源码，从而体会一下这种简洁的统一的美。

对于 Go 语言的格式，除了上面提到的 gofmt 的妙用，其实还是有几个非常简单的细节需要大家知道：

* 缩进

    在 Go 语言中，缩进默认使用制表符（Tab）来缩进，这也是 gofmt 的默认做法；除非必要，否则不建议使用空格进行缩进。

* 每行代码的长度

    Go 语言中每行的代码长度是没有限制的，因此不用担心长度溢出的问题；当然如果一行代码太长了，可以考虑手动换行，建议在新行前面添加一些 Tab 符来示意它和上一行是有关系的，便于阅读。

* 括号

    与 C 和 Java 相比较， Go 语言只需要少量的括号就能实现很多功能：在控制结构 if、for、switch 的声明中不需要把语句括起来就能正常运行。同时，操作符预定义的优先级也很简短明了，比如 `x<<8 + y<<16` 其中空格的存在就已经暗示了计算的优先级（空格经由 `gofmt` 格式化时会自动添加，开发者不需要特别关注这里的空格）。

    比如下面的 `for` 循环的代码:

```go
package main

import "fmt"

func main() {
	for i := 0; i < 5; i++ {
		fmt.Println(i)
	}
}
```


# 注释

Go 语言提供了 C 语言风格里的 `/* */` **块注释**和 C++ 语言风格里的 `//` **行注释**。这两种注释风格中，行注释是主要的注释方式，块注释的风格更多地用在包注释的场合，但是块注释还有一个用处，就是在行内表达式间使用，比如 `fmt.Println(1<<8 /* 这里刻意注释一段话，打印 260 */ + 1<<2)`；当然，当临时删除大块的代码的时候用块注释的方式也很方便。

`godoc` 是一个程序，同时也是一个 Web 服务器，它能从 Go 源码文件中提取各个 package 包里的文档。在包级别作用域的声明语句前直接出现的注释语句（注释和代码之间不能有空格行）会和声明语句一起被抽取出来作为相应条目的文档文本。因此源码中注释的具体内容及其排版风格直接决定了 godoc 生成的文档的质量。

【注】：可通过 `go get -v golang.org/x/tools/cmd/godoc` 安装 godoc 命令。

每个包都应该在包声明前包含一个块注释用来介绍包的功能——即包注释。如果一个包存在很多个文件，只需在任何一个文件中包含包注释就可以了。包注释应该介绍包的功能并提供整个包相关的信息。因为包注释会出现在 godoc 最前面的页面上，因此应该设置紧随其后的详细文档。下面给出了 `regexp` 这个包的注释示例，其首先介绍了 `regexp` 包的主要功能，然后马上给出了其正则表达式的语法。

```go
/*
Package regexp implements a simple library for regular expressions.

The syntax of the regular expressions accepted is:

    regexp:
        concatenation { '|' concatenation }
    concatenation:
        { closure }
    closure:
        term [ '*' | '+' | '?' ]
    term:
        '^'
        '$'
        '.'
        character
        '[' [ '^' ] character-ranges ']'
        '(' regexp ')'
*/
package regexp

```

如果一个包的功能非常简单，包注释也可以简化成为行注释，比如下面 `path` 包的注释：

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

Go 语言的注释里不需要附加格式符号，比如不需要 `**something**` 这种加粗强调的语法。由于自动生成的文档甚至没有固定宽度的字体，因此也不要试图通过空格来对齐某些内容；和前面讲的 gofmt 一样，godoc 会自动处理这些情况。在 Go 语言中，注释是未经特别解析的纯文本，HTML 和其他的注解，比如 `_this_` 会把每个字符重新打印出来 `_this_`，因此不应该使用这些特殊的样式。godoc 会用固定宽度的字体表示缩进，因此可以非常完美地展示注释中的代码片段；在核心代码库中，包 `fmt` 的[包注释](https://github.com/golang/go/blob/release-branch.go1.13/src/fmt/doc.go)就很好地利用了这个特性。

godoc 会根据上下文来决策要不要重新格式化注释，因此请确认你的注释看上去已经很不错了，比如使用正确的拼写、合适的标点符号、简洁的句子结构，折叠过长的句子，等等。

包中任何一个在顶层声明（在包作用域的声明）之前的注释均被认为是相应声明的注释文档，会被 godoc 自动抽取展示出来。程序中每一个导出的名字（首字母大写的名称，包括常量、变量、函数、方法等）都应该有一个注释文档，从而方便其他库引用这个名字时知道这个名字的具体含义。

注释文档最好是一个完整的句子，这可以增加展示方式的灵活度。同时，注释文档的第一句最好是对应名称的一句话总结。

```go
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```

如果每个注释文档的第一个单词都是它的描述对象，开发者就可以使用 `go` 命令行工具的子命令 `doc` 生成文档然后使用 `grep` 查验想要寻找的内容。比如你想不起来名称 `Compile` 了，但是想找正则表达式的**解析**函数，此时你可以运行下面的命令找到相应的内容：

```bash
$ go doc -all regexp | grep -i parse
```

如果所有的包注释文档都是 ”this function ...” 开始的，grep 过滤帮你找不到相应的名称。但正因为 regexp 包里每个名称的注释的第一个单词都是对应的名称，因此使用 grep 过滤的时候很容易就能看到这个名称，从而让你找到你想要的单词。

```bash
$ go doc -all regexp | grep -i parse
    Compile parses a regular expression and returns, if successful, a Regexp
    MustCompile is like Compile but panics if the expression cannot be parsed.
    parsed. It simplifies safe initialization of global variables holding
$
```

Go 的声明语法允许对一组变量进行声明，然后可以用单个注释文档介绍这一组常量或者变量。当然，这种注释内容往往比较笼统。

```go
// Error codes returned by failures to parse an expression.（表达式解析失败的错误码）
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

**组**也可用来暗示某几个名词之间的某种关联关系，比如下面的代码声明了一个 mutex 以及由它保护的一组变量。

```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```