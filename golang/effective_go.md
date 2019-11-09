# Effective Go

## Introduction（简介）

Go is a new language. Although it borrows ideas from existing languages, it has unusual properties that make effective Go programs different in character from programs written in its relatives. A straightforward translation of a C++ or Java program into Go is unlikely to produce a satisfactory result—Java programs are written in Java, not Go. On the other hand, thinking about the problem from a Go perspective could produce a successful but quite different program. In other words, to write Go well, it's important to understand its properties and idioms. It's also important to know the established conventions for programming in Go, such as naming, formatting, program construction, and so on, so that programs you write will be easy for other Go programmers to understand.

Go 是一个新的编程语言。虽然它是从现有语言中借鉴了很多想法，但是它有不同的属性，使得 Go 程序的效率与它的借鉴语言不同。如果直接按照 C++ 或者 Java 的程序逻辑转换成 Go 语言程序，很大可能不会有一个满意的结果——毕竟 Java 语言是用 Java 写的，而不是 Go 写的。另一方面，从 Go 的视角来考虑问题能够产生一个完全不同但成功的程序。换句话说，为了写好 Go 语言，用好它的特点和技巧很重要。当然了解 Go 语言编程的约定也很有必要，比如命名、格式化、程序构建等，如此你写出来的程序其他的 Go 开发者能够很容易的理解。

This document gives tips for writing clear, idiomatic Go code. It augments the [language specification](https://golang.google.cn/ref/spec), the [Tour of Go](https://tour.golang.org/), and [How to Write Go Code](https://golang.google.cn/doc/code.html), all of which you should read first.

本文档给出了一些编写清晰、规范的 Go 代码的技巧。它是“语言特性”、“Go学习”和“如何写 Go 程序”的增强版，因此在看本文前应该先看看这些。

### Examples（例子）

The [Go package sources](https://golang.google.cn/src/) are intended to serve not only as the core library but also as examples of how to use the language. Moreover, many of the packages contain working, self-contained executable examples you can run directly from the [golang.org](https://golang.org/) web site, such as [this one](https://golang.org/pkg/strings/#example_Map) (if necessary, click on the word "Example" to open it up). If you have a question about how to approach a problem or how something might be implemented, the documentation, code and examples in the library can provide answers, ideas and background.

Go 包源码不仅仅期望能成为核心的库，同时也作为如何使用这门语言的离子。除此外，许多的包都涵盖了可运行且自包含的例子，你可以直接在 golang.org 这个网页上运行，比如 [这个](https://golang.org/pkg/strings/#example_Map) 例子。如果你想知道怎么解决某个问题，或者如何实现某个东西，代码库中的文档、源码和例子可以提供一些答案、想法以及背景。

## Formatting（格式化）

Formatting issues are the most contentious but the least consequential. People can adapt to different formatting styles but it's better if they don't have to, and less time is devoted to the topic if everyone adheres to the same style. The problem is how to approach this Utopia without a long prescriptive style guide.

格式化问题是最具争议性但最不重要的问题。人们往往可以接受不同的格式化风格，但是最好不让他们做选择，如果每个人都使用同一种风格，格式化问题就不会占用大家那么多时间了。问题是如何在没有长期风格指南的情况下达成这个乌托邦。

With Go we take an unusual approach and let the machine take care of most formatting issues. The `gofmt` program (also available as `go fmt`, which operates at the package level rather than source file level) reads a Go program and emits the source in a standard style of indentation and vertical alignment, retaining and if necessary reformatting comments. If you want to know how to handle some new layout situation, run `gofmt`; if the answer doesn't seem right, rearrange your program (or file a bug about `gofmt`), don't work around it.

对于 Go 语言，我们可以通过一种不寻常的方式来让机器帮我们处理尽可能多的格式问题。程序 `gofmt` （也是 `go fmt` ，它在包级别上做处理而不是文件级别上做处理）读取一个 Go 代码，然后按照标准风格中的缩进、垂直对齐产生规范后的源代码，保留并在必要时重新格式化注释。如果你想知道怎么处理一些新的布局问题，执行 `gofmt` 就好了；如果效果和预期的不一样，重新调整一下你的代码（或者给 `gofmt` 提一个 bug）就可以了，不要就这个问题花太多时间。

As an example, there's no need to spend time lining up the comments on the fields of a structure. `Gofmt` will do that for you. Given the declaration

作为一个例子，对于结构体中的字段，我们没有必要花时间对齐注释，`Gofmt` 可以帮你自动完成。如果有下面的声明：

```go
type T struct {
    name string // name of the object
    value int // its value
}
```

`gofmt` will line up the columns:

`gofmt` 将会对齐每一行的内容如下：

```go
type T struct {
    name    string // name of the object
    value   int    // its value
}
```

All Go code in the standard packages has been formatted with `gofmt`.

所有在标准库中的 Go 源码都被 `gofmt` 格式化过。

Some formatting details remain. Very briefly:【还有一些格式化的细节。简单来讲：

- Indentation【缩进

  We use tabs for indentation and `gofmt` emits them by default. Use spaces only if you must.

  我们使用 tabs 来进行缩进，而且 `gofmt` 默认会自动生成他们。除非必要，否则不使用空格。

- Line length【每行的长度

  Go has no line length limit. Don't worry about overflowing a punched card. If a line feels too long, wrap it and indent with an extra tab

  Go 没有行长度的限制。不用担心会有长度溢出的问题。如果一行太长了，可以折叠它并适当另外的 tab 空格。

- Parentheses【括号

  Go needs fewer parentheses than C and Java: control structures (`if`, `for`, `switch`) do not have parentheses in their syntax. Also, the operator precedence hierarchy is shorter and clearer, so`x<<8 + y<<16 `means what the spacing implies, unlike in the other languages.
  
  相比较 C 和 Java，Go 需要更少的括号即可完成功能：控制结构（ if、for、switch）在他们的语法中不需要括号就可以正常工作。同时，操作符预定义的优先级很短且明了，比如 `x<<8 + y<<16` 空格已经暗示了计算优先级，这一点上不像其他的语言。

## Commentary（注释）

Go provides C-style `/* */` block comments and C++-style `//` line comments. Line comments are the norm; block comments appear mostly as package comments, but are useful within an expression or to disable large swaths of code.

Go 提供了 C 风格的块注释和 C++ 风格的 `//` 行注释。行注释是主要的注释方式，块注释的风格一般用于包注释，但是在行内表达式间使用比较方便，当注释大块的代码的时候也很方便。

The program—and web server—`godoc` processes Go source files to extract documentation about the contents of the package. Comments that appear before top-level declarations, with no intervening newlines, are extracted along with the declaration to serve as explanatory text for the item. The nature and style of these comments determines the quality of the documentation `godoc` produces.

程序，同时也是一个 Web 服务器，`godoc` 会处理 Go 源码文件并从中提取关于包内容的文档。在最外层声明语句的前面出现且中间没有出现空格行的注释语句，会和声明一起被抽取出来作为相应条目的解释文本。这些注释的性质和风格决定了 `godoc` 生成的文档的质量。

Every package should have a *package comment*, a block comment preceding the package clause. For multi-file packages, the package comment only needs to be present in one file, and any one will do. The package comment should introduce the package and provide information relevant to the package as a whole. It will appear first on the `godoc` page and should set up the detailed documentation that follows.

每个包都应该有一个包注释，即在包声明语句前的块注释。对于存在多个文件的包来说，包注释只需要在任何一个文件中存在就可以了。包注释应该介绍包功能并提供整个包相关的的信息。包注释会首先出现在 `godoc` 的页面上，并且应该设置一下紧跟着的详细文档。

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

If the package is simple, the package comment can be brief.

如果包功能简单，包注释也可以比较简单：

```go
// Package path implements utility routines for
// manipulating slash-separated filename paths.
```

Comments do not need extra formatting such as banners of stars. The generated output may not even be presented in a fixed-width font, so don't depend on spacing for alignment—`godoc`, like `gofmt`, takes care of that. The comments are uninterpreted plain text, so HTML and other annotations such as `_this_` will reproduce *verbatim* and should not be used. One adjustment `godoc` does do is to display indented text in a fixed-width font, suitable for program snippets. The package comment for the [`fmt` package](https://golang.google.cn/pkg/fmt/) uses this to good effect.

注释不需要另外的格式，比如带星星的展示位等。自动生成的文档展示的时候甚至没有固定宽度的字体，因此不要视图通过空格来对齐内容；就像 `gofmt` 那样，`godoc`会考虑这些的。注释是未经解析的白文本，因此 HTML 和其他的注解，比如 `_this_` 会逐字地重新生成 `_this_`，因此不应该使用这些特殊样式。`godoc` 做的一个调整是用固定宽度的字体显示缩进的文本，更好的展示代码片段。包 `fmt` 的包注释很好地利用了这个特性。

Depending on the context, `godoc` might not even reformat comments, so make sure they look good straight up: use correct spelling, punctuation, and sentence structure, fold long lines, and so on.

依赖于上下文， `godoc` 可能不会重新格式化注释，因此请确认你的注释看上去已经很漂亮：使用正确的拼写、合适的标点符号、简洁的句子结构，折叠长句，等等。

Inside a package, any comment immediately preceding a top-level declaration serves as a *doc comment* for that declaration. Every exported (capitalized) name in a program should have a doc comment.

在包中，任何一个在顶层声明之前的注释被认为是那个声明的注释文档。程序中每一个导出（首字母大写）的名字都应该有一个注释文档。

Doc comments work best as complete sentences, which allow a wide variety of automated presentations. The first sentence should be a one-sentence summary that starts with the name being declared.

注释文档最好以完整的句子存在，从而允许各种自动展示。第一句应该是是相应要声明的名称的一句话总结。

```go
// Compile parses a regular expression and returns, if successful,
// a Regexp that can be used to match against text.
func Compile(str string) (*Regexp, error) {
```

If every doc comment begins with the name of the item it describes, you can use the [doc](https://golang.google.cn/cmd/go/#hdr-Show_documentation_for_package_or_symbol) subcommand of the [go](https://golang.google.cn/cmd/go/) tool and run the output through `grep`. Imagine you couldn't remember the name "Compile" but were looking for the parsing function for regular expressions, so you ran the command,

如果每个注释文档都由它描述的名字开始，你就可以使用 go 工具的子命令 `doc` 生成文档然后使用 `grep` 查验想要的内容。假设你无法想起名称 `Compile` 了，但是想找一个正则表达式的解析函数，此时你可以运行下面的命令：

```bash
$ go doc -all regexp | grep -i parse
```

If all the doc comments in the package began, "This function...", `grep` wouldn't help you remember the name. But because the package starts each doc comment with the name, you'd see something like this, which recalls the word you're looking for.

如果所有的包注释文档都是 ”this function ...”，`grep` 不会帮你想起名称。但是因为每个包都以其名称开始其注释文档，因此你会看到它的名称，从而让你找到你想要的单词。

```bash
$ go doc -all regexp | grep -i parse
    Compile parses a regular expression and returns, if successful, a Regexp
    MustCompile is like Compile but panics if the expression cannot be parsed.
    parsed. It simplifies safe initialization of global variables holding
$
```

Go's declaration syntax allows grouping of declarations. A single doc comment can introduce a group of related constants or variables. Since the whole declaration is presented, such a comment can often be perfunctory.

Go 的声明语法允许分组进行声明。单个注释文档可以介绍一组相关的常量或者变量。因为注释包含了所有的声明，因此这种注释内容往往偏笼统的。

```go
// Error codes returned by failures to parse an expression.
var (
    ErrInternal      = errors.New("regexp: internal error")
    ErrUnmatchedLpar = errors.New("regexp: unmatched '('")
    ErrUnmatchedRpar = errors.New("regexp: unmatched ')'")
    ...
)
```

Grouping can also indicate relationships between items, such as the fact that a set of variables is protected by a mutex.

组也可以暗示各个条目之间的关系，比如一个 mutext 以及由它保护的一组变量。

```go
var (
    countLock   sync.Mutex
    inputCount  uint32
    outputCount uint32
    errorCount  uint32
)
```

## Names（名称）

Names are as important in Go as in any other language. They even have semantic effect: the visibility of a name outside a package is determined by whether its first character is upper case. It's therefore worth spending a little time talking about naming conventions in Go programs.

就像其他语言一样，名称在 Go 中也很重要。Go 中的名称甚至有语法上的影响：名称的首字母是否大写影响一个名字在包外的可见性。因此值得花一些时间来聊聊 Go 程序中的名称惯例及规范。

### Package names（包名）

When a package is imported, the package name becomes an accessor for the contents. After

```go
import "bytes"
```

the importing package can talk about `bytes.Buffer`. It's helpful if everyone using the package can use the same name to refer to its contents, which implies that the package name should be good: short, concise, evocative. By convention, packages are given lower case, single-word names; there should be no need for underscores or mixedCaps. Err on the side of brevity, since everyone using your package will be typing that name. And don't worry about collisions *a priori*. The package name is only the default name for imports; it need not be unique across all source code, and in the rare case of a collision the importing package can choose a different name to use locally. In any case, confusion is rare because the file name in the import determines just which package is being used.

当包被导入后，包名会成为其内容的访问器。在 `import "byte"` 后，导入此包的文件可以使用 `bytes.Buffer`。如果使用这个包的人都通过包名称调用包里的内容，这就意味着包名称应该足够好：短、简介、释义。按照惯例，包名应该是小写字符、独立单词的名称；他们应该不包含下划线或驼峰式名称。简而言之，因为每个使用你的包的人都会输入你的包名。不需要担心和已经存在的包重名冲突；包名只用于导入此包时的默认名称，并没有必要在所有的源码中都保持唯一，在少有的导入包名称冲突的情况下，可以给冲突的包定义一个不冲突的本地名称来使用。因为文件名称决定了使用的是哪个包，因此任何情况下都很少出现混淆。

Another convention is that the package name is the base name of its source directory; the package in `src/encoding/base64` is imported as `"encoding/base64"` but has name `base64`, not `encoding_base64` and not `encodingBase64`.

另一个惯例，是包名一般设置为它的源文件目录名，目录 `src/encoding/base64` 定义的包引入时是 `"encoding/base64"`，可以使用 `base64` 来引用包里的内容，不能写成 `encoding_base64`，也不能写成 `encodingBase64`。

The importer of a package will use the name to refer to its contents, so exported names in the package can use that fact to avoid stutter. (Don't use the `import .` notation, which can simplify tests that must run outside the package they are testing, but should otherwise be avoided.) For instance, the buffered reader type in the `bufio` package is called `Reader`, not `BufReader`, because users see it as `bufio.Reader`, which is a clear, concise name. Moreover, because imported entities are always addressed with their package name, `bufio.Reader` does not conflict with `io.Reader`. Similarly, the function to make new instances of `ring.Ring`—which is the definition of a *constructor* in Go—would normally be called `NewRing`, but since `Ring` is the only type exported by the package, and since the package is called `ring`, it's called just `New`, which clients of the package see as `ring.New`. Use the package structure to help you choose good names.

一个包的导入器会使用它的名称来引用其内容，因此在包中导出的名字可以避免含糊不清。（不要使用 `import .` 符号，虽然它可以简化必须在包外面进行的测试，但是仍然应该避免使用）比如，在 `bufio` 包中带缓存的读类型是 `Reader` 而不是 `BufReader`, 因为当用户看到 `bufio.Reader` 的时候就已经知道这是个带缓存的 `Reader`，简洁又准确。进一步讲，因为被导入的实体经常与他们的包名一起取用，因此 `bufio.Reader` 和 `io.Reader` 是不冲突的。类似的，一个创建 `ring.Ring`（它是 Go 中的一个构造器） 新对象的函数正常情况下可以命名为 `NewRing`，但是因为 `Ring` 是包唯一导出的类型，并且因为包名叫 `ring`，因此这个方法可以命名为 `New`，这样这个包的客户端就可以通过 `ring.New` 使用它了。使用包的结构来帮助你选择好的名称吧。

Another short example is `once.Do`; `once.Do(setup)` reads well and would not be improved by writing `once.DoOrWaitUntilDone(setup)`. Long names don't automatically make things more readable. A helpful doc comment can often be more valuable than an extra long name.

另一个短小的例子是 `once.Do`，`once.Do(setup)` 读起来很通顺，如果写成 `once.DoOrWaitUntilDone(setup)` 好像也不能让它变得更好。长的名字并不会自然而然地让事物更具有可读性。帮助性的注释文档往往有更高的价值，而不是特别长的名称。

### Getters（获取器）

Go doesn't provide automatic support for getters and setters. There's nothing wrong with providing getters and setters yourself, and it's often appropriate to do so, but it's neither idiomatic nor necessary to put `Get` into the getter's name. If you have a field called `owner` (lower case, unexported), the getter method should be called `Owner` (upper case, exported), not `GetOwner`. The use of upper-case names for export provides the hook to discriminate the field from the method. A setter function, if needed, will likely be called `SetOwner`. Both names read well in practice:

Go 并没有对获取器（getter）和设置器（setter）提供自动的支持。如果你自己提供获取器和设置器，也没有什么问题，而且一般推荐这么做；但是把 `Get` 放进获取器的名字里显得既不明智也没有必要。如果有一个名为 `owner` 的字段（小写，非导出），获取器的名字应该叫 `Owner`（大写字母，可导出），而不是 `GetOwner`。大写字母名称可以导出，这种用法提供了一种机制使方法名与字段名可以不同。在设置器函数必要的情况下，可以取名为 `SetOwner`。如此，两个名字读起来都很流畅：

```go
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

### Interface names（接口名）

By convention, one-method interfaces are named by the method name plus an -er suffix or similar modification to construct an agent noun: `Reader`, `Writer`, `Formatter`, `CloseNotifier` etc.

通常情况下，只有一个方法的接口被命名为其包含的方法名加一个 "-er" 后缀，或者同样的方式构造一个代理名词：`Reader, Writer, Formatter, CloseNotifier` 等。

There are a number of such names and it's productive to honor them and the function names they capture. `Read`, `Write`, `Close`, `Flush`, `String` and so on have canonical signatures and meanings. To avoid confusion, don't give your method one of those names unless it has the same signature and meaning. Conversely, if your type implements a method with the same meaning as a method on a well-known type, give it the same name and signature; call your string-converter method `String` not `ToString`.

有许多这种卓有成效的名称，对应的包以及包内的方法都被人们熟知。`Read`, `Write`, `Close`, `Flush`, `String`有规范的签名和含义。为了避免混淆，除非你的方法有相同的签名和含义，否则不要以这些名字命名。相反，如果你自己的类型有一个方法和某个已存在类型的方法功能相似，此时建议取相同的名字就可以。比如你定义了一个字符串转换器的方法，应该直接命名为 `String` ，不要命名为 `ToString`。

### MixedCaps（混合驼峰）

Finally, the convention in Go is to use `MixedCaps` or `mixedCaps` rather than underscores to write multiword names.

最后，Go 惯例中使用混合驼峰 `MixedCaps` 或 `mixedCaps` 来组织多个单词的名称，不推荐使用下划线的方式。

## Semicolons（分号）

Like C, Go's formal grammar uses semicolons to terminate statements, but unlike in C, those semicolons do not appear in the source. Instead the lexer uses a simple rule to insert semicolons automatically as it scans, so the input text is mostly free of them.

像 C 语言那样，Go 的常规语法使用分号来结束一条状态语句；但是和 C 不同的是，这些分号不会出现在源码中。实际上，词法分析器在扫码源码的时候会通过一种简单的规则自动插入分号，从而允许在源码中不写分号。

The rule is this. If the last token before a newline is an identifier (which includes words like `int` and `float64`), a basic literal such as a number or string constant, or one of the tokens

```go
break continue fallthrough return ++ -- ) }
```

the lexer always inserts a semicolon after the token. This could be summarized as, “if the newline comes after a token that could end a statement, insert a semicolon”.

分号插入规则是：1）行尾符号的最后一个符号是一个标识符（包括 `int` 和 `float64` 等）、2）类似数字或字符串常量这种基础语法、或者3）是 `break continue fallthrough return ++ -- ) }` 其中一个标识符，只要是其中一种情况，词法分析器就会在这些符号后面插入一个分号。可以总结为：“如果一个标识符的下一个字符就是行尾符，且这个标识符可以终止一个状态表达式，就插入一个分号”。

A semicolon can also be omitted immediately before a closing brace, so a statement such as

```go
    go func() { for { dst <- <-src } }()
```

needs no semicolons. Idiomatic Go programs have semicolons only in places such as `for` loop clauses, to separate the initializer, condition, and continuation elements. They are also necessary to separate multiple statements on a line, should you write code that way.

可以省略紧接在右大括号前面的分号，因此表达式 `go func() { for { dst <- <-src } }()` 不需要任何的分号。Go 程序中只有一种惯用分号的情况就是 `for` 循环语句，通过分号来分割初始化语句、条件语句和迭代变量改变语句。如果你想在一行里写多个表达式，也应该使用分号来分割这些语句。

One consequence of the semicolon insertion rules is that you cannot put the opening brace of a control structure (`if`, `for`, `switch`, or `select`) on the next line. If you do, a semicolon will be inserted before the brace, which could cause unwanted effects. Write them like this

分号插入规则的一个结果是，你不可以把控制结构语句（`if`, `for`, `switch`, 或者 `select`）的左大括号放在新的一行（应该放在一行）。如果你把 `if`, `for`, `switch`, 或者 `select` 紧跟着的大括号放在新的一行，此时左大括号前面会插入一个分号，这肯定不是我们想要的效果。可以看下面的例子及其反例：

```go
if i < f() {
    g()
}
```

not like this（不应该写成下面的样子）

```go
if i < f()  // wrong!
{           // wrong!
    g()
}
```

## Control structures（控制结构）

The control structures of Go are related to those of C but differ in important ways. There is no `do` or `while` loop, only a slightly generalized `for`; `switch` is more flexible; `if` and `switch` accept an optional initialization statement like that of `for`; `break` and `continue` statements take an optional label to identify what to break or continue; and there are new control structures including a type switch and a multiway communications multiplexer, `select`. The syntax is also slightly different: there are no parentheses and the bodies must always be brace-delimited.

Go 中的控制结构和 C 语言有点像但是也有很不一样的地方：在 Go 语言中没有 `do` 和 `while` 循环，只有一个通用的 `for` 循环；`switch` 相对更灵活；`if` 和 `switch` 可以像 `for` 语句那样接收一个可选的初始化语句；`break` 和 `continue` 语句可以通过一个可选的标签来指定 break 或 continue 的位置；可以根据类型选择执行的逻辑分支；多出来一个新的控制结构 `select`，可以用来做多路通信分发器。和 C 语言相比， Go 的语法也有一些不同的地方：Go 语言中没有括号，且他们的执行体都必须以大括号进行分割。

### If（If 结构）

In Go a simple `if` looks like this:【在 Go 语言中，简单的 `if` 语句如下：

```go
if x > 0 {
    return y
}
```

Mandatory braces encourage writing simple `if` statements on multiple lines. It's good style to do so anyway, especially when the body contains a control statement such as a `return` or `break`.

强制添加大括号可以促使大家把简单的 `if` 语句写成多行的样式，无疑这种多行的样式会更好看，尤其当执行体内部包含 `return` 或 `break` 这种控制语句的时候尤其明显。

Since `if` and `switch` accept an initialization statement, it's common to see one used to set up a local variable.

鉴于 `if` 和 `switch` 中允许有一个初始化语句，因此可以经常看到类似下面的用法（创建了一个本地临时变量）：

```go
if err := file.Chmod(0664); err != nil {
    log.Print(err)
    return err
}
```

In the Go libraries, you'll find that when an `if` statement doesn't flow into the next statement—that is, the body ends in `break`, `continue`, `goto`, or `return`—the unnecessary `else` is omitted.

在 Go 的库源码中，可以很明显看到这种情况，如果 `if` 的执行体以 `break`, `continue`, `goto`, 或者 `return` 结束，逻辑上没任何作用的 `else` 就会被省略掉。

```go
f, err := os.Open(name)
if err != nil {
    return err
}
codeUsing(f)
```

This is an example of a common situation where code must guard against a sequence of error conditions. The code reads well if the successful flow of control runs down the page, eliminating error cases as they arise. Since error cases tend to end in `return` statements, the resulting code needs no `else` statements.

上面是一个很常见的例子，代码必须要考虑一系列的错误情况。如果去除那些错误处理的情况，控制流成功地贯穿页面，代码很容易阅读。同时因为错误处理的那个 `if` 执行体以 `return` 结束，因此代码中没有必要出现 `else` 语句。【也就是说，如果 `os.Open` 调用出错，整个函数会直接 `return err` 返回，后面的代码（`codeUsing`）也就不会执行；如果 `os.Open` 调用不出错，代码不会进入到 `if` 的执行体，后面的代码顺理成章地被执行】

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

### Redeclaration and reassignment（重声明和重赋值）

An aside: The last example in the previous section demonstrates a detail of how the `:=` short declaration form works. The declaration that calls `os.Open` reads,

```go
f, err := os.Open(name)
```

This statement declares two variables, `f` and `err`. A few lines later, the call to `f.Stat` reads,

```go
d, err := f.Stat()
```

which looks as if it declares `d` and `err`. Notice, though, that `err` appears in both statements. This duplication is legal: `err` is declared by the first statement, but only *re-assigned* in the second. This means that the call to `f.Stat` uses the existing `err` variable declared above, and just gives it a new value.

旁白：上一小节的最后一个例子很好地阐释了 `:=` 短声明语句的工作方式。声明语句 `f, err := os.Open(name)` 声明了两个变量：`f` 和 `err`。紧接着后面的 `d, err := f.Stat()` 貌似又声明了变量 `d` 和 `err`。但是需要注意 `err` 出现在了两个声明语句中。在 Go 中，这种重复是允许的：`err` 首先由第一个语句**声明**，在第二个语句中被**重新赋值**。这意味着调用 `f.Stat` 时使用了上面已经被声明的 `err` 变量，然后又给了它一个新的值。

In a `:=` declaration a variable `v` may appear even if it has already been declared, provided:

- this declaration is in the same scope as the existing declaration of `v` (if `v` is already declared in an outer scope, the declaration will create a new variable §),
- the corresponding value in the initialization is assignable to `v`, and
- there is at least one other variable in the declaration that is being declared anew.

This unusual property is pure pragmatism, making it easy to use a single `err` value, for example, in a long `if-else` chain. You'll see it used often.

在 `:=` 声明中，即使变量 `v` 已经被声明过也可以再次出现，只需满足：

- `:=` 语法声明和已存在的 `v` 在同一个作用域内（如果 `v` 是在外面的作用域声明的，这种情况下会创建一个新的变量）；
- 初始化语句中的值可以赋值给 `v`，并且
- 在声明语句中至少有一个未声明过的新变量。

§ It's worth noting here that in Go the scope of function parameters and return values is the same as the function body, even though they appear lexically outside the braces that enclose the body.

§（脚注）需要注意，在 Go 语言中，虽然函数参数及其返回值出现在函数执行体的花括号外面，但是它们的作用域是函数的整个执行体。


### For（For 语句）

The Go `for` loop is similar to—but not the same as—C's. It unifies `for` and `while` and there is no `do-while`. There are three forms, only one of which has semicolons.

Go 中的 `for` 循环和 C 语言的类似，但是也不完全一样。`for` 统一了 `for` 和 `while` 循环，因此在 Go 中没有 `do-while` 循环。在 Go 中有三种形式的 `for` 循环，其中只有一种带分号。

```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

Short declarations make it easy to declare the index variable right in the loop.
可以通过短声明很容易地生成 `for` 循环中的索引变量。

```go
sum := 0
for i := 0; i < 10; i++ {
    sum += i
}
```

If you're looping over an array, slice, string, or map, or reading from a channel, a `range` clause can manage the loop.

如果在数组、切片、字符串、哈希 map 上进行循环，或者循环读取信道中的元素，可以使用 `range` 语法来控制整个循环。

```go
for key, value := range oldMap {
    newMap[key] = value
}
```

If you only need the first item in the range (the key or index), drop the second:
在 `range` 语法中如果只需要第一个元素的值，可以省略第二个变量：

```go
for key := range m {
    if key.expired() {
        delete(m, key)
    }
}
```

If you only need the second item in the range (the value), use the *blank identifier*, an underscore, to discard the first:

在 `range` 语法中，如果只需要第二个元素的值，可以使用空标识符（下划线）丢弃第一个元素：

```go
sum := 0
for _, value := range array {
    sum += value
}
```

The blank identifier has many uses, as described in [a later section](https://golang.google.cn/doc/effective_go.html#blank).

For strings, the `range` does more work for you, breaking out individual Unicode code points by parsing the UTF-8. Erroneous encodings consume one byte and produce the replacement rune U+FFFD. (The name (with associated builtin type) `rune` is Go terminology for a single Unicode code point. See [the language specification](https://golang.google.cn/ref/spec#Rune_literals) for details.) The loop

对于字符串来说，`range` 会做更多的事情，它会检索出以 UTF-8 编码的每个 Unicode 字节码。错误的编码会产生一个替代符文 `U+FFFD`，此时相应的索引会前进一个字节（跳过引起错误的字节）。（`rune` 是 Go 内建的一个类型，用来表示 Unicode 字节码，可以到相关的章节查看细节）。具体的可以查看下面的例子：

```go
for pos, char := range "日本\x80語" { // \x80 is an illegal UTF-8 encoding
    fmt.Printf("character %#U starts at byte position %d\n", char, pos)
}
```

prints（上面的代码打印结果为：）

```go
character U+65E5 '日' starts at byte position 0
character U+672C '本' starts at byte position 3
character U+FFFD '�' starts at byte position 6
character U+8A9E '語' starts at byte position 7
```

Finally, Go has no comma operator and `++` and `--` are statements not expressions. Thus if you want to run multiple variables in a `for` you should use parallel assignment (although that precludes `++` and `--`).

Go 没有逗号操作符；符号 `++` 和 `--` 是状态表达式而不是值表达式。因此，如果想在 `for` 循环中维护多个变量应该使用平行赋值语句（此时不能使用 `++` 和 `--` 运算符）。

```go
// Reverse a
for i, j := 0, len(a)-1; i < j; i, j = i+1, j-1 {
    a[i], a[j] = a[j], a[i]
}
```

### Switch （switch 语句）

Go's `switch` is more general than C's. The expressions need not be constants or even integers, the cases are evaluated top to bottom until a match is found, and if the `switch` has no expression it switches on `true`. It's therefore possible—and idiomatic—to write an `if`-`else`-`if`-`else` chain as a `switch`.

Go 里的 `switch` 比 C 语言中的更通用。它的表达式不必是常量，甚至可以不是整数，它的 case 会从上到下依次执行直到匹配到某个 case 为止。如果 `switch` 后没有跟任何的表达式，则默认是一个 `true` 值。一般情况下可以选择把 `if`-`else`-`if`-`else` 写成 `switch` 的语句。

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

There is no automatic fall through, but cases can be presented in comma-separated lists.

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

Although they are not nearly as common in Go as some other C-like languages, `break` statements can be used to terminate a `switch` early. Sometimes, though, it's necessary to break out of a surrounding loop, not the switch, and in Go that can be accomplished by putting a label on the loop and "breaking" to that label. This example shows both uses.

不像其他类 C 的编程语言那样常见，在 Go 语言中，`break` 语句可以用来提前终止 `switch` 语句。但是，有时候我们需要终止的是一个包围起来的循环而不是 switch 逻辑，这种情况下我们可以在循环上定义一个标签，然后就可以通过标签标注 `break` 终止那个循环了。下面的例子展示了两种应用：

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

Of course, the `continue` statement also accepts an optional label but it applies only to loops.

当然，`continue` 表达式也可以设置一个可选的标签，不过 `continue` 只能用在循环中。

To close this section, here's a comparison routine for byte slices that uses two `switch` statements:

作为本小节的结尾，下面使用两个 `switch` 来比较两个字节切片：

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

### Type switch（类型 switch）

A switch can also be used to discover the dynamic type of an interface variable. Such a *type switch* uses the syntax of a type assertion with the keyword `type` inside the parentheses. If the switch declares a variable in the expression, the variable will have the corresponding type in each clause. It's also idiomatic to reuse the name in such cases, in effect declaring a new variable with the same name but a different type in each case.

switch 语句还可以用来探索接口变量的动态类型。**类型 switch**使用带括号的关键字`type`来对一个变量的类型进行断言。如果在 switch 表达式中声明了变量，这个变量会保存每个 case 对应的类型的值。在这些 case 中一般会复用名称，实际上会为每个 case 声明一个同名不同类的新变量。

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

## Functions（函数）

### Multiple return values（多返回值）

One of Go's unusual features is that functions and methods can return multiple values. This form can be used to improve on a couple of clumsy idioms in C programs: in-band error returns such as `-1` for `EOF` and modifying an argument passed by address.

Go 非同寻常的一个特性是它的函数和方法可以返回多个值。这个特性可以提升 C 程序中的几个粗苯的习惯：把 `EOF` 错误以 `-1` 的形式返回，修改某个传址的参数。

In C, a write error is signaled by a negative count with the error code secreted away in a volatile location. In Go, `Write` can return a count *and* an error: “Yes, you wrote some bytes but not all of them because you filled the device”. The signature of the `Write` method on files from package `os` is:

```go
func (file *File) Write(b []byte) (n int, err error)
```

and as the documentation says, it returns the number of bytes written and a non-nil `error` when `n != len(b)`. This is a common style; see the section on error handling for more examples.

在 C 语言中，写错误是一个负数，代表某种错误的错误码，错误码与错误类型的对应关系则保存在内存里。在 Go 语言中， `Write` 可以返回一个数值**和**一个错误：“你写成功了一些字节，但是因为设备满了导致剩了一些字节。”包 `os` 中的 `Write` 方法的定义是 `func (file *File) Write(b []byte) (n int, err error)`，就像它的文档说的，它返回写成功的数目，当 `n != len(b)` 的时候还会返回一个非空的 错误。这是一种很常见的形式，可以在错误一节查看更多的例子。

A similar approach obviates the need to pass a pointer to a return value to simulate a reference parameter. Here's a simple-minded function to grab a number from a position in a byte slice, returning the number and the next position.

同样的，多返回值的特性还可以避免给返回值传指针来模拟引用类型的参数。下面给出了一个简单明了的函数，其功能是在一个字节数组的某个位置开始获取一个数字，然后返回这个数字以及下一个位置。

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

You could use it to scan the numbers in an input slice `b` like this:

你可以用上面的函数来扫描在输入切片 `b` 中的数字。

```go
    for i := 0; i < len(b); {
        x, i = nextInt(b, i)
        fmt.Println(x)
    }
```

### Named result parameters（命名的返回值变量）

The return or result "parameters" of a Go function can be given names and used as regular variables, just like the incoming parameters. When named, they are initialized to the zero values for their types when the function begins; if the function executes a `return` statement with no arguments, the current values of the result parameters are used as the returned values.

可以给 Go 函数的返回值参数或者结果参数指定一个名称，然后就可以在函数体中像传入的普通变量那样使用它了。返回值变量一旦被命名，它们在函数执行之初就被初始化了他们对应类型的零值；如果函数执行了 `return` 命令且没有显示指明任何参数，则在 `return` 语句执行时返回值变量的值就是实际意义上的返回值。

The names are not mandatory but they can make code shorter and clearer: they're documentation. If we name the results of `nextInt` it becomes obvious which returned `int` is which.

函数返回值的命名不是必须的，但是他们可以让代码更简洁明了：他们属于文档的一部分。如果我们把返回值命名为 `nextInt`，返回的值的含义就变得显而易见了。

```go
func nextInt(b []byte, pos int) (value, nextPos int) {
```

Because named results are initialized and tied to an unadorned return, they can simplify as well as clarify. Here's a version of `io.ReadFull` that uses them well:

同时因为命名的返回值在函数原型之初就已经被初始化好了，可以与裸的 `return` 一起使用，从而让代码更简洁。下面的给出了 `io.ReadFull` 的一个版本就很好地使用了这个特性：

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

### Defer（推迟函数）

Go's `defer` statement schedules a function call (the *deferred* function) to be run immediately before the function executing the `defer` returns. It's an unusual but effective way to deal with situations such as resources that must be released regardless of which path a function takes to return. The canonical examples are unlocking a mutex or closing a file.

Go 的 `defer` 表达式规划了一个函数（被 deferred 的函数）可以在**当前函数**返回给调用者之前再及时运行。`defer` 的用法有点不寻常，但是很有用；当代码中引用了某个资源，不管程序执行到哪个分支都必须需要释放这个资源，这种情况使用 `defer` 就很方便处理了。`defer` 典型的应用是在解锁一个互斥锁或者关闭一个文件句柄。

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

Deferring a call to a function such as `Close` has two advantages. First, it guarantees that you will never forget to close the file, a mistake that's easy to make if you later edit the function to add a new return path. Second, it means that the close sits near the open, which is much clearer than placing it at the end of the function.

推迟执行类似 `Close` 这种函数的调用有两个好处。第一，它保证了调用的必然性，防止你后来修改函数逻辑的时候（比如加了另一个 `return` 的分支）忘记关闭文件句柄。第二，这意味着打开与关闭的代码是可以放在一起的，比起把它们两个分开（打开文件句柄的代码在函数开始的地方，关闭的代码在函数末尾）更容易理解。

The arguments to the deferred function (which include the receiver if the function is a method) are evaluated when the *defer* executes, not when the *call* executes. Besides avoiding worries about variables changing values as the function executes, this means that a single deferred call site can defer multiple function executions. Here's a silly example.

当 `defer` 开始执行的时候，传递给 `defer` 函数（也可能是一个有接收器的方法）的参数表达式会立马运算，并不会推迟到被 deferred 的函数执行。除了需要想办法来避免函数执行过程中参数值被修改外，上面的特性意味着一个独立的推迟点可以推迟多个函数的执行。下面是一个看起来很蠢的例子：

```go
for i := 0; i < 5; i++ {
    defer fmt.Printf("%d ", i)
}
```

Deferred functions are executed in LIFO order, so this code will cause `4 3 2 1 0` to be printed when the function returns. A more plausible example is a simple way to trace function execution through the program. We could write a couple of simple tracing routines like this:

被延迟的多函数会按照后进先出（LIFO）的顺序依次执行，因此上面的代码在函数返回后会打印出 `4 3 2 1 0`。另一个具有象征意义的例子是跟踪整个函数的执行顺序。我们可以写一个简单的跟踪样例如下：

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

We can do better by exploiting the fact that arguments to deferred functions are evaluated when the `defer` executes. The tracing routine can set up the argument to the untracing routine. This example:

当 `defer` 表达式开始执行时，传递给 `defer` 函数的参数表达式马上就会执行，利用这个特性我们可以写出更优雅的代码。跟踪的函数可以作为解除跟踪的函数的参数，代码如下：

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

prints

```go
entering: b
in b
entering: a
in a
leaving: a
leaving: b
```

For programmers accustomed to block-level resource management from other languages, `defer` may seem peculiar, but its most interesting and powerful applications come precisely from the fact that it's not block-based but function-based. In the section on `panic` and `recover` we'll see another example of its possibilities.

对于习惯了其他语言进行块级资源管理的开发者来说，可能 `defer` 看起来有点奇特，但是它最有意思并且最有力的应用恰好因为 `defer` 不基于块，而是基于函数来使用。在 `panic` 和 `recover` 小节我们会看到它的一些其他使用例子。

## Data（数据）

### Allocation with `new` （分配器 new）

Go has two allocation primitives, the built-in functions `new` and `make`. They do different things and apply to different types, which can be confusing, but the rules are simple. Let's talk about `new` first. It's a built-in function that allocates memory, but unlike its namesakes in some other languages it does not *initialize* the memory, it only *zeros* it. That is, `new(T)` allocates zeroed storage for a new item of type `T` and returns its address, a value of type `*T`. In Go terminology, it returns a pointer to a newly allocated zero value of type `T`.

Go 有两个内存分配的命令，分别是内建的函数 `new` 和 `make`。他们做不同的事情，作用的类型也不一样，很容易混淆，不过区分的规则很简单。这里首先讨论 `new` 函数。`new` 是一个用来分配内存的内建函数，但是不像在其他编程语言中的同名函数那样，它并不**初始化**内存，只是让内存块全部数据置零。换句话说，`new(T)` 分配一块全是 0 的内存给类型为 `T` 的新条目，然后返回这个条目的地址（返回值的类型为 `*T`）。在 Go 编程术语中，`new` 返回类型 `T` 的指针，这个指针指向一块数据类型为 `T` 且全是零值的内存。

Since the memory returned by `new` is zeroed, it's helpful to arrange when designing your data structures that the zero value of each type can be used without further initialization. This means a user of the data structure can create one with `new` and get right to work. For example, the documentation for `bytes.Buffer` states that "the zero value for `Buffer` is an empty buffer ready to use." Similarly, `sync.Mutex` does not have an explicit constructor or `Init` method. Instead, the zero value for a `sync.Mutex` is defined to be an unlocked mutex.

因为 `new` 返回的是被置零的内存，因此如果你的数据结构可以直接使用这些零值而不需要其他的初始化的时候，使用 `new` 就很方便。这意味着通过 `new` 创建一个这种数据结构的实例，接着就可以用它来做事情了。比如，`bytes.Buffer` 的文档中就注明了：`Buffer` 的零值是一个使用就绪的空缓存。同样的，`sync.Mutex` 也并没有显式的构造器或者 `Init` 函数；相反，`sync.Mutex` 的零值被定义为一个没有上锁的互斥锁。

The zero-value-is-useful property works transitively. Consider this type declaration.

**零值即有用**的属性临时很有作用。考虑下面的类型：

```go
type SyncedBuffer struct {
    lock    sync.Mutex
    buffer  bytes.Buffer
}
```

Values of type `SyncedBuffer` are also ready to use immediately upon allocation or just declaration. In the next snippet, both `p` and `v` will work correctly without further arrangement.

`SyncedBuffer` 类型的值一旦被分配内存或者一旦声明后就可以使用了（不需要再花费其他精力去初始化它）。下面的代码片段里，不再需要其他的操作，`p` 和 `v` 都可以正确地工作，

```go
p := new(SyncedBuffer)  // type *SyncedBuffer
var v SyncedBuffer      // type  SyncedBuffer
```

### Constructors and composite literals（构造器与复合表达式）

Sometimes the zero value isn't good enough and an initializing constructor is necessary, as in this example derived from package `os`.

有时候类型的零值不能直接使用，此时初始化构造器就显得很有必要存在，比如下面从包 `os` 里抽取出来的一个例子：

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

There's a lot of boiler plate in there. We can simplify it using a *composite literal*, which is an expression that creates a new instance each time it is evaluated.

代码里有很多样板语句。我们可以通过**复合文本**来简化它；而**复合文本**每次执行时都会创建一个新的实例从而达成目的，使用方式如下：

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```

Note that, unlike in C, it's perfectly OK to return the address of a local variable; the storage associated with the variable survives after the function returns. In fact, taking the address of a composite literal allocates a fresh instance each time it is evaluated, so we can combine these last two lines.

可以注意到，不像 C 语言，在 Go 语言中可以返回一个临时变量的地址；函数返回后临时变量的内存会被保留（并不会被回收掉）。实际上，每当取复合文本的地址的时候都会分配一个新对象（TODO 分配到栈上？每次都分配一个？），因此我们可以把上面例子中最后两行代码写成 `return &File{fd, name, nil, 0}`。

```go
    return &File{fd, name, nil, 0}
```

The fields of a composite literal are laid out in order and must all be present. However, by labeling the elements explicitly as *field*`:`*value* pairs, the initializers can appear in any order, with the missing ones left as their respective zero values. Thus we could say

复合文本中的字段必须要按顺序排列且必须全部给出来。但是，通过 **字段 : 值** 的形式显式地指定元素及其值，初始化时就可以以任意顺序传入值，如果有字段没有指定值则默认被设置为其对应的零值，因此上面的代码还可以简化为 `return &File{fd: fd, name: name}`。

```go
    return &File{fd: fd, name: name}
```

As a limiting case, if a composite literal contains no fields at all, it creates a zero value for the type. The expressions `new(File)` and `&File{}` are equivalent.

作为一种限制，如果符合文本中不包含任何的字段，它会创建其类型的零值，此时 `new(File)` 和 `&File{}` 是等价的。

Composite literals can also be created for arrays, slices, and maps, with the field labels being indices or map keys as appropriate. In these examples, the initializations work regardless of the values of `Enone`, `Eio`, and `Einval`, as long as they are distinct.

符合文本还可以创建数组、切片、哈希 map，此时他的字段标签就变成了切片的索引或者 map 的键。在下面的例子中，只要 `Enone`, `Eio`, 和 `Einval`是不一样的，就可以进行正常的初始化。

```go
a := [...]string   {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
s := []string      {Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
m := map[int]string{Enone: "no error", Eio: "Eio", Einval: "invalid argument"}
```

### Allocation with `make`（分配器 make）

Back to allocation. The built-in function `make(T, args)` serves a purpose different from `new(T)`. It creates slices, maps, and channels only, and it returns an *initialized* (not *zeroed*) value of type `T` (not `*T`). The reason for the distinction is that these three types represent, under the covers, references to data structures that must be initialized before use. A slice, for example, is a three-item descriptor containing a pointer to the data (inside an array), the length, and the capacity, and until those items are initialized, the slice is `nil`. For slices, maps, and channels, `make` initializes the internal data structure and prepares the value for use. For instance,

```go
make([]int, 10, 100)
```

allocates an array of 100 ints and then creates a slice structure with length 10 and a capacity of 100 pointing at the first 10 elements of the array. (When making a slice, the capacity can be omitted; see the section on slices for more information.) In contrast, `new([]int)` returns a pointer to a newly allocated, zeroed slice structure, that is, a pointer to a `nil` slice value.

回到内存分配。内建函数 `make(T, args)` 的作用于 `new(T)` 的不同。`make` 函数只能用来创建切片、哈希 map 和 信道，并且返回一个类型为 `T` 且被初始化的值（内存不是全零）。区别的原因是，这三个类型的数据底层引用了其他的数据结构，而这些数据结构在使用前必须先初始化才可以使用。比如，切片是一个**三组件**的描述符，包含一个指向具体数据的指针（指向数组）、切片长度、切片容量（TODO 是不是能说明切片的线程安全性？）。在这些组件被初始化之前，切片的值是 `nil`。对于切片、map 和信道来说，`make` 初始化了他们底层的数据结构让他们的值可以使用。

比如 `make([]int, 10, 100)` 初始化了一个 100 个整数的数组，同时创建了一个长度为 10、容量为 100 、指向数组的前 10 个元素的切片（当创建切片的时候，其容量是可以省略的，见切片小节查看更多内容）。作为对比，`new([]int)` 返回了一个新创建且被置零的切片的指针，指向的是值为 `nil` 的切片值。

These examples illustrate the difference between `new` and `make`.
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

Remember that `make` applies only to maps, slices and channels and does not return a pointer. To obtain an explicit pointer allocate with `new` or take the address of a variable explicitly.

记住，`make` 只可以作用于 map、切片和信道，并且返回的不是指针。如果想得到指针，可以使用 `new` 或者显式取变量的地址获得。

### Arrays（数组）

Arrays are useful when planning the detailed layout of memory and sometimes can help avoid allocation, but primarily they are a building block for slices, the subject of the next section. To lay the foundation for that topic, here are a few words about arrays.

在设计内存的详细布局的时候**数组**会很有用，有时候数组还可以避免内存分配；不过数组的主要作用是给切片提供底层的数据块，这将在下一节讲。为了给切片做铺垫，这里简单介绍一下数组：

There are major differences between the ways arrays work in Go and C. In Go,

- Arrays are values. Assigning one array to another copies all the elements.
- In particular, if you pass an array to a function, it will receive a *copy* of the array, not a pointer to it.
- The size of an array is part of its type. The types `[10]int` and `[20]int` are distinct.

数组在 Go 和 C 中的主要区别是它的工作方式。在 Go 语言中，

- 数组是值；如果把一个数组赋值给另一个数组会把所有的元素复制一份；
- 如果一个函数的参数是数组，调用函数的时候得到的是数组的完全拷贝而不是它的指针；
- 数组的大小属于它的类型的一部分，类型 `[10]int` 和 `[20]int` 不一样。

The value property can be useful but also expensive; if you want C-like behavior and efficiency, you can pass a pointer to the array.

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

But even this style isn't idiomatic Go. Use slices instead.
但是在 Go 中这种风格的使用很不常见，更多情况是使用切片。

### Slices（切片）

Slices wrap arrays to give a more general, powerful, and convenient interface to sequences of data. Except for items with explicit dimension such as transformation matrices, most array programming in Go is done with slices rather than simple arrays.

切片封装了数组，并给出操作一个数据序列的更通用、更有效也更方便的接口。除了具有明确维数的项（比如变换矩阵），在 Go 中大部分都是使用切片来进行数组编程，不会直接使用数组。

Slices hold references to an underlying array, and if you assign one slice to another, both refer to the same array. If a function takes a slice argument, changes it makes to the elements of the slice will be visible to the caller, analogous to passing a pointer to the underlying array. A `Read` function can therefore accept a slice argument rather than a pointer and a count; the length within the slice sets an upper limit of how much data to read. Here is the signature of the `Read` method of the `File` type in package `os`:

切片保存了底层数组的引用，如果把一个切片赋值给另一个切片，两个切片会指向同一个数组。如果函数把切片作为参数，且在函数内部修改了切片的某个元素，调用者将会感知到这种变化，效果就好像是传入了底层数组的指针一样。因此，`Read` 函数可以接收一个切片类型的参数而不再需要指针和数量；切片的长度是指能够读取的元素的最大数量。下面是包 `os` 中类型 `File` 的 `Read` 方法的声明签名：

```go
func (f *File) Read(buf []byte) (n int, err error)
```

The method returns the number of bytes read and an error value, if any. To read into the first 32 bytes of a larger buffer `buf`, *slice* (here used as a verb) the buffer.

`Read` 方法返回了读取的字节数量和一个错误值（如果存在错误）。如果想读取一个很大的缓存 `buf` 的前 32 个字节的数据，可以把缓存**切**一部分出来：

```go
    n, err := f.Read(buf[0:32])
```

Such slicing is common and efficient. In fact, leaving efficiency aside for the moment, the following snippet would also read the first 32 bytes of the buffer.

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

The length of a slice may be changed as long as it still fits within the limits of the underlying array; just assign it to a slice of itself. The *capacity* of a slice, accessible by the built-in function `cap`, reports the maximum length the slice may assume. Here is a function to append data to a slice. If the data exceeds the capacity, the slice is reallocated. The resulting slice is returned. The function uses the fact that `len` and `cap` are legal when applied to the `nil` slice, and return 0.

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

We must return the slice afterwards because, although `Append` can modify the elements of `slice`, the slice itself (the run-time data structure holding the pointer, length, and capacity) is passed by value.

上面的源码中，当使用 `Apend` 函数的时候最后必须要返回一个切片，虽然 `Append` 会修改 `slice` 的元素，但是 slice （运行时数据结构包含了指针、大小、容量）是以传值的方式到达函数的，切片内在的数组指针、大小、容量改变后必须通过返回值通知调用方。

The idea of appending to a slice is so useful it's captured by the `append` built-in function. To understand that function's design, though, we need a little more information, so we'll return to it later.

给切片追加元素的点子非常有用，内建的函数 `append` 的实现就是同样的逻辑。如果想理解 `append` 函数的设计，我们还需要更多的知识，后面的小节再继续聊。

### Two-dimensional slices（两维切片）

Go's arrays and slices are one-dimensional. To create the equivalent of a 2D array or slice, it is necessary to define an array-of-arrays or slice-of-slices, like this:

Go 的数组和切片都是一维结构。如果想创建一个等效的二维数组或者切片，必须定义一个数组的数组或者切片的切片，就像下面的定义：

```go
type Transform [3][3]float64  // A 3x3 array, really an array of arrays.
type LinesOfText [][]byte     // A slice of byte slices.
```

Because slices are variable-length, it is possible to have each inner slice be a different length. That can be a common situation, as in our `LinesOfText` example: each line has an independent length.

因为切片的长度可变，因此高维数组的切片可以有不同的长度，这种情况还是很常见的，比如下面 `LinesOfText` 的例子：其中每行有独立的长度。

```go
text := LinesOfText{
	[]byte("Now is the time"),
	[]byte("for all good gophers"),
	[]byte("to bring some fun to the party."),
}
```

Sometimes it's necessary to allocate a 2D slice, a situation that can arise when processing scan lines of pixels, for instance. There are two ways to achieve this. One is to allocate each slice independently; the other is to allocate a single array and point the individual slices into it. Which to use depends on your application. If the slices might grow or shrink, they should be allocated independently to avoid overwriting the next line; if not, it can be more efficient to construct the object with a single allocation. For reference, here are sketches of the two methods. First, a line at a time:

有时候分配一个二维的切片是必要的，比如当需要扫描每行的像素的时候就需要二维切片。有两种方式来完成这个目的。其中一种方式是分配独立的切片，另一种方式是分配一个独立的数组然后把每个切片映射到这个数组。使用哪种方式取决于你的需求。如果切片可能会增长或者收缩，为了避免行与行之间的数据应该分配独立的切片；否则就可以一次分配然后构造各个对象，这种方式会更高效。为了说明问题，下面给出了两种方式的用法，首先是每次一行的方式：

```go
// Allocate the top-level slice.
picture := make([][]uint8, YSize) // One row per unit of y.
// Loop over the rows, allocating the slice for each row.
for i := range picture {
	picture[i] = make([]uint8, XSize)
}
```

And now as one allocation, sliced into lines:
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

### Maps（映射）

Maps are a convenient and powerful built-in data structure that associate values of one type (the *key*) with values of another type (the *element* or *value*). The key can be of any type for which the equality operator is defined, such as integers, floating point and complex numbers, strings, pointers, interfaces (as long as the dynamic type supports equality), structs and arrays. Slices cannot be used as map keys, because equality is not defined on them. Like slices, maps hold references to an underlying data structure. If you pass a map to a function that changes the contents of the map, the changes will be visible in the caller.

映射（map）是一种方便且强大的内建数据结构，它把一种类型的值（键）和另一种类型的值（元素或值）关联起来。任意一种可以比较是否相等的类型都可以作为键类型，比如整数、浮点数、复数、字符串、指针、接口（只要它的动态类型可以判定相等性）、结构和数组。切片不可以作为映射的键，因为不能判定两个切片是否相等。和切片类似，映射持有底层数据结构的引用。当给函数传递了一个映射，如果函数修改了映射里的内容，函数调用者是可以感知到其改变的。

Maps can be constructed using the usual composite literal syntax with colon-separated key-value pairs, so it's easy to build them during initialization.

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

Assigning and fetching map values looks syntactically just like doing the same for arrays and slices except that the index doesn't need to be an integer.

给哈希的值赋值和从哈希中拿数据的语法和数组以及切片的语法类似，只不过哈希的键可能不是整形。

```go
offset := timeZone["EST"]
```

An attempt to fetch a map value with a key that is not present in the map will return the zero value for the type of the entries in the map. For instance, if the map contains integers, looking up a non-existent key will return `0`. A set can be implemented as a map with value type `bool`. Set the map entry to `true` to put the value in the set, and then test it by simple indexing.

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

Sometimes you need to distinguish a missing entry from a zero value. Is there an entry for `"UTC"` or is that 0 because it's not in the map at all? You can discriminate with a form of multiple assignment.

因为在哈希中不存在的键值的值是零值，而有时候可能需要区分是键不存在还是对应的键的值确实是零值。哈希中有 `UTC` 的键吗？返回 `0` 是以为它不存在与哈希吗？其实可以通过一种多值赋值的方式来区分这种情况。 

```go
var seconds int
var ok bool
seconds, ok = timeZone[tz]
```

For obvious reasons this is called the “comma ok” idiom. In this example, if `tz` is present, `seconds` will be set appropriately and `ok` will be true; if not, `seconds` will be set to zero and `ok` will be false. Here's a function that puts it together with a nice error report:

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

To test for presence in the map without worrying about the actual value, you can use the [blank identifier](https://golang.google.cn/doc/effective_go.html#blank) (`_`) in place of the usual variable for the value.

如果只是想知道哈希中是否存在某个元素而不需要知道其具体的值，可以用**空标识符**替代接收元素值的变量的位置。

```go
_, present := timeZone[tz]
```

To delete a map entry, use the `delete` built-in function, whose arguments are the map and the key to be deleted. It's safe to do this even if the key is already absent from the map.

如果想删除哈希中的某个元素，可以使用内建函数 `delete`；它的参数是哈希和对应要删除的键。这个函数可以在键存在的时候调用也可以在键不存在的时候调用，不会报错 :)

```go
delete(timeZone, "PDT")  // Now on Standard Time
```

### Printing（打印）

Formatted printing in Go uses a style similar to C's `printf` family but is richer and more general. The functions live in the `fmt` package and have capitalized names: `fmt.Printf`, `fmt.Fprintf`, `fmt.Sprintf` and so on. The string functions (`Sprintf` etc.) return a string rather than filling in a provided buffer.

在 Go 中格式化的输出使用类似 C 中的 `printf` 家族函数，但是功能更丰富和通用。这些函数在包 `fmt` 包中，且有首字母大写的名字：`fmt.Printf`, `fmt.Fprintf`, `fmt.Sprintf`，等等。字符串函数（`Sprintf`等）会返回一个字符串而不是填充某个缓存。 

You don't need to provide a format string. For each of `Printf`, `Fprintf` and `Sprintf` there is another pair of functions, for instance `Print` and `Println`. These functions do not take a format string but instead generate a default format for each argument. The `Println` versions also insert a blank between arguments and append a newline to the output while the `Print` versions add blanks only if the operand on neither side is a string. In this example each line produces the same output.

你不需要提供格式化字符串。对于 `Printf`, `Fprintf` 和 `Sprintf` 来说，每个都有对应类似的函数，比如 `Print` 和 `Println`。后面的这些函数不需要格式化字符串，而是给每个参数生成一个默认的格式。函数 `Println` 会给参数之间插入一个空白符，并且在最后向输出追加一个行尾符；函数 `Print` 则会在非字符串之间添加一个空白符。下面的例子总每行都会产生相同的输出。

```go
fmt.Printf("Hello %d\n", 23)
fmt.Fprint(os.Stdout, "Hello ", 23, "\n")
fmt.Println("Hello", 23)
fmt.Println(fmt.Sprint("Hello ", 23))
```

The formatted print functions `fmt.Fprint` and friends take as a first argument any object that implements the `io.Writer` interface; the variables `os.Stdout` and `os.Stderr` are familiar instances.

格式化打印函数 `fmt.Fprint` 和它的友函数的第一个参数是实现了 `io.Writer` 接口的任意对象；变量 `os.Stdout` 和 `os.Stderr` 是相似的实例。

Here things start to diverge from C. First, the numeric formats such as `%d` do not take flags for signedness or size; instead, the printing routines use the type of the argument to decide these properties.

这里开始有一些和 C 不同的事情。首先，数字化格式符，比如 `%d` ，不接受符号类型或大小；相反，打印例程通过类型来决定这些属性。

```go
var x uint64 = 1<<64 - 1
fmt.Printf("%d %x; %d %x\n", x, x, int64(x), int64(x))
```

prints

```go
18446744073709551615 ffffffffffffffff; -1 -1
```

If you just want the default conversion, such as decimal for integers, you can use the catchall format `%v` (for “value”); the result is exactly what `Print` and `Println` would produce. Moreover, that format can print *any* value, even arrays, slices, structs, and maps. Here is a print statement for the time zone map defined in the previous section.

如果只是想要默认的转换，比如小数到整数的转换，你可以使用万能格式符 `%v`（value 的首字母）；结果就是调用 `Print` 和 `Println` 会产生的值。另外，`%v` 葛师傅可以用来打印任意类型的值，甚至包含数组、切片、结构和哈希 map。下面的离子打印了之前小节定义的时区哈希值。

```go
fmt.Printf("%v\n", timeZone)  // or just fmt.Println(timeZone)
```

which gives output:
上面的代码会输出下面的内容：

```go
map[CST:-21600 EST:-18000 MST:-25200 PST:-28800 UTC:0]
```

For maps, `Printf` and friends sort the output lexicographically by key.

对于哈希， `Printf` 及其友函数会按照索引的字母排序进行输出。

When printing a struct, the modified format `%+v` annotates the fields of the structure with their names, and for any value the alternate format `%#v` prints the value in full Go syntax.

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

prints

```go
&{7 -2.35 abc   def}
&{a:7 b:-2.35 c:abc     def}
&main.T{a:7, b:-2.35, c:"abc\tdef"}
map[string]int{"CST":-21600, "EST":-18000, "MST":-25200, "PST":-28800, "UTC":0}
```

(Note the ampersands.) That quoted string format is also available through `%q` when applied to a value of type `string` or `[]byte`. The alternate format `%#q` will use backquotes instead if possible. (The `%q` format also applies to integers and runes, producing a single-quoted rune constant.) Also, `%x` works on strings, byte arrays and byte slices as well as on integers, generating a long hexadecimal string, and with a space in the format (`% x`) it puts spaces between the bytes.

注意 **&** 符号。当需要打印的值是 `string` 或 `[]byte` 时，双引号括起来的字符串格式可以通过 `%q` 获得；而 `%#q` 会给字符串加上反引号。（`%q` 也可以格式化整形和符文类型，生成单引号括起来的符文常量。）与此同时， `%x` 可以作用于字符串、字节数组、字节切片，如果传入的是一个整数会生成一个长十六进制字符串；如果加一个空格，`% x` 会在每个字节之间加一个空格。

Another handy format is `%T`, which prints the *type* of a value.
另一个便利的格式符是 `%T`，它会把值的类型打印出来。`fmt.Printf("%T\n", timeZone)` 的打印结果是 `map[string]int`。

```go
fmt.Printf("%T\n", timeZone)
```

prints【打印】

```go
map[string]int
```

If you want to control the default format for a custom type, all that's required is to define a method with the signature `String() string` on the type. For our simple type `T`, that might look like this.

如果想控制自定义类型的打印格式，只需要给相应的类型定义一个签名是 `String() string` 的方法。对于简单类型 `T`，其可能的方法如下：

```go
func (t *T) String() string {
    return fmt.Sprintf("%d/%g/%q", t.a, t.b, t.c)
}
fmt.Printf("%v\n", t)
```

to print in the format
上面的代码会打印出下面的结果：

```go
7/-2.35/"abc\tdef"
```

(If you need to print *values* of type `T` as well as pointers to `T`, the receiver for `String` must be of value type; this example used a pointer because that's more efficient and idiomatic for struct types. See the section below on [pointers vs. value receivers](https://golang.google.cn/doc/effective_go.html#pointers_vs_values) for more information.)

（如果你想打印类型 `T` 的值以及 `T` 类型的指针，`String` 方法的接收者必须是值类型（什么鬼？如果是值的话其指针岂不是临时变量的了？）；上面的例子使用了指针，因为对于结构体来说指针更高效且惯用。可以查看《[指针接收器与值接收器](#pointers_vs_values)》一节了解更多内容）

Our `String` method is able to call `Sprintf` because the print routines are fully reentrant and can be wrapped this way. There is one important detail to understand about this approach, however: don't construct a `String` method by calling `Sprintf` in a way that will recur into your `String` method indefinitely. This can happen if the `Sprintf` call attempts to print the receiver directly as a string, which in turn will invoke the method again. It's a common and easy mistake to make, as this example shows.

自定义的 `String` 函数可以调用 `Sprintf` 函数，因为打印的例程完全可重入且可以以这种方式封装起来。但是，这里有一个很重要的细节需要理解：不能构造这样的 `String` 方法——里面以某种方式调用 `Sprintf` 函数，然后 `Sprintf` 函数在调用定义的 `String` 方法，如此往复形成无限循环。如果 `Sprintf` 调用尝试以字符串的形式直接打印接受者就可能会触发这个问题，无限触发自定义的 `String` 函数。这个错误很容易犯，就像下面的例子：

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

It's also easy to fix: convert the argument to the basic string type, which does not have the method.

上面的问题也很容易修复：只需要把参数转换为基础的字符串类型即可，这样就不会循环调用 `String` 方法了，如下面的代码所示：

```go
type MyString string
func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

In the [initialization section](https://golang.google.cn/doc/effective_go.html#initialization) we'll see another technique that avoids this recursion.

在“初始化一节”我们还会看到另一个避免这种循环调用的技术。

Another printing technique is to pass a print routine's arguments directly to another such routine. The signature of `Printf` uses the type `...interface{}` for its final argument to specify that an arbitrary number of parameters (of arbitrary type) can appear after the format.

另一种打印技术是把一个打印例程的参数直接传给另一个类似的例程。函数 `Printf` 的声明中使用了类型 `...interface{}` 作为它的最后一个参数，因此可以在格式字符串后面传递任意数量的任意类型的参数。

```go
func Printf(format string, v ...interface{}) (n int, err error) {
```

Within the function `Printf`, `v` acts like a variable of type `[]interface{}` but if it is passed to another variadic function, it acts like a regular list of arguments. Here is the implementation of the function `log.Println` we used above. It passes its arguments directly to `fmt.Sprintln` for the actual formatting.

在函数 `Printf` 中，`v` 就像是类型 `[]interface{}` 的值，但是如果它被传递给另一个不定参数的函数，它会表现得像一个正常的参数列表。下面的代码是 `log.Println` 的一种实现，它把自己的参数直接传给了`fmt.Sprintln` 进行实际的格式输出。

```go
// Println prints to the standard logger in the manner of fmt.Println.
func Println(v ...interface{}) {
    std.Output(2, fmt.Sprintln(v...))  // Output takes parameters (int, string)
}
```

We write `...` after `v` in the nested call to `Sprintln` to tell the compiler to treat `v` as a list of arguments; otherwise it would just pass `v` as a single slice argument.

上面的代码示例中，当在内部调用 `Sprintln` 时，我们在 `v` 后添加了 `...` 从而把 `v` 视为一个参数的列表；如果没有这三个点，会把 `v` 作为单独的一个切片参数传递给 `Sprintln`。

There's even more to printing than we've covered here. See the `godoc` documentation for package `fmt` for the details.

其实除了这里讨论的内容外还有很多打印相关的内容。可以查看 `godoc` 中 `fmt` 的文档了解更多的内容。

By the way, a `...` parameter can be of a specific type, for instance `...int` for a min function that chooses the least of a list of integers:

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

Now we have the missing piece we needed to explain the design of the `append` built-in function. The signature of `append` is different from our custom `Append` function above. Schematically, it's like this:

```go
func append(slice []T, elements ...T) []T
```

接下来我们继续聊内建函数 `append`的相关内容。内建函数 `append` 的声明方式和我们上面自定义的 `Append` 函数不同，如下面的形式：

```go
func append(slice []T, elements ...T) []T
```

where *T* is a placeholder for any given type. You can't actually write a function in Go where the type `T` is determined by the caller. That's why `append` is built in: it needs support from the compiler.

这里的 *T* 是一个占位符，用来表示任意给定的类型。实际上，在 Go 语言中我们不能写一个带泛型的函数，这也是 `append` 是内建函数的原因——它需要编译器层面的支持。

What `append` does is append the elements to the end of the slice and return the result. The result needs to be returned because, as with our hand-written `Append`, the underlying array may change. This simple example

`append` 做的事情就是在一个切片的尾部追加元素，然后返回追加以后的结果。`append` 函数必须要返回结果，原因就像我们手写的 `Append` 函数一样，在追加过程中，切片底层的数组可能会发生变化，此时必须通过返回结果来通知这种变化。

```go
x := []int{1,2,3}
x = append(x, 4, 5, 6)
fmt.Println(x)
```

prints `[1 2 3 4 5 6]`. So `append` works a little like `Printf`, collecting an arbitrary number of arguments.

上面的例子打印出 `[1 2 3 4 5 6]`。一定程度上可以认为 `append` 的传参方式和 `Printf` 有点像，可以传入任意多数目的参数。

But what if we wanted to do what our `Append` does and append a slice to a slice? Easy: use `...` at the call site, just as we did in the call to `Output` above. This snippet produces identical output to the one above.

如果想使用 `append` 函数把一个切片追加到另一个切片上面，就必须使用 `...` 运算符，类似上面中 `Output` 展示的用法。下面的代码输出的内容和上面的代码输出的内容是一样的。

```go
x := []int{1,2,3}
y := []int{4,5,6}
x = append(x, y...)
fmt.Println(x)
```

Without that `...`, it wouldn't compile because the types would be wrong; `y` is not of type `int`.

在上面的例子中，如果不带 `...`，代码时没有办法编译通过的，因为 `y` 的类型是一个切片而不是需要的 `int`。

## Initialization（初始化）

Although it doesn't look superficially very different from initialization in C or C++, initialization in Go is more powerful. Complex structures can be built during initialization and the ordering issues among initialized objects, even among different packages, are handled correctly.

虽然表面上看 Go 的初始化与 C/C++ 的初始化没有太大区别，但是 Go 的初始化方式更强大。在初始化过程中可以直接创建复合结构，且正确地处理了初始化对象之间的排序问题，即使在不同的包之间也是如此。

### Constants（常量）

Constants in Go are just that—constant. They are created at compile time, even when defined as locals in functions, and can only be numbers, characters (runes), strings or booleans. Because of the compile-time restriction, the expressions that define them must be constant expressions, evaluatable by the compiler. For instance, `1<<3` is a constant expression, while `math.Sin(math.Pi/4)` is not because the function call to `math.Sin` needs to happen at run time.

在 Go 中，常量仅仅指常量。常量在编译时就被创建，甚至函数内的本地常量也是如此；常量只能是数字、字符列表（符文列表）、字符串或布尔值。受到编译时的限制，定义常量的表达式必须是编译器可以执行的常量表达式。比如 `1<<3` 是一个常量表达式，`math.Sin(math.Pi/4)` 不是常量表达式，因为 `math.Sin` 需要在运行时才可以调用。

In Go, enumerated constants are created using the `iota` enumerator. Since `iota` can be part of an expression and expressions can be implicitly repeated, it is easy to build intricate sets of values.

在 Go 中，枚举型常量通过使用 `iota` 枚举器来创建。鉴于 `iota` 可以作为表达式的一部分，同时表达式可以隐式地重复，因此很方便创建复杂的值集合。

```
type ByteSize float64

const (
    _           = iota // ignore first value by assigning to blank identifier
    KB ByteSize = 1 << (10 * iota)
    MB
    GB
    TB
    PB
    EB
    ZB
    YB
)
```

The ability to attach a method such as `String` to any user-defined type makes it possible for arbitrary values to format themselves automatically for printing. Although you'll see it most often applied to structs, this technique is also useful for scalar types such as floating-point types like `ByteSize`.

可以给任何用户定义的类型添加方法（比如 `String` 方法），这种能力使得任意值被打印时都能定义自己的输出格式。最常见的是给结构体添加方法，但是这项技术也可以用来给标量类型（比如 浮点类型 `ByteSize`）添加方法。

```
func (b ByteSize) String() string {
    switch {
    case b >= YB:
        return fmt.Sprintf("%.2fYB", b/YB)
    case b >= ZB:
        return fmt.Sprintf("%.2fZB", b/ZB)
    case b >= EB:
        return fmt.Sprintf("%.2fEB", b/EB)
    case b >= PB:
        return fmt.Sprintf("%.2fPB", b/PB)
    case b >= TB:
        return fmt.Sprintf("%.2fTB", b/TB)
    case b >= GB:
        return fmt.Sprintf("%.2fGB", b/GB)
    case b >= MB:
        return fmt.Sprintf("%.2fMB", b/MB)
    case b >= KB:
        return fmt.Sprintf("%.2fKB", b/KB)
    }
    return fmt.Sprintf("%.2fB", b)
}
```

The expression `YB` prints as `1.00YB`, while `ByteSize(1e13)` prints as `9.09TB`.

表达式 `YB` 会打印 `1.00YB`， 而 `ByteSize(1e13)` 会打印 `9.09TB`。

The use here of `Sprintf` to implement `ByteSize`'s `String` method is safe (avoids recurring indefinitely) not because of a conversion but because it calls `Sprintf` with `%f`, which is not a string format: `Sprintf` will only call the `String` method when it wants a string, and `%f` wants a floating-point value.

上面的代码通过使用 `Sprintf` 来实现 `ByteSize` 的 `String` 方法是安全（避免循环调用）的，倒不是因为类型转换，而是因为使用了 `%f` 格式符，它不是一个字符串格式化符：`Sprintf` 只在需要字符串类型的时候才会调用 `String` 方法，上面代码中 `%f` 希望的是浮点值。

### Variables（变量）

Variables can be initialized just like constants but the initializer can be a general expression computed at run time.

变量会向常量那样被初始化，但是初始化表达式可以是在运行时进行计算的通用表达式。

```
var (
    home   = os.Getenv("HOME")
    user   = os.Getenv("USER")
    gopath = os.Getenv("GOPATH")
)
```

### The init function（init 函数）

Finally, each source file can define its own niladic `init` function to set up whatever state is required. (Actually each file can have multiple `init` functions.) And finally means finally: `init` is called after all the variable declarations in the package have evaluated their initializers, and those are evaluated only after all the imported packages have been initialized.

每个源文件可以定义它自己的 `init` 函数用来初始化需要的状态。（事实上每个文件可以有多个 `init` 函数）。最后真的意味着最后：`init` 会在包中所有的变量声明被执行后再调用，而且只在所有被导入的包初始化以后才会调用。

Besides initializations that cannot be expressed as declarations, a common use of `init` functions is to verify or repair correctness of the program state before real execution begins.

在初始化时不能声明新东西，`init` 函数大都是为了在真实运行开始前校验或者修复程序的状态。

```
func init() {
    if user == "" {
        log.Fatal("$USER not set")
    }
    if home == "" {
        home = "/home/" + user
    }
    if gopath == "" {
        gopath = home + "/go"
    }
    // gopath may be overridden by --gopath flag on command line.
    flag.StringVar(&gopath, "gopath", gopath, "override default GOPATH")
}
```

## Methods（方法）

### Pointers vs. Values（指针与值）

As we saw with `ByteSize`, methods can be defined for any named type (except a pointer or an interface); the receiver does not have to be a struct.

就像我们看到的 `ByteSize` 那样，可以给任何命名的类型（除了指针和接口）定义方法；接收器不必一定是结构。

In the discussion of slices above, we wrote an `Append` function. We can define it as a method on slices instead. To do this, we first declare a named type to which we can bind the method, and then make the receiver for the method a value of that type.

在切片一节的讨论中，我们编写了一个 `Append` 函数。其实我们可以把它定义为切片的一个方法。为了达成目的，我们首先需要声明一个命名的类型从而方便把函数绑定到这个类型上，然后就可以给相应的方法传递一个相应类型的接收者。

```go
type ByteSlice []byte

func (slice ByteSlice) Append(data []byte) []byte {
    // Body exactly the same as the Append function defined above.
}
```

This still requires the method to return the updated slice. We can eliminate that clumsiness by redefining the method to take a *pointer* to a `ByteSlice` as its receiver, so the method can overwrite the caller's slice.

上面的函数仍然需要方法返回更新后的切片。我们可以把接受者改为`ByteSlice`的**指针**来修改上面笨拙的写法，这样就可以复写调用者里的切片了，如下所示：

```go
func (p *ByteSlice) Append(data []byte) {
    slice := *p
    // Body as above, without the return.
    *p = slice
}
```

In fact, we can do even better. If we modify our function so it looks like a standard `Write` method, like this,

实际上我们可以做的更好。如果我们模仿标准的 `Write` 方法继续修改我们的函数，如下：

```go
func (p *ByteSlice) Write(data []byte) (n int, err error) {
    slice := *p
    // Again as above.
    *p = slice
    return len(data), nil
}
```

then the type `*ByteSlice` satisfies the standard interface `io.Writer`, which is handy. For instance, we can print into one.

如此，类型 `*ByteSlice` 实现了便利的标准接口 `io.Writer`。比如下面的代码：

```go
    var b ByteSlice
    fmt.Fprintf(&b, "This hour has %d days\n", 7)
```

We pass the address of a `ByteSlice` because only `*ByteSlice` satisfies `io.Writer`. The rule about pointers vs. values for receivers is that value methods can be invoked on pointers and values, but pointer methods can only be invoked on pointers.

上面的代码中我们传递了 `ByteSlice` 的地址，因为只有 `*ByteSlice` 实现了 `io.Writer` 接口。上面关于**指针接受者和值接受者**的规则是：**值方法可以在指针和值上面触发调用，但是指针方法只能被指针触发调用**。

This rule arises because pointer methods can modify the receiver; invoking them on a value would cause the method to receive a copy of the value, so any modifications would be discarded. The language therefore disallows this mistake. There is a handy exception, though. When the value is addressable, the language takes care of the common case of invoking a pointer method on a value by inserting the address operator automatically. In our example, the variable `b` is addressable, so we can call its `Write` method with just `b.Write`. The compiler will rewrite that to `(&b).Write` for us.

之所以有上面的规则，是因为指针方法可以修改接受者；如果在值接受者上面触发调用，会导致方法接收一个值的拷贝，因此任何的修改都会被丢弃；正因为这个原因 Go 语言层面上不允许这种错误。不过，这里有一个特例。如果值是可取址的，当在一个值上调用指针方法的时候，Go 语言会照顾到这种常见的用法在值上自动添加一个取址运算符。比如上面的例子中，`b` 是可取址的，因此我们可以通过 `b.Write` 调用它的 `Write` 方法，编译器会为我们把它重写为 `(&b).Write`。

By the way, the idea of using `Write` on a slice of bytes is central to the implementation of `bytes.Buffer`.

顺便一提，在字节的切片上使用 `Write` 是 `bytes.Buffer` 的核心实现方式。

## Interfaces and other types（接口与其他类型）

### Interfaces（接口）

Interfaces in Go provide a way to specify the behavior of an object: if something can do *this*, then it can be used *here*. We've seen a couple of simple examples already; custom printers can be implemented by a `String` method while `Fprintf` can generate output to anything with a `Write` method. Interfaces with only one or two methods are common in Go code, and are usually given a name derived from the method, such as `io.Writer` for something that implements `Write`.

在 Go 中，可以用接口来指定一个对象的行为：如果一个东西可以做**这个**，那么它可以用在**那儿**。之前已经有看过几个简单例子；实现 `String` 方法可以定义类型的打印格式，`Fprintf` 可以向任何实现了 `Write` 方法的类型打印输出。在 Go 源码中，只有一两个方法的接口有很多，一般情况下这些接口的名称就衍生与他们的方法，比如 `io.Writer` 接口就有一个 `Write` 方法。

A type can implement multiple interfaces. For instance, a collection can be sorted by the routines in package `sort` if it implements `sort.Interface`, which contains `Len()`, `Less(i, j int) bool`, and `Swap(i, j int)`, and it could also have a custom formatter. In this contrived example `Sequence` satisfies both.

一个类型可以实现多个接口。比如，包 `sort` 中的例程可以用来给实现了 `sort.Interface` 接口的集合排序，也就是需要集合类型实现 `Len()`, `Less(i, j int) bool`, 和 `Swap(i, j int)` 方法；当然集合可以有自定义的输出格式。下面的 `Sequence` 同时满足两个接口：

```go
type Sequence []int

// Methods required by sort.Interface.
func (s Sequence) Len() int {
    return len(s)
}
func (s Sequence) Less(i, j int) bool {
    return s[i] < s[j]
}
func (s Sequence) Swap(i, j int) {
    s[i], s[j] = s[j], s[i]
}

// Copy returns a copy of the Sequence.
func (s Sequence) Copy() Sequence {
    copy := make(Sequence, 0, len(s))
    return append(copy, s...)
}

// Method for printing - sorts the elements before printing.
func (s Sequence) String() string {
    s = s.Copy() // Make a copy; don't overwrite argument.
    sort.Sort(s)
    str := "["
    for i, elem := range s { // Loop is O(N²); will fix that in next example.
        if i > 0 {
            str += " "
        }
        str += fmt.Sprint(elem)
    }
    return str + "]"
}
```

### Conversions（类型转换）

The `String` method of `Sequence` is recreating the work that `Sprint` already does for slices. (It also has complexity O(N²), which is poor.) We can share the effort (and also speed it up) if we convert the `Sequence` to a plain `[]int` before calling `Sprint`.

`Sequence` 的 `String` 方法一直在重复做 `Sprint` 做完的事情（复杂度为 O(N²)）。如果调用 `Sprint` 前如果我们把 `Sequence` 转换为普通的 `[]int` 类型的值，如此可以避免很多的重复工作（同时会加快速度）。

```go
func (s Sequence) String() string {
    s = s.Copy()
    sort.Sort(s)
    return fmt.Sprint([]int(s))
}
```

This method is another example of the conversion technique for calling `Sprintf` safely from a `String` method. Because the two types (`Sequence` and `[]int`) are the same if we ignore the type name, it's legal to convert between them. The conversion doesn't create a new value, it just temporarily acts as though the existing value has a new type. (There are other legal conversions, such as from integer to floating point, that do create a new value.)

这个方法是另一个在 `String` 方法中安全调用 `Sprintf` 函数的例子（不会触发循环调用）。因为两个类型（`Sequence` 和 `[]int`）只有类型名不同其他都是一样的，因此可以在它们之间进行转换。类型转换并不会创建新的值，它只是临时把一个存在的值作为另一种类型的值来看待（还有一些类型转换也是合法的，比如从整数转换成为浮点数，不过这种转换会产生一个新的值。）

It's an idiom in Go programs to convert the type of an expression to access a different set of methods. As an example, we could use the existing type `sort.IntSlice` to reduce the entire example to this:

在 Go 程序中，可以通过类型转换的方式获取新类型上的方法的调用权限，这种方式算是一种惯用手法吧。比如，我们可以使用已经存在的类型 `sort.IntSlice` 来减少代码量：

```go
type Sequence []int

// Method for printing - sorts the elements before printing
func (s Sequence) String() string {
    s = s.Copy()
    sort.IntSlice(s).Sort()
    return fmt.Sprint([]int(s))
}
```

Now, instead of having `Sequence` implement multiple interfaces (sorting and printing), we're using the ability of a data item to be converted to multiple types (`Sequence`, `sort.IntSlice` and `[]int`), each of which does some part of the job. That's more unusual in practice but can be effective.

上面的代码，可以允许我们不用实现 sorting 和 printing 接口，我们把 `Sequence` 转换为不同的类型从而使用每种类型具有的一些功能，每种转换完成一部分的工作，一起打成最后的效果。这种用法可能不同寻常，但是很高效。

### Interface conversions and type assertions（接口转换与类型断言）

[Type switches](https://golang.google.cn/doc/effective_go.html#type_switch) are a form of conversion: they take an interface and, for each case in the switch, in a sense convert it to the type of that case. Here's a simplified version of how the code under `fmt.Printf` turns a value into a string using a type switch. If it's already a string, we want the actual string value held by the interface, while if it has a `String` method we want the result of calling the method.

类型 switch 是类型转换的一种方式：他们接收一个接口变量，然后在每个 case 里把接口变量转换为对应的类型。下面的代码是简化版的 `fmt.Printf` 方法，用来展示如何通过类型 switch 把一个接口变量转换成字符串。如果接口变量已经是一个字符串，只需要把接口变量的真实的字符串值返回就可以了；否则如果它实现了 `String` 方法（实现了 `Stringer` 接口），我们可以通过调用这个方法并返回其结果。

```go
type Stringer interface {
    String() string
}

var value interface{} // Value provided by caller.
switch str := value.(type) {
case string:
    return str
case Stringer:
    return str.String()
}
```

The first case finds a concrete value; the second converts the interface into another interface. It's perfectly fine to mix types this way.

上面的代码中，第一个 case 直接使用其底层数据；第二个 case 则把接口转换成另一个接口（`interface{}` 转换为 `Stringer`），在这种情况下把类型杂糅在一起使用是可行的。

What if there's only one type we care about? If we know the value holds a `string` and we just want to extract it? A one-case type switch would do, but so would a *type assertion*. A type assertion takes an interface value and extracts from it a value of the specified explicit type. The syntax borrows from the clause opening a type switch, but with an explicit type rather than the `type` keyword:

如果我们只关系一种类型怎么简化代码呢？如果我们知道一个值中保存的是 `string` 类型的值，我们怎么直接把这个值解析出来？当然可以使用只有一个 case 的 switch 语句来实现，但是也可以通过**类型断言**来实现。类型断言可以在接口类型的值上面抽取特定类型的值。它的语法是从类型 switch 中借鉴的，但是 switch 中使用的是 `value.(type)`，类型断言的括号中直接显式指定类型 `value.(typeName)`。

```go
value.(typeName)
```

and the result is a new value with the static type `typeName`. That type must either be the concrete type held by the interface, or a second interface type that the value can be converted to. To extract the string we know is in the value, we could write:

`value.(typeName)` 的结果是静态类型 `typeName` 对应的新值。这个静态类型要么是接口变量的底层数据类型，要么是接口变量的底层数据类型实现的另一个接口。如果我们已经知道某个值存储的是字符串，直接通过 `str := value.(string)` 就可以获取到。

```go
str := value.(string)
```

But if it turns out that the value does not contain a string, the program will crash with a run-time error. To guard against that, use the "comma, ok" idiom to test, safely, whether the value is a string:

类似 `str := value.(string)` 的写法很简单易用，但是如果在 `value` 中保存的不是一个字符串，会产生运行时错误导致代码崩溃。为了避免这种情况，可以使用 “逗号与ok” 的惯用语法来检测相应的值是否是字符串：

```go
str, ok := value.(string)
if ok {
    fmt.Printf("string value is: %q\n", str)
} else {
    fmt.Printf("value is not a string\n")
}
```

If the type assertion fails, `str` will still exist and be of type string, but it will have the zero value, an empty string.

如果类型断言失败了，`str` 依然存在且是字符串类型，只不过它的值会是空字符串。

As an illustration of the capability, here's an `if`-`else` statement that's equivalent to the type switch that opened this section.

为了说明这种类型断言的能力，下面的 `if - else` 表达式与上面的类型 switch 实现了相同的功能。

```go
if str, ok := value.(string); ok {
    return str
} else if str, ok := value.(Stringer); ok {
    return str.String()
}
```

### Generality（泛型）

If a type exists only to implement an interface and will never have exported methods beyond that interface, there is no need to export the type itself. Exporting just the interface makes it clear the value has no interesting behavior beyond what is described in the interface. It also avoids the need to repeat the documentation on every instance of a common method.

如果类型的存在只是为了实现某个接口，除此之外都没有其他可导出的方法，此时就没有必要导出这个类型。只需要导出接口即可，表明这个类型的值除了接口里定义的方法外没有其他的方法。这样还能避免重复地给普通方法写文档。

In such cases, the constructor should return an interface value rather than the implementing type. As an example, in the hash libraries both `crc32.NewIEEE` and `adler32.New` return the interface type `hash.Hash32`. Substituting the CRC-32 algorithm for Adler-32 in a Go program requires only changing the constructor call; the rest of the code is unaffected by the change of algorithm.

在这种情况下，构造器应该返回接口的值而不是相应的类型的值。比如，在哈希库 `crc32.NewIEEE` 和 `adler32.New`中返回的是 `hash.Hash32` 接口类型的值。把 CRC-32 的算法替代为 Adler-32 的算法只需要改变一下调用的构造器，其他的就不需要任何改变了。

A similar approach allows the streaming cipher algorithms in the various `crypto` packages to be separated from the block ciphers they chain together. The `Block` interface in the `crypto/cipher` package specifies the behavior of a block cipher, which provides encryption of a single block of data. Then, by analogy with the `bufio` package, cipher packages that implement this interface can be used to construct streaming ciphers, represented by the `Stream` interface, without knowing the details of the block encryption.

同样的方式，允许不同 `crypto` 包中的流密码算法可以与把它们链接在一起的块加密算法分开。在 `crypto/cipher` 包中的 `Block` 接口限定了块加密算法的行为，它可以对某个独立的数据块进行加密。然后，和 `bufio` 包中使用 `io.Reader` 和 `io.Writer` 的做法类似，实现 `Block` 接口的加密包可以被用来创建流加密器，并且以 `Stream` 接口类型的数据返回，从而封装了块密码算法的细节，把使用者的注意力解放出来。

The `crypto/cipher` interfaces look like this:

接口 `crypto/cipher` 的源码如下：

```go
type Block interface {
    BlockSize() int
    Encrypt(dst, src []byte)
    Decrypt(dst, src []byte)
}

type Stream interface {
    XORKeyStream(dst, src []byte)
}
```

Here's the definition of the counter mode (CTR) stream, which turns a block cipher into a streaming cipher; notice that the block cipher's details are abstracted away:

下面是 CTR（计数器模式）流的定义，它可以把块密码转换成为流密码；可以看到块密码相关的细节被抽象了：

```go
// NewCTR returns a Stream that encrypts/decrypts using the given Block in
// counter mode. The length of iv must be the same as the Block's block size.
func NewCTR(block Block, iv []byte) Stream
```

`NewCTR` applies not just to one specific encryption algorithm and data source but to any implementation of the `Block` interface and any `Stream`. Because they return interface values, replacing CTR encryption with other encryption modes is a localized change. The constructor calls must be edited, but because the surrounding code must treat the result only as a `Stream`, it won't notice the difference.

`NewCTR` 不仅仅只能应用在特定的加密算法和数据源，而是能够应用在所有实现了 `Block` 接口和 `Stream` 接口的加密算法。因为返回的是接口类型而不是被定制过的 CTR 加密过程。如果逻辑要进行修改，这个构造器的调用部分必须要修改，但是因为构造器返回的是 `Sream` 返回值类型，因此代码的其他地方不会感知到这种变化。

### Interfaces and methods（接口与方法）

Since almost anything can have methods attached, almost anything can satisfy an interface. One illustrative example is in the `http` package, which defines the `Handler` interface. Any object that implements `Handler` can serve HTTP requests.

几乎任何东西都可以关联方法，任何东西都可以满足某个接口。其中一个例子是定义了 `Handler` 接口的 `http` 包。任何实现了 `Handler` 接口的对象都可以处理 HTTP 请求。

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

`ResponseWriter` is itself an interface that provides access to the methods needed to return the response to the client. Those methods include the standard `Write` method, so an `http.ResponseWriter` can be used wherever an `io.Writer` can be used. `Request` is a struct containing a parsed representation of the request from the client.

`ResponseWriter` 它自己本身就是一个接口，提供了响应客户端的一些方法。其中包含标准的 `Write` 方法，因此 `http.ResponseWriter` 可以被用在任何使用了 `io.Writer` 类型的地方。`Request` 是一个结构体，包含了从客户端解析出来的请求体。

For brevity, let's ignore POSTs and assume HTTP requests are always GETs; that simplification does not affect the way the handlers are set up. Here's a trivial but complete implementation of a handler to count the number of times the page is visited.

为了简化，让我们忽略 POSTs 并且假设 HTTP 请求只有 GETs；当然这种简化并不会影响请求处理器的创建。下面是一个短小但是完整的处理器（实现了 ServerHTTP 方法），用于计数页面被查看的次数。

```go
// Simple counter server.
type Counter struct {
    n int
}

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ctr.n++
    fmt.Fprintf(w, "counter = %d\n", ctr.n)
}
```

(Keeping with our theme, note how `Fprintf` can print to an `http.ResponseWriter`.) For reference, here's how to attach such a server to a node on the URL tree.

（可以稍微关注一下 `Fprintf` 是如何把内容打印到 `http.ResponseWriter` 的）作为参考，下面的代码展示了如果把这样的服务绑定到一个 URL 树上的：

```go
import "net/http"
...
ctr := new(Counter)
http.Handle("/counter", ctr)
```

But why make `Counter` a struct? An integer is all that's needed. (The receiver needs to be a pointer so the increment is visible to the caller.)

在这个例子里其实整数也是可以满足需求的，为什么要把 `Counter` 声明成一个结构体呢？（接收者必须是指针才可以让调用者感知到它的增加）

```go
// Simpler counter server.
type Counter int

func (ctr *Counter) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    *ctr++
    fmt.Fprintf(w, "counter = %d\n", *ctr)
}
```

What if your program has some internal state that needs to be notified that a page has been visited? Tie a channel to the web page.

如果当页面被访问时需要一些内部状态的通知，代码应该怎么写？可以给网页绑一个信道：

```go
// A channel that sends a notification on each visit.
// (Probably want the channel to be buffered.)
type Chan chan *http.Request

func (ch Chan) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    ch <- req
    fmt.Fprint(w, "notification sent")
}
```

Finally, let's say we wanted to present on `/args` the arguments used when invoking the server binary. It's easy to write a function to print the arguments.

最后，如果我们想展示服务器启动时在 `/args` 路径里传递的参数，可以很容易地写一个函数来打印这些参数。

```go
func ArgServer() {
    fmt.Println(os.Args)
}
```

How do we turn that into an HTTP server? We could make `ArgServer` a method of some type whose value we ignore, but there's a cleaner way. Since we can define a method for any type except pointers and interfaces, we can write a method for a function. The `http` package contains this code:

那额我们该怎么把它变成一个 HTTP 服务呢？我们可以定义任意的类型并给整个类型写一个 `ArgServer` 方法，然后忽略掉接收者的值；但是有一个更好的方式。因为我们可以给任何类型（除了指针和接口）定义方法，因此我们可以给函数定义方法。在 `http` 中就包含这样的代码：

```go
// The HandlerFunc type is an adapter to allow the use of
// ordinary functions as HTTP handlers.  If f is a function
// with the appropriate signature, HandlerFunc(f) is a
// Handler object that calls f.
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, req).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, req *Request) {
    f(w, req)
}
```

`HandlerFunc` is a type with a method, `ServeHTTP`, so values of that type can serve HTTP requests. Look at the implementation of the method: the receiver is a function, `f`, and the method calls `f`. That may seem odd but it's not that different from, say, the receiver being a channel and the method sending on the channel.

`HandlerFunc` 是一个定义了 `ServeHTTP` 方法的类型，因此这个类型的值可以处理 HTTP 请求。可以查看这个方法的实现：接受者是一个函数 `f`，在方法内部调用了 `f`。可能这看起来有一点奇怪，但是这和 “接收者是信道并且在对应的方法中向信道中发送内容” 没什么不一样的。

To make `ArgServer` into an HTTP server, we first modify it to have the right signature.

为了让 `ArgServer` 注册为 HTTP 服务器，我们可以先修改它让它有正确的签名：

```go
// Argument server.
func ArgServer(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintln(w, os.Args)
}
```

`ArgServer` now has same signature as `HandlerFunc`, so it can be converted to that type to access its methods, just as we converted `Sequence` to `IntSlice` to access `IntSlice.Sort`. The code to set it up is concise:

上面的代码中 `ArgServer` 和 `HandlerFunc` 有相同的签名，因此我们可以把它转换为`HandlerFunc` 类型从而可以使用这种类型的方法，就好像我们把 `Sequence` 转换为 `IntSlice` 使用 `IntSlice.Sort` 一样。下面的代码简洁明了：

```go
http.Handle("/args", http.HandlerFunc(ArgServer))
```

When someone visits the page `/args`, the handler installed at that page has value `ArgServer` and type `HandlerFunc`. The HTTP server will invoke the method `ServeHTTP` of that type, with `ArgServer` as the receiver, which will in turn call `ArgServer` (via the invocation `f(w, req)` inside `HandlerFunc.ServeHTTP`). The arguments will then be displayed.

当有人访问了页面 `/args`，处理器在这个页面安装了类型为 `HandlerFunc` 的 `ArgServer`。HTTP 服务器会触发 `HandlerFunc` 类型的方法 `ServeHTTP`，当然这里的接收器是 `ArgServer`，并且会调用 `ArgServer` 方法（通过 `HandlerFunc.ServeHTTP` 内的 `f(w, req)`）。然后参数就被展示出来了。

In this section we have made an HTTP server from a struct, an integer, a channel, and a function, all because interfaces are just sets of methods, which can be defined for (almost) any type.

在这一小节中我们在结构体、整数、信道和函数上面分别创建了 HTTP 服务器，可以做到这一切都是因为接口的纯粹性，它只包含方法且几乎所有类型（除了指针和接口）都可以实现接口。

## The blank identifier（空白标识符）

We've mentioned the blank identifier a couple of times now, in the context of [`for` `range` loops](https://golang.google.cn/doc/effective_go.html#for) and [maps](https://golang.google.cn/doc/effective_go.html#maps). The blank identifier can be assigned or declared with any value of any type, with the value discarded harmlessly. It's a bit like writing to the Unix `/dev/null` file: it represents a write-only value to be used as a place-holder where a variable is needed but the actual value is irrelevant. It has uses beyond those we've seen already.

至此为止我们提到过很多次空白标识符，在 《for-range 循环》和 《映射 maps》都有提到。空白标识符可以被任意类型的任意值赋值声明，而且这些值会无害地被丢弃掉。它就像向 Unix 的 `/dev/null` 写东西一样：它表示一个只写的值，可以作为占位符使用，如果有一个变量的值不会被使用但是又必须要设置一个变量来接收相应的值，此时就可以使用空占位符。当然它的用处还有很多。

### The blank identifier in multiple assignment（多值赋值语句中的空白占位符）

The use of a blank identifier in a `for range` loop is a special case of a general situation: multiple assignment.

for-range 循环中的空白占位符是一种通用情况的特例：多值赋值语句。

If an assignment requires multiple values on the left side, but one of the values will not be used by the program, a blank identifier on the left-hand-side of the assignment avoids the need to create a dummy variable and makes it clear that the value is to be discarded. For instance, when calling a function that returns a value and an error, but only the error is important, use the blank identifier to discard the irrelevant value.

如果一个赋值语句需要在左侧有多个变量，但是其中部分变量不会被代码使用，这时候用空白标识符就可以避免创建显得多余的变量并且指明丢弃对应的值。比如，当我们调用一个返回一个值和一个错误的函数的时候，如果我们只关注它的错误，就可以使用空白标识符来丢弃这个函数返回的值：

```go
if _, err := os.Stat(path); os.IsNotExist(err) {
	fmt.Printf("%s does not exist\n", path)
}
```

Occasionally you'll see code that discards the error value in order to ignore the error; this is terrible practice. Always check error returns; they're provided for a reason.

偶尔情况下你还可以看到忽略错误值的代码，当然这种习惯是非常不好的（不推荐）；最佳实践中应该总是检查错误返回值，因为错误中总包含一些错误的原因应该处理啊。

```go
// Bad! This code will crash if path does not exist.
fi, _ := os.Stat(path)
if fi.IsDir() {
    fmt.Printf("%s is a directory\n", path)
}
```

### Unused imports and variables（未使用的导入与未使用的变量）

It is an error to import a package or to declare a variable without using it. Unused imports bloat the program and slow compilation, while a variable that is initialized but not used is at least a wasted computation and perhaps indicative of a larger bug. When a program is under active development, however, unused imports and variables often arise and it can be annoying to delete them just to have the compilation proceed, only to have them be needed again later. The blank identifier provides a workaround.

如果导入了没有使用的包或者声明了没有使用的变量，对编译器来说都会报错。没有应用过的导入会让程序显得臃肿且拖慢编译速度，初始化但未经使用的变量意味着计算资源的浪费有时候甚至意味着很大的缺陷。但是，代码在开发过程中总会出现未使用的导入和没有使用过的变量，如果仅仅为了编译通过而删掉他们然后再把它们加回来，这显得有点烦人，此时就可以使用空白描述符：

This half-written program has two unused imports (`fmt` and `io`) and an unused variable (`fd`), so it will not compile, but it would be nice to see if the code so far is correct.

下面写了一半的代码中有两个没有使用过的导入（`fmt` 和 `io`），有一个没有使用过的变量 `fd`，因此这段代码时不会被编译通过的。但是如果能知道这段代码是不是正确的感觉会很不错。

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
}
```

To silence complaints about the unused imports, use a blank identifier to refer to a symbol from the imported package. Similarly, assigning the unused variable `fd` to the blank identifier will silence the unused variable error. This version of the program does compile.

为了让编译器正常编译而不报错，可以使用空白标识符来标记导入的包，同样的可以把未使用的 `fd` 变量赋值给空白标识符。下面的代码就可以成功编译了：

```go
package main

import (
    "fmt"
    "io"
    "log"
    "os"
)

var _ = fmt.Printf // For debugging; delete when done.
var _ io.Reader    // For debugging; delete when done.

func main() {
    fd, err := os.Open("test.go")
    if err != nil {
        log.Fatal(err)
    }
    // TODO: use fd.
    _ = fd
}
```

By convention, the global declarations to silence import errors should come right after the imports and be commented, both to make them easy to find and as a reminder to clean things up later.

传统意义上，为了避免导入错误的全局声明应该紧跟导入语句的后面并且注释一下情况，从而便于找到这些语句，而且能够作为一种提醒让自己删除掉。

### Import for side effect（为了副作用的导入）

An unused import like `fmt` or `io` in the previous example should eventually be used or removed: blank assignments identify code as a work in progress. But sometimes it is useful to import a package only for its side effects, without any explicit use. For example, during its `init` function, the `net/http/pprof` package registers HTTP handlers that provide debugging information. It has an exported API, but most clients need only the handler registration and access the data through a web page. To import the package only for its side effects, rename the package to the blank identifier:

在前面的例子中，像 `fmt` 和 `io` 这种未经使用的包最后要么被使用要么被溢出：空白标识符仅仅意味着代码还在开发状态。不过有时候我们可能只需要导入包以后引起的副作用，但是确实不会显式地使用这个包。

比如，在 `init` 函数中，包 `net/http/pprof` 包会注册提供调试信息的 HTTP 处理器。虽然这个包有导出的 API， 但是客户端需要的仅仅是处理器注册然后通过它提供的网页页面获取信息。为了某个副作用来导入某个包而不使用这个包的变量，可以给这个包重命名为空白标识符。

```go
import _ "net/http/pprof"
```

This form of import makes clear that the package is being imported for its side effects, because there is no other possible use of the package: in this file, it doesn't have a name. (If it did, and we didn't use that name, the compiler would reject the program.)

这种方式的导入显而易见只是为了这个包的副作用，因为没有其他地方使用这个包：在这个文件中它甚至没有名字（如果导入的包有名字但是我们却没有使用这个名字，编译器会报错）。

### Interface checks（接口检查）

As we saw in the discussion of [interfaces](https://golang.google.cn/doc/effective_go.html#interfaces_and_types) above, a type need not declare explicitly that it implements an interface. Instead, a type implements the interface just by implementing the interface's methods. In practice, most interface conversions are static and therefore checked at compile time. For example, passing an `*os.File` to a function expecting an `io.Reader` will not compile unless `*os.File` implements the `io.Reader` interface.

就像我们在《接口》那一节看到的，一个类型不需要显式指明自己实现了哪些接口。相反，类型仅仅通过实现接口中包含的方法来实现接口。实际中，大部分的接口转换都是静态的因此在编译时就会进行检查。比如，给一个希望得到 `io.Reader` 类型的函数传递一个 `*os.File` 类型，如果 `*os.File` 没有实现`io.Reader` 接口，代码不能通过编译。

Some interface checks do happen at run-time, though. One instance is in the `encoding/json` package, which defines a `Marshaler` interface. When the JSON encoder receives a value that implements that interface, the encoder invokes the value's marshaling method to convert it to JSON instead of doing the standard conversion. The encoder checks this property at run time with a [type assertion](https://golang.google.cn/doc/effective_go.html#interface_conversions) like:

但是，有时候接口的检查是在运行时发生的。比如在 `encoding/json` 这个包里面有一个 `Marshaler` 接口，当 JSON 编码器接收到一个实现了这个接口的值的时候，编码器调用这个值的 marshal 犯法把这个值转化为 JSON，此时不会把值转换为标准的类型。编码器会在运行时检查它的属性：

```go
m, ok := val.(json.Marshaler)
```

If it's necessary only to ask whether a type implements an interface, without actually using the interface itself, perhaps as part of an error check, use the blank identifier to ignore the type-asserted value:

如果仅需要知道某个类型是否实现了某个接口，不需要使用接口本身，可以像错误检查那里的用法一样把一个空白标识符放在那里从而忽略掉类型检查得到的值，如下：

```go
if _, ok := val.(json.Marshaler); ok {
    fmt.Printf("value %v of type %T implements json.Marshaler\n", val, val)
}
```

One place this situation arises is when it is necessary to guarantee within the package implementing the type that it actually satisfies the interface. If a type—for example, `json.RawMessage`—needs a custom JSON representation, it should implement `json.Marshaler`, but there are no static conversions that would cause the compiler to verify this automatically. If the type inadvertently fails to satisfy the interface, the JSON encoder will still work, but will not use the custom implementation. To guarantee that the implementation is correct, a global declaration using the blank identifier can be used in the package:

这种情况发生的场合，我们必须在实现了该类型的包内保证它实现了相应的接口。比如，如果类型 `json.RawMessage` 需要一个自定义的 JSON 表示，它应该实现 `json.Marshaler` 接口，但是代码中没有任何静态转换让编译器自动确认这种实现。如果有类型不经意地没有实现接口的方法，JSON 编码器仍然会工作，只是这个时候不会使用自定义的实现而是调用默认的方法。为了保证实现了正确的方法，可以在包中写一个使用了空白标识符的声明：

```go
var _ json.Marshaler = (*RawMessage)(nil)
```

In this declaration, the assignment involving a conversion of a `*RawMessage` to a `Marshaler` requires that `*RawMessage` implements `Marshaler`, and that property will be checked at compile time. Should the `json.Marshaler` interface change, this package will no longer compile and we will be on notice that it needs to be updated.

在这个声明中，赋值语句包含一次转换，从 `*RawMessage` 转换为 `Marshaler` 需要 `*RawMessage` 实现了 `Marshaler` 接口，这种检查会在编译时进行。假如接口 `json.Marshaler` 变化了，这个包将不会编译通过，然后我们就知道它需要更新了。

The appearance of the blank identifier in this construct indicates that the declaration exists only for the type checking, not to create a variable. Don't do this for every type that satisfies an interface, though. By convention, such declarations are only used when there are no static conversions already present in the code, which is a rare event.

在上面的这种情况下，空白标识符的出现标明声明语句仅仅是为了类型检查，并不会创建一个变量。但是，如果类型满足某个接口则没有必要做这种声明。通常情况下，这种用法仅在源码中不存在静态类型转换的情况使用，当然，这种情况其实是很少见到的。

## Embedding（嵌套）

Go does not provide the typical, type-driven notion of subclassing, but it does have the ability to “borrow” pieces of an implementation by *embedding* types within a struct or interface.

Go 不支持子类的概念，但是这个概念可以通过在结构体或接口中**嵌套**类型来实现。

Interface embedding is very simple. We've mentioned the `io.Reader` and `io.Writer` interfaces before; here are their definitions.

接口的嵌套非常简单，我们前面提到过 `io.Reader` 和 `io.Writer` 接口，下面是它们的声明：

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}

type Writer interface {
    Write(p []byte) (n int, err error)
}
```

The `io` package also exports several other interfaces that specify objects that can implement several such methods. For instance, there is `io.ReadWriter`, an interface containing both `Read` and `Write`. We could specify `io.ReadWriter` by listing the two methods explicitly, but it's easier and more evocative to embed the two interfaces to form the new one, like this:

上面的定义每个接口只有一个方法，在 `io` 包中还导出了几个其他的接口用来指定对象应该实现多个方法。比如，`io.ReadWriter` 接口，包含了 `Read` 和 `Write` 两个方法。我们当然可以把两个方法直接写在 `io.ReadWriter` 接口定义里，但是更简单更明了的方式是把上面的两个接口嵌进新的接口中，就像下面的代码：

```go
// ReadWriter is the interface that combines the Reader and Writer interfaces.
type ReadWriter interface {
    Reader
    Writer
}
```

This says just what it looks like: A `ReadWriter` can do what a `Reader` does *and* what a `Writer` does; it is a union of the embedded interfaces (which must be disjoint sets of methods). Only interfaces can be embedded within interfaces.

上面的代码的含义是：一个 `ReadWriter` 既是一个 `Reader` 又是一个 `Writer`；它是嵌套起来的接口的联合体（这些联合体之间的方法不能有重名）。需要注意的是，接口只能被嵌套到接口里。

The same basic idea applies to structs, but with more far-reaching implications. The `bufio` package has two struct types, `bufio.Reader` and `bufio.Writer`, each of which of course implements the analogous interfaces from package `io`. And `bufio` also implements a buffered reader/writer, which it does by combining a reader and a writer into one struct using embedding: it lists the types within the struct but does not give them field names.

结构体也有类似的用法，但是含义更深远（更复杂）。包 `bufio` 有两个结构体类型 `bufio.Reader` 和 `bufio.Writer`，每个都实现了包 `io` 中的类似接口。同时 `bufio` 包实现了带缓存的读写器（reader/writer），通过嵌套的方式在一个结构中组合 reader 和 writer：在组合结构体声明中列出子结构体，但是不给定字段名。如下示例：

```go
// ReadWriter stores pointers to a Reader and a Writer.
// It implements io.ReadWriter.
type ReadWriter struct {
    *Reader  // *bufio.Reader
    *Writer  // *bufio.Writer
}
```

The embedded elements are pointers to structs and of course must be initialized to point to valid structs before they can be used. The `ReadWriter` struct could be written as

嵌套元素是子结构体的指针，因此初始化时必须要指向合法的结构体才能使用。`ReadWriter` 结构体也可以写为：

```go
type ReadWriter struct {
    reader *Reader
    writer *Writer
}
```

but then to promote the methods of the fields and to satisfy the `io` interfaces, we would also need to provide forwarding methods, like this:

但是但是如果指定了字段名，为了提升字段里的方法，同时满足 `io` 接口，不得不定义代理方法，比如下面的 `Read` 方法：

```go
func (rw *ReadWriter) Read(p []byte) (n int, err error) {
    return rw.reader.Read(p)
}
```

By embedding the structs directly, we avoid this bookkeeping. The methods of embedded types come along for free, which means that `bufio.ReadWriter` not only has the methods of `bufio.Reader` and `bufio.Writer`, it also satisfies all three interfaces: `io.Reader`, `io.Writer`, and `io.ReadWriter`.

通过直接嵌套结构，我们可以避免上面啰嗦的簿记方式。嵌套类型的方法自动提升为组合体的方法，也就是说，`bufio.ReadWriter` 不仅实现了 `bufio.Reader` 和 `bufio.Writer`这两个接口中的方法，而且满足所有下面的三个接口：`io.Reader`, `io.Writer`, 和 `io.ReadWriter`。

There's an important way in which embedding differs from subclassing. When we embed a type, the methods of that type become methods of the outer type, but when they are invoked the receiver of the method is the inner type, not the outer one. In our example, when the `Read` method of a `bufio.ReadWriter` is invoked, it has exactly the same effect as the forwarding method written out above; the receiver is the `reader` field of the `ReadWriter`, not the `ReadWriter` itself.

嵌套与子类还是很不一样的。如果我们嵌套了一个类型，则这个类型所有的方法都会成为外面类型的方法，但是当方法被触发调用的时候其接收者依然是里面的类型而不是外面的类型。比如上面的例子中，当 `bufio.ReadWriter` 的 `Read` 方法被触发的时候，和上面写的代理方法是一样的效果：接收者是 `ReadWriter` 的 `reader` 字段，而不是 `ReadWriter` 自己。

Embedding can also be a simple convenience. This example shows an embedded field alongside a regular, named field.

嵌套也可以很便利的应用。下面的例子中，嵌套的字段和普通的命名字段一起使用：

```go
type Job struct {
    Command string
    *log.Logger
}
```

The `Job` type now has the `Print`, `Printf`, `Println` and other methods of `*log.Logger`. We could have given the `Logger` a field name, of course, but it's not necessary to do so. And now, once initialized, we can log to the `Job`:

`Job` 类型有 `Print`, `Printf`, `Println` 和 `*log.Logger` 的其他方法。我们可以给 `Logger` 一个字段名，但是没有必要这么做。现在，我们可以向初始化后的 `Job` 类型的对象打日志了：

```go
job.Println("starting now...")
```

The `Logger` is a regular field of the `Job` struct, so we can initialize it in the usual way inside the constructor for `Job`, like this,

`Logger` 是结构体 `Job` 的一个常规字段，因此我们可以通过常规的方式来构建 `Job`，比如下面的方式：

```go
func NewJob(command string, logger *log.Logger) *Job {
    return &Job{command, logger}
}
```

or with a composite literal,
也可以使用符合语法：

```go
job := &Job{command, log.New(os.Stderr, "Job: ", log.Ldate)}
```

If we need to refer to an embedded field directly, the type name of the field, ignoring the package qualifier, serves as a field name, as it did in the `Read` method of our `ReadWriter` struct. Here, if we needed to access the `*log.Logger` of a `Job` variable `job`, we would write `job.Logger`, which would be useful if we wanted to refine the methods of `Logger`.

如果我们需要直接引用嵌套的字段，字段的名称默认是忽略包修饰符的名称，就像 `ReadWriter` 结构体中的 `Read` 方法一样。如果我们需要访问 `Job` 变量中的 `*log.Logger`，我们可以写成 `job.Logger`，如果想重新定义 `Logger` 的方法，这会非常有用。

```go
func (job *Job) Printf(format string, args ...interface{}) {
    job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
}
```

Embedding types introduces the problem of name conflicts but the rules to resolve them are simple. First, a field or method `X` hides any other item `X` in a more deeply nested part of the type. If `log.Logger` contained a field or method called `Command`, the `Command` field of `Job` would dominate it.

嵌套类型引入了命名冲突的问题，不过解决的方式也很简单。首先，字段或方法 `X` 会覆盖任何深层嵌套类型的 `X`。比如，如果 `log.Logger` 包含了一个叫 `Command` 的字段或方法，`Job` 中的 `Command` 字段会覆盖它。

Second, if the same name appears at the same nesting level, it is usually an error; it would be erroneous to embed `log.Logger` if the `Job` struct contained another field or method called `Logger`. However, if the duplicate name is never mentioned in the program outside the type definition, it is OK. This qualification provides some protection against changes made to types embedded from outside; there is no problem if a field is added that conflicts with another field in another subtype if neither field is ever used.

其次，如果同样的名字在同一级的嵌套中出现（对比不同级的情况），这会导致错误。如果 `Job` 结构包含了另一个名称为 `Logger` 的字段或者方法，此时如果再嵌套 `log.Logger` 就会报错。然而，如果重复的名称不会再类型定义之外使用，也不会引起问题。这个特性可以保护结构体不会因为被嵌套的外部类型的变化而变得不可用。当添加的字段与另一个子类型的字段冲突时，如果两个字段都不会被使用，是不会报错了。

## Concurrency（并发）

### Share by communicating（通过通信共享变量）

Concurrent programming is a large topic and there is space only for some Go-specific highlights here.

并发变成是一个很大的话题，这里只强调 Go 相关的特性。

Concurrent programming in many environments is made difficult by the subtleties required to implement correct access to shared variables. Go encourages a different approach in which shared values are passed around on channels and, in fact, never actively shared by separate threads of execution. Only one goroutine has access to the value at any given time. Data races cannot occur, by design. To encourage this way of thinking we have reduced it to a slogan:

> Do not communicate by sharing memory; instead, share memory by communicating.

在很多环境中，并发编程都很困难，为了保证正确访问共享变量，里面有太多的实现细节要解决。Go 鼓励一种不同的方式——通过信道共享变量的值，不要直接在不同的线程间共享变量。在任何时间点只有一种协程有权访问变量的值；在设计上就杜绝了数据竞争。为了鼓励这种方式，我们还把它变成一句口号：**不要通过共享内存进行通信，相反，通过通信来共享内存**。

This approach can be taken too far. Reference counts may be best done by putting a mutex around an integer variable, for instance. But as a high-level approach, using channels to control access makes it easier to write clear, correct programs.

这种方式不是万能的。比如，引用计数的最佳实现方式就是给一个整数添加一个互斥锁。但是作为一个高级别的方式，使用信道来控制访问可以更容易写出简明正确的代码。

One way to think about this model is to consider a typical single-threaded program running on one CPU. It has no need for synchronization primitives. Now run another such instance; it too needs no synchronization. Now let those two communicate; if the communication is the synchronizer, there's still no need for other synchronization. Unix pipelines, for example, fit this model perfectly. Although Go's approach to concurrency originates in Hoare's Communicating Sequential Processes (CSP), it can also be seen as a type-safe generalization of Unix pipes.

思考这个模型的一种方式是假设程序是运行在一个 CPU 核上的独立线程的代码，这种情况下没有必要有同步命令。现在思考另一个实例，它也不需要任何的同步。接着，让这两个线程进行通信，如果通信是同步的，那么就不需要其他的同步了。比如 Unix 的流水线就很契合这种模型。虽然 Go 解决并发的解决方案源自 Hoare 的CSP（通信序列过程）模型，但是它也可以看做是 Unix 管道的一个类型安全的通用版。

### Goroutines（Go 协程）

They're called *goroutines* because the existing terms—threads, coroutines, processes, and so on—convey inaccurate connotations. A goroutine has a simple model: it is a function executing concurrently with other goroutines in the same address space. It is lightweight, costing little more than the allocation of stack space. And the stacks start small, so they are cheap, and grow by allocating (and freeing) heap storage as required.

之所以叫 **goroutines** ，是因为已有的术语——线程、协程、进程等——传达的含义不明确。一个 goroutine 有一个简单的模型：它是一个函数，与其他的 goroutine 在同一个地址空间一起运行。它是轻量级的，只比分配堆栈空间花费的多一点资源。同时它们也很廉价，因为它的堆栈开始时很小，只会根据需要增加（或释放）堆存储。

Goroutines are multiplexed onto multiple OS threads so if one should block, such as while waiting for I/O, others continue to run. Their design hides many of the complexities of thread creation and management.

Go协程多路复用多个操作系统线程，因此如果一个线程因为 I/O 调用阻塞了，其他的协程会继续运行。Go协程的设计隐藏了线程创建和管理的很多复杂度。

Prefix a function or method call with the `go` keyword to run the call in a new goroutine. When the call completes, the goroutine exits, silently. (The effect is similar to the Unix shell's `&` notation for running a command in the background.)

在一个函数或者方法的前面加一个关键词 `go` 就可以让它在一个新的 goroutine 中运行。当调用结束时，goroutine 会静态地退出。（类似于 Unix Shell 中的 `&` 符号，让命令在后台运行）

```go
go list.Sort()  // run list.Sort concurrently; don't wait for it.
```

A function literal can be handy in a goroutine invocation.
函数声明语法可以在 goroutine 调用中方便地使用。

```go
func Announce(message string, delay time.Duration) {
    go func() {
        time.Sleep(delay)
        fmt.Println(message)
    }()  // Note the parentheses - must call the function.
}
```

In Go, function literals are closures: the implementation makes sure the variables referred to by the function survive as long as they are active.

在 Go 中，**函数即闭包：实现上被函数引用的变量会与函数一起存活**。

These examples aren't too practical because the functions have no way of signaling completion. For that, we need channels.

上面的这些例子都没有太大的实践意义，因为函数并没有标识自己结束。为此需要引入信道。

### Channels（信道）

Like maps, channels are allocated with `make`, and the resulting value acts as a reference to an underlying data structure. If an optional integer parameter is provided, it sets the buffer size for the channel. The default is zero, for an unbuffered or synchronous channel.

就像映射一样，信道通过 `make` 来分配内存，返回的结果就好像底层数据结构的引用。在使用 `make` 时如果传入了一个可选的整数参数，会创建信道的缓存；如果不传默认就是 0，这时候标明是一个非缓存或同步信道。

```go
ci := make(chan int)            // unbuffered channel of integers
cj := make(chan int, 0)         // unbuffered channel of integers
cs := make(chan *os.File, 100)  // buffered channel of pointers to Files
```

Unbuffered channels combine communication—the exchange of a value—with synchronization—guaranteeing that two calculations (goroutines) are in a known state.

非缓存的信道把同步和通信（交换值）组合在一起，保证两个计算（Go协程）在已知的状态。

There are lots of nice idioms using channels. Here's one to get us started. In the previous section we launched a sort in the background. A channel can allow the launching goroutine to wait for the sort to complete.

存在很多应用信道的好习惯。我们可以从下面这个例子开始。在前面章节里我们在后台启用了一个排序例程；信道可以允许我们启动一个 Goroutine 并等待这个排序结束。

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

Receivers always block until there is data to receive. If the channel is unbuffered, the sender blocks until the receiver has received the value. If the channel has a buffer, the sender blocks only until the value has been copied to the buffer; if the buffer is full, this means waiting until some receiver has retrieved a value.

在接收到数据前接收器会一直阻塞。如果信道是非缓冲的，在接收器接收到值之前发送器会一直阻塞。如果信道有缓冲，发送器只在值拷贝到缓冲前是阻塞的；如果缓冲区满了，发送器会一直等待直到有接收器消费了一个值。

A buffered channel can be used like a semaphore, for instance to limit throughput. In this example, incoming requests are passed to `handle`, which sends a value into the channel, processes the request, and then receives a value from the channel to ready the “semaphore” for the next consumer. The capacity of the channel buffer limits the number of simultaneous calls to `process`.

带缓存的信道可以用来作为信号量来使用，比如可以用它来控制吞吐量。在下面的例子中，过来的请求被传到 `handle`，然后它会在把值传到一个信道中，然后处理这个请求，接着再从信道中接收一个值释放“信号量”给下一个消费者。信道的缓冲区容量限定了 `process` 的并发数量。

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

Once `MaxOutstanding` handlers are executing `process`, any more will block trying to send into the filled channel buffer, until one of the existing handlers finishes and receives from the buffer.

一旦 `MaxOutstanding` 个处理器运行 `process`，任何其他的 Goroutine 尝试发送数据到信道的行为都会被阻塞，直到一个已存在的处理器结束并且消费缓冲中的一个数据。

This design has a problem, though: `Serve` creates a new goroutine for every incoming request, even though only `MaxOutstanding` of them can run at any moment. As a result, the program can consume unlimited resources if the requests come in too fast. We can address that deficiency by changing `Serve` to gate the creation of the goroutines. Here's an obvious solution, but beware it has a bug we'll fix subsequently:

但是，这种设计是有问题的： `Serve` 会给每个进来的请求都创建一个新的 Goroutine，虽然最多只有 `MaxOutstanding` 个 Goroutine 同时执行。因此只要请求进来的够快，程序会无限消费资源。我们可以修改 `Serve` 函数解决这个缺陷，比如限定 goroutine 的创建数量。下面是一个显而易见的解决方案，不过下面有一个 bug 需要我们修复：

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

The bug is that in a Go `for` loop, the loop variable is reused for each iteration, so the `req` variable is shared across all goroutines. That's not what we want. We need to make sure that `req` is unique for each goroutine. Here's one way to do that, passing the value of `req` as an argument to the closure in the goroutine:

上面代码的缺陷发生在 Go 语言中的 `for` 循环，循环变量在每次迭代时会复用，因此 `req` 变量在各个 goroutine 之间是共享的。但是我们不希望 req 被共享，因此需要给每个 Goroutine 传一个唯一的 `req`。如下面的代码，给每个 Goroutine 的闭包传递了 `req` 作为其参数：

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

Compare this version with the previous to see the difference in how the closure is declared and run. Another solution is just to create a new variable with the same name, as in this example:

可以把上面的代码和前一个代码进行比较，看看闭包是如何声明并运行的。上面的 bug 还有一个解决方案，就是每次循环时创建一个同名的临时变量，如下面的代码：

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

It may seem odd to write

```go
req := req
```

but it's legal and idiomatic in Go to do this. You get a fresh version of the variable with the same name, deliberately shadowing the loop variable locally but unique to each goroutine.

可能 `req := req` 的写法很怪异，但是在 Go 中这是合法且惯用的写法。通过这种方法可以得到一个同名的新变量，刻意覆盖了本地的循环变量但是给每个 Goroutine 提供了唯一的变量。

Going back to the general problem of writing the server, another approach that manages resources well is to start a fixed number of `handle` goroutines all reading from the request channel. The number of goroutines limits the number of simultaneous calls to `process`. This `Serve` function also accepts a channel on which it will be told to exit; after launching the goroutines it blocks receiving from that channel.

回到写服务器的常见的问题，另一个管理资源的好方法是开始固定数目的 `handle` Goroutine ，所有这些 Goroutine 都从请求信道中读数据。Goroutine 的数目限定了同时执行 `process` 的并发数量。`Serve` 函数也接收了一个信道，可以在这个信道接收退出信号，Goroutine 开始运转后它会读取信道中的值并被阻塞在这里。

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

### Channels of channels（信道的信道）

One of the most important properties of Go is that a channel is a first-class value that can be allocated and passed around like any other. A common use of this property is to implement safe, parallel demultiplexing.

Go最重要的特性之一就是**信道是第一级类型**，可以像其他的类型那样初始化和传递。这个特性的一个应用就是实现安全并行的多路复用。

In the example in the previous section, `handle` was an idealized handler for a request but we didn't define the type it was handling. If that type includes a channel on which to reply, each client can provide its own path for the answer. Here's a schematic definition of type `Request`.

在前面的例子中，`handle` 对请求来说是一个理想的处理器，但是我们并没有定义它处理的类型。如果类型包含一个用来响应的信道，每个客户端都可以提供自己的路径获取响应。下面给出类型 `Request` 的语法定义：

```go
type Request struct {
    args        []int
    f           func([]int) int
    resultChan  chan int
}
```

The client provides a function and its arguments, as well as a channel inside the request object on which to receive the answer.

在客户端提供了一个函数和函数的参数，同时在请求的实例中有一个信道用来接收响应。

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

On the server side, the handler function is the only thing that changes.

在服务器端，只需要修改处理器函数。

```go
func handle(queue chan *Request) {
    for req := range queue {
        req.resultChan <- req.f(req.args)
    }
}
```

There's clearly a lot more to do to make it realistic, but this code is a framework for a rate-limited, parallel, non-blocking RPC system, and there's not a mutex in sight.

为了让代码功能更理想，显然上面的例子还有很多东西可以做，但是上面的代码可以作为一个限流、并行、非阻塞的 RPC 系统的框架，且里面没有出现任何锁。

### Parallelization（并行）

Another application of these ideas is to parallelize a calculation across multiple CPU cores. If the calculation can be broken into separate pieces that can execute independently, it can be parallelized, with a channel to signal when each piece completes.

Goroutine 和 信道的另一个应用是在多 CPU 核心上并发地进行运算。如果运算可以被分解成可以同步运行的独立片段，它就可以进行并行运算，当某个 Goroutine 结束后可以通过信道来通知其结束。

Let's say we have an expensive operation to perform on a vector of items, and that the value of the operation on each item is independent, as in this idealized example.

假设我们有一个耗时的向量运算，向量中每个值的运算都是独立的，如下：

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

We launch the pieces independently in a loop, one per CPU. They can complete in any order but it doesn't matter; we just count the completion signals by draining the channel after launching all the goroutines.

我们可以把循环中可以独立出来的逻辑拆分开，让每个 CPU 都去执行。这些拆分出来的逻辑可以以任何的次序执行，在启动所有的 Goroutine 以后我们只需要计数完成信号的数量（读取所有信道中的值）。

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

Rather than create a constant value for numCPU, we can ask the runtime what value is appropriate. The function `runtime.NumCPU` returns the number of hardware CPU cores in the machine, so we could write

相对于为 numCPU 创建一个常量的值，我们可以通过运行时来获取准确的 CPU 数量。函数 `runtime.NumCPU` 返回硬件 CPU 的核心数量，因此我们可以写 `var numCPU = runtime.NumCPU()`。

```go
var numCPU = runtime.NumCPU()
```

There is also a function `runtime.GOMAXPROCS`, which reports (or sets) the user-specified number of cores that a Go program can have running simultaneously. It defaults to the value of `runtime.NumCPU` but can be overridden by setting the similarly named shell environment variable or by calling the function with a positive number. Calling it with zero just queries the value. Therefore if we want to honor the user's resource request, we should write

还有一个函数 `runtme.GOMAXPROCS`，它可以报告（或设置）用户定义过的 Go 程序可以并发运行的核心数。它的默认值是 `runtime.NumCPU`，不过可以通过设置相同名称的环境变量名，或者通过调用这个函数（传一个正整数）来覆盖这个值。如果调用这个函数时传入的是 0，则会得到这个值的大小。因此如果我们想兑现用户的资源请求，我们可以这样写 `var numCPU = runtime.GOMAXPROCS(0)`。

```go
var numCPU = runtime.GOMAXPROCS(0)
```

Be sure not to confuse the ideas of concurrency—structuring a program as independently executing components—and parallelism—executing calculations in parallel for efficiency on multiple CPUs. Although the concurrency features of Go can make some problems easy to structure as parallel computations, Go is a concurrent language, not a parallel one, and not all parallelization problems fit Go's model. For a discussion of the distinction, see the talk cited in [this blog post](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html).

请确认不要模糊并发的概念——把代码构造为独立的可执行模块——即并发——高效地在多 CPU 核心上并行的执行。尽管 Go 的并发特性可以让构造平行计算变得简单，但是 Go 毕竟是一个并发的语言，它并不是一个并行语言，而且并不是所有的并发问题都契合 Go 的模型。可以查看 [这里](https://blog.golang.org/2013/01/concurrency-is-not-parallelism.html)查看更多相关的讨论。

### A leaky buffer（漏斗缓存）

The tools of concurrent programming can even make non-concurrent ideas easier to express. Here's an example abstracted from an RPC package. The client goroutine loops receiving data from some source, perhaps a network. To avoid allocating and freeing buffers, it keeps a free list, and uses a buffered channel to represent it. If the channel is empty, a new buffer gets allocated. Once the message buffer is ready, it's sent to the server on `serverChan`.

并发编程工具甚至可以让非并发的想法更容易表达。下面是从 RPC 包抽象出来的一个例子。客户端协程循环从某些源中接收数据（可能是网络）；为了避免重复分配和回收缓存，它保存了一个空闲列表，并用一个缓存的信道承载它。如果信道是空的，会分配一个新的缓存。一旦消息的缓存就绪，它就会把它发送到 `serverChan` 给服务端。

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

The server loop receives each message from the client, processes it, and returns the buffer to the free list.

服务端从客户端循环获取每个消息，处理它然后把缓存返回给空闲列表。

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

The client attempts to retrieve a buffer from `freeList`; if none is available, it allocates a fresh one. The server's send to `freeList` puts `b` back on the free list unless the list is full, in which case the buffer is dropped on the floor to be reclaimed by the garbage collector. (The `default` clauses in the `select` statements execute when no other case is ready, meaning that the `selects` never block.) This implementation builds a leaky bucket free list in just a few lines, relying on the buffered channel and the garbage collector for bookkeeping.

客户端试图从 `freeList` 获取缓存，如果没有可用的缓存它就分配一个新的缓存。服务端用完缓存后再把 `b` 返回给 `freeList`，除非空闲列表满了，此时会缓存会被丢弃，然后被垃圾回收期回收掉。（在 `select` 中 `default` 的分支会在没有其他的 case 匹配的时候执行，意味着 `select` 不会阻塞）上面的代码通过几行代码构建了一个漏斗桶的空闲列表，通过使用带缓存的信道和垃圾回收器进行簿记。

## Errors（错误）

Library routines must often return some sort of error indication to the caller. As mentioned earlier, Go's multivalue return makes it easy to return a detailed error description alongside the normal return value. It is good style to use this feature to provide detailed error information. For example, as we'll see, `os.Open` doesn't just return a `nil` pointer on failure, it also returns an error value that describes what went wrong.

第三方库应该返回一些有指示含义的错误信息给调用者。就像之前提到的那样，Go 的多返回值特性让**同时返回普通返回值和一个详尽的错误描述**变得很简单。通过这个特性提供来提供详细的错误信息是一个很好的风格。比如，就像我们看到的，`os.Open` 在发生错误的时候不仅返回一个 `nil` 的指针，同时返回一个错误值描述哪里出了问题。

By convention, errors have type `error`, a simple built-in interface.
为了方便，错误有一个类型 `error`，它是一个内建的接口。

```go
type error interface {
    Error() string
}
```

A library writer is free to implement this interface with a richer model under the covers, making it possible not only to see the error but also to provide some context. As mentioned, alongside the usual `*os.File` return value, `os.Open` also returns an error value. If the file is opened successfully, the error will be `nil`, but when there is a problem, it will hold an `os.PathError`:

包作者可以自由地使用更丰富的模型来实现此接口，让调用者不仅能看到错误而且能提供一些其他的上下文内容。就像刚刚提到的，`os.Open` 函数除了返回 `*os.File` 类型的值，还会返回一个错误值。如果文件成功打开了，错误值是 nil，当时如果打开失败，错误值会是 `os.PathError`：

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

`PathError`'s `Error` generates a string like this:
`PathError` 的 `Error` 会产生下面类似的字符串：

```bash
open /etc/passwx: no such file or directory
```

Such an error, which includes the problematic file name, the operation, and the operating system error it triggered, is useful even if printed far from the call that caused it; it is much more informative than the plain "no such file or directory".

这样的错误里面包含有问题的文件名、操作以及它触发的操作系统错误，这个信息很有用，即使这个打印距离调用的问题很远也是如此。这种输出比起 “找不到相关的文件或目录” 这种提示要有用的多。

When feasible, error strings should identify their origin, such as by having a prefix naming the operation or package that generated the error. For example, in package `image`, the string representation for a decoding error due to an unknown format is "image: unknown format".

如果可行，错误字符串可以表示他们的源信息，比如什么操作导致的什么错误，比如哪个包的什么错误，等等。以包 `image` 为例，由未知格式导致的编码错误的字符串提示是：“图片：未知的格式”。

Callers that care about the precise error details can use a type switch or a type assertion to look for specific errors and extract details. For `PathErrors` this might include examining the internal `Err` field for recoverable failures.

如果调用者关注详细的错误类型，那就可以用一个类型 switch 或者一个类型断言来查找特定的错误并抽取它的详细信息。对于下面代码中的 `PathErrors`，为了恢复失败操作，它可能包含了一个内部的 `Err` 字段。

```go
for try := 0; try < 2; try++ {
    file, err = os.Create(filename)
    if err == nil {
        return
    }
    if e, ok := err.(*os.PathError); ok && e.Err == syscall.ENOSPC {
        deleteTempFiles()  // Recover some space.
        continue
    }
    return
}
```

The second `if` statement here is another [type assertion](https://golang.google.cn/doc/effective_go.html#interface_conversions). If it fails, `ok` will be false, and `e` will be `nil`. If it succeeds, `ok` will be true, which means the error was of type `*os.PathError`, and then so is `e`, which we can examine for more information about the error.

第二个 `if` 语句是另一个类型断言。如果断言失败了，`ok` 会是 false，此时 `e` 的值是 `nil`。如果成功，`ok` 会是 true，意味着错误值的类型是 `*os.PathError`，接着 `e`就可以做进一步的检查了。

### Panic（Panic）

The usual way to report an error to a caller is to return an `error` as an extra return value. The canonical `Read` method is a well-known instance; it returns a byte count and an `error`. But what if the error is unrecoverable? Sometimes the program simply cannot continue.

报出错误最常见的方式是给调用者返回一个额外的 `error`类型的值。典型的 `Read` 方法是一个有名的例子：它返回了一个字节数和一个 `error`。但是如果错误没有被覆盖怎么办呢？有时候程序可能不能进行下去。

For this purpose, there is a built-in function `panic` that in effect creates a run-time error that will stop the program (but see the next section). The function takes a single argument of arbitrary type—often a string—to be printed as the program dies. It's also a way to indicate that something impossible has happened, such as exiting an infinite loop.

为了达成这个目的，内建的 `panic` 函数会创建一个运行时错误，并且会终止程序（看下节）。这个函数会接收一个任意类型的参数——经常是一个字符串——当程序终止后会打印出来。这种方式也可以表明发生了一些不应该发生的事情，比如存在一个无限循环逻辑。

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

This is only an example but real library functions should avoid `panic`. If the problem can be masked or worked around, it's always better to let things continue to run rather than taking down the whole program. One possible counterexample is during initialization: if the library truly cannot set itself up, it might be reasonable to panic, so to speak.

上面只是一个例子，真实的库函数中应该避免使用 `panic`。如果问题可以被处理，更好的方式是让代码继续运行，比起终止整个程序要好。在计数例子的初始化时可以使用 panic：如果包不能成功启动，它可以很合理地 panic。

```go
var user = os.Getenv("USER")

func init() {
    if user == "" {
        panic("no value for $USER")
    }
}
```

### Recover（恢复）

When `panic` is called, including implicitly for run-time errors such as indexing a slice out of bounds or failing a type assertion, it immediately stops execution of the current function and begins unwinding the stack of the goroutine, running any deferred functions along the way. If that unwinding reaches the top of the goroutine's stack, the program dies. However, it is possible to use the built-in function `recover` to regain control of the goroutine and resume normal execution.

当 `panic` 被调用，包含一些运行时错误的情况（比如越界访问切片，后者类型断言失败），它会马上停止当前函数的执行，并且开始展开 Goroutine 的堆栈，整个过程中会运行被推迟执行的函数。如果堆栈一直展开到达了 goroutine 的栈顶，代码就退出了。但是，我们可以通过内建的 `recover` 来重新获取 goutine 的控制权并且恢复其正常的执行。

A call to `recover` stops the unwinding and returns the argument passed to `panic`. Because the only code that runs while unwinding is inside deferred functions, `recover` is only useful inside deferred functions.

调用 `recover` 可以停止堆栈的展开，并返回传递给 `panic` 的参数。因为在堆栈展开的过程中只有被推迟的函数的代码会继续执行，因为 `recover` 只能放在被推迟的函数中才有意义。

One application of `recover` is to shut down a failing goroutine inside a server without killing the other executing goroutines.

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

In this example, if `do(work)` panics, the result will be logged and the goroutine will exit cleanly without disturbing the others. There's no need to do anything else in the deferred closure; calling `recover` handles the condition completely.

在上面的例子中，如果 `do(work)` 报运行时错误（panic），结果会被记录下来，并且 Goroutine 会干净利落地退出而不会影响到其他的 Goroutine。对于上面的例子的情况，在**推迟**语法中没必要做其他的事情，调用 `recover` 就可以完全处理相应的状况。

Because `recover` always returns `nil` unless called directly from a deferred function, deferred code can call library routines that themselves use `panic` and `recover` without failing. As an example, the deferred function in `safelyDo` might call a logging function before calling `recover`, and that logging code would run unaffected by the panicking state.

如果不是在被推迟的函数中调用，`recover` 总是返回 `nil` 值；正是因为这个原因，被推迟的执行逻辑里可以调用使用了 `panic` 和 `recover` 第三方库而不会失败退出。比如，在 `safelyDo` 的推迟函数里可以在调用 `recover` 之前调用一个日志函数，日志相关的代码不会受 panicking 的状态的影响。

With our recovery pattern in place, the `do` function (and anything it calls) can get out of any bad situation cleanly by calling `panic`. We can use that idea to simplify error handling in complex software. Let's look at an idealized version of a `regexp` package, which reports parsing errors by calling `panic` with a local error type. Here's the definition of `Error`, an `error` method, and the `Compile` function.

使用上面例子中的 recovery 的用法，`do` 函数（包含它调用的任何东西）可以通过调用 `panic` 干净利落地处理任何坏状况。我们可以把这种模式应用在复杂的软件中从而简化错误处理。让我们看一下理想状态下的 `regexp`包，可以通过调用 `panic` 并传入一个本地的错误类型来解析错误信息（准确地解析，继续看下面的解释）。下面的代码定义了 `Error` 类型， `error` 和 `Compile` 方法：

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

If `doParse` panics, the recovery block will set the return value to `nil`—deferred functions can modify named return values. It will then check, in the assignment to `err`, that the problem was a parse error by asserting that it has the local type `Error`. If it does not, the type assertion will fail, causing a run-time error that continues the stack unwinding as though nothing had interrupted it. This check means that if something unexpected happens, such as an index out of bounds, the code will fail even though we are using `panic` and `recover` to handle parse errors.

如果 `doParse` 报错，恢复的代码块会把返回值 regexp 设置为 `nil`——推迟的函数可以修改命名的返回值。然后它会在 `err` 的赋值语句断言 `e` 是不是一个本地类型的 `Error`。如果它不是本地的 `Error`，类型断言会失败，从而造成一个新的运行时错误，因此堆栈会继续展开就好像没有被中断过一样。这种检查意味着如果有未预知的事情发生，比如索引溢出，即使我们使用了 `panic` 和 `recover` 处理了解析错误的报错，代码依然会报错。

With error handling in place, the `error` method (because it's a method bound to a type, it's fine, even natural, for it to have the same name as the builtin `error` type) makes it easy to report parse errors without worrying about unwinding the parse stack by hand:

和错误处理一起的，还有一个 `error` 方法（因为 `error` 方法绑定到了类型上，而且与内建的 `error` 类型有相同的名字，因此使用起来更优雅自然）使得报告解析错误很容易，不用再想着手动释放堆栈。

```go
if pos == 0 {
    re.error("'*' illegal at start of expression")
}
```

Useful though this pattern is, it should be used only within a package. `Parse` turns its internal `panic` calls into `error` values; it does not expose `panics` to its client. That is a good rule to follow.

虽然这种模式很有用，但是应该只在一个包内部来使用。`Parse` 把内部的 `panic` 转换成了 `error` 值；它并没有把 `panic` 暴露给自己的调用者，这是一个最佳实践。

By the way, this re-panic idiom changes the panic value if an actual error occurs. However, both the original and new failures will be presented in the crash report, so the root cause of the problem will still be visible. Thus this simple re-panic approach is usually sufficient—it's a crash after all—but if you want to display only the original value, you can write a little more code to filter unexpected problems and re-panic with the original error. That's left as an exercise for the reader.

需要注意，如果真的有报错，recover 后再重新发起 panic 的习惯改变了 panic 的值。不过，在崩溃报告中原始报错和新的报错都会打印出来，因此导致错误的根源是可见的。虽然这种方式经常不够用——毕竟最后还是崩溃了——但是如果只想展示原始值，可以写一点代码来过滤未知问题然后用原生的错误重新发起 panic。这个可以作为课后练习大家自己来完成。

## A web server

Let's finish with a complete Go program, a web server. This one is actually a kind of web re-server. Google provides a service at `chart.apis.google.com` that does automatic formatting of data into charts and graphs. It's hard to use interactively, though, because you need to put the data into the URL as a query. The program here provides a nicer interface to one form of data: given a short piece of text, it calls on the chart server to produce a QR code, a matrix of boxes that encode the text. That image can be grabbed with your cell phone's camera and interpreted as, for instance, a URL, saving you typing the URL into the phone's tiny keyboard.

Here's the complete program. An explanation follows.

```go
package main

import (
    "flag"
    "html/template"
    "log"
    "net/http"
)

var addr = flag.String("addr", ":1718", "http service address") // Q=17, R=18

var templ = template.Must(template.New("qr").Parse(templateStr))

func main() {
    flag.Parse()
    http.Handle("/", http.HandlerFunc(QR))
    err := http.ListenAndServe(*addr, nil)
    if err != nil {
        log.Fatal("ListenAndServe:", err)
    }
}

func QR(w http.ResponseWriter, req *http.Request) {
    templ.Execute(w, req.FormValue("s"))
}

const templateStr = `
<html>
<head>
<title>QR Link Generator</title>
</head>
<body>
{{if .}}
<img src="http://chart.apis.google.com/chart?chs=300x300&cht=qr&choe=UTF-8&chl={{.}}" />
<br>
{{.}}
<br>
<br>
{{end}}
<form action="/" name=f method="GET"><input maxLength=1024 size=70
name=s value="" title="Text to QR Encode"><input type=submit
value="Show QR" name=qr>
</form>
</body>
</html>
`
```

The pieces up to `main` should be easy to follow. The one flag sets a default HTTP port for our server. The template variable `templ` is where the fun happens. It builds an HTML template that will be executed by the server to display the page; more about that in a moment.

The `main` function parses the flags and, using the mechanism we talked about above, binds the function `QR` to the root path for the server. Then `http.ListenAndServe` is called to start the server; it blocks while the server runs.

`QR` just receives the request, which contains form data, and executes the template on the data in the form value named `s`.

The template package `html/template` is powerful; this program just touches on its capabilities. In essence, it rewrites a piece of HTML text on the fly by substituting elements derived from data items passed to `templ.Execute`, in this case the form value. Within the template text (`templateStr`), double-brace-delimited pieces denote template actions. The piece from `{{if .}}` to `{{end}}` executes only if the value of the current data item, called `.` (dot), is non-empty. That is, when the string is empty, this piece of the template is suppressed.

The two snippets `{{.}}` say to show the data presented to the template—the query string—on the web page. The HTML template package automatically provides appropriate escaping so the text is safe to display.

The rest of the template string is just the HTML to show when the page loads. If this is too quick an explanation, see the [documentation](https://golang.google.cn/pkg/html/template/) for the template package for a more thorough discussion.

And there you have it: a useful web server in a few lines of code plus some data-driven HTML text. Go is powerful enough to make a lot happen in a few lines.