# golang语言规范

# Introduction

This is a reference manual for the Go programming language. For more information and other documents, see [golang.org](https://golang.google.cn/).

这是 Go 语言参考手册。可以访问 [golang.org](https://golang.org)（需科学上网） 或 [golang.google.cn](https://golang.google.cn)(国内可访问) 查看更多内容。

Go is a general-purpose language designed with systems programming in mind. It is strongly typed and garbage-collected and has explicit support for concurrent programming. Programs are constructed from *packages*, whose properties allow efficient management of dependencies.

Go 是一个通用的系统性语言。它支持强类型检查，有垃圾回收机制，同时语言层面支持并发编程。Go 源码由各个不同的包进行组织，以此高效地管理依赖关系。

The grammar is compact and regular, allowing for easy analysis by automatic tools such as integrated development environments.

Go 语言的语法紧凑且规范，方便 IDE（继承开发环境）等自动化工具进行语法分析，让开发者更好地开发。

# Notation（符号系统）

The syntax is specified using Extended Backus-Naur Form (EBNF):

本手册中的语法使用的是 EBNF（扩展的 BN 形式）

```go
Production  = production_name "=" [ Expression ] "." .
Expression  = Alternative { "|" Alternative } .
Alternative = Term { Term } .
Term        = production_name | token [ "…" token ] | Group | Option | Repetition .
Group       = "(" Expression ")" .
Option      = "[" Expression "]" .
Repetition  = "{" Expression "}" .
```

Productions are expressions constructed from terms and the following operators, in increasing precedence:

项目是一个个的表达式，由语法词汇和下面的操作符构成，下面的语法优先级依次增加：

```go
|   alternation
()  grouping
[]  option (0 or 1 times)
{}  repetition (0 to n times)
```

Lower-case production names are used to identify lexical tokens. Non-terminals are in CamelCase. Lexical tokens are enclosed in double quotes `""` or back quotes ````.

小写的名称用来表示字母符号。词汇符号一般包含在双引号或者反引号中。

The form `a … b` represents the set of characters from `a` through `b` as alternatives. The horizontal ellipsis `…` is also used elsewhere in the spec to informally denote various enumerations or code snippets that are not further specified. The character `…` (as opposed to the three characters `...`) is not a token of the Go language.

`a ... b` 表示从 `a` 到 `b` 之间的字符集合，且彼此之间可以互相替代。有地方会使用省略号来表示其他的情况或者其他代码片段。需要说明的是，省略号并不是 Go 语言里的语法，但是三个点是，二者应该区别开。

# Source code representation（源码）

Source code is Unicode text encoded in [UTF-8](https://en.wikipedia.org/wiki/UTF-8). The text is not canonicalized, so a single accented code point is distinct from the same character constructed from combining an accent and a letter; those are treated as two code points. For simplicity, this document will use the unqualified term *character* to refer to a Unicode code point in the source text.

源码是 UTF-8 编码的 Unicode 文本。文本并没有被规范化，因此一个带重音符号的字节码不同于一个重音符和一个字符组合成形成的字符；他们被当做两个不同的字节码。为了简化，本文使用术语“字符”来表示 Unicode 的字节码。

Each code point is distinct; for instance, upper and lower case letters are different characters.

每个字节码都是不同的，比如大写字符和小写字符是不同的字符。

Implementation restriction: For compatibility with other tools, a compiler may disallow the NUL character (U+0000) in the source text.

实现上的约束：为了其他工具的兼容性，编译器可能不允许在源码中出现 NUL 字符（U+0000）。

Implementation restriction: For compatibility with other tools, a compiler may ignore a UTF-8-encoded byte order mark (U+FEFF) if it is the first Unicode code point in the source text. A byte order mark may be disallowed anywhere else in the source.

实现上的约束：为了与其他工具兼容，如果源文件的第一个字符是 UTF-8 的字节序标识 （U+FEFE），编译器可以忽略它。文件中其他地方的字节序表示都是不允许存在的。

## Characters（字符）

The following terms are used to denote specific Unicode character classes:

下面的属于用来表示特殊的 Unicode 字符集：

```go
newline        = /* the Unicode code point U+000A */ .
unicode_char   = /* an arbitrary Unicode code point except newline */ .
unicode_letter = /* a Unicode code point classified as "Letter" */ .
unicode_digit  = /* a Unicode code point classified as "Number, decimal digit" */ .
```

In [The Unicode Standard 8.0](https://www.unicode.org/versions/Unicode8.0.0/), Section 4.5 "General Category" defines a set of character categories. Go treats all characters in any of the Letter categories Lu, Ll, Lt, Lm, or Lo as Unicode letters, and those in the Number category Nd as Unicode digits.

## Letters and digits（字母和数字）

The underscore character `_` (U+005F) is considered a letter.

下划线也被认为是一个字母。

```go
letter        = unicode_letter | "_" .
decimal_digit = "0" … "9" .
binary_digit  = "0" | "1" .
octal_digit   = "0" … "7" .
hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .
```

# Lexical elements（词汇元素）

## Comments（注释）

Comments serve as program documentation. There are two forms:

注释被认为是程序的文档，有两种形式：

1. *Line comments* start with the character sequence `//` and stop at the end of the line.【第一种形式是以 `//` 开始的字符串，一直到行尾都属于注释。
2. *General comments* start with the character sequence `/*` and stop with the first subsequent character sequence `*/`.【另一种是通用的注释，以标识 `/*`开始，以 `*/` 结束。

A comment cannot start inside a [rune](https://golang.google.cn/ref/spec#Rune_literals) or [string literal](https://golang.google.cn/ref/spec#String_literals), or inside a comment. A general comment containing no newlines acts like a space. Any other comment acts like a newline.

注释不能在 `rune` （符文）和字符串中开始，也不能在注释中声明注释。不包含换行符的常规注释就像空格一样对待。 其他的注释都像换行符。比如下面的例子：

```go

type Point struct {
	X float64
	Y float64 // 这种注释类似换行符
}

func (p Point) getX() float64 {
	return /*返回，这里是可以有注释的，编译时被视为空格*/ p.X
}

```



## Tokens（符号）

Tokens form the vocabulary of the Go language. There are four classes: *identifiers*, *keywords*, *operators and punctuation*, and *literals*. *White space*, formed from spaces (U+0020), horizontal tabs (U+0009), carriage returns (U+000D), and newlines (U+000A), is ignored except as it separates tokens that would otherwise combine into a single token. Also, a newline or end of file may trigger the insertion of a [semicolon](https://golang.google.cn/ref/spec#Semicolons). While breaking the input into tokens, the next token is the longest sequence of characters that form a valid token.

符号构成了 Go 语言的词汇。Go 语言中有四种类型的符号：标识符、关键词、操作符和标点符号、文字。空格（U+0020）、制表符（Tab，U+0009）、回车（U+000D）和行尾符（U+000A），这几个符号总是用来分割不同的符号，多于的会被直接忽略掉。行尾和文件尾可能会被编译器自动插入分号。

从源文件解析得到符号的算法是，从一个符号结束点开始，下一个可以形成有效的符号的最长的字符序列就是下一个符号。

## Semicolons（分号）

The formal grammar uses semicolons ";"  as terminators in a number of productions. Go programs may omit most of these semicolons using the following two rules:

形式语法使用分号  ";"  作为项目的终止标识。Go 程序可以根据下面的规则省略大部分的分号

1. When the input is broken into tokens, a semicolon is automatically inserted into the token stream immediately after a line's final token if that token is【当源文件被解析成为符号后，如果标识符是一行的最后一个标识符，且是下面的任意一种情况，则会自动添加分号
   - an [identifier](https://golang.google.cn/ref/spec#Identifiers)【标识符
   - an [integer](https://golang.google.cn/ref/spec#Integer_literals), [floating-point](https://golang.google.cn/ref/spec#Floating-point_literals), [imaginary](https://golang.google.cn/ref/spec#Imaginary_literals), [rune](https://golang.google.cn/ref/spec#Rune_literals), or [string](https://golang.google.cn/ref/spec#String_literals) literal【数字、浮点数、虚数、字节码或字符串
   - one of the [keywords](https://golang.google.cn/ref/spec#Keywords) `break`, `continue`, `fallthrough`, or `return`【是 break、continue、fallthrough 或 return 其中的一个关键词
   - one of the [operators and punctuation](https://golang.google.cn/ref/spec#Operators_and_punctuation) `++`, `--`, `)`, `]`, or `}`【是 ++、--、)、]、或 } 中的一个。
2. To allow complex statements to occupy a single line, a semicolon may be omitted before a closing `")"` or `"}"`.【当在一行中写一个复杂的语句，在 ) 和 } 之前的分号也可以省略。

To reflect idiomatic use, code examples in this document elide semicolons using these rules.

为了反映惯用方式，本文档中的代码示例使用这些规则省略分号。

## Identifiers（标识符）

Identifiers name program entities such as variables and types. An identifier is a sequence of one or more letters and digits. The first character in an identifier must be a letter.

标识符用以命名变量和类型，一个标识符是一串字符和数字的序列，且第一个字符必须是一个字母。

```go
identifier = letter { letter | unicode_digit } .
a
_x9
ThisVariableIsExported
αβ
```

Some identifiers are [predeclared](https://golang.google.cn/ref/spec#Predeclared_identifiers).

## Keywords（关键字）

The following keywords are reserved and may not be used as identifiers.

下面的关键字是预定义的，不可以作为标识符。

```go
break        default      func         interface    select
case         defer        go           map          struct
chan         else         goto         package      switch
const        fallthrough  if           range        type
continue     for          import       return       var
```

## Operators and punctuation（操作符与标点符号）

The following character sequences represent [operators](https://golang.google.cn/ref/spec#Operators) (including [assignment operators](https://golang.google.cn/ref/spec#assign_op)) and punctuation:

下面的符号表示操作符（包括赋值运算符）和标点符号。

```go
+    &     +=    &=     &&    ==    !=    (    )
-    |     -=    |=     ||    <     <=    [    ]
*    ^     *=    ^=     <-    >     >=    {    }
/    <<    /=    <<=    ++    =     :=    ,    ;
%    >>    %=    >>=    --    !     ...   .    :
     &^          &^=
```

## Integer literals（整数）

An integer literal is a sequence of digits representing an [integer constant](https://golang.google.cn/ref/spec#Constants). An optional prefix sets a non-decimal base: `0b` or `0B` for binary, `0`, `0o`, or `0O` for octal, and `0x` or `0X` for hexadecimal. A single `0` is considered a decimal zero. In hexadecimal literals, letters `a` through `f` and `A` through `F` represent values 10 through 15.

整数时一串用来表示整数常量的数字。可选的前缀用来表示非十进制数：`0b` 和 `0B` 表示二进制，`0`、`0o` 或 `0O` 用来表示八进制，`0x` 和 `0X` 用来表示十六进制。如果只有一个 `0` 则被解析为十进制的 `0`。在十六进制中，字母 `a` 到字母 `f` 和 字母 `A` 到 `F` 表示 10 到 15 的数值。

For readability, an underscore character `_` may appear after a base prefix or between successive digits; such underscores do not change the literal's value.

为了可读性，下划线（`_`）可以出现在表示几进制的前缀以后和连续的数字之间，这些下划线不会影响文本表示的数值。

```go
int_lit        = decimal_lit | binary_lit | octal_lit | hex_lit .
decimal_lit    = "0" | ( "1" … "9" ) [ [ "_" ] decimal_digits ] .
binary_lit     = "0" ( "b" | "B" ) [ "_" ] binary_digits .
octal_lit      = "0" [ "o" | "O" ] [ "_" ] octal_digits .
hex_lit        = "0" ( "x" | "X" ) [ "_" ] hex_digits .

decimal_digits = decimal_digit { [ "_" ] decimal_digit } .
binary_digits  = binary_digit { [ "_" ] binary_digit } .
octal_digits   = octal_digit { [ "_" ] octal_digit } .
hex_digits     = hex_digit { [ "_" ] hex_digit } .
42
4_2
0600
0_600
0o600
0O600       // second character is capital letter 'O'
0xBadFace
0xBad_Face
0x_67_7a_2f_cc_40_c6
170141183460469231731687303715884105727
170_141183_460469_231731_687303_715884_105727

_42         // an identifier, not an integer literal
42_         // invalid: _ must separate successive digits
4__2        // invalid: only one _ at a time
0_xBadFace  // invalid: _ must separate successive digits
```

## Floating-point literals（浮点数）

A floating-point literal is a decimal or hexadecimal representation of a [floating-point constant](https://golang.google.cn/ref/spec#Constants).

浮点数文本可以是十进制或者十六进制，用来表示浮点数常量。

A decimal floating-point literal consists of an integer part (decimal digits), a decimal point, a fractional part (decimal digits), and an exponent part (`e` or `E` followed by an optional sign and decimal digits). One of the integer part or the fractional part may be elided; one of the decimal point or the exponent part may be elided. An exponent value exp scales the mantissa (integer and fractional part) by 10exp.

十进制浮点数包含整数（十进制数），一个十进制的小数点，小数部分（十进制数），和一个指数部分（ `e` 或者 `E` ，后面跟着一个可选的符号和一个十进制数）。整数部分或者小数部分可以忽略一个，小数点或者指数部分可以忽略一个。指数部分的值 exp 表示把基数放大 10^exp 倍

A hexadecimal floating-point literal consists of a `0x` or `0X` prefix, an integer part (hexadecimal digits), a radix point, a fractional part (hexadecimal digits), and an exponent part (`p` or `P` followed by an optional sign and decimal digits). One of the integer part or the fractional part may be elided; the radix point may be elided as well, but the exponent part is required. (This syntax matches the one given in IEEE 754-2008 §5.12.3.) An exponent value exp scales the mantissa (integer and fractional part) by 2exp.

十六进制浮点数包含一个 `0x` 或者 `0X` 前缀，整数部分（十六进制数字）、小数点、小数部分（十六进制数字）、和一个指数部分（`p` 或者 `P`，后面跟着一个可选的符号和十进制的数字）。整数部分和小数部分可以省略一个，小数点可以省略，但是指数部分必须存在。指数部分的值可以把基数放大 2^exp 倍。

For readability, an underscore character `_` may appear after a base prefix or between successive digits; such underscores do not change the literal value.

为了可读性，下划线可以出现在前缀与连续的数字之间，这些下划线不会影响数字的值。

```go
float_lit         = decimal_float_lit | hex_float_lit .

decimal_float_lit = decimal_digits "." [ decimal_digits ] [ decimal_exponent ] |
                    decimal_digits decimal_exponent |
                    "." decimal_digits [ decimal_exponent ] .
decimal_exponent  = ( "e" | "E" ) [ "+" | "-" ] decimal_digits .

hex_float_lit     = "0" ( "x" | "X" ) hex_mantissa hex_exponent .
hex_mantissa      = [ "_" ] hex_digits "." [ hex_digits ] |
                    [ "_" ] hex_digits |
                    "." hex_digits .
hex_exponent      = ( "p" | "P" ) [ "+" | "-" ] decimal_digits .
0.
72.40
072.40       // == 72.40
2.71828
1.e+0
6.67428e-11
1E6
.25
.12345E+5
1_5.         // == 15.0
0.15e+0_2    // == 15.0

0x1p-2       // == 0.25
0x2.p10      // == 2048.0
0x1.Fp+0     // == 1.9375
0X.8p-0      // == 0.5
0X_1FFFP-16  // == 0.1249847412109375
0x15e-2      // == 0x15e - 2 (integer subtraction)

0x.p1        // invalid: mantissa has no digits
1p-2         // invalid: p exponent requires hexadecimal mantissa
0x1.5e-2     // invalid: hexadecimal mantissa requires p exponent
1_.5         // invalid: _ must separate successive digits
1._5         // invalid: _ must separate successive digits
1.5_e1       // invalid: _ must separate successive digits
1.5e_1       // invalid: _ must separate successive digits
1.5e1_       // invalid: _ must separate successive digits
```

## Imaginary literals（虚数）

An imaginary literal represents the imaginary part of a [complex constant](https://golang.google.cn/ref/spec#Constants). It consists of an [integer](https://golang.google.cn/ref/spec#Integer_literals) or [floating-point](https://golang.google.cn/ref/spec#Floating-point_literals) literal followed by the lower-case letter `i`. The value of an imaginary literal is the value of the respective integer or floating-point literal multiplied by the imaginary unit *i*.

虚数表示复数常量的虚数部分。它可以是正数也可以是浮点数，后面跟着一个 `i`表示它是虚数。虚数的值是它对应的整数或者浮点数的值与虚数的基础单元 `i` 的乘机。

```go
imaginary_lit = (decimal_digits | int_lit | float_lit) "i" .
```

For backward compatibility, an imaginary literal's integer part consisting entirely of decimal digits (and possibly underscores) is considered a decimal integer, even if it starts with a leading `0`.

为了向后兼容，虚数的整数部分如果可以解析为纯十进制数字（可能包含下划线），不管是不是以 0 为前缀开始都认为是十进制的数。（区别整数如果以 0 为前缀会被认为是八进制）

```go
0i
0123i         // == 123i for backward-compatibility
0o123i        // == 0o123 * 1i == 83i
0xabci        // == 0xabc * 1i == 2748i
0.i
2.71828i
1.e+0i
6.67428e-11i
1E6i
.25i
.12345E+5i
0x1p-2i       // == 0x1p-2 * 1i == 0.25i
```

## Rune literals（符文）

A rune literal represents a [rune constant](https://golang.google.cn/ref/spec#Constants), an integer value identifying a Unicode code point. A rune literal is expressed as one or more characters enclosed in single quotes, as in `'x'` or `'\n'`. Within the quotes, any character may appear except newline and unescaped single quote. A single quoted character represents the Unicode value of the character itself, while multi-character sequences beginning with a backslash encode values in various formats.

符文表示的是符文常量，一个整形数值表示一个 Unicode 的字节码。符文可以由包含在单引号中的一个或多个字符来表示，比如 `'x'` 和 `'\n'`。在单引号内，除了行尾符号和未经转义的单引号，任何字符都可能出现。单引号中一个单独的字符表示这个字符的 Unicode 值，也可以用一个反斜线加多个字符来表示不同编码形式的值。

The simplest form represents the single character within the quotes; since Go source text is Unicode characters encoded in UTF-8, multiple UTF-8-encoded bytes may represent a single integer value. For instance, the literal `'a'` holds a single byte representing a literal `a`, Unicode U+0061, value `0x61`, while `'ä'` holds two bytes (`0xc3` `0xa4`) representing a literal `a`-dieresis, U+00E4, value `0xe4`.

最简单的形式是用单个字符表示字符值。由于 Go 源码是以 UTF-8 编码的 Unicode 值，多个 UTF-8 编码的字节可能仅表示一个数字的值。比如，文本 `a` 只包含一个字节来表示文本 `a`，Unicode 的数值是 U+0061，值为 `0x61`，但是 `'ä'` 包含了两个字节 (`0xc3, 0xa4`)，表示的是一个 a 加一个分音符，Unicode 的数值是 U+00E4，真实值是 `0xe4`。

Several backslash escapes allow arbitrary values to be encoded as ASCII text. There are four ways to represent the integer value as a numeric constant: `\x` followed by exactly two hexadecimal digits; `\u` followed by exactly four hexadecimal digits; `\U` followed by exactly eight hexadecimal digits, and a plain backslash `\` followed by exactly three octal digits. In each case the value of the literal is the value represented by the digits in the corresponding base.

几个反斜线转义符号允许把数值编码成为 ASCII 文本。有四种方式表示整形数字常量：`\x` 后面跟两位十六进制数字， `\u` 后面跟四位十六进制， `\U` 后面跟八位十六进制，如果只有一个 `\` 符号可以跟三个八进制数字。在每种情况中，文本的数值都是对应进制下代表的数值。

Although these representations all result in an integer, they have different valid ranges. Octal escapes must represent a value between 0 and 255 inclusive. Hexadecimal escapes satisfy this condition by construction. The escapes `\u` and `\U` represent Unicode code points so within them some values are illegal, in particular those above `0x10FFFF` and surrogate halves.

虽然这些转义表示的都是整数，但是他们有不同的值范围。八进制转义必须表示 0 到 255 之间的数。十六进制数也是一样。转义 `\u` 和 `\U` 表示 Unicode 的字节码，所以有一些值是不合法的，特别像大于 `0x10FFFF` 的数以及半个字节码的那种数值。

After a backslash, certain single-character escapes represent special values:

反斜线可以转义一些特殊值的符号：

```go
\a   U+0007 alert or bell
\b   U+0008 backspace
\f   U+000C form feed
\n   U+000A line feed or newline
\r   U+000D carriage return
\t   U+0009 horizontal tab
\v   U+000b vertical tab
\\   U+005c backslash
\'   U+0027 single quote  (valid escape only within rune literals)
\"   U+0022 double quote  (valid escape only within string literals)
```

All other sequences starting with a backslash are illegal inside rune literals.

除了上面的符号，在符文中其他的转义符号都是不合法的。

```go
rune_lit         = "'" ( unicode_value | byte_value ) "'" .
unicode_value    = unicode_char | little_u_value | big_u_value | escaped_char .
byte_value       = octal_byte_value | hex_byte_value .
octal_byte_value = `\` octal_digit octal_digit octal_digit .
hex_byte_value   = `\` "x" hex_digit hex_digit .
little_u_value   = `\` "u" hex_digit hex_digit hex_digit hex_digit .
big_u_value      = `\` "U" hex_digit hex_digit hex_digit hex_digit
                           hex_digit hex_digit hex_digit hex_digit .
escaped_char     = `\` ( "a" | "b" | "f" | "n" | "r" | "t" | "v" | `\` | "'" | `"` ) .
'a'
'ä'
'本'
'\t'
'\000'
'\007'
'\377'
'\x07'
'\xff'
'\u12e4'
'\U00101234'
'\''         // rune literal containing single quote character
'aa'         // illegal: too many characters
'\xa'        // illegal: too few hexadecimal digits
'\0'         // illegal: too few octal digits
'\uDFFF'     // illegal: surrogate half
'\U00110000' // illegal: invalid Unicode code point
```

## String literals（字符串）

A string literal represents a [string constant](https://golang.google.cn/ref/spec#Constants) obtained from concatenating a sequence of characters. There are two forms: raw string literals and interpreted string literals.

字符串通过拼接遗传字符形成字符串常量。有两种形式：原生字符串和解析字符串。

Raw string literals are character sequences between back quotes, as in ``foo``. Within the quotes, any character may appear except back quote. The value of a raw string literal is the string composed of the uninterpreted (implicitly UTF-8-encoded) characters between the quotes; in particular, backslashes have no special meaning and the string may contain newlines. Carriage return characters ('\r') inside raw string literals are discarded from the raw string value.

原生字符串是包含在反引号之间的字符序列，比如 ``foo`。在反引号内，除了反引号外可以出现任何的字符。原生字符串的值是反引号之间未经解析的 UTF-8 编码的字符序列；在这种形式下，反引号没有任何特殊含义，且字符可以包含行尾符。这种模式下 `\r` 符号会被丢弃。

Interpreted string literals are character sequences between double quotes, as in `"bar"`. Within the quotes, any character may appear except newline and unescaped double quote. The text between the quotes forms the value of the literal, with backslash escapes interpreted as they are in [rune literals](https://golang.google.cn/ref/spec#Rune_literals) (except that `\'` is illegal and `\"` is legal), with the same restrictions. The three-digit octal (`\`*nnn*) and two-digit hexadecimal (`\x`*nn*) escapes represent individual *bytes* of the resulting string; all other escapes represent the (possibly multi-byte) UTF-8 encoding of individual *characters*. Thus inside a string literal `\377` and `\xFF` represent a single byte of value `0xFF`=255, while `ÿ`, `\u00FF`, `\U000000FF` and `\xc3\xbf` represent the two bytes `0xc3` `0xbf` of the UTF-8 encoding of character U+00FF.

解析字符串是在双引号之间的字符序列，比如 `"bar"`。在双引号之间，除了行尾符合未经转义的双引号，其他都可以出现。双引号之间的文本形成了它的值，反斜线如同在 "符文" 一节描述的那样对特殊字符进行解析（除了 `\'` 和 `\"` 的区别，字符串中前者是不合法的，后者是合法的）。三个数字的八进制（`\nnn`）和两个数字的十六进制 `\xnn`转义符号表示字符串中独立字节的值，其他的转义符号表示 UTF-8 的独立符号的值。因此在字符串中 `\377` 和 `\xFF` 表示一个独立的字节且其值为 `\xFF = 255`，而  `ÿ`, `\u00FF`, `\U000000FF` 和 `\xc3\xbf` 表示两个字节 `0xc3` 和 `0xbf` ，它是 UTF-8 编码的字符 U+00FF.

```go
string_lit             = raw_string_lit | interpreted_string_lit .
raw_string_lit         = "`" { unicode_char | newline } "`" .
interpreted_string_lit = `"` { unicode_value | byte_value } `"` .
`abc`                // same as "abc"
`\n
\n`                  // same as "\\n\n\\n"
"\n"
"\""                 // same as `"`
"Hello, world!\n"
"日本語"
"\u65e5本\U00008a9e"
"\xff\u00FF"
"\uD800"             // illegal: surrogate half
"\U00110000"         // illegal: invalid Unicode code point
```

These examples all represent the same string:

下面的例子表示的是同一个字符串：

```go
"日本語"                                 // UTF-8 input text
`日本語`                                 // UTF-8 input text as a raw literal
"\u65e5\u672c\u8a9e"                    // the explicit Unicode code points
"\U000065e5\U0000672c\U00008a9e"        // the explicit Unicode code points
"\xe6\x97\xa5\xe6\x9c\xac\xe8\xaa\x9e"  // the explicit UTF-8 bytes
```

If the source code represents a character as two code points, such as a combining form involving an accent and a letter, the result will be an error if placed in a rune literal (it is not a single code point), and will appear as two code points if placed in a string literal.

如果源码把一个字符表示成两个字节码，比如包含一个重音符和一个字符，如果把它作为一个符文使用会报错，如果把它当做字符串会变成两个字节码而存在。

# Constants（常量）

There are *boolean constants*, *rune constants*, *integer constants*, *floating-point constants*, *complex constants*, and *string constants*. Rune, integer, floating-point, and complex constants are collectively called *numeric constants*.

Go 中有布尔常量、符文常量、整数常量、浮点数常量、复数常量和字符串常量等，符文、整数、浮点数和复数统称为数字常量。

A constant value is represented by a [rune](https://golang.google.cn/ref/spec#Rune_literals), [integer](https://golang.google.cn/ref/spec#Integer_literals), [floating-point](https://golang.google.cn/ref/spec#Floating-point_literals), [imaginary](https://golang.google.cn/ref/spec#Imaginary_literals), or [string](https://golang.google.cn/ref/spec#String_literals) literal, an identifier denoting a constant, a [constant expression](https://golang.google.cn/ref/spec#Constant_expressions), a [conversion](https://golang.google.cn/ref/spec#Conversions) with a result that is a constant, or the result value of some built-in functions such as `unsafe.Sizeof` applied to any value, `cap` or `len` applied to [some expressions](https://golang.google.cn/ref/spec#Length_and_capacity), `real` and `imag` applied to a complex constant and `complex` applied to numeric constants. The boolean truth values are represented by the predeclared constants `true` and `false`. The predeclared identifier [iota](https://golang.google.cn/ref/spec#Iota) denotes an integer constant.

常量通过符文、整数、浮点数、虚数或者字符串的文本值来表示，它可以是一个表示常量、常量表达式或者转换成的常量的标识符，也可以是一些内建函数（比如 `unsafe.Sizeof`对任何值的运算，`cap` 和 `len` 对一些表达式的运算、`real` 和 `imag` 对复数常量的运算以及 `complex` 对一些数值常量的运算，等等 ）的运算结果。布尔真值可以用预定义的常量 true  和 false 来表示。预定义的标识符 `iota` 表示整数常量。

In general, complex constants are a form of [constant expression](https://golang.google.cn/ref/spec#Constant_expressions) and are discussed in that section.

一般情况下，复数常量是一种形式的常量表达式，对应的章节有所描述。

Numeric constants represent exact values of arbitrary precision and do not overflow. Consequently, there are no constants denoting the IEEE-754 negative zero, infinity, and not-a-number values.

数值常量可以用来表示任意的精度且不会造成溢出。也正因为这个原因，在 Go 数值常量中没有 IEEE-754 定义的负0、无穷大和非数值。

Constants may be [typed](https://golang.google.cn/ref/spec#Types) or *untyped*. Literal constants, `true`, `false`, `iota`, and certain [constant expressions](https://golang.google.cn/ref/spec#Constant_expressions) containing only untyped constant operands are untyped.

常量可以是有类型的也可以是没有类型的。`true, false, iota` 和一些只包含未定型常量操作数的特定的常量表达式都是未定型的。

A constant may be given a type explicitly by a [constant declaration](https://golang.google.cn/ref/spec#Constant_declarations) or [conversion](https://golang.google.cn/ref/spec#Conversions), or implicitly when used in a [variable declaration](https://golang.google.cn/ref/spec#Variable_declarations) or an [assignment](https://golang.google.cn/ref/spec#Assignments) or as an operand in an [expression](https://golang.google.cn/ref/spec#Expressions). It is an error if the constant value cannot be [represented](https://golang.google.cn/ref/spec#Representability) as a value of the respective type.

一个常量可以通过常量声明或者转换显式给定一个类型，或者在变量声明、赋值语句、作为表达式操作符时隐式地确定。如果常量的值不能赋值给对应的类型，就会报错。

An untyped constant has a *default type* which is the type to which the constant is implicitly converted in contexts where a typed value is required, for instance, in a [short variable declaration](https://golang.google.cn/ref/spec#Short_variable_declarations) such as `i := 0` where there is no explicit type. The default type of an untyped constant is `bool`, `rune`, `int`, `float64`, `complex128` or `string` respectively, depending on whether it is a boolean, rune, integer, floating-point, complex, or string constant.

当需要的是一个有类型的值，而传入的是一个未定义类型的常量时，这个未定型的常量会根据上下文隐式地转换成目标类型，比如在变量的短声明 `i := 0` 语句中就没有显式指定类型（但是 `i` 的值是 `int` 类型）。未定型常量的类型的默认类型一般是 `bool, rune, int, float64, complex128, string`，具体的取决于这个常量时什么类型的值。

Implementation restriction: Although numeric constants have arbitrary precision in the language, a compiler may implement them using an internal representation with limited precision. That said, every implementation must:

实现约束：虽然数字常量可以代表任意精度的值，但是编译器可以使用内部带精度的表示来实现他们。也就是说，任意一种实现必须满足：

- Represent integer constants with at least 256 bits.【至少用 256 位表示一个整形常量；
- Represent floating-point constants, including the parts of a complex constant, with a mantissa of at least 256 bits and a signed binary exponent of at least 16 bits.【浮点常量，包含复数常量的各个部分，其基数部分至少用 256 位表示，指数部分至少 16 位；
- Give an error if unable to represent an integer constant precisely.【如果不能够精确表示一个整数常量的值，报错；
- Give an error if unable to represent a floating-point or complex constant due to overflow.【如果因为溢出不能表示一个浮点数或者复数常量的值，报错；
- Round to the nearest representable constant if unable to represent a floating-point or complex constant due to limits on precision.【如果因为精度问题导致不能表示一个浮点数或者复数，则舍入一些精度取最接近的浮点数。

These requirements apply both to literal constants and to the result of evaluating [constant expressions](https://golang.google.cn/ref/spec#Constant_expressions).

上面的这些要求同时可以用在常量及其表达式中。

# Variables（变量）
A variable is a storage location for holding a value. The set of permissible values is determined by the variable's type.

可以把变量理解为一个值的存储位置，在 golang 代码中一个变量就代表了一个“坑位”，而这个“坑位”上能保存什么类型的值（整数、小数、数组、结构等）是由这个变量的类型决定的。

A variable declaration or, for function parameters and results, the signature of a function declaration or function literal reserves storage for a named variable. Calling the built-in function new or taking the address of a composite literal allocates storage for a variable at run time. Such an anonymous variable is referred to via a (possibly implicit) pointer indirection.

变量“坑位”的声明无处不在，比如在我们显式声明变量的时候，比如在函数声明参数列表和返回值列表的时候，比如声明函数变量的时候（ 类似 `var f := func(){}`)，比如按照函数的一般形式声明函数的时候，其实都是在某个变量声明“坑位”。上面提到的这些坑位在编译阶段就已经占好了，其实在运行阶段也可以声明“坑位”（这些坑位是没有变量与之对应的，可以认为是匿名的“坑位”），比如使用关键词 `new`的时候，还比如获取某个复合实体[todo link]的地址的时候。一般情况下，这些匿名的坑位通过**指针**间接地被引用。

Structured variables of array, slice, and struct types have elements and fields that may be addressed individually. Each such element acts like a variable.

数组、切片和结构体所对应的的变量比较特殊，这些变量要么包含一些基础元素，要么有子字段，需要注意的是，这些基础元素和子字段都是可以分别取地址信息，然后就可以像普通变量一样操作这些变量对应的元素了。

The static type (or just type) of a variable is the type given in its declaration, the type provided in the new call or composite literal, or the type of an element of a structured variable. Variables of interface type also have a distinct dynamic type, which is the concrete type of the value assigned to the variable at run time (unless the value is the predeclared identifier nil, which has no type). The dynamic type may vary during execution but values stored in interface variables are always assignable to the static type of the variable.

一个变量的类型在变量声明的时候就确定了（静态类型），当我们调用**new**方法时需要指定类型，当我们声明复合类型（比如数组、序列、结构等）时也要指定相应的类型，当我们定义复合类型时这些类型的子元素或者子字段也需要指定类型。接口类型的变量也有类型的概念（动态类型），只不过这个类型相对来说不太固定，因为它和我们赋值给接口变量的值有关；如果我们赋的值是一个整形数字，那么接口变量的类型就是整形，如果我们赋的值是一个小数，那么接口变量的类型就是小数，以此类推。当然，`nil` 是没有类型的，因为好几个类型都可以被赋值为 `nil`。值得注意的是，在代码执行过程中，动态类型可能会发生改变，但是在接口变量中保存的值总可以再赋值给相应的静态类型；换句话说，一个值被赋值给接口变量后，这个值的“本质”不会改变。

```go
var x interface{}  // x is nil and has static type interface{}
var v *T           // v has value nil, static type *T
x = 42             // x has value 42 and dynamic type int
x = v              // x has value (*T)(nil) and dynamic type *T
```

A variable's value is retrieved by referring to the variable in an expression; it is the most recent value assigned to the variable. If a variable has not yet been assigned a value, its value is the zero value for its type.

当我们在表达式中引用一个变量的时候，其实是在引用这个变量的“坑位”上所保存的值；当然，这个值是我们最后一次赋值给这个变量的，而不是之前赋过的值。而如果一个变量被声明后还没有被赋过值，那么这个变量的值默认是它对应类型的零值。


# Types（类型）
A type determines a set of values together with operations and methods specific to those values. A type may be denoted by a type name, if it has one, or specified using a type literal, which composes a type from existing types.

确定了一个类型后，这个类型可能的值也就确定了，这个类型的值可能的一些操作和方法也确定了。如果一个类型已经有名称了（比如 int、float、bool 等），可以直接通过类型名称来表征；否则也可以通过已有的类型来组装新的类型。

```go
Type      = TypeName | TypeLit | "(" Type ")" .
TypeName  = identifier | QualifiedIdent .
TypeLit   = ArrayType | StructType | PointerType | FunctionType | InterfaceType |
	    SliceType | MapType | ChannelType .
```

The language predeclares certain type names. Others are introduced with type declarations. Composite types—array, struct, pointer, function, interface, slice, map, and channel types—may be constructed using type literals.

Golang 预先定义了几个特定的类型，除此之外都是通过类型声明的方式创建的。复合类型，比如数组、结构体、指针、函数、接口、切片、哈希以及信道，这些都可以通过类型声明的语法来创建。

Each type T has an underlying type: If T is one of the predeclared boolean, numeric, or string types, or a type literal, the corresponding underlying type is T itself. Otherwise, T's underlying type is the underlying type of the type to which T refers in its type declaration.

每个类型都可以对应到一个底层的数据类型。比如有一个自主创建的类型 T，如果 T 是 Golang 内建的（布尔、数字、字符串），或者 数组、结构体、指针、函数、接口、切片、哈希、信道等，那么 T 所对应的底层的数据类型就是 T。否则，T 的底层数据类型就是声明 T 时指定的那个类型（比如 `type newInt int` 中 `newInt` 的底层数据类型时 `int`）。

```go
type (
	A1 = string
	A2 = A1
)

type (
	B1 string
	B2 B1
	B3 []B1
	B4 B3
)
```

The underlying type of string, A1, A2, B1, and B2 is string. The underlying type of []B1, B3, and B4 is []B1.

对于上面的代码示例，string、A1、A2、B1 和 B2 的底层数据类型是 string，[]B1、B3 和 B4 的底层数据类型是 `[]B1`。


## Method sets（方法集）
A type may have a method set associated with it. The method set of an interface type is its interface. The method set of any other type T consists of all methods declared with receiver type T. The method set of the corresponding pointer type *T is the set of all methods declared with receiver *T or T (that is, it also contains the method set of T). Further rules apply to structs containing embedded fields, as described in the section on struct types. Any other type has an empty method set. In a method set, each method must have a unique non-blank method name.

一般情况下，类型都会绑定一个方法集。一个接口类型的方法集是这个接口包含的那些方法集。而一般类型（假设为 T） 的方法集是在这个类型（T）上声明的所有的方法。一个类型的指针（假设为 `*T`）的方法集是在 `*T` 和 `T` 上**声明的所有方法的集合**（也就是说，`*T` 对应的方法涵盖了 `T` 对应的方法）。如[结构类型](#)一节中所描述的那样，对于包含嵌入字段的结构，有另外的规则来确定其方法集。当然，还有一些类型是不包含任何方法的。

在一个方法集合中，每个方法必须是非空且唯一的，当然这里和很多语言不一样。一个类型的集合里定义两个函数，假如它们有相同的函数名，无论他们的传入参数是什么，也无论他们的返回类型是什么，都认为这两个函数是冲突的，因此代码编译的时候会报错。

The method set of a type determines the interfaces that the type implements and the methods that can be called using a receiver of that type.

一个类型的方法集直接确定了这个类型实现了什么接口。调用类型的方法集也很简单，在类型的值上直接通过 “.” 运算符调用就可以了。

## Boolean types（布尔类型）
A boolean type represents the set of Boolean truth values denoted by the predeclared constants true and false. The predeclared boolean type is bool; it is a defined type.

布尔类型表示由预定义的常量 `true` 和 `false` 的真值集合。golang 预定义的布尔类型是 `bool`，和很多语言一样，属于内置类型，没什么好说的。

## Numeric types（数字类型）
A numeric type represents sets of integer or floating-point values. The predeclared architecture-independent numeric types are:

数字类型表示整数和浮点数的集合。各个精度的预定义数字类型有下面这些：

```go
uint8       the set of all unsigned  8-bit integers (0 to 255)
uint16      the set of all unsigned 16-bit integers (0 to 65535)
uint32      the set of all unsigned 32-bit integers (0 to 4294967295)
uint64      the set of all unsigned 64-bit integers (0 to 18446744073709551615)

int8        the set of all signed  8-bit integers (-128 to 127)
int16       the set of all signed 16-bit integers (-32768 to 32767)
int32       the set of all signed 32-bit integers (-2147483648 to 2147483647)
int64       the set of all signed 64-bit integers (-9223372036854775808 to 9223372036854775807)

float32     the set of all IEEE-754 32-bit floating-point numbers
float64     the set of all IEEE-754 64-bit floating-point numbers

complex64   the set of all complex numbers with float32 real and imaginary parts
complex128  the set of all complex numbers with float64 real and imaginary parts

byte        alias for uint8
rune        alias for int32
```
The value of an n-bit integer is n bits wide and represented using two's complement arithmetic.

一个 n 位的整数是由 n 个二进制位构成的，可以认为它的字节宽度是 n。

There is also a set of predeclared numeric types with implementation-specific sizes:

为了方便使用，golang 预定义了几个不显示包含位数的数字类型，这些数字类型的位数会根据平台的不同而变化（32位平台与64位平台）。

```go
uint     either 32 or 64 bits
int      same size as uint
uintptr  an unsigned integer large enough to store the uninterpreted bits of a pointer value
```
To avoid portability issues all numeric types are defined types and thus distinct except byte, which is an alias for uint8, and rune, which is an alias for int32. Explicit conversions are required when different numeric types are mixed in an expression or assignment. For instance, int32 and int are not the same type even though they may have the same size on a particular architecture.

为了避免移植的问题，所有数字类型都根据可能的宽度定义了具体的类型，并且彼此之间是有区别的。不过也是有特例的，比如 byte 和 rune，前者其实是 uint8 的别称，后者是 int32 的别称。当不同数字类型之间转换时，显式的转换时必要的，比如 虽然 int32 和 int 可能有一样的字节宽度（在32位系统上），但是它们两个依然会被认为是两个不同的类型，如果要把 int 类型的值赋值给 int32 类型的变量，必须要显式地声明一下，`int32(valOfInt)`。

## Array types（数组类型）
An array is a numbered sequence of elements of a single type, called the element type. The number of elements is called the length of the array and is never negative.
数组是一个特定类型的元素的有数序列（即数目也是数组类型中的一部分，区别于切片）。一个数组的长度即数组中元素的个数，因此这个长度肯定是一个非负数。

```go
ArrayType   = "[" ArrayLength "]" ElementType .
ArrayLength = Expression .
ElementType = Type .
```

The length is part of the array's type; it must evaluate to a non-negative constant representable by a value of type int. The length of array a can be discovered using the built-in function len. The elements can be addressed by integer indices 0 through len(a)-1. Array types are always one-dimensional but may be composed to form multi-dimensional types.

正如上面所说的，数组的长度是数组类型的一部分，数组长度必须是非负的值，且必须是一个常量或者能计算得出常量的表达式。我们可以通过内建的 `len` 函数来得到数组的长度。

和其他语言类似，数组中的元素通过 0 到 len(a)-1 进行索引。数组类型表示的是一个一维的结构，不过可以通过组合的方式形成多维的结构（如果数组 A 中的每个元素都是一个数组 B，那么可以认为数组 A 是多维的，我们首先通过 A 的索引确定某个元素，然后再通过索引找到 B 数组中的某个值）。


```go
[32]byte
[2*N] struct { x, y int32 }
[1000]*float64
[3][5]int
[2][2][2]float64  // same as [2]([2]([2]float64))
```

## Slice types（切片类型）
A slice is a descriptor for a contiguous segment of an underlying array and provides access to a numbered sequence of elements from that array. A slice type denotes the set of all slices of arrays of its element type. The number of elements is called the length of the slice and is never negative. The value of an uninitialized slice is nil.

切片是一个描述符，用来获取数组（成为底层数组）中一段连续的数据值。一个切片类型能够表示其底层数组的所有切片集合（区别于数组长度属于数组类型的一部分）。与数组类似，切片中元素的个数就是切片的长度，因此切片的长度也是一个非负数。如果一个切片的变量没有被初始化，那么这个变量的值是 nil。

```go
SliceType = "[" "]" ElementType .
```

The length of a slice s can be discovered by the built-in function len; unlike with arrays it may change during execution. The elements can be addressed by integer indices 0 through len(s)-1. The slice index of a given element may be less than the index of the same element in the underlying array.

切片的长度可以通过内建函数 `len` 获取得到，不过切片的长度是可以改变的，这一点和数组不一样，数组的长度是固定的。切片中的元素可以通过 0 到 len(s)-1 进行索引。需要说明的是，给定元素的切片索引可能小于底层数组中相同元素的索引。

A slice, once initialized, is always associated with an underlying array that holds its elements. A slice therefore shares storage with its array and with other slices of the same array; by contrast, distinct arrays always represent distinct storage.

切片一旦初始化后，就和底层的数组绑定在一起了。如果好几个切片有一个共同的底层数组，那么这几个切片共享数组的存储空间，更详细地，假如两个切片共享同一个底层数组，当我们通过一个切片修改了某个元素的值以后，另一个切片就可能会感知到这种改变。作为对比，不同的数据之间是不存在内存共享的，它们是不同的值，彼此之间不会干扰。

The array underlying a slice may extend past the end of the slice. The capacity is a measure of that extent: it is the sum of the length of the slice and the length of the array beyond the slice; a slice of length up to that capacity can be created by slicing a new one from the original slice. The capacity of a slice a can be discovered using the built-in function cap(a).

对于切片来说，它对应的底层数组的长度能够超过切片的长度。我们一般用“容量”（capacity）来表征一个底层数组的容量，其实也是切片的最大容量。如果一个切片的长度超过了其底层数组的容量，就会在老的切片基础上动态创建一个新的切片，当然这时候其对应的底层数组也已经改变了（扩容了）。一个切片的容量可以通过内建的 `cap` 函数来获取。

A new, initialized slice value for a given element type T is made using the built-in function make, which takes a slice type and parameters specifying the length and optionally the capacity. A slice created with make always allocates a new, hidden array to which the returned slice value refers. That is, executing

```go
make([]T, length, capacity)
```

produces the same slice as allocating an array and slicing it, so these two expressions are equivalent:

```go
make([]int, 50, 100)
new([100]int)[0:50]
```

我们可以通过 Golang 内建的 `make` 函数创建一个新的且初始化过的切片，只需要传入切片的类型、切片长度和切片的容量即可。通过 `make` 函数创建的切片总是会隐式地创建一个新的数组作为所创建的切片的底层数组。鉴于此，我们可以知道 `make([]int, 50, 100)` 等效于 `new([100]int)[0:50]`。

Like arrays, slices are always one-dimensional but may be composed to construct higher-dimensional objects. With arrays of arrays, the inner arrays are, by construction, always the same length; however with slices of slices (or arrays of slices), the inner lengths may vary dynamically. Moreover, the inner slices must be initialized individually.

和数组一样，切片是一个一维的结构，但是可以通过组装的方式构建高维的数组。对与一个高维的数组来说，每个内部数组（低维数组）的长度需要保持一致（毕竟长度也是数组类型的一部分）；但是对于切片来说则没有上面这种限制，内部切片（低维切片）的长度是可以不一致的。需要一提的是，高维切片的内部切片（低维切片）必须分别初始化才可以（否则低维切片就是 nil 了，编译的时候就会报错）。


## Struct types（结构体类型）
A struct is a sequence of named elements, called fields, each of which has a name and a type. Field names may be specified explicitly (IdentifierList) or implicitly (EmbeddedField). Within a struct, non-blank field names must be unique.

所谓结构体，可以理解成包含一系列元素的结构块。一般把结构体中的元素称为字段，每个字段会有自己的名称和类型。字段的名称可以是显式定义的（普通的字段），也可以是隐式定义的（嵌套字段）。在结构体中，非空字段的名称必须是唯一的，不能重名。

```go
StructType    = "struct" "{" { FieldDecl ";" } "}" .
FieldDecl     = (IdentifierList Type | EmbeddedField) [ Tag ] .
EmbeddedField = [ "*" ] TypeName .
Tag           = string_lit .
```

```go
// An empty struct.
struct {}

// A struct with 6 fields.
struct {
	x, y int
	u float32
	_ float32  // padding
	A *[]int
	F func()
}
```
A field declared with a type but no explicit field name is called an embedded field. An embedded field must be specified as a type name T or as a pointer to a non-interface type name *T, and T itself may not be a pointer type. The unqualified type name acts as the field name.

如果一个字段只有类型但没有显式的字段名称，这种字段称为嵌入字段。一个嵌入字段必然是由类型的名称 T 或者 非接口 类型的指针 *T 来定义的，这里的类型 T 可能本身就是一个指针，也可能不是指针。对于嵌入字段，因为没有显示定义名称，一般情况下会把类型名称充当字段名称。

```go
// A struct with four embedded fields of types T1, *T2, P.T3 and *P.T4
struct {
	T1        // field name is T1
	*T2       // field name is T2
	P.T3      // field name is T3
	*P.T4     // field name is T4
	x, y int  // field names are x and y
}
```

The following declaration is illegal because field names must be unique in a struct type:

下面结构体的声明是不合法的，因为字段名称重复了（T，*T 和 *P.T 三个嵌入字段的默认名称都是 T）。

```go
struct {
	T     // conflicts with embedded field *T and *P.T
	*T    // conflicts with embedded field T and *P.T
	*P.T  // conflicts with embedded field T and *T
}
```
A field or method f of an embedded field in a struct x is called promoted if x.f is a legal selector that denotes that field or method f.

如果结构体 x 中的嵌入字段的子字段或者方法可以通过 `x.f` 的方式调用，我们就说子字段和方法被提升到了 x。

Promoted fields act like ordinary fields of a struct except that they cannot be used as field names in composite literals of the struct.

结构体中，被提升的字段和普通字段一样使用。唯一不同的点是，如果我们通过结构体的声明语法声明一个结构值，嵌入字段的初始化比较麻烦。

Given a struct type S and a defined type T, promoted methods are included in the method set of the struct as follows:

* If S contains an embedded field T, the method sets of S and *S both include promoted methods with receiver T. The method set of *S also includes promoted methods with receiver *T.
* If S contains an embedded field *T, the method sets of S and *S both include promoted methods with receiver T or *T.

再说一个嵌套字段的方法提升的规则，给定一个结构体类型 S 和一个已定义的结构体 T，方法提升的规则如下：
1. 如果 S 包含嵌套字段 T（区别于 *T），那么 S 和 *S 的方法集均包含在类型 T 上定义的方法， *S 的方法集还包含在类型 *T 上定义的方法。
2. 如果 S 包含嵌套字段 *T（区别于 T），那么 S 和 *S 的方法集包含在类型 T 和类型 *T 上定义的方法。


A field declaration may be followed by an optional string literal tag, which becomes an attribute for all the fields in the corresponding field declaration. An empty tag string is equivalent to an absent tag. The tags are made visible through a reflection interface and take part in type identity for structs but are otherwise ignored.

结构体中，字段声明时可以选择性地定义标签，这个标签会成为那条字段声明语句中包含的所有字段的属性。如果一个标签只包含一个空的字符串，那么这个标签就是一个空标签（就等于没有定义）。目前标签只在反射时会使用，在运行时环境中动态地表征结构体，其他的应用场合标签会被直接忽略。

```
struct {
	x, y float64 ""  // an empty tag string is like an absent tag
	name string  "any string is permitted as a tag"
	_    [4]byte "ceci n'est pas un champ de structure"
}

// A struct corresponding to a TimeStamp protocol buffer.
// The tag strings define the protocol buffer field numbers;
// they follow the convention outlined by the reflect package.
struct {
	microsec  uint64 `protobuf:"1"`
	serverIP6 uint64 `protobuf:"2"`
}
```


## Pointer types（指针类型）
A pointer type denotes the set of all pointers to variables of a given type, called the base type of the pointer. The value of an uninitialized pointer is nil.

```go
PointerType = "*" BaseType .
BaseType    = Type .
*Point
*[4]int
```

指针类型表示所有特定类型的变量的指针集合，我们可以把这个特定的类型称为指针的基类型。一个指针变量如果没有初始化，那么这个指针变量的值是 nil。


## Function types（函数类型）
A function type denotes the set of all functions with the same parameter and result types. The value of an uninitialized variable of function type is nil.

一个函数类型表示所有具有相同参数和返回类型的函数集合。如果一个函数类型的变量没有初始化，则其值为 nil。

```go
FunctionType   = "func" Signature .
Signature      = Parameters [ Result ] .
Result         = Parameters | Type .
Parameters     = "(" [ ParameterList [ "," ] ] ")" .
ParameterList  = ParameterDecl { "," ParameterDecl } .
ParameterDecl  = [ IdentifierList ] [ "..." ] Type .
```

Within a list of parameters or results, the names (IdentifierList) must either all be present or all be absent. If present, each name stands for one item (parameter or result) of the specified type and all non-blank names in the signature must be unique. If absent, each type stands for one item of that type. Parameter and result lists are always parenthesized except that if there is exactly one unnamed result it may be written as an unparenthesized type.

在函数定义的参数列表中，参数值的名称要么全部都指定，要么全部都不指定；返回列表同样的道理。如果制定了参数的名称，那么每个名称就代表了一个特定类型的元素，并且这些所有非空的名称必须是唯一的；对返回列表来说，同样的道理。如果参数的名称未指定，只有类型的值，那么每个类型代表这个类型的一个元素（用来占坑）。对于参数列表来说，必须要包含在一对小括号里面；对于返回列表来说，除非只返回一个值且未指定返回值名称，否则也要包含在一对小括号里面。

The final incoming parameter in a function signature may have a type prefixed with .... A function with such a parameter is called variadic and may be invoked with zero or more arguments for that parameter.

函数定义中最后一个参数可以是一个带有三个“.”前缀的类型，包含这种参数的函数叫做可变参数函数。如果一个函数只包含一个可变参数，在调用这个函数的时候，就可以随意传入 0 个或者多个指定类型的参数。

如果把函数声明中的每个参数都当做一个坑位，普通类型的参数占的坑都必须且只能放一个变量值，但是可变参数所占的坑比较特殊，可以放任意多指定类型的值（包含0个）。

```go
func()
func(x int) int
func(a, _ int, z float32) bool
func(a, b int, z float32) (bool)
func(prefix string, values ...int)
func(a, b int, z float64, opt ...interface{}) (success bool)
func(int, int, float64) (float64, *[]int)
func(n int) func(p *T)
```

## Interface types（接口类型）
An interface type specifies a method set called its interface. A variable of interface type can store a value of any type with a method set that is any superset of the interface. Such a type is said to implement the interface. The value of an uninitialized variable of interface type is nil.

接口类型定义了这个接口所包含的方法集合。一个特定接口类型的变量，可以保存任何一个实现了其接口所有方法的类型的值。此时我们说这个类型实现了这个接口。如果接口的变量没有初始化，这个变量的值是 nil。

```go
InterfaceType      = "interface" "{" { MethodSpec ";" } "}" .
MethodSpec         = MethodName Signature | InterfaceTypeName .
MethodName         = identifier .
InterfaceTypeName  = TypeName .
```

As with all method sets, in an interface type, each method must have a unique non-blank name.

一个接口类型定义的所有方法，每个方法的名称都必须是非空且唯一的。

```go
// A simple File interface.
interface {
	Read([]byte) (int, error)
	Write([]byte) (int, error)
	Close() error
}
interface {
	String() string
	String() string  // illegal: String not unique
	_(x int)         // illegal: method must have non-blank name
}
```

More than one type may implement an interface. For instance, if two types S1 and S2 have the method set
```go
func (p T) Read(p []byte) (n int, err error)   { return … }
func (p T) Write(p []byte) (n int, err error)  { return … }
func (p T) Close() error                       { return … }
```
(where T stands for either S1 or S2) then the File interface is implemented by both S1 and S2, regardless of what other methods S1 and S2 may have or share.

对于一个接口类型，可能有很多类型都实现了这个接口。比如，如果两个类型 S1 和 S2 都有方法 `Read([]byte) (int, error)`、`Write([]byte) (int, error)`、`Close() error`，不管有没有定义其他的方法，我们都可以说 S1 和 S2 都实现了接口 `File`。

A type implements any interface comprising any subset of its methods and may therefore implement several distinct interfaces. For instance, all types implement the empty interface:
```go
interface{}
```

如果一个类型实现了一个方法集合，那么这个类型实现了由其任意方法子集合组成的接口，因此这个类型有可能实现了好几个接口。比如，因为空接口中不包含任意的方法，我们可以说所有的类型（内建、自建）都实现了空接口。

Similarly, consider this interface specification, which appears within a type declaration to define an interface called Locker:

```go
type Locker interface {
	Lock()
	Unlock()
}
```
If S1 and S2 also implement
```go
func (p T) Lock() { … }
func (p T) Unlock() { … }
```
they implement the Locker interface as well as the File interface.

同样的代理，如果一个接口 `Locker` 只包含 `Lock()` 和 `Unlock` 两个方法，假设上面提到的 S1 和 S2 同时也实现了这两个方法，那么我们就说 S1 和 S2 同时实现了接口 `File` 和 接口 `Locker`。

An interface T may use a (possibly qualified) interface type name E in place of a method specification. This is called embedding interface E in T; it adds all (exported and non-exported) methods of E to the interface T.

接口是是可以嵌套的。如果定义一个接口 T，可以通过引用接口 E 来简介引入接口 E 中限定的所有方法（包含可导出的方法和不可导出的方法）。

```go
type ReadWriter interface {
	Read(b Buffer) bool
	Write(b Buffer) bool
}

type File interface {
	ReadWriter  // same as adding the methods of ReadWriter
	Locker      // same as adding the methods of Locker
	Close()
}

type LockedFile interface {
	Locker
	File        // illegal: Lock, Unlock not unique
	Lock()      // illegal: Lock not unique
}
```
An interface type T may not embed itself or any interface type that embeds T, recursively.

当然，接口的嵌入是不允许循环导入的情况产生的。

```go
// illegal: Bad cannot embed itself
type Bad interface {
	Bad
}

// illegal: Bad1 cannot embed itself using Bad2
type Bad1 interface {
	Bad2
}
type Bad2 interface {
	Bad1
}
```

## Map types（map类型）
A map is an unordered group of elements of one type, called the element type, indexed by a set of unique keys of another type, called the key type. The value of an uninitialized map is nil.

一个 map 是一组特定类型的元素的组合，元素之间是无序的，且这些元素可以通过一个唯一性的键值进行索引。如果一个 map 变量没有初始化，那么它的值是 nil。

```go
MapType     = "map" "[" KeyType "]" ElementType .
KeyType     = Type .
```

The comparison operators == and != must be fully defined for operands of the key type; thus the key type must not be a function, map, or slice. If the key type is an interface type, these comparison operators must be defined for the dynamic key values; failure will cause a run-time panic.

作为 map 的键值类型，必须可以通过 `==` 和 `!=` 进行比较，因此 map 的键值不可以是函数、map、或 切片，这几个类型的值都是不可比较的。如果键值是一个接口类型，那么传入的键值的数值也应该是可以比较的（定义了 `==` 和 `!=` 操作符；否则会造成运行时错误。

```go
map[string]int
map[*T]struct{ x, y float64 }
map[string]interface{}
```

The number of map elements is called its length. For a map m, it can be discovered using the built-in function len and may change during execution. Elements may be added during execution using assignments and retrieved with index expressions; they may be removed with the delete built-in function.

map 中元素的个数称作 map 的长度。对于一个 map 的变量 m，它的长度可以通过内置的函数 `len` 获取到，而且代码执行过程中这个值可能是会改变的。map 中的元素可以在代码执行过程中动态的增加和索引获取，也可以通过内置的 `delete` 函数删除相应的索引及其值。

A new, empty map value is made using the built-in function make, which takes the map type and an optional capacity hint as arguments:
```go
make(map[string]int)
make(map[string]int, 100)
```

一个新的空 map 的值可以通过内置的 `make` 函数进行创建并初始化，只需要传入 map 的类型可一个可选的容量值即可。

The initial capacity does not bound its size: maps grow to accommodate the number of items stored in them, with the exception of nil maps. A nil map is equivalent to an empty map except that no elements may be added.

值得注意的是，通过 `make` 初始 map 的时候容量并不会限制 map 的长度：除非 map 还没有初始化（还是 nil），否则 map 会根据其元素的数量动态的增长。一个值为 nil 的 map 和一个元素个数为 0 的 map 是等效的，不同的是可以向后者增加元素，但是绝不可以向 nil 中增加元素，否则会报错的。

## Channel types（信道类型）
A channel provides a mechanism for concurrently executing functions to communicate by sending and receiving values of a specified element type. The value of an uninitialized channel is nil.
```go
ChannelType = ( "chan" | "chan" "<-" | "<-" "chan" ) ElementType .
```
信道类型提供了一种机制，通过发送和接收特定类型的元素来完成并发执行的函数之间的通信（多协程之间的通信）。如果一个信道变量没有初始化，则变量的值为 nil。

The optional `<-` operator specifies the channel direction, send or receive. If no direction is given, the channel is bidirectional. A channel may be constrained only to send or only to receive by assignment or explicit conversion.

```go
chan T          // can be used to send and receive values of type T
chan<- float64  // can only be used to send float64s
<-chan int      // can only be used to receive ints
```
可选的操作符 `<-` 限定了信道的方向：发送还是接收。如果没有方向上的限定，信道默认是双向的，既可以发送也可以接收。一个信道可以通过加 `<-` 操作符来显式限定只能发送数据或者只能接收数据。我们可以发散思考一下，如果一个信道只能发送不能接收，那么这个信道有意义吗？其实我们可以这么思考，如果在一个作用域里我们把一个双向的信道显示转换为只能发送或者只能接收的信道，是不是能够提升我们代码的可读性和可维护性？（答案是yes）。


The `<-` operator associates with the leftmost chan possible:
```go
chan<- chan int    // same as chan<- (chan int)
chan<- <-chan int  // same as chan<- (<-chan int)
<-chan <-chan int  // same as <-chan (<-chan int)
chan (<-chan int)
```

值得一提的是， 当嵌套层级比较深的时候，操作符`<-`总是和最左的信道绑定到一起。不过为了可读性和可维护性，就像一个很长的算式一样，多个信道嵌套的情况建议大家添加括号用来表意。

A new, initialized channel value can be made using the built-in function make, which takes the channel type and an optional capacity as arguments:
```go
make(chan int, 100)
```
我们可以使用 Golang 内建的 `make` 函数来创建并初始化信道变量，只需要传入信道的类型和一个可选的信道容量即可。

The capacity, in number of elements, sets the size of the buffer in the channel. If the capacity is zero or absent, the channel is unbuffered and communication succeeds only when both a sender and receiver are ready. Otherwise, the channel is buffered and communication succeeds without blocking if the buffer is not full (sends) or not empty (receives). A nil channel is never ready for communication.

如果我们在使用 `make` 函数创建信道的时候传入了大于 0 的容量值，那么就是声明我们创建的信道是一个带缓存（buffer）的信道。如果我们传入了 0 或者省略容量值，那么创建的就是一个不带缓冲的信道。不带缓冲的信道只会在发送者和接受者同时就绪的时候才可以正常工作。作为比较，带缓冲的信道只要缓冲区没有满，无论接受者是否就绪，发送者都可以一直发送数据；只要缓冲区没有空，无论发送者是否还是就绪状态，接受者都可以一直获取数据。如果一个信道变量没有初始化，则是 nil 值，这种情况想使没有办法用来发送数据或接收数据的。

A channel may be closed with the built-in function close. The multi-valued assignment form of the receive operator reports whether a received value was sent before the channel was closed.

我们可以使用 golang 内建的 `close` 函数来关闭一个信道。获取信道中的数值时，我们可以通过多值赋值的方式（类似于 `i, ok := <-myIntChan` ）来感知获取到的值是否是在信道关闭之前传入的。

A single channel may be used in send statements, receive operations, and calls to the built-in functions cap and len by any number of goroutines without further synchronization. Channels act as first-in-first-out queues. For example, if one goroutine sends values on a channel and a second goroutine receives them, the values are received in the order sent.

单个通道可用于发送语句，可以用于接收操作，也可以在任意多的协程（goroutine）中安全地调用内建的 `cap` 和 `len` 函数获取信道的容量和当前信道中元素的数目，这里是不需要显式加锁的，因为信道的实现中已经考虑了这种情况在内部加了隐式的锁。信道是一个先进先出的结构，比如，如果一个协程在一个信道中有序地发送了几个值，那么在另一个协程中会依序接收到相同的值。


# Properties of types and values（类型及其值的属性）
## Type identity（类型标识）
Two types are either identical or different.

两个类型要么是一样的要么是不一样的。

A defined type is always different from any other type. Otherwise, two types are identical if their underlying type literals are structurally equivalent; that is, they have the same literal structure and corresponding components have identical types. In detail:

一个已定义好的类型大概率和其他的类型是不一样的。除非这两个类型的底层的类型是一样的，也就是说除非他们有相同的结构且各个属性保持一致，比如：

* Two array types are identical if they have identical element types and the same array length.
* Two slice types are identical if they have identical element types.
* Two struct types are identical if they have the same sequence of fields, and if corresponding fields have the same names, and identical types, and identical tags. Non-exported field names from different packages are always different.
* Two pointer types are identical if they have identical base types.
* Two function types are identical if they have the same number of parameters and result values, corresponding parameter and result types are identical, and either both functions are variadic or neither is. Parameter and result names are not required to match.
* Two interface types are identical if they have the same set of methods with the same names and identical function types. Non-exported method names from different packages are always different. The order of the methods is irrelevant.
* Two map types are identical if they have identical key and element types.
* Two channel types are identical if they have identical element types and the same direction.

* 如果两个数组类型有相同的元素类型且长度相同，那么这两个数组类型就是一样的。
* 如果两个切片有相同的元素类型，那么这两个切片类型就是一样的。
* 如果两个结构体有顺序一致且相同的字段，且对应的字段有相同的名称、相同的类型、和相同的标签定义，那么这两个结构体就是一样的。在不同包中如果某个字段是非导出的，那么这两个结构体将是不一样的。
* 如果两个指针类型其基础类型时一样的，那么这两个指针类型时一样的。
* 如果两个函数类型，它们有相同数量的参数和返回列表，对应的参数和返回值的类型时一致的，两个函数要么都是不定参数函数要么都不是不定参数函数，那么这两个函数类型就是一样的。提一下，参数和返回值的名称是无所谓的。
* 如果两个接口类型，它们包含的函数集是一致的，函数名、函数类型都一致，无所谓这些函数的声明顺序，这两个接口是一样的。不同包中的非导出函数总是不相同的。

Given the declarations

给定下面的声明：

```go
type (
	A0 = []string
	A1 = A0
	A2 = struct{ a, b int }
	A3 = int
	A4 = func(A3, float64) *A0
	A5 = func(x int, _ float64) *[]string
)

type (
	B0 A0
	B1 []string
	B2 struct{ a, b int }
	B3 struct{ a, c int }
	B4 func(int, float64) *B0
	B5 func(x int, y float64) *A1
)

type	C0 = B0
```

these types are identical:

```go
A0, A1, and []string
A2 and struct{ a, b int }
A3 and int
A4, func(int, float64) *[]string, and A5

B0 and C0
[]int and []int
struct{ a, b *T5 } and struct{ a, b *T5 }
func(x int, y float64) *[]string, func(int, float64) (result *[]string), and A5
```

B0 and B1 are different because they are new types created by distinct type definitions; func(int, float64) *B0 and func(x int, y float64) *[]string are different because B0 is different from []string.

B0 和 B1 是不一样的，因为他们的定义方式是不一样的，`func(int, float64) *B0` 和 `func(x int, y float64) *[]string` 也不一样，是因为 B0 与 []string 是不一样的。

## Assignability（可赋值性）
A value x is assignable to a variable of type T ("x is assignable to T") if one of the following conditions applies:

* x’s type is identical to T.
* x's type V and T have identical underlying types and at least one of V or T is not a defined type.
* T is an interface type and x implements T.
* x is a bidirectional channel value, T is a channel type, x's type V and T have identical element types, and at least one of V or T is not a defined type.
* x is the predeclared identifier nil and T is a pointer, function, slice, map, channel, or interface type.
* x is an untyped constant representable by a value of type T.

如果满足下面的一个条件，x 值就可以赋值给类型为 T 的变量：
* x 的类型就是 T；
* x 的类型 V 和 T 有相同的底层类型，并且 V 或 T 至少有一个不是一个新定义的类型；
* T 是一个接口类型，且 x 实现了 T 接口中的所有方法；
* x 是一个双向的信道值，T 是一个信道类型，x 的类型 V 和 T 有相同的类型元素，且 V 和 T 中至少有一个不是新定义的类型；
* x 是预定的 nil 值，而 T 是一个指针、函数、切片、map、信道或者接口类型；
* x 是一个为指定类型的常量，其可以通过类型 T 的值来表示。

## Representability（代表性）
A constant x is representable by a value of type T if one of the following conditions applies:

* x is in the set of values determined by T.
* T is a floating-point type and x can be rounded to T's precision without overflow. Rounding uses IEEE 754 round-to-even rules but with an IEEE negative zero further simplified to an unsigned zero. Note that constant values never result in an IEEE negative zero, NaN, or infinity.
* T is a complex type, and x's components real(x) and imag(x) are representable by values of T's component type (float32 or float64).

如果满足下列条件之一，则常数x可由类型 T 的值表示：
* x 的值在类型为 T 的值的所有集合中；
* T 是一个浮点数类型，x 可以四舍五入到 T 的精度且不会造成溢出。对于一个无符号的 0，使用 IEEE754 四舍五入到基数的规范，比 IEEE的负零的规范更简单。需要注意的，常量值永远不会等于 IEEE 的负0、无穷小（NaN） 和 无穷大。
* T 是一个符合类型，并且 x 的实数部分和虚数部分可以分别由 T 类型对应的部分来代表（基本上就是两种类型，float32 或者 float64）。

```go
x                   T           x is representable by a value of T because

'a'                 byte        97 is in the set of byte values
97                  rune        rune is an alias for int32, and 97 is in the set of 32-bit integers
"foo"               string      "foo" is in the set of string values
1024                int16       1024 is in the set of 16-bit integers
42.0                byte        42 is in the set of unsigned 8-bit integers
1e10                uint64      10000000000 is in the set of unsigned 64-bit integers
2.718281828459045   float32     2.718281828459045 rounds to 2.7182817 which is in the set of float32 values
-1e-1000            float64     -1e-1000 rounds to IEEE -0.0 which is further simplified to 0.0
0i                  int         0 is an integer value
(42 + 0i)           float32     42.0 (with zero imaginary part) is in the set of float32 values
```

```go
x                   T           x is not representable by a value of T because
0                   bool        0 is not in the set of boolean values
'a'                 string      'a' is a rune, it is not in the set of string values
1024                byte        1024 is not in the set of unsigned 8-bit integers
-1                  uint16      -1 is not in the set of unsigned 16-bit integers
1.1                 int         1.1 is not an integer value
42i                 float32     (0 + 42i) is not in the set of float32 values
1e1000              float64     1e1000 overflows to IEEE +Inf after rounding
```

# Blocks（代码块）
A block is a possibly empty sequence of declarations and statements within matching brace brackets.

一个代码块可能是一个空的声明和语法序列，被包裹在一对**大括号**内部。

```go
Block = "{" StatementList "}" .
StatementList = { Statement ";" } .
```

In addition to explicit blocks in the source code, there are implicit blocks:

1. The universe block encompasses all Go source text.
2. Each package has a package block containing all Go source text for that package.
3. Each file has a file block containing all Go source text in that file.
4. Each "if", "for", and "switch" statement is considered to be in its own implicit block.
5. Each clause in a "switch" or "select" statement acts as an implicit block.

Blocks nest and influence scoping.

在代码中除了显式地定义块，还存在很多隐式的代码块：
1. 包含所有 Go 源码的全局的块；
2. 每个包有一个包范围的块，包含了整个包所有的源码；
3. 每个文件有一个文件范围的块，包含所有当前文件的源码；
4. 每个 `if`、`for`、`switch` 语法中都可以认为存在一个它们自己的隐式的块；
5. `switch`、`select` 中的每个条目都可以认为是一个隐式的块。

块可以嵌套并且会影响作用域。

# Declarations and scope（声明与作用域）
A declaration binds a non-blank identifier to a constant, type, variable, function, label, or package. Every identifier in a program must be declared. No identifier may be declared twice in the same block, and no identifier may be declared in both the file and package block.

一个声明可以给一个非空的标识符绑定常量、类型、变量、函数、标签或包。程序中的每个标识符都必须要声明。在同一个代码块中没有标识符可以被声明两次，在文件和包中也没有标识符可以被声明两次。

The blank identifier may be used like any other identifier in a declaration, but it does not introduce a binding and thus is not declared. In the package block, the identifier init may only be used for init function declarations, and like the blank identifier it does not introduce a new binding.

在一个声明中，空标识符可以像其他表示符那样使用，不过它不会绑定任何值因此它不会被声明。在一个包代码块中，标识符 “init” 只能用来声明 `init` 函数，就像空标识符那样它也不会绑定新的其他值，如果绑定会编译出错。

```go
Declaration   = ConstDecl | TypeDecl | VarDecl .
TopLevelDecl  = Declaration | FunctionDecl | MethodDecl .
```

The scope of a declared identifier is the extent of source text in which the identifier denotes the specified constant, type, variable, function, label, or package.

声明的标识符的作用域是源文本的范围，在这个范围里，标识符表示特定的常量、类型、变量、函数、标签或包。

Go is lexically scoped using blocks:

1. The scope of a predeclared identifier is the universe block.
2. The scope of an identifier denoting a constant, type, variable, or function (but not method) declared at top level (outside any function) is the package block.
3. The scope of the package name of an imported package is the file block of the file containing the import declaration.
4. The scope of an identifier denoting a method receiver, function parameter, or result variable is the function body.
5. The scope of a constant or variable identifier declared inside a function begins at the end of the ConstSpec or VarSpec (ShortVarDecl for short variable declarations) and ends at the end of the innermost containing block.
6. The scope of a type identifier declared inside a function begins at the identifier in the TypeSpec and ends at the end of the innermost containing block.

Go 使用**块**来确定语法上的作用域：
1. Go 中预定义的标识符的作用域是全局块；
2. 在高层（任何函数外）声明的表示常量、类型、变量或函数（不是方法）的标识符的作用域是包所确定的块；
3. 被导入的包的包名，其所用域是导入包的文件；
4. 函数接收者、函数参数、返回值变量的作用域是整个函数体；
5. 在函数体中声明的常量标识符、变量标识符，它们的作用域是自声明的位置开始到声明所在的作用域的最后。（？）
6. 在函数内部声明的类型标识符的作用域是从声明开始到声明所在的作用域的最后位置。

An identifier declared in a block may be redeclared in an inner block. While the identifier of the inner declaration is in scope, it denotes the entity declared by the inner declaration.

在一个块中声明的标识符可能会在此块内部的一个块里面重新声明。虽然内部块声明的标识符同时也在外部作用域里，但是它表示的是在内部作用域中声明的实体（类似于被覆盖了）。

The package clause is not a declaration; the package name does not appear in any scope. Its purpose is to identify the files belonging to the same package and to specify the default package name for import declarations.

包语句不是一种声明，包名不出现在任何的作用域中。包语句的存在是为了表征哪些文件处于同一个包内，同时也定义了在导入此包时默认的包名。

## Label scopes（标签的作用域）
Labels are declared by labeled statements and are used in the "break", "continue", and "goto" statements. It is illegal to define a label that is never used. In contrast to other identifiers, labels are not block scoped and do not conflict with identifiers that are not labels. The scope of a label is the body of the function in which it is declared and excludes the body of any nested function.

标签通过标签声明语句进行声明，并与 `break`、`continue`、`goto` 结合使用。不允许定义未经使用的标签，否则会编译报错。与其他的标识符不同，标签是没有块作用域概念的，当然标签不能和已有的普通标识符冲突。标签的作用域是标签声明语句所在的函数体（假如有嵌套函数，不包含嵌套函数的函数体）。

## Blank identifier（空白标识符）
The blank identifier is represented by the underscore character _. It serves as an anonymous placeholder instead of a regular (non-blank) identifier and has special meaning in declarations, as an operand, and in assignments.

空白表示符用下划线 “_” 符号来表示。在声明语句，可以用它来作为一个匿名的占位符而不是给定一个非空的具有特定含义的标识符；在赋值语句中，也可以用它来作为操作数使用（在赋值符号左边）。

## Predeclared identifiers（预定义的标识符）
The following identifiers are implicitly declared in the universe block:

下面的标识符是 Go 预定义的标识符，全局作用域都可以直接调用：

```go
Types:
	bool byte complex64 complex128 error float32 float64
	int int8 int16 int32 int64 rune string
	uint uint8 uint16 uint32 uint64 uintptr

Constants:
	true false iota

Zero value:
	nil

Functions:
	append cap close complex copy delete imag len
	make new panic print println real recover
```

## Exported identifiers（导出的标识符）
An identifier may be exported to permit access to it from another package. An identifier is exported if both:

1. the first character of the identifier's name is a Unicode upper case letter (Unicode class "Lu"); and
2. the identifier is declared in the package block or it is a field name or method name.

All other identifiers are not exported.

一个标识符可以被导出，从而允许从其他的包中直接使用这个标识符。一个可导出的标识符满足下面两个条件：

1. 标识符的第一个字符是 Unicode 的大写格式（Unicode 类 “Lu”）；同时
2. 这个标识符时在包的块中声明的，或者它是一个字段的名称或者方法的名称。

所有其他的标识符都没有导出。


## Uniqueness of identifiers（标识符的唯一性）
Given a set of identifiers, an identifier is called unique if it is different from every other in the set. Two identifiers are different if they are spelled differently, or if they appear in different packages and are not exported. Otherwise, they are the same.

给定一个标识符的集合，如果其中一个标识符与其他的任何一个标识符都不一样，那么就说这个标识符是唯一的。如果两个标识符的拼写不一样、或者他们出现在不同的包且没有导出，那么这两个标识符就是不同的；否则就是相同的。

## Constant declarations（常量的声明）
A constant declaration binds a list of identifiers (the names of the constants) to the values of a list of constant expressions. The number of identifiers must be equal to the number of expressions, and the nth identifier on the left is bound to the value of the nth expression on the right.

常量的声明可以把常量表达式列表绑定到一个一一对应的标识符列表（常量名）。标识符的个数必须要与表达书个数相同，等号左边的第 n 个标识符会绑定到等号右边的第 n 个表达式上面。

```go
ConstDecl      = "const" ( ConstSpec | "(" { ConstSpec ";" } ")" ) .
ConstSpec      = IdentifierList [ [ Type ] "=" ExpressionList ] .

IdentifierList = identifier { "," identifier } .
ExpressionList = Expression { "," Expression } .
```

If the type is present, all constants take the type specified, and the expressions must be assignable to that type. If the type is omitted, the constants take the individual types of the corresponding expressions. If the expression values are untyped constants, the declared constants remain untyped and the constant identifiers denote the constant values. For instance, if the expression is a floating-point literal, the constant identifier denotes a floating-point constant, even if the literal's fractional part is zero.

如果给定了常量的类型，所有的常量都使用这个类型，所有的表达式都必须能够赋值成为这个类型。如果类型是缺省的，常量会被判定为各个相关表达式的类型值。如果一个表达式的值也没有指定类型，那么声明的常量也会是没有类型的，此时常量标识符表示的是常量的值（类型待定）。比如，如果一个表达式是浮点字面值，则常量标识符表示的是一个浮点型常量，即使它的小数部分是0。

```go
const Pi float64 = 3.14159265358979323846
const zero = 0.0         // untyped floating-point constant
const (
	size int64 = 1024
	eof        = -1  // untyped integer constant
)
const a, b, c = 3, 4, "foo"  // a = 3, b = 4, c = "foo", untyped integer and string constants
const u, v float32 = 0, 3    // u = 0.0, v = 3.0
```

Within a parenthesized const declaration list the expression list may be omitted from any but the first ConstSpec. Such an empty list is equivalent to the textual substitution of the first preceding non-empty expression list and its type if any. Omitting the list of expressions is therefore equivalent to repeating the previous list. The number of identifiers must be equal to the number of expressions in the previous list. Together with the iota constant generator this mechanism permits light-weight declaration of sequential values:



如果是包含在括号里的常量声明方式，且声明的是一个常量列表，此时除了第一个常量的常量表达式不可以省略，其他跟随的常量都可以省略常量表达式。此时，所有常量表达式为空的常量都从第一个非空的常量表达式继承其值和其类型（如果类型存在的情况下）。省略常量表达式其实就等效于重复前面的常量表达式。标识符的数量必须等于上一个列表中的表达式数量。通过与 iota 常量产生器搭配使用，这种机制可以以轻量级地声明序列值：

```go
const (
	Sunday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Partyday
	numberOfDays  // this constant is not exported
)
```

## Iota（iota）
Within a constant declaration, the predeclared identifier iota represents successive untyped integer constants. Its value is the index of the respective ConstSpec in that constant declaration, starting at zero. It can be used to construct a set of related constants:

在一个常量表达式中，预定义的标识符 `iota` 表示连续的无类型整数常量。iota 的值从 0 开始一直递增。可以使用它来构建一个相关联的常量的集合：

```go
const (
	c0 = iota  // c0 == 0
	c1 = iota  // c1 == 1
	c2 = iota  // c2 == 2
)

const (
	a = 1 << iota  // a == 1  (iota == 0)
	b = 1 << iota  // b == 2  (iota == 1)
	c = 3          // c == 3  (iota == 2, unused)
	d = 1 << iota  // d == 8  (iota == 3)
)

const (
	u         = iota * 42  // u == 0     (untyped integer constant)
	v float64 = iota * 42  // v == 42.0  (float64 constant)
	w         = iota * 42  // w == 84    (untyped integer constant)
)

const x = iota  // x == 0
const y = iota  // y == 0
```

By definition, multiple uses of iota in the same ConstSpec all have the same value:

通过定义，在一个常量声明语句中的多次使用，iota 的值是一样的：

```go
const (
	bit0, mask0 = 1 << iota, 1<<iota - 1  // bit0 == 1, mask0 == 0  (iota == 0)
	bit1, mask1                           // bit1 == 2, mask1 == 1  (iota == 1)
	_, _                                  //                        (iota == 2, unused)
	bit3, mask3                           // bit3 == 8, mask3 == 7  (iota == 3)
)
```

This last example exploits the implicit repetition of the last non-empty expression list.

最后一个例子利用了最后一个非空表达式列表的隐式重复机制，跳过了 iota=2 的情况。


## Type declarations （类型声明）
A type declaration binds an identifier, the type name, to a type. Type declarations come in two forms: alias declarations and type definitions.

一个类型声明把一个已定义的类型名称绑定到另一个类型上面。类型声明有两种方式：别名声明 和 类型定义。

```go
TypeDecl = "type" ( TypeSpec | "(" { TypeSpec ";" } ")" ) .
TypeSpec = AliasDecl | TypeDef .
```

### Alias declarations（别名声明）
An alias declaration binds an identifier to the given type.
一个别名声明把一个标识绑定到了给定的类型。

```go
AliasDecl = identifier "=" Type .
```

Within the scope of the identifier, it serves as an alias for the type.

在这个别名标识的定义域内，这个表示就代表这个类型。

```go
type (
	nodeList = []*Node  // nodeList and []*Node are identical types
	Polar    = polar    // Polar and polar denote identical types
)
```

### Type definitions（类型定义）
A type definition creates a new, distinct type with the same underlying type and operations as the given type, and binds an identifier to it.

区别于类型别名，类型定义是创建一个新的**完全不同**的类型并用一个新的标识符进行标识，这个类型与给定的类型有相同的底层类型和相同的操作（仅限于操作，并不意味着有相同的方法，见下面的内容）。

```go
TypeDef = identifier Type .
```

The new type is called a defined type. It is different from any other type, including the type it is created from.

通过类型定义创建的新类型与其他的类型是不一样的；即使是创建这个新类型时指定的那个类型，它们两个也是不一样的。

```go
type (
	Point struct{ x, y float64 }  // Point and struct{ x, y float64 } are different types
	polar Point                   // polar and Point denote different types
)

type TreeNode struct {
	left, right *TreeNode
	value *Comparable
}

type Block interface {
	BlockSize() int
	Encrypt(src, dst []byte)
	Decrypt(src, dst []byte)
}
```

A defined type may have methods associated with it. It does not inherit any methods bound to the given type, but the method set of an interface type or of elements of a composite type remains unchanged:

一个定义好的新类型可以定义与之相关的方法。值得注意的是，新类型不会继承创建这个新类型时给定的类型的方法；但是如果给定的类型时一个接口类型，或者是通过嵌入的方式创建一个新的类型（结构体），那么方法集合是会被继承的。

```go
// A Mutex is a data type with two methods, Lock and Unlock.
type Mutex struct         { /* Mutex fields */ }
func (m *Mutex) Lock()    { /* Lock implementation */ }
func (m *Mutex) Unlock()  { /* Unlock implementation */ }

// NewMutex has the same composition as Mutex but its method set is empty.
type NewMutex Mutex

// The method set of PtrMutex's underlying type *Mutex remains unchanged,
// but the method set of PtrMutex is empty.
type PtrMutex *Mutex

// The method set of *PrintableMutex contains the methods
// Lock and Unlock bound to its embedded field Mutex.
type PrintableMutex struct {
	Mutex
}

// MyBlock is an interface type that has the same method set as Block.
type MyBlock Block
```

Type definitions may be used to define different boolean, numeric, or string types and associate methods with them:

类型定义可以用来定义区别于基础的 布尔、数字或字符串类型的新类型，并给这些新类型定义一些新的方法。

```go
type TimeZone int

const (
	EST TimeZone = -(5 + iota)
	CST
	MST
	PST
)

func (tz TimeZone) String() string {
	return fmt.Sprintf("GMT%+dh", tz)
}
```

## Variable declarations（变量声明）
A variable declaration creates one or more variables, binds corresponding identifiers to them, and gives each a type and an initial value.

变量声明可以用来创建一个或多个变量，给这些变量绑定标识符，同时给每个变量一个类型和初始值。

```go
VarDecl     = "var" ( VarSpec | "(" { VarSpec ";" } ")" ) .
VarSpec     = IdentifierList ( Type [ "=" ExpressionList ] | "=" ExpressionList ) .
```

```go
var i int
var U, V, W float64
var k = 0
var x, y float32 = -1, -2
var (
	i       int
	u, v, s = 2.0, 3.0, "bar"
)
var re, im = complexSqrt(-1)
var _, found = entries[name]  // map lookup; only interested in "found"
```

If a list of expressions is given, the variables are initialized with the expressions following the rules for assignments. Otherwise, each variable is initialized to its zero value.

如果给定了一个表达式列表，变量会根据赋值语句的规则依次被这些表达式赋值。如果没有表达式列表，那么这些变量会被初始化为他们对应类型的零值。

If a type is present, each variable is given that type. Otherwise, each variable is given the type of the corresponding initialization value in the assignment. If that value is an untyped constant, it is first implicitly converted to its default type; if it is an untyped boolean value, it is first implicitly converted to type bool. The predeclared value nil cannot be used to initialize a variable with no explicit type.

如果类型给定了，那么每个变量都被定义为这个类型。如果类型不给定，那么每个变量的类型都会根据被赋的值的类型来确定。如果赋的值是一个未指定类型的常量，这个常量首先会被隐式地转换为这个值的默认类型；比如如果赋的值是一个布尔值，那么它首先就会被隐式地转换为布尔类型。golang 中预先定义的 nil 值不能用来初始化一个未显式定义类型的变量。

```go
var d = math.Sin(0.5)  // d is float64
var i = 42             // i is int
var t, ok = x.(T)      // t is T, ok is bool
var n = nil            // illegal
```

Implementation restriction: A compiler may make it illegal to declare a variable inside a function body if the variable is never used.

实现上的一个限制：如果在一个函数体内声明了一个变量但是却没有使用过这个变量，编译器会报错。

## Short variable declarations（变量声明短方式）
A short variable declaration uses the syntax:

可以通过一种短方式来声明变量，语法是：

```go
ShortVarDecl = IdentifierList ":=" ExpressionList .
```

It is shorthand for a regular variable declaration with initializer expressions but no types:

这种短方式是那种带初始值的常规的变量声明方式的一种变形：

```go
"var" IdentifierList = ExpressionList .
```


```go
i, j := 0, 10
f := func() int { return 7 }
ch := make(chan int)
r, w, _ := os.Pipe()  // os.Pipe() returns a connected pair of Files and an error, if any
_, y, _ := coord(p)   // coord() returns three values; only interested in y coordinate
```

Unlike regular variable declarations, a short variable declaration may redeclare variables provided they were originally declared earlier in the same block (or the parameter lists if the block is the function body) with the same type, and at least one of the non-blank variables is new. As a consequence, redeclaration can only appear in a multi-variable short declaration. Redeclaration does not introduce a new variable; it just assigns a new value to the original.

不想常规的变量声明方式，变量声明的短方式可能会重新声明一些之前已经存在的同类型的变量（比如重新声明的变量存在当前函数的参数列表中），此时应该至少有一个变量是新声明的才可以。正是因为这个原因，重新声明仅可以出现在一个多变量声明的短方式中。重声明并不会引入新的变量，它只是把一个新的值赋值给之前的变量。

```go
field1, offset := nextField(str, 0)
field2, offset := nextField(str, offset)  // redeclares offset
a, a := 1, 2                              // illegal: double declaration of a or no new variable if a was declared elsewhere
```

Short variable declarations may appear only inside functions. In some contexts such as the initializers for "if", "for", or "switch" statements, they can be used to declare local temporary variables.

变量声明的短方式可能只会在函数中出现，尤其在`if`、`for`、`switch`这些语句中，经常被用来声明各种临时变量。

## Function declarations（函数声明）
A function declaration binds an identifier, the function name, to a function.

一个函数声明把一个标识符，即函数名，绑定到一个函数上面去。

```go
FunctionDecl = "func" FunctionName Signature [ FunctionBody ] .
FunctionName = identifier .
FunctionBody = Block .
```

If the function's signature declares result parameters, the function body's statement list must end in a terminating statement.

如果一个函数的声明中包含返回的参数，此时函数体中必须包含一个 `return` 语句表中止。

```go
func IndexRune(s string, r rune) int {
	for i, c := range s {
		if c == r {
			return i
		}
	}
	// invalid: missing return statement
}
```

A function declaration may omit the body. Such a declaration provides the signature for a function implemented outside Go, such as an assembly routine.

如果一个函数声明中省略了函数体，此时函数声明的是一个在 Go 外面实现的函数，比如 Go 源码中包含大量使用汇编语言写的函数，然后链接到对应的 Go 函数使用。

```go
func min(x int, y int) int {
	if x < y {
		return x
	}
	return y
}

func flushICache(begin, end uintptr)  // implemented externally
```

## Method declarations（方法声明，有接受者）
A method is a function with a receiver. A method declaration binds an identifier, the method name, to a method, and associates the method with the receiver's base type.

如果一个函数有接受者，此时称这种函数为方法。方法生命可以把一个标识符，即方法名称，绑定到一个函数上，同时会把这个函数关联到接受者的基础类型上。

```go
MethodDecl = "func" Receiver MethodName Signature [ FunctionBody ] .
Receiver   = Parameters .
```

The receiver is specified via an extra parameter section preceding the method name. That parameter section must declare a single non-variadic parameter, the receiver. Its type must be a defined type T or a pointer to a defined type T. T is called the receiver base type. A receiver base type cannot be a pointer or interface type and it must be defined in the same package as the method. The method is said to be bound to its receiver base type and the method name is visible only within selectors for type T or *T.

接受者在方法名的前面，通过一个小括号里的特殊变量来指定。小括号里的参数必须声明一个非可变参数，这个参数就是接受者。接受者的类型必须是一个定义好的新类型 T 或者这个新类型的指针，此时 T 称为接受者的基础类型。接受者的基础类型不能是一个指针，也不能是一个接口类型；而且它必须和它的方法一起声明在同一个包里。方法是被绑定到其接受者的基础类型上面的，且方法的名称只对基础类型 T 或其指针类型 *T 可见。

A non-blank receiver identifier must be unique in the method signature. If the receiver's value is not referenced inside the body of the method, its identifier may be omitted in the declaration. The same applies in general to parameters of functions and methods.

一个非空的接收者标识符在方法签名中必须是唯一的。如果接收者的值在方法体内没有被使用，接受者的标识符是可以省略的。这同样适用于函数和方法的参数。

For a base type, the non-blank names of methods bound to it must be unique. If the base type is a struct type, the non-blank method and field names must be distinct.

对于一个基础类型，绑定到这个类型上的方法名必须是唯一的。如果一个基础类型时一个结构体，那么非空的方法名和字段名之间也必须是唯一的。

Given defined type Point, the declarations

给定一个定义好的类型 Point，声明为：

```go
func (p *Point) Length() float64 {
	return math.Sqrt(p.x * p.x + p.y * p.y)
}

func (p *Point) Scale(factor float64) {
	p.x *= factor
	p.y *= factor
}
```
bind the methods Length and Scale, with receiver type *Point, to the base type Point.

通过在 `*Point` 的接受者身上添加 `Length` 和 `Scale` 方法，给基础类型 Point 绑定了这两个方法。

The type of a method is the type of a function with the receiver as first argument. For instance, the method Scale has type

```go
func(p *Point, factor float64)
```

However, a function declared this way is not a method.

**方法**所对应的的类型对应的是一个带有接受者且为其第一个参数的**函数**的类型。比如上面的 `Scale` 的类型时 `func(p *Point, factor float64)`， 但是一个函数按照这种方式声明出来以后依然只是一个函数，而不是一个方法。

# Expressions（表达式）

An expression specifies the computation of a value by applying operators and functions to operands.

表达式通过将操作符和函数应用于操作数来指定值的计算。

## Operands（操作数）

Operands denote the elementary values in an expression. An operand may be a literal, a (possibly [qualified](https://golang.google.cn/ref/spec#Qualified_identifiers)) non-[blank](https://golang.google.cn/ref/spec#Blank_identifier) identifier denoting a [constant](https://golang.google.cn/ref/spec#Constant_declarations), [variable](https://golang.google.cn/ref/spec#Variable_declarations), or [function](https://golang.google.cn/ref/spec#Function_declarations), or a parenthesized expression.

The [blank identifier](https://golang.google.cn/ref/spec#Blank_identifier) may appear as an operand only on the left-hand side of an [assignment](https://golang.google.cn/ref/spec#Assignments).

```go
Operand     = Literal | OperandName | "(" Expression ")" .
Literal     = BasicLit | CompositeLit | FunctionLit .
BasicLit    = int_lit | float_lit | imaginary_lit | rune_lit | string_lit .
OperandName = identifier | QualifiedIdent.
```

操作数表示表达式中的基本值。一个操作数可以是一个文字，也可以是一个（可能是带包名的）非空的标识符用来表示常量、变量或者函数，也可以是一个被括号括起来的表达式。

当空白标识符作为操作数时，只能存在赋值运算符的左边。

## Qualified identifiers（全局有效标识符）

A qualified identifier is an identifier qualified with a package name prefix. Both the package name and the identifier must not be [blank](https://golang.google.cn/ref/spec#Blank_identifier).

一个全局有效的标识符是指带包名前缀的标识符。包名和标识符必须都不为空。

```go
QualifiedIdent = PackageName "." identifier .
```

A qualified identifier accesses an identifier in a different package, which must be [imported](https://golang.google.cn/ref/spec#Import_declarations). The identifier must be [exported](https://golang.google.cn/ref/spec#Exported_identifiers) and declared in the [package block](https://golang.google.cn/ref/spec#Blocks) of that package.

一旦被导入，一个全局有效的标识符可以在不同的包中引用某个标识符。因此，此类标识符必须是可导出的，且在包块范围内声明。

```go
math.Sin	// denotes the Sin function in package math
```



## Composite literals（复合类型字面量）

Composite literals construct values for structs, arrays, slices, and maps and create a new value each time they are evaluated. They consist of the type of the literal followed by a brace-bound list of elements. Each element may optionally be preceded by a corresponding key.

复合语法用来构建结构体、数组、切片和 map 的值，每次运行都创建一个新的值。语法主要由一个类型跟着一个大括号括起来的元素列表组成，这些元素可能有对应的键值，也可能没有键值。

```go
CompositeLit  = LiteralType LiteralValue .
LiteralType   = StructType | ArrayType | "[" "..." "]" ElementType |
                SliceType | MapType | TypeName .
LiteralValue  = "{" [ ElementList [ "," ] ] "}" .
ElementList   = KeyedElement { "," KeyedElement } .
KeyedElement  = [ Key ":" ] Element .
Key           = FieldName | Expression | LiteralValue .
FieldName     = identifier .
Element       = Expression | LiteralValue .
```

The LiteralType's underlying type must be a struct, array, slice, or map type (the grammar enforces this constraint except when the type is given as a TypeName). The types of the elements and keys must be [assignable](https://golang.google.cn/ref/spec#Assignability) to the respective field, element, and key types of the literal type; there is no additional conversion. The key is interpreted as a field name for struct literals, an index for array and slice literals, and a key for map literals. For map literals, all elements must have a key. It is an error to specify multiple elements with the same field name or constant key value. For non-constant map keys, see the section on [evaluation order](https://golang.google.cn/ref/spec#Order_of_evaluation).

文本类型底层的类型必须是结构体、数组、切片或 map（除非把类型作为 类型名 给出，否则语法上会强制这条约束）。元素和键值的类型必须可以分配给对应的字段、元素、以及文本类型的键类型；这里没有额外的转换。键值可以解释为结构体中的字段名、数组和切片中的索引、以及 map 中的键。对于 map 来说，所有的元素必须有一个键。如果同一个字段或者特定的一个键指定多个元素就会出错（看需求是不是这样）。如果想了解 map 的非常数键值相关的内容，可以看 《map 的执行顺序》这一小节的内容。

For struct literals the following rules apply:

- A key must be a field name declared in the struct type.
- An element list that does not contain any keys must list an element for each struct field in the order in which the fields are declared.
- If any element has a key, every element must have a key.
- An element list that contains keys does not need to have an element for each struct field. Omitted fields get the zero value for that field.
- A literal may omit the element list; such a literal evaluates to the zero value for its type.
- It is an error to specify an element for a non-exported field of a struct belonging to a different package.

对于一个结构体来说，有下面的规则：

- 键必须是在结构体中声明的字段名；
- 如果只给出元素列表而不给出具体对应的字段名，那么必须按照结构体定义时的顺序把每个字段的值都给出来（否则就要显示指出哪个字段对应哪个元素）；
- 只要有元素被指定了字段名，那么所有的元素都要有一个字段名；
- 如果元素列表中包含了字段名，就不需要把结构体中所有的字段名都列出来了，值需要指定有值的字段就可以，没有指定的字段的默认值是那个字段的零值；
- 如果声明结构体实例时省略了所有的元素，则声明的是这个类型的零值；
- 在另一个包中是不能给一个非导出的字段赋值的，否则会报错。

Given the declarations

对于下面的声明：

```go
type Point3D struct { x, y, z float64 }
type Line struct { p, q Point3D }
```

one may write

可以写成下面的样子：

```go
origin := Point3D{}                            // zero value for Point3D
line := Line{origin, Point3D{y: -4, z: 12.3}}  // zero value for line.q.x
```

For array and slice literals the following rules apply:

- Each element has an associated integer index marking its position in the array.
- An element with a key uses the key as its index. The key must be a non-negative constant [representable](https://golang.google.cn/ref/spec#Representability) by a value of type `int`; and if it is typed it must be of integer type.
- An element without a key uses the previous element's index plus one. If the first element has no key, its index is zero.

对于数组和切片来说，有下面的规则：

- 在数组中每个元素都有一个整数来索引其位置；
- 每个元素由其对应的键进行索引，键必须是一个非负整数；
- 可以通过索引 `+1` 的方式获取当前元素后面一个元素的索引（只要别超过数组或切片的长度），第一个元素的索引值是 0。

[Taking the address](https://golang.google.cn/ref/spec#Address_operators) of a composite literal generates a pointer to a unique [variable](https://golang.google.cn/ref/spec#Variables) initialized with the literal's value.

```go
var pointer *Point3D = &Point3D{y: 1000}
```

可以取复合文本对象的地址，此时会生成一个指向该文本对象值的指针，可以把这个指针赋值给一个变量。

Note that the [zero value](https://golang.google.cn/ref/spec#The_zero_value) for a slice or map type is not the same as an initialized but empty value of the same type. Consequently, taking the address of an empty slice or map composite literal does not have the same effect as allocating a new slice or map value with [new](https://golang.google.cn/ref/spec#Allocation).

特别说明一下，切片和 map 的零值与它们初始化但不包含任何元素的值是不一样的。因此，取一个空切片或空 map 的地址与通过 `new` 方法初始化一个切片和 map 的方式，二者的效果是不一样的。(试验发现当前好像已经一样了)

```go
p1 := &[]int{}    // p1 points to an initialized, empty slice with value []int{} and length 0
p2 := new([]int)  // p2 points to an uninitialized slice with value nil and length 0
```

The length of an array literal is the length specified in the literal type. If fewer elements than the length are provided in the literal, the missing elements are set to the zero value for the array element type. It is an error to provide elements with index values outside the index range of the array. The notation `...` specifies an array length equal to the maximum element index plus one.

数组的长度是声明数组时指定的长度。如果数组中的元素占不满整个数组，剩下的位置填充的是数组元素的零值。当然，不能在数组中保存超过其长度的元素，否则会指针溢出报错。在初始化数组时如果通过 `...` 来指定数组长度，则数组长度被指定为提供的元素的个数。

```go
buffer := [10]string{}             // len(buffer) == 10
intSet := [6]int{1, 2, 3, 5}       // len(intSet) == 6
days := [...]string{"Sat", "Sun"}  // len(days) == 2
```

A slice literal describes the entire underlying array literal. Thus the length and capacity of a slice literal are the maximum element index plus one. A slice literal has the form

切片描述整个底层数组的数据。因此切片的容量是底层数组能容纳的最多的元素个数。一个切片的语法是：

```go
[]T{x1, x2, … xn}
```

and is shorthand for a slice operation applied to an array:

上面的语法是下面语法的简写：

```go
tmp := [n]T{x1, x2, … xn}
tmp[0 : n]
```

Within a composite literal of array, slice, or map type `T`, elements or map keys that are themselves composite literals may elide the respective literal type if it is identical to the element or key type of `T`. Similarly, elements or keys that are addresses of composite literals may elide the `&T` when the element or key type is `*T`.

在数组、切片和 map 中，如果元素或者 map 的键本身也是复合类型，在初始化时则可以省略相应的类型（比如 `[...]Point{{1.5, -3.5}, {0, 0}}     // same as [...]Point{Point{1.5, -3.5}, Point{0, 0}}` ）。类似的，如果元素和键是复合文本的地址指针，在初始化时也可以省略类似 `&T` 的语法。

```go
[...]Point{{1.5, -3.5}, {0, 0}}     // same as [...]Point{Point{1.5, -3.5}, Point{0, 0}}
[][]int{{1, 2, 3}, {4, 5}}          // same as [][]int{[]int{1, 2, 3}, []int{4, 5}}
[][]Point{{{0, 1}, {1, 2}}}         // same as [][]Point{[]Point{Point{0, 1}, Point{1, 2}}}
map[string]Point{"orig": {0, 0}}    // same as map[string]Point{"orig": Point{0, 0}}
map[Point]string{{0, 0}: "orig"}    // same as map[Point]string{Point{0, 0}: "orig"}

type PPoint *Point
[2]*Point{{1.5, -3.5}, {}}          // same as [2]*Point{&Point{1.5, -3.5}, &Point{}}
[2]PPoint{{1.5, -3.5}, {}}          // same as [2]PPoint{PPoint(&Point{1.5, -3.5}), PPoint(&Point{})}
```

A parsing ambiguity arises when a composite literal using the TypeName form of the LiteralType appears as an operand between the [keyword](https://golang.google.cn/ref/spec#Keywords) and the opening brace of the block of an "if", "for", or "switch" statement, and the composite literal is not enclosed in parentheses, square brackets, or curly braces. In this rare case, the opening brace of the literal is erroneously parsed as the one introducing the block of statements. To resolve the ambiguity, the composite literal must appear within parentheses.

当使用 **类名称** 形式的复合文本作为关键字与“if”、“for”、“switch”的左括号之间的操作数时，如果不包含在小括号、中括号或者大括号中，则存在解析歧义。在这种罕见的情况下，复合文本中的大括号会被错误地解析为 “if”、“for”、“switch” 等的块。为了避免这种歧义，复合文本必须要用小括号包含起来，如下面两条语句：

```go
if x == (T{a,b,c}[i]) { … }
if (x == T{a,b,c}[i]) { … }
```

Examples of valid array, slice, and map literals:

下面展示了合法的数组、切片和 map 的声明语法：

```go
// list of prime numbers
primes := []int{2, 3, 5, 7, 9, 2147483647}

// vowels[ch] is true if ch is a vowel
vowels := [128]bool{'a': true, 'e': true, 'i': true, 'o': true, 'u': true, 'y': true}

// the array [10]float32{-1, 0, 0, 0, -0.1, -0.1, 0, 0, 0, -1}
filter := [10]float32{-1, 4: -0.1, -0.1, 9: -1}

// frequencies in Hz for equal-tempered scale (A4 = 440Hz)
noteFrequency := map[string]float32{
	"C0": 16.35, "D0": 18.35, "E0": 20.60, "F0": 21.83,
	"G0": 24.50, "A0": 27.50, "B0": 30.87,
}
```



## Function literals（函数字面量）

A function literal represents an anonymous [function](https://golang.google.cn/ref/spec#Function_declarations).

一个函数字面量表示一个匿名的函数。

```go
FunctionLit = "func" Signature FunctionBody .
func(a, b int, z float64) bool { return a*b < int(z) }
```

A function literal can be assigned to a variable or invoked directly.

函数字面量可以被赋值给一个变量，并可以通过这个字面量直接调用。

```go
f := func(x, y int) int { return x + y }
func(ch chan int) { ch <- ACK }(replyChan)
```

Function literals are *closures*: they may refer to variables defined in a surrounding function. Those variables are then shared between the surrounding function and the function literal, and they survive as long as they are accessible.

函数字面量也是闭包：他们可以引用定义在这个函数外面的变量。这些被闭包引用的变量在闭包与闭包外的函数之间共享，如果闭包外的函数执行完毕了，这些被闭包引用的变量会依然存在（这些变量会逃逸到堆上）。



## Primary expressions（主表达式）

Primary expressions are the operands for unary and binary expressions.

主表达式是一元和二元表达式的操作数。

```go
PrimaryExpr =
	Operand |
	Conversion |
	MethodExpr |
	PrimaryExpr Selector |
	PrimaryExpr Index |
	PrimaryExpr Slice |
	PrimaryExpr TypeAssertion |
	PrimaryExpr Arguments .

Selector       = "." identifier .
Index          = "[" Expression "]" .
Slice          = "[" [ Expression ] ":" [ Expression ] "]" |
                 "[" [ Expression ] ":" Expression ":" Expression "]" .
TypeAssertion  = "." "(" Type ")" .
Arguments      = "(" [ ( ExpressionList | Type [ "," ExpressionList ] ) [ "..." ] [ "," ] ] ")" .
```
```go
x
2
(s + ".txt")
f(3.1415, true)
Point{1, 2}
m["foo"]
s[i : j + 1]
obj.color
f.p[i].x()
```



## Selectors（选择表达式）

For a [primary expression](https://golang.google.cn/ref/spec#Primary_expressions) `x` that is not a [package name](https://golang.google.cn/ref/spec#Package_clause), the *selector expression*

```go
x.f
```

denotes the field or method `f` of the value `x` (or sometimes `*x`; see below). The identifier `f` is called the (field or method) *selector*; it must not be the [blank identifier](https://golang.google.cn/ref/spec#Blank_identifier). The type of the selector expression is the type of `f`. If `x` is a package name, see the section on [qualified identifiers](https://golang.google.cn/ref/spec#Qualified_identifiers).

对主表达式中的 x，如果它不是一个包名，那么选择表达式 `x.f` 表示的是 x（或 *x） 的字段 `f` 或者方法 `f`。标识符 `f` 被称为（字段或方法）选择器；当然，它不能是一个空标识符。选择表达式的类型就是 `f` 的类型。如果`x` 是一个包，我们称这种用法是有效标识符而不是选择表达式，详细可以参见《全局有效标识符》一节。

A selector `f` may denote a field or method `f` of a type `T`, or it may refer to a field or method `f` of a nested [embedded field](https://golang.google.cn/ref/spec#Struct_types) of `T`. The number of embedded fields traversed to reach `f` is called its *depth* in `T`. The depth of a field or method `f` declared in `T` is zero. The depth of a field or method `f` declared in an embedded field `A` in `T` is the depth of `f` in `A` plus one.

选择器 `f` 可以表示类型 T 的一个字段或者一个方法，也可以表示类型 T 中某个嵌套的结构的字段或方法。如果 `f` 是嵌套字段上的属性或者方法，嵌套的深度称为 `f` 的深度。如果一个字段或者方法 `f` 是在类型 `T` 上直接声明的，则认为 `f` 的深度为 0；如果 `f` 是在类型 T 的嵌套字段 A 上定义的，则 `f` 的深度就是 `f` 在 `A` 上的深度加 1。

The following rules apply to selectors:

1. For a value `x` of type `T` or `*T` where `T` is not a pointer or interface type, `x.f` denotes the field or method at the shallowest depth in `T` where there is such an `f`. If there is not exactly [one `f`](https://golang.google.cn/ref/spec#Uniqueness_of_identifiers) with shallowest depth, the selector expression is illegal.
2. For a value `x` of type `I` where `I` is an interface type, `x.f` denotes the actual method with name `f` of the dynamic value of `x`. If there is no method with name `f` in the [method set](https://golang.google.cn/ref/spec#Method_sets) of `I`, the selector expression is illegal.
3. As an exception, if the type of `x` is a [defined](https://golang.google.cn/ref/spec#Type_definitions) pointer type and `(*x).f` is a valid selector expression denoting a field (but not a method), `x.f` is shorthand for `(*x).f`.
4. In all other cases, `x.f` is illegal.
5. If `x` is of pointer type and has the value `nil` and `x.f` denotes a struct field, assigning to or evaluating `x.f` causes a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics).
6. If `x` is of interface type and has the value `nil`, [calling](https://golang.google.cn/ref/spec#Calls) or [evaluating](https://golang.google.cn/ref/spec#Method_values) the method `x.f` causes a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics).

选择器表达式有下面的规则：

1. 如果类型 T 不是指针类型（对比第3条）或接口类型，对于类型为 `T` 或 `*T` 的值 x，`x.f` 表示类型 T 中深度最浅的字段或方法。如果不存在一个最浅深度的 `f` ，此时选择表达式是不合法的。
2. 如果类型 I 是一个接口类型，对于它的一个值 x，`x.f` 表示动态值 `x` 在其底层类型上实现的一个方法 `f`，如果在接口定义的方法集合中不包含 `f`，此时选择表达式是不合法的。
3. 如果 x 的类型是自定义类型的指针，同时 `(*x).f`  是一个合法的选择表达式且表示的是一个字段（而不是一个方法），此时 `x.f`是 `(*x).f`  的简写形式。
4. 其他情况下 `x.f` 都是不合法的。
5. 如果 `x` 是一个指针类型且值为 nil，而`x.f` 表示结构中的字段，给 `x.f` 赋值或执行它都会报运行时错误。
6. 如果 `x` 是一个接口且值为 nil，调用或执行 `x.f` 都会导致运行时错误。

For example, given the declarations:

```go
type T0 struct {
	x int
}

func (*T0) M0()

type T1 struct {
	y int
}

func (T1) M1()

type T2 struct {
	z int
	T1
	*T0
}

func (*T2) M2()

type Q *T2

var t T2     // with t.T0 != nil
var p *T2    // with p != nil and (*p).T0 != nil
var q Q = p
```

one may write:

```go
t.z          // t.z
t.y          // t.T1.y
t.x          // (*t.T0).x

p.z          // (*p).z
p.y          // (*p).T1.y
p.x          // (*(*p).T0).x

q.x          // (*(*q).T0).x        (*q).x is a valid field selector

p.M0()       // ((*p).T0).M0()      M0 expects *T0 receiver
p.M1()       // ((*p).T1).M1()      M1 expects T1 receiver
p.M2()       // p.M2()              M2 expects *T2 receiver
t.M2()       // (&t).M2()           M2 expects *T2 receiver, see section on Calls
```

but the following is invalid:

下面的语句是不合法的（触犯了第 3 条规则）：

```go
q.M0()       // (*q).M0 is valid but not a field selector
```





## Method expressions（方法表达式）

If `M` is in the [method set](https://golang.google.cn/ref/spec#Method_sets) of type `T`, `T.M` is a function that is callable as a regular function with the same arguments as `M` prefixed by an additional argument that is the receiver of the method.

如果 `M` 是类型 `T` 的方法集中的一个方法，`T.M` 可以像普通的函数那样调用，而且这个普通的函数与 `T.M` 有类似的参数和返回值列表，只不过后者多了一个接受者作为参数。

```go
MethodExpr    = ReceiverType "." MethodName .
ReceiverType  = Type .
```

Consider a struct type `T` with two methods, `Mv`, whose receiver is of type `T`, and `Mp`, whose receiver is of type `*T`.

假设有一个类型 `T` 有两个方法，其中 `Mv` 的接受者是类型 `T`，而 `Mp` 的接受者是类型 `*T`：

```go
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
```

The expression

```go
T.Mv
```

yields a function equivalent to `Mv` but with an explicit receiver as its first argument; it has signature

```go
func(tv T, a int) int
```

表达式 `T.Mv` 产生一个等效的函数 `func(tv T, a int) int`, 这个函数把接受者显式声明为自己的第一个参数。

That function may be called normally with an explicit receiver, so these five invocations are equivalent:

当显式指定了接受者，函数可以正常调用，因此下面五种形式的调用是等效的。

```go
t.Mv(7)
T.Mv(t, 7)
(T).Mv(t, 7)
f1 := T.Mv; f1(t, 7)
f2 := (T).Mv; f2(t, 7)
```

Similarly, the expression

```go
(*T).Mp
```

yields a function value representing `Mp` with signature

```go
func(tp *T, f float32) float32
```

同样的，表达式 `(*T).Mp` 生成一个等效的函数 `func(tp *T, f float32) float32`。

For a method with a value receiver, one can derive a function with an explicit pointer receiver, so

```go
(*T).Mv
```

yields a function value representing `Mv` with signature

```go
func(tv *T, a int) int
```

对于一个有**接受者值**的方法，我们可以导出一个显式地带有**指针接受器**的函数，因此 `(*T).Mv` 生成了一个函数值表示 `Mv` 方法：`func(tv *T, a int) int`。

Such a function indirects through the receiver to create a value to pass as the receiver to the underlying method; the method does not overwrite the value whose address is passed in the function call.

这样的函数通过接收器（指针）向底层方法传递了一个值（根据指针创建一个新的值）；基于上面的理由，这个方法不会覆盖修改被传入了地址的值。

The final case, a value-receiver function for a pointer-receiver method, is illegal because pointer-receiver methods are not in the method set of the value type.

最后的例子，**值作为接收器的函数替代指针作为接收器的方法，这种形式上的做法就不合法了，因为指针接受者的方法不包含在值类型的方法集合中**。

Function values derived from methods are called with function call syntax; the receiver is provided as the first argument to the call. That is, given `f := T.Mv`, `f` is invoked as `f(t, 7)` not `t.f(7)`. To construct a function that binds the receiver, use a [function literal](https://golang.google.cn/ref/spec#Function_literals) or [method value](https://golang.google.cn/ref/spec#Method_values).

从方法中导出的函数值是可以通过函数调用的语法进行调用的；此时接受者会作为调用的第一个参数传给函数。比如，假设 `f := T.Mv`，`f` 可以通过 `f(t,7)` 进行调用，但是不能通过 `t.f(7)` 进行调用。如果想构建一个绑定了接受者的函数，参考函数定义语法和方法值这两个小节。

It is legal to derive a function value from a method of an interface type. The resulting function takes an explicit receiver of that interface type.

从一个接口类型中是可以到处函数值的。这时导出的函数值需要显式传入那个接口类型的接受者。



## Method values（方法值）

If the expression `x` has static type `T` and `M` is in the [method set](https://golang.google.cn/ref/spec#Method_sets) of type `T`, `x.M` is called a *method value*. The method value `x.M` is a function value that is callable with the same arguments as a method call of `x.M`. The expression `x` is evaluated and saved during the evaluation of the method value; the saved copy is then used as the receiver in any calls, which may be executed later.

如果表达式 `x` 的静态类型是 `T`，同时 `M` 在类型 `T` 的方法结合中，那么 `X.M` 就被称为方法值。方法值 `x.M` 也是一个函数值，可以通过传入调用 `x.M` 时所传入的相同的参数进行调用。表达式 `x` 在计算方法值的时候会被计算并保存，因此在调用 `x.M` 的时候这个接受器会默认被传入而不需要再传入了。

The type `T` may be an interface or non-interface type.

类型 `T` 可以是接口也可以不是接口。

As in the discussion of [method expressions](https://golang.google.cn/ref/spec#Method_expressions) above, consider a struct type `T` with two methods, `Mv`, whose receiver is of type `T`, and `Mp`, whose receiver is of type `*T`.

向《方法表达式》一节所介绍的，如果一个结构类型 `T` 有两个方法，其中 `Mv` 的接收器是类型 `T`，`Mp`的接收器是类型 `*T`。

```go
type T struct {
	a int
}
func (tv  T) Mv(a int) int         { return 0 }  // value receiver
func (tp *T) Mp(f float32) float32 { return 1 }  // pointer receiver

var t T
var pt *T
func makeT() T
```

The expression

```go
t.Mv
```

yields a function value of type

```go
func(int) int
```

These two invocations are equivalent:

```go
t.Mv(7)
f := t.Mv; f(7)
```

表达式 `t.MV` 生成了函数值的类型为 `func(int) int`。此时这两种方式的调用是等效的：`t.Mv(7)` 和`f := t.MV; f(7)`。

Similarly, the expression

```go
pt.Mp
```

yields a function value of type

```go
func(float32) float32
```

同样的，表达式 `pt.Mp` 生成了函数值的类型为 `func(float32) float32`。

As with [selectors](https://golang.google.cn/ref/spec#Selectors), a reference to a non-interface method with a value receiver using a pointer will automatically dereference that pointer: `pt.Mv` is equivalent to `(*pt).Mv`.

像选择器一节讲的，如果一个非接口方法的引用，其引用的方法有一个值接收器，使用指针调用此方法时指针会被自动解为对应类型的值：`pt.Mv` 和 `(*pt).Mv` 是等效的。

As with [method calls](https://golang.google.cn/ref/spec#Calls), a reference to a non-interface method with a pointer receiver using an addressable value will automatically take the address of that value: `t.Mp` is equivalent to `(&t).Mp`.

在方法调用方面，如果一个非接口方法的引用，其引用的方法有一个指针类型的接收器，使用可取地址的值调用这个方法时，会自动取这个值的地址进行使用：`t.Mp` 和 `(&t).Mp` 是等效的。

**这里应该注意 `f := makeT().Mp` 不合法的原因**。

```go
f := t.Mv; f(7)   // like t.Mv(7)
f := pt.Mp; f(7)  // like pt.Mp(7)
f := pt.Mv; f(7)  // like (*pt).Mv(7)
f := t.Mp; f(7)   // like (&t).Mp(7)
f := makeT().Mp   // invalid: result of makeT() is not addressable
```

Although the examples above use non-interface types, it is also legal to create a method value from a value of interface type.

```go
var i interface { M(int) } = myVal
f := i.M; f(7)  // like i.M(7)
```

虽然上面的例子都是用的非接口类型，不过对于接口类型中的方法也能从接口类型的变量上创建方法值。

## Index expressions（索引表达式）

A primary expression of the form

```go
a[x]
```

denotes the element of the array, pointer to array, slice, string or map `a` indexed by `x`. The value `x` is called the *index* or *map key*, respectively. The following rules apply:

主要的索引表达式类似于 `a[x]`，表示某个实体 `a` 中索引为 `x` 的元素，包括数组、数组的指针、切片、字符串和 map 等。其中 `x` 称为 索引 或 map 的键。

If `a` is not a map:

- the index `x` must be of integer type or an untyped constant
- a constant index must be non-negative and [representable](https://golang.google.cn/ref/spec#Representability) by a value of type `int`
- a constant index that is untyped is given type `int`
- the index `x` is *in range* if `0 <= x < len(a)`, otherwise it is *out of range*

如果 `a` 不是一个 map：

- 索引 `x` 必须是一个整数或者一个没有指定类型的常数；
- 如果是常数索引的话，必须是一个非负且可以用 int 类型的数代表的常数；
- 如果传入的是一个未指定类型的常量索引的话，其默认类型是 `int`；
- 索引 `x` 的范围是 `0 <= x < len(a)`，否则会越界。

For `a` of [array type](https://golang.google.cn/ref/spec#Array_types) `A`:

- a [constant](https://golang.google.cn/ref/spec#Constants) index must be in range
- if `x` is out of range at run time, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs
- `a[x]` is the array element at index `x` and the type of `a[x]` is the element type of `A`

如果 `a` 是一个数组类型 `A` 的实例：

- 常量索引必须在`[0,len(a)-1]`范围内；
- 如果 `x` 越界了，会发生运行时错误；
- `a[x]`表示在索引 `x` 处的元素值，且 `a[x]` 的类型是 `A` 数组的元素类型。

For `a` of [pointer](https://golang.google.cn/ref/spec#Pointer_types) to array type:

- `a[x]` is shorthand for `(*a)[x]`

如果 `a` 是一个数组类型的指针：

- `a[x]` 是 `(*a)[x]` 的缩写。

For `a` of [slice type](https://golang.google.cn/ref/spec#Slice_types) `S`:

- if `x` is out of range at run time, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs
- `a[x]` is the slice element at index `x` and the type of `a[x]` is the element type of `S`

如果 `a` 是一个切片 `S` 的实例：

- 如果 `x`在运行时超过`[0,len(a)-1]`，会发生运行时错误；
- `a[x]`表示在索引 `x` 处的元素值，且 `a[x]` 的类型是 `S` 切片的元素类型。

For `a` of [string type](https://golang.google.cn/ref/spec#String_types):

- a [constant](https://golang.google.cn/ref/spec#Constants) index must be in range if the string `a` is also constant
- if `x` is out of range at run time, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs
- `a[x]` is the non-constant byte value at index `x` and the type of `a[x]` is `byte`
- `a[x]` may not be assigned to

如果 `a` 是一个字符串：

- 如果字符 `a` 也是一个常量，那么 a 的常量索引必须在 `[0,len(a)-1]`范围内。
- 如果 `x`在运行时超过`[0,len(a)-1]`，会发生运行时错误；
- `a[x]` 是一个在索引 `x` 处类型为 byte 的非常量，且其类型是就是 `byte`；
- `a[x]` 可能无法被赋值。

For `a` of [map type](https://golang.google.cn/ref/spec#Map_types) `M`:

- `x`'s type must be [assignable](https://golang.google.cn/ref/spec#Assignability) to the key type of `M`
- if the map contains an entry with key `x`, `a[x]` is the map element with key `x` and the type of `a[x]` is the element type of `M`
- if the map is `nil` or does not contain such an entry, `a[x]` is the [zero value](https://golang.google.cn/ref/spec#The_zero_value) for the element type of `M`

如果 `a` 是 map 类型 `M` 的实例：

- `x` 的类型必须能赋值给 `M` 的键值；
- 如果 map 包含一个键 `x`，则 `a[x]` 是 map 中键为 `x` 的元素，且 `a[x]` 的类型是 `M` 的元素类型。
- 如果 map 的值为 nil，或者不存在一个这样的键，那么 `a[x]` 是 `M` 的元素的零值。（**如果 map 的值为 nil，可以从值中读数据，但是如果写数据会 panic**）

Otherwise `a[x]` is illegal.

其他情况下，`a[x]` 是不合法的。

An index expression on a map `a` of type `map[K]V` used in an [assignment](https://golang.google.cn/ref/spec#Assignments) or initialization of the special form

```go
v, ok = a[x]
v, ok := a[x]
var v, ok = a[x]
```

yields an additional untyped boolean value. The value of `ok` is `true` if the key `x` is present in the map, and `false` otherwise.

Assigning to an element of a `nil` map causes a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics).

对于 map 来说，如果 `a` 是类型 `map[K]V` 的一个实例值，在赋值语句中可以通过指定第二个接收值来生成一个附加的布尔值。如果这个值是 true 说明索引 `k` 是存在于 map 的，否则就说明不存在。

不能向值为 nil 的 map 中赋值，会 panic。



## Slice expressions（切片表达式）

Slice expressions construct a substring or slice from a string, array, pointer to array, or slice. There are two variants: a simple form that specifies a low and high bound, and a full form that also specifies a bound on the capacity.

切片表达式能够从字符串、数组、数组指针、切片等上重新构建一个新的子字符串或切片。有两个变形：单一形式，只地你故意低边界和高边界；全部形式，定义边界同时定义容量。

### Simple slice expressions（简单切片表达式）

For a string, array, pointer to array, or slice `a`, the primary expression

```go
a[low : high]
```

constructs a substring or slice. The *indices* `low` and `high` select which elements of operand `a` appear in the result. The result has indices starting at 0 and length equal to `high` - `low`. After slicing the array `a`

对于字符串、数组、数组指针或切片 `a`, 主表达式 `a[low:high]` 能够构建一个子字符串或新切片。低索引和高索引选择了操作数中的那些元素会出现在结果中。生成的结果的索引从 0 开始，长度等于 `high - low`。比如下面的数组 `a`：

```go
a := [5]int{1, 2, 3, 4, 5}
s := a[1:4]
```

the slice `s` has type `[]int`, length 3, capacity 4, and elements

生成的切片 `s` 的类型是 `[]int`，长度是 3， 容量是 4，且每个元素的情况如下：

```go
s[0] == 2
s[1] == 3
s[2] == 4
```

For convenience, any of the indices may be omitted. A missing `low` index defaults to zero; a missing `high` index defaults to the length of the sliced operand:

```go
a[2:]  // same as a[2 : len(a)]
a[:3]  // same as a[0 : 3]
a[:]   // same as a[0 : len(a)]
```

If `a` is a pointer to an array, `a[low : high]` is shorthand for `(*a)[low : high]`.

为了便利性，任何一个索引都可能被忽略。如果索引 `low` 被忽略了，则默认为 0；如果 `high` 被忽略了，则默认为操作数的长度。

For arrays or strings, the indices are *in range* if `0` <= `low` <= `high` <= `len(a)`, otherwise they are *out of range*. For slices, the upper index bound is the slice capacity `cap(a)` rather than the length. A [constant](https://golang.google.cn/ref/spec#Constants) index must be non-negative and [representable](https://golang.google.cn/ref/spec#Representability) by a value of type `int`; for arrays or constant strings, constant indices must also be in range. If both indices are constant, they must satisfy `low <= high`. If the indices are out of range at run time, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs.

对数组和字符串来说，如果 `0 <= low <= high <= len(a)` 则说明索引在正常范围内，否则就是超出正常范围。对于切片来说，最大的索引边界是切片的容量 `cap(a)` 而不是其长度。一个常量的索引必须是非负数并且可以表示为 `int` 类型的数值；对于数组和常量字符串来说，常数索引也必须是在正常范围内才可以。如果两个索引都是常量，必须满足 `low <= high`。如果索引在运行时超出了正常范围，会报 panic 错误。

Except for [untyped strings](https://golang.google.cn/ref/spec#Constants), if the sliced operand is a string or slice, the result of the slice operation is a non-constant value of the same type as the operand. For untyped string operands the result is a non-constant value of type `string`. If the sliced operand is an array, it must be [addressable](https://golang.google.cn/ref/spec#Address_operators) and the result of the slice operation is a slice with the same element type as the array.

除了未定义类型的字符串，如果切片的操作数是一个字符串或者切片，切片运算的结果是一个非常量的值，且其值的类型和操作数的类型一致。对于未定义类型的字符串来说，结果则仍然是一个字符串（`string`）。如果切片操作数是一个数组，它必须是可寻址的；切片的结果是一个新切片，且它的元素的类型与数组元素类型一样。

If the sliced operand of a valid slice expression is a `nil` slice, the result is a `nil` slice. Otherwise, if the result is a slice, it shares its underlying array with the operand.

如果一个合法的切片操作的操作数是一个值为 nil 的切片，结果也是一个 nil 的切片。如果结果依然是一个切片，新切片会和操作数共享底层存储；换句话说，修改新的切片中的值，老切片是能感知到的。

```go
var a [10]int
s1 := a[3:7]   // underlying array of s1 is array a; &s1[2] == &a[5]
s2 := s1[1:4]  // underlying array of s2 is underlying array of s1 which is array a; &s2[1] == &a[5]
s2[1] = 42     // s2[1] == s1[2] == a[5] == 42; they all refer to the same underlying array element
```

### Full slice expressions（全切片表达式）

For an array, pointer to array, or slice `a` (but not a string), the primary expression

```go
a[low : high : max]
```

constructs a slice of the same type, and with the same length and elements as the simple slice expression `a[low : high]`. Additionally, it controls the resulting slice's capacity by setting it to `max - low`. Only the first index may be omitted; it defaults to 0. After slicing the array `a`

对于数组、数组指针或切片 `a`（不能使字符串）来说，主表达式 `a[low:hight:max]` 构建了有相同类型的切片，新切片的长度以及元素的情况和简单切片表达式 `a[low:high]` 一样，除此之外，它控制了新切片的容量，其值为 `max-low`。这种形式下，只有第一个索引可以被省略，默认为 0。对一个数组 `a` 进行切片后，

```go
a := [5]int{1, 2, 3, 4, 5}
t := a[1:3:5]
```

the slice `t` has type `[]int`, length 2, capacity 4, and elements

新生成的切片 `t` 的类型是 `[]int`，长度是 2，容量是 4，每个元素的情况如下：

```go
t[0] == 2
t[1] == 3
```

As for simple slice expressions, if `a` is a pointer to an array, `a[low : high : max]` is shorthand for `(*a)[low : high : max]`. If the sliced operand is an array, it must be [addressable](https://golang.google.cn/ref/spec#Address_operators).

和简单切片表达式一样，如果 `a` 是一个数组的指针，`a[low:high:max]` 是 `(*a)[low:high:max]` 的简写。如果切片的操作数时一个数组，它必须是可寻址的。

The indices are *in range* if `0 <= low <= high <= max <= cap(a)`, otherwise they are *out of range*. A [constant](https://golang.google.cn/ref/spec#Constants) index must be non-negative and [representable](https://golang.google.cn/ref/spec#Representability) by a value of type `int`; for arrays, constant indices must also be in range. If multiple indices are constant, the constants that are present must be in range relative to each other. If the indices are out of range at run time, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs.

如果索引满足 `0<=low<=high<=max<=cap(a)`，则说明索引在正常范围内，否则就越界。一个常量的索引必须是非负数并且可以表示为 `int` 类型的数值；对于数组来说，常数索引也必须是在正常范围内才可以。如果多个索引都是常量，常量之间必须也是自洽的。如果索引在运行时越界，会产生运行时 panic 错误。

## Type assertions（类型断言）

For an expression `x` of [interface type](https://golang.google.cn/ref/spec#Interface_types) and a type `T`, the primary expression

```go
x.(T)
```

asserts that `x` is not `nil` and that the value stored in `x` is of type `T`. The notation `x.(T)` is called a *type assertion*.

对于接口类型的表达式 `x` 以及类型 `T`，主表达式 `x.(T)` 断言 `x` 的值不是 `nil`，且存储在 `x` 中的值的类型是 `T`。 符号 `x.(T)` 被称为**类型断言**。

More precisely, if `T` is not an interface type, `x.(T)` asserts that the dynamic type of `x` is [identical](https://golang.google.cn/ref/spec#Type_identity) to the type `T`. In this case, `T` must [implement](https://golang.google.cn/ref/spec#Method_sets) the (interface) type of `x`; otherwise the type assertion is invalid since it is not possible for `x` to store a value of type `T`. If `T` is an interface type, `x.(T)` asserts that the dynamic type of `x` implements the interface `T`.

更精确地说，如果 `T` 不是一个接口类型，`x.(T)` 断言 `x` 的类型是`T`。这时，`T` 必须实现了 x 的接口类型；否则，因为 `x` 不可能保存一个类型为 `T` 的值，因此类型断言就是无效的。如果 `T` 是一个接口类型，`x.(T)` 断言 `x` 的动态类型实现了接口 `T`。

If the type assertion holds, the value of the expression is the value stored in `x` and its type is `T`. If the type assertion is false, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs. In other words, even though the dynamic type of `x` is known only at run time, the type of `x.(T)` is known to be `T` in a correct program.

如果类型断言成功了，断言表达式的结果是保存在 `x` 中的值，且这个值的类型是 `T`。如果类型断言失败了，则会发生一个运行时的 panic 错误。换句话说，虽然 `x` 的动态类型只有在运行时才能知道，但是 `x.(T)` 应该别确认是合法有效的。

```go
var x interface{} = 7          // x has dynamic type int and value 7
i := x.(int)                   // i has type int and value 7

type I interface { m() }

func f(y I) {
	s := y.(string)        // illegal: string does not implement I (missing method m)
	r := y.(io.Reader)     // r has type io.Reader and the dynamic type of y must implement both I and io.Reader
	…
}
```

A type assertion used in an [assignment](https://golang.google.cn/ref/spec#Assignments) or initialization of the special form

在赋值语句中，类型断言的表达式可以如下所示：

```go
v, ok = x.(T)
v, ok := x.(T)
var v, ok = x.(T)
var v, ok T1 = x.(T)
```

yields an additional untyped boolean value. The value of `ok` is `true` if the assertion holds. Otherwise it is `false` and the value of `v` is the [zero value](https://golang.google.cn/ref/spec#The_zero_value) for type `T`. No [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs in this case.

上面的表达式生成了未指定类型的布尔值。如果断言成功，那么 `ok` 的值就是 `true`；如果断言失败，`ok` 的值就是`false`，且 `v` 的值是类型 `T` 的零值。此时不会发生运行时错误。



## Calls（调用）

Given an expression `f` of function type `F`,

```go
f(a1, a2, … an)
```

calls `f` with arguments `a1, a2, … an`. Except for one special case, arguments must be single-valued expressions [assignable](https://golang.google.cn/ref/spec#Assignability) to the parameter types of `F` and are evaluated before the function is called. The type of the expression is the result type of `F`. A method invocation is similar but the method itself is specified as a selector upon a value of the receiver type for the method.

给定函数类型为 `F` 的表达式值 `f`，`f(a1,a2, ... an)` 调用 `f`，且传入了参数 `a1,a2, ... an`。除了一种特殊的情况，**一般情况下**参数必须是单值的表达式且可以赋值给 `F` 中指定的参数类型，这些表达式在函数调用前会运算得到结果。函数调用表达式的结果是 `F` 返回结果的类型。方法的调用相似，只是方法本身有一个接收者，且在这个接收者进行调用。

```go
math.Atan2(x, y)  // function call
var pt *Point
pt.Scale(3.5)     // method call with receiver pt
```

In a function call, the function value and arguments are evaluated in [the usual order](https://golang.google.cn/ref/spec#Order_of_evaluation). After they are evaluated, the parameters of the call are passed by value to the function and the called function begins execution. The return parameters of the function are passed by value back to the calling function when the function returns.

在一个函数调用里，函数值和参数值按照常规的顺序进行计算。一旦运算结束，函数调用的参数会以传值的方式传给函数，然后函数开始运行。同样的，函数也是以传值的方式返回参数给调用方。

Calling a `nil` function value causes a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics).

在值为 nil 的函数值上进行函数调用会导致运行时错误。

As a special case, if the return values of a function or method `g` are equal in number and individually assignable to the parameters of another function or method `f`, then the call `f(g(*parameters_of_g*))` will invoke `f` after binding the return values of `g` to the parameters of `f` in order. The call of `f` must contain no parameters other than the call of `g`, and `g` must have at least one return value. If `f` has a final `...` parameter, it is assigned the return values of `g` that remain after assignment of regular parameters.

**作为一种特例**（TODO 确认这里具体什么含义），如果一个函数或方法 `g` 的返回值在数量和类型上与另一个函数或方法 `f` 完全一致，那么函数调用 `f(g(*parameters_of_g*))` 时，在完成 `g` 的返回值到 `f` 的参数的绑定后立马调用 `f`。这种情况下，`f` 的调用只依赖 `g`，不包含其他的参数，并且 `g` 必须至少有一个返回值。如果 `f` 有一个 `...` 不定项参数，这个参数会被赋值 `g` 返回值列表中未经匹配的所有的值。 

```go
func Split(s string, pos int) (string, string) {
	return s[0:pos], s[pos:]
}

func Join(s, t string) string {
	return s + t
}

if Join(Split(value, len(value)/2)) != value {
	log.Panic("test fails")
}
```

A method call `x.m()` is valid if the [method set](https://golang.google.cn/ref/spec#Method_sets) of (the type of) `x` contains `m` and the argument list can be assigned to the parameter list of `m`. If `x` is [addressable](https://golang.google.cn/ref/spec#Address_operators) and `&x`'s method set contains `m`, `x.m()` is shorthand for `(&x).m()`:

```go
var p Point
p.Scale(3.5)
```

There is no distinct method type and there are no method literals.

如果 `x` 的类型的方法集中包含 `m` ，方法调用 `x.m()` 是合法的，且可以给 `m` 赋值参数。如果 `x` 可以取值并且 `&x` 的方法集中包含 `m`，`x.m()` 是 `(&x).m()` 的缩写。

没有明确的方法类型，也没有方法文字。（？）

## Passing arguments to `...` parameters（传参给不定参数）

If `f` is [variadic](https://golang.google.cn/ref/spec#Function_types) with a final parameter `p` of type `...T`, then within `f` the type of `p` is equivalent to type `[]T`. If `f` is invoked with no actual arguments for `p`, the value passed to `p` is `nil`. Otherwise, the value passed is a new slice of type `[]T` with a new underlying array whose successive elements are the actual arguments, which all must be [assignable](https://golang.google.cn/ref/spec#Assignability) to `T`. The length and capacity of the slice is therefore the number of arguments bound to `p` and may differ for each call site.

如果 `f` 是不定参函数，包含一个类型为 `...T` 的参数 `p`，那么在 `f` 内部 `p` 的类型是 `[]T`。如果 `f` 被调用时没有参数传给 `p`，此时 `p` 的值是 `nil`。否则，传递给 `f` 的是一个新切片，切片的类型是 `[]T`，底层是一个新的数组，里面包含了传给函数与方法的参数。当然，这些参数必须都可以赋值给 `T` 的变量。传递给 `f` 的新切片的长度和容量与绑定到 `p` 上的参数的个数有关，且每次调用时根据传入参数的不同发生变化。

Given the function and calls

```go
func Greeting(prefix string, who ...string)
Greeting("nobody")
Greeting("hello:", "Joe", "Anna", "Eileen")
```

within `Greeting`, `who` will have the value `nil` in the first call, and `[]string{"Joe", "Anna", "Eileen"}` in the second.

对于上面的方法及其调用，`Greeting("nobody")`的调用，在 `Greeting` 内部， `who` 的值是 nil；`Greeting("hello:", "Joe", "Anna", "Eileen")` 的调用 `who` 的值是 `[]string{"Joe", "Anna", "Eileen"}`。

If the final argument is assignable to a slice type `[]T`, it is passed unchanged as the value for a `...T` parameter if the argument is followed by `...`. In this case no new slice is created.

如果最后的一个参数可以赋值给切片类型 `[]T`，如果传参的时候跟着 `...` 操作符，那么就可以直接作为 `...T` 参数传入函数。这种情况下不会产生新的切片。

Given the slice `s` and call

```go
s := []string{"James", "Jasmine"}
Greeting("goodbye:", s...)
```

within `Greeting`, `who` will have the same value as `s` with the same underlying array.

比如有切片 `s` 和上面的调用，在 `Greeting` 内部，`who` 与 `s` 有相同的值，且二者共享同一个底层数组。

## Operators（操作符）

Operators combine operands into expressions.

操作符把操作数组合成为表达式。

```go
Expression = UnaryExpr | Expression binary_op Expression .
UnaryExpr  = PrimaryExpr | unary_op UnaryExpr .

binary_op  = "||" | "&&" | rel_op | add_op | mul_op .
rel_op     = "==" | "!=" | "<" | "<=" | ">" | ">=" .
add_op     = "+" | "-" | "|" | "^" .
mul_op     = "*" | "/" | "%" | "<<" | ">>" | "&" | "&^" .

unary_op   = "+" | "-" | "!" | "^" | "*" | "&" | "<-" .
```

Comparisons are discussed [elsewhere](https://golang.google.cn/ref/spec#Comparison_operators). For other binary operators, the operand types must be [identical](https://golang.google.cn/ref/spec#Type_identity) unless the operation involves shifts or untyped [constants](https://golang.google.cn/ref/spec#Constants). For operations involving constants only, see the section on [constant expressions](https://golang.google.cn/ref/spec#Constant_expressions).

**比较**将在其他小结进行专项讨论。对于二进制操作符，它们的操作数的类型必须是一致的，除非包含左移右移或未定义类型的常数。如果操作数只涉及到常数，可以查看《常数表达式》的内容。

Except for shift operations, if one operand is an untyped [constant](https://golang.google.cn/ref/spec#Constants) and the other operand is not, the constant is implicitly [converted](https://golang.google.cn/ref/spec#Conversions) to the type of the other operand.

除非是移位操作，否则在一个操作数时未定义类型的常量而另一个操作数不是的情况下，常量会隐式地转换成为另一个操作数的类型。

The right operand in a shift expression must have integer type or be an untyped constant [representable](https://golang.google.cn/ref/spec#Representability) by a value of type `uint`. If the left operand of a non-constant shift expression is an untyped constant, it is first implicitly converted to the type it would assume if the shift expression were replaced by its left operand alone.

移位表达式中的右操作数必须是一个整形的数值，或者是一个未定义类型但是可以用 `uint` 类型的值表示的常量。如果非常量移位表达式的左操作数是一个未声明类型的常量，它会先被隐式地转换为一个可能的类型（假设此时求的不是一个移位表达式，而是只包含当前的常量）。

```go
var s uint = 33
var i = 1<<s                  // 1 has type int
var j int32 = 1<<s            // 1 has type int32; j == 0
var k = uint64(1<<s)          // 1 has type uint64; k == 1<<33
var m int = 1.0<<s            // 1.0 has type int; m == 0 if ints are 32bits in size
var n = 1.0<<s == j           // 1.0 has type int32; n == true
var o = 1<<s == 2<<s          // 1 and 2 have type int; o == true if ints are 32bits in size
var p = 1<<s == 1<<33         // illegal if ints are 32bits in size: 1 has type int, but 1<<33 overflows int
var u = 1.0<<s                // illegal: 1.0 has type float64, cannot shift
var u1 = 1.0<<s != 0          // illegal: 1.0 has type float64, cannot shift
var u2 = 1<<s != 1.0          // illegal: 1 has type float64, cannot shift
var v float32 = 1<<s          // illegal: 1 has type float32, cannot shift
var w int64 = 1.0<<33         // 1.0<<33 is a constant shift expression
var x = a[1.0<<s]             // 1.0 has type int; x == a[0] if ints are 32bits in size
var a = make([]byte, 1.0<<s)  // 1.0 has type int; len(a) == 0 if ints are 32bits in size
```

### Operator precedence（操作符优先级）

Unary operators have the highest precedence. As the `++` and `--` operators form statements, not expressions, they fall outside the operator hierarchy. As a consequence, statement `*p++` is the same as `(*p)++`.

一元操作符有最高的优先级。当 `++` 和 `--` 操作符组成语句而不是表达式时，他们不属于运算符的结构层次。因此，语句 `*p++` 和 `(*p)++` 是一样的。

There are five precedence levels for binary operators. Multiplication operators bind strongest, followed by addition operators, comparison operators, `&&` (logical AND), and finally `||` (logical OR):

对二元操作符来说，有 5 个优先级的等级。乘法操作符的优先级最高，然后是加法操作符、比较云算法、与运算符，最后是或运算符：

```go
Precedence    Operator
    5             *  /  %  <<  >>  &  &^
    4             +  -  |  ^
    3             ==  !=  <  <=  >  >=
    2             &&
    1             ||
```

Binary operators of the same precedence associate from left to right. For instance, `x / y * z` is the same as `(x / y) * z`.

同一优先级的运算符，按照从左到右的顺序依次执行，比如 `x / y *z` 和 `(x / y) * z` 是一样的。

```go
+x
23 + 3*x[i]
x <= f()
^a >> b
f() || g()
x == y+1 && <-chanPtr > 0
```

## Arithmetic operators（算术运算符）

Arithmetic operators apply to numeric values and yield a result of the same type as the first operand. The four standard arithmetic operators (`+`, `-`, `*`, `/`) apply to integer, floating-point, and complex types; `+` also applies to strings. The bitwise logical and shift operators apply to integers only.

算术运算符可以作用于数字的值，并生成与第一个操作数相同类型的结果值。标准的四个运算符（`+, -, *, /`）可以作用于整数、小数和虚数；`+` 也可以作用于字符串。位逻辑和移位操作只能作用于整数。

```go
+    sum                    integers, floats, complex values, strings
-    difference             integers, floats, complex values
*    product                integers, floats, complex values
/    quotient               integers, floats, complex values
%    remainder              integers

&    bitwise AND            integers
|    bitwise OR             integers
^    bitwise XOR            integers
&^   bit clear (AND NOT)    integers

<<   left shift             integer << unsigned integer
>>   right shift            integer >> unsigned integer
```

### Integer operators（整数运算符）

For two integer values `x` and `y`, the integer quotient `q = x / y` and remainder `r = x % y` satisfy the following relationships:

对于两个整数值 `x` 和 `y`，整数的商 `q = x / y` 和余数`r = x % y` 满足下面的关系式：

```go
x = q*y + r  and  |r| < |y|
```

with `x / y` truncated towards zero (["truncated division"](https://en.wikipedia.org/wiki/Modulo_operation)).

```go
 x     y     x / y     x % y
 5     3       1         2
-5     3      -1        -2
 5    -3      -1         2
-5    -3       1        -2
```

The one exception to this rule is that if the dividend `x` is the most negative value for the int type of `x`, the quotient `q = x / -1` is equal to `x` (and `r = 0`) due to two's-complement [integer overflow](https://golang.google.cn/ref/spec#Integer_overflow):

`x / y` 会向 0 进行去尾。有一种特例，如果被除数 `x` 是其类型的最小的复数，则 `q = x / -1` 的商依然是 `x` （否则就溢出了），余数 `r = 0`。

```go
			 x, q
int8                     -128
int16                  -32768
int32             -2147483648
int64    -9223372036854775808
```

If the divisor is a [constant](https://golang.google.cn/ref/spec#Constants), it must not be zero. If the divisor is zero at run time, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs. If the dividend is non-negative and the divisor is a constant power of 2, the division may be replaced by a right shift, and computing the remainder may be replaced by a bitwise AND operation:

如果除数是一个常量，这个常量的值不能为 0。如果在运行时除数是 0 则会发生运行时错误。如果被除数是一个非负数且除数是一个 2 的幂数的常量，除法会被替换成为右移，求余会被替换成为按位 与 运算。

```go
 x     x / 4     x % 4     x >> 2     x & 3
 11      2         3         2          3
-11     -2        -3        -3          1
```

The shift operators shift the left operand by the shift count specified by the right operand, which must be positive. If the shift count is negative at run time, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs. The shift operators implement arithmetic shifts if the left operand is a signed integer and logical shifts if it is an unsigned integer. There is no upper limit on the shift count. Shifts behave as if the left operand is shifted `n` times by 1 for a shift count of `n`. As a result, `x << 1` is the same as `x*2` and `x >> 1` is the same as `x/2` but truncated towards negative infinity.

移位操作符，操作符左边的数移位右边操作符指定的位数，右操作符必须是正数。如果移位的数量是一个负值，运行时会发生 panic 错误。如果左操作数时一个有符号正数，移位操作符实现的是算术移位；如果左操作数是一个无符号整数，移位操作符实现的是逻辑移位。在移位的数量上是没有上限限制的。移位操作次数 n 的表现就好像是进行 n 次移 1 位的操作。因此，`x << 1` 和 `x * 2` 一样的效果，`x >> 1` 和 `x/2` 是一样的效果，但是会被截断为负无穷大。

For integer operands, the unary operators `+`, `-`, and `^` are defined as follows:

对于整数运算符，一元运算符 `+, -, ^` 的定义如下（注意取非的操作）：

```go
+x                          is 0 + x
-x    negation              is 0 - x
^x    bitwise complement    is m ^ x  with m = "all bits set to 1" for unsigned x
                                      and  m = -1 for signed x
```

### Integer overflow（整数溢出）

For unsigned integer values, the operations `+`, `-`, `*`, and `<<` are computed modulo 2*n*, where *n* is the bit width of the [unsigned integer](https://golang.google.cn/ref/spec#Numeric_types)'s type. Loosely speaking, these unsigned integer operations discard high bits upon overflow, and programs may rely on "wrap around".

对于无符号整形数值，操作符 `+, -, *, <<`  计算的模是 2n，其中 n 是无符号数的字节宽度。不严谨地讲，这些无符号整数在会丢弃溢出的位，程序会依赖于环绕。（？）

For signed integers, the operations `+`, `-`, `*`, `/`, and `<<` may legally overflow and the resulting value exists and is deterministically defined by the signed integer representation, the operation, and its operands. Overflow does not cause a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics). A compiler may not optimize code under the assumption that overflow does not occur. For instance, it may not assume that `x < x + 1` is always true.

对于有符号数，操作符 `+, -, *, <<` 可以进行溢出，结果值肯定会存在且其值决定于有符号数、操作符和操作数。有符号数的溢出不会造成运行时错误。如果编译器假设溢出不会发生，则不会对代码进行优化。比如，编译器不会假设 `x < x +1` 总是成立。

### Floating-point operators（浮点数运算符）

For floating-point and complex numbers, `+x` is the same as `x`, while `-x` is the negation of `x`. The result of a floating-point or complex division by zero is not specified beyond the IEEE-754 standard; whether a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs is implementation-specific.

对浮点数和复数， `+x` 和 `x` 是一样的，`-x` 是 `x` 的负值。无论是浮点数还是复数，根据 `IEEE-754`标准 除数不能为 0，当除数是 0 的时候根据实现不同也可能有不同的结果。

An implementation may combine multiple floating-point operations into a single fused operation, possibly across statements, and produce a result that differs from the value obtained by executing and rounding the instructions individually. An explicit floating-point type [conversion](https://golang.google.cn/ref/spec#Conversions) rounds to the precision of the target type, preventing fusion that would discard that rounding.

一种实现上可能会组合多个浮点数的运算为一个独立的复合运算，其中浮点数的运算可能是一些语句；最后生成的结果可能与各个表达式分别计算（包含一些精度的忽略）。一个显式的浮点数的类型转换会舍入到目标类型的精度，从而避免丢弃舍入的复合运算。

For instance, some architectures provide a "fused multiply and add" (FMA) instruction that computes `x*y + z` without rounding the intermediate result `x*y`. These examples show when a Go implementation can use that instruction:

比如，一些架构提供“乘法加法复合运算”（FMA）指令，专门用于计算 `x*y + z`，此时中间结果 `x*y` 不会进行舍入操作。下面的例子说明了 Go 实现中什么时候可以使用这个指令：

```go
// FMA allowed for computing r, because x*y is not explicitly rounded:
r  = x*y + z
r  = z;   r += x*y
t  = x*y; r = t + z
*p = x*y; r = *p + z
r  = x*y + float64(z)

// FMA disallowed for computing r, because it would omit rounding of x*y:
r  = float64(x*y) + z
r  = z; r += float64(x*y)
t  = float64(x*y); r = t + z
```

### String concatenation（字符串拼接）

Strings can be concatenated using the `+` operator or the `+=` assignment operator:

字符串可以通过 `+` 或 `+=` 运算符进行拼接。

```go
s := "hi" + string(c)
s += " and good bye"
```

String addition creates a new string by concatenating the operands.

通过拼接运算符，会创建一个新的字符串。

## Comparison operators（比较运算符）

Comparison operators compare two operands and yield an untyped boolean value.

比较运算符比较两个操作数并产生一个布尔值。

```go
==    equal
!=    not equal
<     less
<=    less or equal
>     greater
>=    greater or equal
```

In any comparison, the first operand must be [assignable](https://golang.google.cn/ref/spec#Assignability) to the type of the second operand, or vice versa.

在任何的比较中，第一个操作数必须能赋值给第二个操作数（二者具有可比性），反之亦然。

The equality operators `==` and `!=` apply to operands that are *comparable*. The ordering operators `<`, `<=`, `>`, and `>=` apply to operands that are *ordered*. These terms and the result of the comparisons are defined as follows:

相等运算符 `==` 和 `!=` 用于比较两个操作符。顺序运算符 `<, <=, >, >=` 用于排序操作数。这些比较的语法和结果被定义为：

- Boolean values are comparable. Two boolean values are equal if they are either both `true` or both `false`.
- 布尔值是可比较的。两个布尔值如果全是 true 或者全是 false，则他们是相等的。
- Integer values are comparable and ordered, in the usual way.
- 通常情况下，整形数字可以比较且可以排序。
- Floating-point values are comparable and ordered, as defined by the IEEE-754 standard.
- 根据 IEEE-754 规范，浮点数可以比较也可以排序。
- Complex values are comparable. Two complex values `u` and `v` are equal if both `real(u) == real(v)` and `imag(u) == imag(v)`.
- 复数值是可以比较的。两个复数 `u` 和 `v`，如果 `real(u) == real(v)` 且 `imag(u) == imag(v)`则二者相等。
- String values are comparable and ordered, lexically byte-wise.
- 字符串是可比较且可按位按字典排序。
- Pointer values are comparable. Two pointer values are equal if they point to the same variable or if both have value `nil`. Pointers to distinct [zero-size](https://golang.google.cn/ref/spec#Size_and_alignment_guarantees) variables may or may not be equal.
- 指针是可比较的。如果两个指针指向同一个变量值或者二者都是 nil 则二者是相等的。如果指针指向不同的非 0 值，则二者可能相等也可能不相等。
- Channel values are comparable. Two channel values are equal if they were created by the same call to [`make`](https://golang.google.cn/ref/spec#Making_slices_maps_and_channels) or if both have value `nil`.
- 信道的值可以比较。如果两个信道通过一样的 `make` 调用创建的，或者二者都是 nil，则二者是相等的。
- Interface values are comparable. Two interface values are equal if they have [identical](https://golang.google.cn/ref/spec#Type_identity) dynamic types and equal dynamic values or if both have value `nil`.
- 接口的值是可以比较的。如果两个接口值有相同的动态类型值（类型和值都一样），或者二者都是 nil，则二者是相等的。
- A value `x` of non-interface type `X` and a value `t` of interface type `T` are comparable when values of type `X` are comparable and `X` implements `T`. They are equal if `t`'s dynamic type is identical to `X` and `t`'s dynamic value is equal to `x`.
- 一个非接口类型 `X` 的值 `x` 和一个接口类型 `T` 的值 `t`，当类型 `X` 的值可比较且 `X` 实现了接口 `T` 时， 二者是可比较的。如果 `t` 的动态类型是 `X` 并且 `t` 的动态值和 `x` 相等，则二者是相等的。
- Struct values are comparable if all their fields are comparable. Two struct values are equal if their corresponding non-[blank](https://golang.google.cn/ref/spec#Blank_identifier) fields are equal.
- 如果结构体的所有的字段都是可以比较的，那么结构体是可以比较的。当两个结构体的素有相关非空字段都相等时，二者就是相等的。
- Array values are comparable if values of the array element type are comparable. Two array values are equal if their corresponding elements are equal.
- 如果数组的元素类型是可比较的，则数组的值也是可比较的。如果两个数组中相应的值都相等，则两个数组相等。

A comparison of two interface values with identical dynamic types causes a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) if values of that type are not comparable. This behavior applies not only to direct interface value comparisons but also when comparing arrays of interface values or structs with interface-valued fields.

两个接口值，如果他们的动态类型是相同的但是动态类型是不可比较的，运行时会报错。这不仅发生在接口类值的直接比较上面，当比较元素是接口值的列表或者字段是接口值的结构体时，都会发生问题。

Slice, map, and function values are not comparable. However, as a special case, a slice, map, or function value may be compared to the predeclared identifier `nil`. Comparison of pointer, channel, and interface values to `nil` is also allowed and follows from the general rules above.

切片、map 和函数的值都是不可比较的。但是有一个特例，切片、map 和函数的值可以与 Go 预定义的 nil 进行比较。指针、信道和接口值也可以与 nil 进行比较。

```go
const c = 3 < 4            // c is the untyped boolean constant true

type MyBool bool
var x, y int
var (
	// The result of a comparison is an untyped boolean.
	// The usual assignment rules apply.
	b3        = x == y // b3 has type bool
	b4 bool   = x == y // b4 has type bool
	b5 MyBool = x == y // b5 has type MyBool
)
```

## Logical operators（逻辑运算符）

Logical operators apply to [boolean](https://golang.google.cn/ref/spec#Boolean_types) values and yield a result of the same type as the operands. The right operand is evaluated conditionally.

逻辑运算符作用于布尔值毕哥生成一个布尔值。操作符右侧的操作数不一定运行（短路）。

```go
&&    conditional AND    p && q  is  "if p then q else false"
||    conditional OR     p || q  is  "if p then true else q"
!     NOT                !p      is  "not p"
```

## Address operators（地址运算符）

For an operand `x` of type `T`, the address operation `&x` generates a pointer of type `*T` to `x`. The operand must be *addressable*, that is, either a variable, pointer indirection, or slice indexing operation; or a field selector of an addressable struct operand; or an array indexing operation of an addressable array. As an exception to the addressability requirement, `x` may also be a (possibly parenthesized) [composite literal](https://golang.google.cn/ref/spec#Composite_literals). If the evaluation of `x` would cause a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics), then the evaluation of `&x` does too.

对于类型 `T` 的操作数 `x`，地址运算符 `&x` 生成类型为 `*T` 的一个指向 `x` 的指针。操作数必须是可以取地址，也就是操作数是一个变量或间接指针，或者是一个切片索引得到的值；或者是一个可取址结构体的字段值；或者是一个可取址的数组的索引得到的值。作为可取址的需求的特例，`x`可能是一个复合文本（很可能括号括起来，此时复合文本的值没有赋值给变量，也不满足上面提到的几个条件）。如果 x 的执行会导致运行时错误，那么 `&x` 也会导致运行时错误。

For an operand `x` of pointer type `*T`, the pointer indirection `*x` denotes the [variable](https://golang.google.cn/ref/spec#Variables) of type `T` pointed to by `x`. If `x` is `nil`, an attempt to evaluate `*x` will cause a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics).

作为指针类型 `*T` 的操作数 `x`，间接指针 `*x` 表示 `x` 指向的值，值的类型是 `T`。如果 `x` 的值是 nil，`*x` 会导致运行时错误。

```go
&x
&a[f(2)]
&Point{2, 3}
*p
*pf(x)

var x *int = nil
*x   // causes a run-time panic
&*x  // causes a run-time panic
```

## Receive operator （接收运算符）

For an operand `ch` of [channel type](https://golang.google.cn/ref/spec#Channel_types), the value of the receive operation `<-ch` is the value received from the channel `ch`. The channel direction must permit receive operations, and the type of the receive operation is the element type of the channel. The expression blocks until a value is available. Receiving from a `nil` channel blocks forever. A receive operation on a [closed](https://golang.google.cn/ref/spec#Close) channel can always proceed immediately, yielding the element type's [zero value](https://golang.google.cn/ref/spec#The_zero_value) after any previously sent values have been received.

对于类型为信道的操作数 `ch`，接收操作符 `<-ch` 的值是从信道 `ch` 中接收到的值。信道的方向必须允许接收操作，且接收到的值的类型是信道中元素的类型。操作会一直阻塞直到有值可用。从值为 nil 的信道中接收数值的话会一直阻塞在那里。如果从一个关闭的信道中接收值，只要信道中之前发送的值已经被接收完，则会直接返回信道中元素的零值。

```go
v1 := <-ch
v2 = <-ch
f(<-ch)
<-strobe  // wait until clock pulse and discard received value
```

A receive expression used in an [assignment](https://golang.google.cn/ref/spec#Assignments) or initialization of the special form

接收操作会在赋值或初始化语句中使用：

```go
x, ok = <-ch
x, ok := <-ch
var x, ok = <-ch
var x, ok T = <-ch
```

yields an additional untyped boolean result reporting whether the communication succeeded. The value of `ok` is `true` if the value received was delivered by a successful send operation to the channel, or `false` if it is a zero value generated because the channel is closed and empty.

上面的语句产生一个附加的未定型布尔值，用来标识是否通信正常。如果接收的值是通过通过发送操作发送到信道中的，则 `ok` 的值是 true；如果因为信道关闭或空产生了零值，则 `ok` 的值为 false。

## Conversions（转换）

A conversion changes the [type](https://golang.google.cn/ref/spec#Types) of an expression to the type specified by the conversion. A conversion may appear literally in the source, or it may be *implied* by the context in which an expression appears.

类型转换可以把一个表达式的类型转换为一个特定的类型。类型转换可能直接发生在源码的文本中（显式转换），也可能在上下文中隐式地发生转换。

An *explicit* conversion is an expression of the form `T(x)` where `T` is a type and `x` is an expression that can be converted to type `T`.

一个显式的转换的表达式类似于 `T(x)`，其中 `T` 是类型，`x`是一个可以转换为 `T` 的表达式。

```go
Conversion = Type "(" Expression [ "," ] ")" .
```

If the type starts with the operator `*` or `<-`, or if the type starts with the keyword `func` and has no result list, it must be parenthesized when necessary to avoid ambiguity:

如果一个类型是由 `*` 或 `<-` 开始的，或者是由 `func` 关键字开始且没有返回值列表，此时为了避免歧义，需要显式地添加括号：

```go
*Point(p)        // same as *(Point(p))
(*Point)(p)      // p is converted to *Point
<-chan int(c)    // same as <-(chan int(c))
(<-chan int)(c)  // c is converted to <-chan int
func()(x)        // function signature func() x
(func())(x)      // x is converted to func()
(func() int)(x)  // x is converted to func() int
func() int(x)    // x is converted to func() int (unambiguous)
```

A [constant](https://golang.google.cn/ref/spec#Constants) value `x` can be converted to type `T` if `x` is [representable](https://golang.google.cn/ref/spec#Representability) by a value of `T`. As a special case, an integer constant `x` can be explicitly converted to a [string type](https://golang.google.cn/ref/spec#String_types) using the [same rule](https://golang.google.cn/ref/spec#Conversions_to_and_from_a_string_type) as for non-constant `x`.

对于常量值 `x` 来说，如果 `x` 可以由类型 `T` 的值表示，则 `x` 可以被转换为类型 `T` 的值。作为一种特殊情况，就像非常量 `x` 一样，整型常量 `x` 可以被显式地转换为一个字符串类型。

Converting a constant yields a typed constant as result.

转换一个常量会生成一个对应类型的常量。

```go
uint(iota)               // iota value of type uint
float32(2.718281828)     // 2.718281828 of type float32
complex128(1)            // 1.0 + 0.0i of type complex128
float32(0.49999999)      // 0.5 of type float32
float64(-1e-1000)        // 0.0 of type float64
string('x')              // "x" of type string
string(0x266c)           // "♬" of type string
MyString("foo" + "bar")  // "foobar" of type MyString
string([]byte{'a'})      // not a constant: []byte{'a'} is not a constant
(*int)(nil)              // not a constant: nil is not a constant, *int is not a boolean, numeric, or string type
int(1.2)                 // illegal: 1.2 cannot be represented as an int
string(65.0)             // illegal: 65.0 is not an integer constant
```

A non-constant value `x` can be converted to type `T` in any of these cases:

- `x` is [assignable](https://golang.google.cn/ref/spec#Assignability) to `T`.
- ignoring struct tags (see below), `x`'s type and `T` have [identical](https://golang.google.cn/ref/spec#Type_identity) [underlying types](https://golang.google.cn/ref/spec#Types).
- ignoring struct tags (see below), `x`'s type and `T` are pointer types that are not [defined types](https://golang.google.cn/ref/spec#Type_definitions), and their pointer base types have identical underlying types.
- `x`'s type and `T` are both integer or floating point types.
- `x`'s type and `T` are both complex types.
- `x` is an integer or a slice of bytes or runes and `T` is a string type.
- `x` is a string and `T` is a slice of bytes or runes.

一个非常量值 `x` 可以转换为类型 `T` 的值，只需:

- `x` 可以赋值给类型 `T` 的变量；
- 忽略结构体的标签（见下面），`x` 的类型和 `T`有相同的底层类型；
- 忽略结构体的标签（见下面），`x` 的类型和 `T` 都是非自定义的类型的指针类型，并且它们的指针的基类是一样的。
- `x` 类型和 `T` 都是整数类型或浮点数类型；
- `x` 的类型和 `T` 都是复数类型；
- `x` 是一个整数或一个字节或 rune 的切片，`T` 是字符串类型；
- `x` 是字符串并且 `T` 是字节或 rune 的切片。

[Struct tags](https://golang.google.cn/ref/spec#Struct_types) are ignored when comparing struct types for identity for the purpose of conversion:

在比较结构体类型是否一致时，结构体标签会被忽略：

```go
type Person struct {
	Name    string
	Address *struct {
		Street string
		City   string
	}
}

var data *struct {
	Name    string `json:"name"`
	Address *struct {
		Street string `json:"street"`
		City   string `json:"city"`
	} `json:"address"`
}

var person = (*Person)(data)  // ignoring tags, the underlying types are identical
```

Specific rules apply to (non-constant) conversions between numeric types or to and from a string type. These conversions may change the representation of `x` and incur a run-time cost. All other conversions only change the type but not the representation of `x`.

在数字类型和字符串类型的相互转换中，存在一些特殊的规则；这些转换可能会改变 `x` 的表示并且损耗运行时间。其他的转换只改变类型但是不会改变 `x` 的表示。

There is no linguistic mechanism to convert between pointers and integers. The package [`unsafe`](https://golang.google.cn/ref/spec#Package_unsafe) implements this functionality under restricted circumstances.

语言层面不允许指针类型和整数之间的转换。包 `unsafe` 实现了二者之间的严格转换机制。

### Conversions between numeric types（数字类型之间的转换）

For the conversion of non-constant numeric values, the following rules apply

1. When converting between integer types, if the value is a signed integer, it is sign extended to implicit infinite precision; otherwise it is zero extended. It is then truncated to fit in the result type's size. For example, if `v := uint16(0x10F0)`, then `uint32(int8(v)) == 0xFFFFFFF0`. The conversion always yields a valid value; there is no indication of overflow.
2. When converting a floating-point number to an integer, the fraction is discarded (truncation towards zero).
3. When converting an integer or floating-point number to a floating-point type, or a complex number to another complex type, the result value is rounded to the precision specified by the destination type. For instance, the value of a variable `x` of type `float32` may be stored using additional precision beyond that of an IEEE-754 32-bit number, but float32(x) represents the result of rounding `x`'s value to 32-bit precision. Similarly, `x + 0.1` may use more than 32 bits of precision, but `float32(x + 0.1)` does not.

In all non-constant conversions involving floating-point or complex values, if the result type cannot represent the value the conversion succeeds but the result value is implementation-dependent.

对于非常量数值，下面的规则:

1. 当整数之间转换时，如果值是有符号整数，它的符号位进行扩展从而尽可能保证精确；否则扩展 0 位。然后根据结果类型的位数进行截取。比如，有 `v := uint16(0x10F0)` ，那么 `uint32(int8(v)) == 0xFFFFFFF0`。转换总能产生一个有效值，不会出现溢出错误。
2. 当把一个浮点值转换为整数时，小数部分会被丢弃（值趋向于 0）。
3. 当把一个整数或者浮点数转换为浮点数，或者复数转换为两一个复数类型，结果会舍入到结果值类型的精度。比如，`float32`  精度的值 `x` 可以存储在比 IEEE-754 的 32 位数字更精确的变量里，但是 float(32) 表示舍入 `x` 的值到 32 位的精度。同样的，`x + 0.1` 可能会使用 32 位以上的精度，但是 `float32(x + 0.1)` 则不会。

在所有非常量浮点数或者复数转换中，如果结果类型不能表示成功转换后的值，具体结果值会依赖具体的实现。

### Conversions to and from a string type（字符串转换）

1. Converting a signed or unsigned integer value to a string type yields a string containing the UTF-8 representation of the integer. Values outside the range of valid Unicode code points are converted to "\uFFFD". 转换一个有符号或无符号的整数可以产生一个代表相应整数的 UTF-8 字符串。如果给定的整数超过了  Unicode 字节码的范围，会返回 “\uFFFD”。

   ```go
   string('a')       // "a"
   string(-1)        // "\ufffd" == "\xef\xbf\xbd"
   string(0xf8)      // "\u00f8" == "ø" == "\xc3\xb8"
   type MyString string
   MyString(0x65e5)  // "\u65e5" == "日" == "\xe6\x97\xa5"
   ```

2. Converting a slice of bytes to a string type yields a string whose successive bytes are the elements of the slice. 转换字节切片到字符串类型，会根据切片中的元素顺序生成一个字符串。

   ```go
   string([]byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'})   // "hellø"
   string([]byte{})                                     // ""
   string([]byte(nil))                                  // ""
   
   type MyBytes []byte
   string(MyBytes{'h', 'e', 'l', 'l', '\xc3', '\xb8'})  // "hellø"
   ```

3. Converting a slice of runes to a string type yields a string that is the concatenation of the individual rune values converted to strings. 转换 rune 切片到字符串类型，会根据切片中的各个元素顺序生成一个字符串。

   ```go
   string([]rune{0x767d, 0x9d6c, 0x7fd4})   // "\u767d\u9d6c\u7fd4" == "白鵬翔"
   string([]rune{})                         // ""
   string([]rune(nil))                      // ""
   
   type MyRunes []rune
   string(MyRunes{0x767d, 0x9d6c, 0x7fd4})  // "\u767d\u9d6c\u7fd4" == "白鵬翔"
   ```

4. Converting a value of a string type to a slice of bytes type yields a slice whose successive elements are the bytes of the string. 把字符串类型转换为字节切片，可以生成一个字符串中各个字符代表的字节切片。

   ```go
   []byte("hellø")   // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
   []byte("")        // []byte{}
   
   MyBytes("hellø")  // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
   ```

5. Converting a value of a string type to a slice of runes type yields a slice containing the individual Unicode code points of the string. 把字符串类型转换为 rune 切片，可以生成一个字符串中各个字符代表的rune 切片。

   ```go
   []rune(MyString("白鵬翔"))  // []rune{0x767d, 0x9d6c, 0x7fd4}
   []rune("")                 // []rune{}
   
   MyRunes("白鵬翔")           // []rune{0x767d, 0x9d6c, 0x7fd4}
   ```

## Constant expressions （常量表达式）

Constant expressions may contain only [constant](https://golang.google.cn/ref/spec#Constants) operands and are evaluated at compile time.

常量表达式可能只包含常量操作数，它会在编译时进行运算。

Untyped boolean, numeric, and string constants may be used as operands wherever it is legal to use an operand of boolean, numeric, or string type, respectively.

只要期望的操作数时布尔、数字后者字符串，就可以在这些地方使用未定义类型的布尔常量、数字常量和字符常量。

A constant [comparison](https://golang.google.cn/ref/spec#Comparison_operators) always yields an untyped boolean constant. If the left operand of a constant [shift expression](https://golang.google.cn/ref/spec#Operators) is an untyped constant, the result is an integer constant; otherwise it is a constant of the same type as the left operand, which must be of [integer type](https://golang.google.cn/ref/spec#Numeric_types).

常量之间的比较总能生成未定义类型的布尔常量。如果移位表达式中的左操作数时一个未定义类型的常量，结果会是一个整数常量；否则类型会和左操作数保持一致，此时左操作数必须是一个整形类型。

Any other operation on untyped constants results in an untyped constant of the same kind; that is, a boolean, integer, floating-point, complex, or string constant. If the untyped operands of a binary operation (other than a shift) are of different kinds, the result is of the operand's kind that appears later in this list: integer, rune, floating-point, complex. For example, an untyped integer constant divided by an untyped complex constant yields an untyped complex constant.

任何其他对未定型常量的操作都会产生一个相同类型的结果；布尔、整数、浮点数、复数或字符串类型都有这种规律。如果二进制操作（非移位）中的操作数的类型不一致，结果是出现在下面列表中的后者：整数、rune、浮点数、复数。比如，一个未定义类型的整数除一个未定义类型的复数，最后生成的是一个未定型的复数常量。

```go
const a = 2 + 3.0          // a == 5.0   (untyped floating-point constant)
const b = 15 / 4           // b == 3     (untyped integer constant)
const c = 15 / 4.0         // c == 3.75  (untyped floating-point constant)
const Θ float64 = 3/2      // Θ == 1.0   (type float64, 3/2 is integer division)
const Π float64 = 3/2.     // Π == 1.5   (type float64, 3/2. is float division)
const d = 1 << 3.0         // d == 8     (untyped integer constant)
const e = 1.0 << 3         // e == 8     (untyped integer constant)
const f = int32(1) << 33   // illegal    (constant 8589934592 overflows int32)
const g = float64(2) >> 1  // illegal    (float64(2) is a typed floating-point constant)
const h = "foo" > "bar"    // h == true  (untyped boolean constant)
const j = true             // j == true  (untyped boolean constant)
const k = 'w' + 1          // k == 'x'   (untyped rune constant)
const l = "hi"             // l == "hi"  (untyped string constant)
const m = string(k)        // m == "x"   (type string)
const Σ = 1 - 0.707i       //            (untyped complex constant)
const Δ = Σ + 2.0e-4       //            (untyped complex constant)
const Φ = iota*1i - 1/1i   //            (untyped complex constant)
```

Applying the built-in function `complex` to untyped integer, rune, or floating-point constants yields an untyped complex constant.

调用 `complex` 作用在未定义类型的 整数、符文字节、或者浮点数常量上，最后都会生成一个复数常量。

```go
const ic = complex(0, c)   // ic == 3.75i  (untyped complex constant)
const iΘ = complex(0, Θ)   // iΘ == 1i     (type complex128)
```

Constant expressions are always evaluated exactly; intermediate values and the constants themselves may require precision significantly larger than supported by any predeclared type in the language. The following are legal declarations:

常量表达式总是精确地进行运算；无论是运算的中间值还是常量本身，总是会比语言中更大精度的类型进行处理，下面的是一些合法的声明：

```go
const Huge = 1 << 100         // Huge == 1267650600228229401496703205376  (untyped integer constant)
const Four int8 = Huge >> 98  // Four == 4                                (type int8)
```

The divisor of a constant division or remainder operation must not be zero:

常量除法和取余运算中除数都不能为 0：

```go
3.14 / 0.0   // illegal: division by zero
```

The values of *typed* constants must always be accurately [representable](https://golang.google.cn/ref/spec#Representability) by values of the constant type. The following constant expressions are illegal:

指定类型的常量必须能够由指定的常量类型进行标识，否则是不合法的，比如下面的例子：

```go
uint(-1)     // -1 cannot be represented as a uint
int(3.14)    // 3.14 cannot be represented as an int
int64(Huge)  // 1267650600228229401496703205376 cannot be represented as an int64
Four * 300   // operand 300 cannot be represented as an int8 (type of Four)
Four * 100   // product 400 cannot be represented as an int8 (type of Four)
```

The mask used by the unary bitwise complement operator `^` matches the rule for non-constants: the mask is all 1s for unsigned constants and -1 for signed and untyped constants.

对常量来说，一元按位运算中的补位运算符 `^` 与非常量一样的使用规则：对于无符号数常量来说，mask 是全 1 位，对于有符号常量来说 mask 是 -1。

```go
^1         // untyped integer constant, equal to -2
uint8(^1)  // illegal: same as uint8(-2), -2 cannot be represented as a uint8
^uint8(1)  // typed uint8 constant, same as 0xFF ^ uint8(1) = uint8(0xFE)
int8(^1)   // same as int8(-2)
^int8(1)   // same as -1 ^ int8(1) = -2
```

Implementation restriction: A compiler may use rounding while computing untyped floating-point or complex constant expressions; see the implementation restriction in the section on [constants](https://golang.google.cn/ref/spec#Constants). This rounding may cause a floating-point constant expression to be invalid in an integer context, even if it would be integral when calculated using infinite precision, and vice versa.

实现上的限制：当计算未定型的浮点数或者复数常量时，编译器可能会舍入一些精度，可以参考 《常量》 一节的讨论。这种舍入可能会导致在整数上下文中浮点常量表达式无效，即使在无限精确的计算中精度是累积的也不行，反之亦然。（？）

## Order of evaluation（执行顺序)

At package level, [initialization dependencies](https://golang.google.cn/ref/spec#Package_initialization) determine the evaluation order of individual initialization expressions in [variable declarations](https://golang.google.cn/ref/spec#Variable_declarations). Otherwise, when evaluating the [operands](https://golang.google.cn/ref/spec#Operands) of an expression, assignment, or [return statement](https://golang.google.cn/ref/spec#Return_statements), all function calls, method calls, and communication operations are evaluated in lexical left-to-right order.

在包级别，初始化依赖决定了各个变量声明中变量的初始化顺序。在一般表达式的运算中，赋值、返回语句、所有的函数调用、方法调用以及信道的运算，都是按照从左向右的顺序进行运算的。

For example, in the (function-local) assignment

```go
y[f()], ok = g(h(), i()+x[j()], <-c), k()
```

the function calls and communication happen in the order `f()`, `h()`, `i()`, `j()`, `<-c`, `g()`, and `k()`. However, the order of those events compared to the evaluation and indexing of `x` and the evaluation of `y` is not specified.

比如 `y[f()], ok = g(h(), i()+x[j()], <-c), k()`，调用顺序依次是 `f(), h(), i(), j(), <-c, g(), k()`。 但是，这些事件和 x 以及 y 的索引操作之间谁先谁后，则并不能确定。

```go
a := 1
f := func() int { a++; return a }
x := []int{a, f()}            // x may be [1, 2] or [2, 2]: evaluation order between a and f() is not specified
m := map[int]int{a: 1, a: 2}  // m may be {2: 1} or {2: 2}: evaluation order between the two map assignments is not specified
n := map[int]int{a: f()}      // n may be {2: 3} or {3: 3}: evaluation order between the key and the value is not specified
```

At package level, initialization dependencies override the left-to-right rule for individual initialization expressions, but not for operands within each expression:

在包界别，初始化依赖覆盖了上面提到的从左向右的顺序，当然每个表达式中的计算依然按照那个顺序执行。

```go
var a, b, c = f() + v(), g(), sqr(u()) + v()

func f() int        { return c }
func g() int        { return a }
func sqr(x int) int { return x*x }

// functions u and v are independent of all other variables and functions
```

The function calls happen in the order `u()`, `sqr()`, `v()`, `f()`, `v()`, and `g()`.

上面函数调用的顺序依次是：`u(), sqr(), v(), f(), v(), g()`。

Floating-point operations within a single expression are evaluated according to the associativity of the operators. Explicit parentheses affect the evaluation by overriding the default associativity. In the expression `x + (y + z)` the addition `y + z` is performed before adding `x`.

一个表达式中的浮点运算根据相关的运算符的顺序进行执行。通过使用括号可以改变默认的执行顺序，在表达式 `x + (y + z)` 中，加法 `y + z` 会先运行。



# Statements（语句）

Statements control execution.

语句控制执行。

```go
Statement =
	Declaration | LabeledStmt | SimpleStmt |
	GoStmt | ReturnStmt | BreakStmt | ContinueStmt | GotoStmt |
	FallthroughStmt | Block | IfStmt | SwitchStmt | SelectStmt | ForStmt |
	DeferStmt .

SimpleStmt = EmptyStmt | ExpressionStmt | SendStmt | IncDecStmt | Assignment | ShortVarDecl .
```

## Terminating statements（终止语句）

A *terminating statement* prevents execution of all statements that lexically appear after it in the same [block](https://golang.google.cn/ref/spec#Blocks). The following statements are terminating:

终止语句阻止同一个块中其后面语句的执行，下面的语句可以终止：

1. A "return" or "goto" statement. 【“return” 或 “goto” 语句。

2. A call to the built-in function `panic`. 【内建函数 `panic` 的调用语句。

3. A block in which the statement list ends in a terminating statement. 【一个带终止语句的块。

4. An "if" statement in which: 【对于 `if` 语句来说，

  - the "else" branch is present, and【存在 `else` 语句，且 
  - both branches are terminating statements. 【两个分支都有终止语句。
  
5. A "for" statement in which: 【对于 `for` 语句来说

  - there are no "break" statements referring to the "for" statement, and 【没有 `break` 语句，且
  - the loop condition is absent. 【条件语句是空的。（意思就是一直阻塞在 for 循环内部）
  
6. A "switch" statement in which:【对于 `switch` 语句来说，

  - there are no "break" statements referring to the "switch" statement,【没有 `break` 语句。
  - there is a default case, and 【但是有一个 default 分支
  - the statement lists in each case, including the default, end in a terminating statement, or a possibly labeled ["fallthrough" statement](https://golang.google.cn/ref/spec#Fallthrough_statements).【且每种情况，包含 default，最后都以终止语句结束，或者有一个 fallthrough 的语句。

7. A "select" statement in which: 【 对于 `select` 语句

  - there are no "break" statements referring to the "select" statement, and 【没有 `break` 语句，
  - the statement lists in each case, including the default if present, end in a terminating statement.【对于每个情况，包括默认的情况，都以终止语句结束。
  
8. A [labeled statement](https://golang.google.cn/ref/spec#Labeled_statements) labeling a terminating statement.【一个标记到终止语句的标签。

All other statements are not terminating.

除了上面的情况，其他的语句都不是终止语句。

A [statement list](https://golang.google.cn/ref/spec#Blocks) ends in a terminating statement if the list is not empty and its final non-empty statement is terminating.

如果一个语句列表不是空的，且它最后非空的语句是结束，这个语句列表也是终止语句。

## Empty statements（空语句）

The empty statement does nothing.

空语句不做任何事情。

```go
EmptyStmt = .
```

## Labeled statements（标签语句）

A labeled statement may be the target of a `goto`, `break` or `continue` statement.

标签语句可以定义 `goto, break, continue` 语句跳转的目标位置。

​```go
LabeledStmt = Label ":" Statement .
Label       = identifier .
```
```go
Error: log.Panic("error encountered")
```

## Expression statements（表达式语句)

With the exception of specific built-in functions, function and method [calls](https://golang.google.cn/ref/spec#Calls) and [receive operations](https://golang.google.cn/ref/spec#Receive_operator) can appear in statement context. Such statements may be parenthesized.

除了一些内建的函数，普通函数和方法调用以及接收操作都可以出现在声明语句上下文中。这些语句可能会被括号括起来。

```go
ExpressionStmt = Expression .
```

The following built-in functions are not permitted in statement context:

下面的这些内建函数不允许出现在语句上下文中：

```go
append cap complex imag len make new real
unsafe.Alignof unsafe.Offsetof unsafe.Sizeof
```

```go
h(x+y)
f.Close()
<-ch
(<-ch)
len("foo")  // illegal if len is the built-in function
```

## Send statements（发送语句)

A send statement sends a value on a channel. The channel expression must be of [channel type](https://golang.google.cn/ref/spec#Channel_types), the channel direction must permit send operations, and the type of the value to be sent must be [assignable](https://golang.google.cn/ref/spec#Assignability) to the channel's element type.

发送语句可以在一个信道上发送值。信道表达式必须是一个信道类型的值，信道必须允许发送，且发送的值必须能够赋值给信道元素的类型。

```go
SendStmt = Channel "<-" Expression .
Channel  = Expression .
```

Both the channel and the value expression are evaluated before communication begins. Communication blocks until the send can proceed. A send on an unbuffered channel can proceed if a receiver is ready. A send on a buffered channel can proceed if there is room in the buffer. A send on a closed channel proceeds by causing a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics). A send on a `nil` channel blocks forever.

在通信正式开始前，信道以及被传送的值都会先进行运算（如果有必要）。在发送可以进行前通信会一直进行阻塞。一个不带缓冲区的信道只有在接受者就绪后才可以完成数据的发送。对带缓冲的信道的而言，只要有缓冲空间，就可以完成数据的发送。如果在一个关闭了的信道上发送数据会发生运行时错误。如果在一个 nil 的信道上发送数据则会一直阻塞在那里。

```go
ch <- 3  // send value 3 to channel ch
```

## IncDec statements（加减语句)

The "++" and "--" statements increment or decrement their operands by the untyped [constant](https://golang.google.cn/ref/spec#Constants) `1`. As with an assignment, the operand must be [addressable](https://golang.google.cn/ref/spec#Address_operators) or a map index expression.

语句 `++` 和 `--` 会给他们的操作数 **加 **或 **减** 未定义类型的常量 1。作为一个赋值语句，操作数必须是可寻址的，或者是一个 map 的索引表达式。

```go
IncDecStmt = Expression ( "++" | "--" ) .
```

The following [assignment statements](https://golang.google.cn/ref/spec#Assignments) are semantically equivalent:

下面的赋值语句是等效的：

```go
IncDec statement    Assignment
x++                 x += 1
x--                 x -= 1
```

## Assignments（赋值语句)

```go
Assignment = ExpressionList assign_op ExpressionList .

assign_op = [ add_op | mul_op ] "=" .
```

Each left-hand side operand must be [addressable](https://golang.google.cn/ref/spec#Address_operators), a map index expression, or (for `=` assignments only) the [blank identifier](https://golang.google.cn/ref/spec#Blank_identifier). Operands may be parenthesized.

等号左侧的每个操作数要么可寻址，要么是 map 映射的索引表达式，或者是（仅对 `=` 来说）空标识符。操作数可以用括号括起来。

```go
x = 1
*p = f()
a[i] = 23
(k) = <-ch  // same as: k = <-ch
```

An *assignment operation* `x op= y` where *op* is a binary [arithmetic operator](https://golang.google.cn/ref/spec#Arithmetic_operators) is equivalent to `x = x op (y)` but evaluates `x` only once. The `op=` construct is a single token. In assignment operations, both the left- and right-hand expression lists must contain exactly one single-valued expression, and the left-hand expression must not be the blank identifier.

对于赋值表达式 `x op= y`，其中 `op` 是一个二元算术运算符，其等效于 `x = x op (y)`，但是只会执行一次 `x`。`op=` 是一个独立的标识。在这种赋值语句中，左边和右边的表达式列表都必须只能包含单值表达式，且左侧表达式不能是空标识符。

```go
a[i] <<= 2
i &^= 1<<n
```

A tuple assignment assigns the individual elements of a multi-valued operation to a list of variables. There are two forms. In the first, the right hand operand is a single multi-valued expression such as a function call, a [channel](https://golang.google.cn/ref/spec#Channel_types) or [map](https://golang.google.cn/ref/spec#Map_types) operation, or a [type assertion](https://golang.google.cn/ref/spec#Type_assertions). The number of operands on the left hand side must match the number of values. For instance, if `f` is a function returning two values,

```go
x, y = f()
```

assigns the first value to `x` and the second to `y`. In the second form, the number of operands on the left must equal the number of expressions on the right, each of which must be single-valued, and the *n*th expression on the right is assigned to the *n*th operand on the left:

```go
one, two, three = '一', '二', '三'
```

元组赋值语句把多个值分别赋值给一组变量。目前存在两种形式。第一种形式，操作符右边的操作数是一个返回多值的表达式，比如函数调用、信道或 map 操作、或类型断言。操作符左侧的操作数的数量必须要和右边的数量相等。比如，如果 `f` 可以返回两个值， `x, y =f()`，第一个返回值会赋值给 `x`，第二个赋值给 `y`。第二种方式，左侧的操作数必须和右侧的表达式数量一样，且每个表达式都是单值的，这种情况下，右侧第 n 个表达式的值赋给了左侧第 n 个变量。

The [blank identifier](https://golang.google.cn/ref/spec#Blank_identifier) provides a way to ignore right-hand side values in an assignment:

在赋值语句中，空标识符提供了一种忽略右侧值的方式：

```go
_ = x       // evaluate x but ignore it
x, _ = f()  // evaluate f() but ignore second result value
```

The assignment proceeds in two phases. First, the operands of [index expressions](https://golang.google.cn/ref/spec#Index_expressions) and [pointer indirections](https://golang.google.cn/ref/spec#Address_operators) (including implicit pointer indirections in [selectors](https://golang.google.cn/ref/spec#Selectors)) on the left and the expressions on the right are all [evaluated in the usual order](https://golang.google.cn/ref/spec#Order_of_evaluation). Second, the assignments are carried out in left-to-right order.

赋值过程有两个步骤：1) 首先按照顺序计算左侧值和右侧值中的索引表达式和间接指针(包括在选择器运算中的间接指针运算)；2) 依照从左向右依次赋值。

```go
a, b = b, a  // exchange a and b

x := []int{1, 2, 3}
i := 0
i, x[i] = 1, 2  // set i = 1, x[0] = 2

i = 0
x[i], i = 2, 1  // set x[0] = 2, i = 1

x[0], x[0] = 1, 2  // set x[0] = 1, then x[0] = 2 (so x[0] == 2 at end)

x[1], x[3] = 4, 5  // set x[1] = 4, then panic setting x[3] = 5.

type Point struct { x, y int }
var p *Point
x[2], p.x = 6, 7  // set x[2] = 6, then panic setting p.x = 7

i = 2
x = []int{3, 5, 7}
for i, x[i] = range x {  // set i, x[2] = 0, x[0]
	break
}
// after this loop, i == 0 and x == []int{3, 5, 3}
```

In assignments, each value must be [assignable](https://golang.google.cn/ref/spec#Assignability) to the type of the operand to which it is assigned, with the following special cases:

1. Any typed value may be assigned to the blank identifier.
2. If an untyped constant is assigned to a variable of interface type or the blank identifier, the constant is first implicitly [converted](https://golang.google.cn/ref/spec#Conversions) to its [default type](https://golang.google.cn/ref/spec#Constants).
3. If an untyped boolean value is assigned to a variable of interface type or the blank identifier, it is first implicitly converted to type `bool`.

在赋值操作数中，每个值都必须可以赋值给对应的变量（意味着类型上要匹配），有如下一些特例:

1. 任何类型都可以赋值给空标识符。
2. 当一个未定型的常量被赋值给一个接口类型或者空标识符时，常量首先会被转换成它的默认类型。
3. 当一个未定型的二进制数被赋值给一个接口类型或空标识符时，它首先会被转换成 布尔 型。

## If statements（If 语句）

"If" statements specify the conditional execution of two branches according to the value of a boolean expression. If the expression evaluates to true, the "if" branch is executed, otherwise, if present, the "else" branch is executed.

“if” 语句根据条件语句（布尔型）的布尔值选型执行哪个分支。如果条件语句为真，则 `if` 语句会被执行，否则 `else` 语句会被执行。如果没有 `else` 语句，会直接跳过 `if` 语句执行其后面的语句。

```go
IfStmt = "if" [ SimpleStmt ";" ] Expression Block [ "else" ( IfStmt | Block ) ] .
if x > max {
	x = max
}
```

The expression may be preceded by a simple statement, which executes before the expression is evaluated.

表达式中可能包含一个简单的语句，这个语句在 `if` 语句正式运行前执行。

```go
if x := f(); x < y {
	return x
} else if x > z {
	return z
} else {
	return y
}
```

## Switch statements（switch 语句）

"Switch" statements provide multi-way execution. An expression or type specifier is compared to the "cases" inside the "switch" to determine which branch to execute.

“Switch” 语句提供了一种多路执行的方式。一个表达式或类型描述符和 “Switch” 中的  “case” 语句进行比较从而决策执行哪个分支的逻辑。

```go
SwitchStmt = ExprSwitchStmt | TypeSwitchStmt .
```

There are two forms: expression switches and type switches. In an expression switch, the cases contain expressions that are compared against the value of the switch expression. In a type switch, the cases contain types that are compared against the type of a specially annotated switch expression. The switch expression is evaluated exactly once in a switch statement.

有两种形式：表达式形式和类型形式。在表达式形式的 Switch 中，case 中表达式的值会与 switch 表达式的值进行比较；在类型形式的 switch 中，case 中包含的类型与 switch 表达式声明的类型进行比较。在 switch 语句中，只会匹配一个 case，匹配到一个其他的就不继续匹配了（区别于 C 语言中的 switch）。

### Expression switches（表达式形式的 switches）

In an expression switch, the switch expression is evaluated and the case expressions, which need not be constants, are evaluated left-to-right and top-to-bottom; the first one that equals the switch expression triggers execution of the statements of the associated case; the other cases are skipped. If no case matches and there is a "default" case, its statements are executed. There can be at most one default case and it may appear anywhere in the "switch" statement. A missing switch expression is equivalent to the boolean value `true`.

在表达式 switch 中，switch 的表达式会被计算，同时 case 的表达式（可以是常量以及非常量）根据情况从左到右、从上到下进行计算。第一个与 switch 的表达式匹配的 case 语句会得到执行，其他的 case 语句会自动跳过。如果没有 case 匹配且有一个 “default” 分支，则会执行 “default” 分支的语句。在一个 switch 语句中国最多只能有一个默认的分支且它可以出现在 switch 语句的任何位置。如果 switch 后面不带任何内容，则表示传入的是布尔真。

```go
ExprSwitchStmt = "switch" [ SimpleStmt ";" ] [ Expression ] "{" { ExprCaseClause } "}" .
ExprCaseClause = ExprSwitchCase ":" StatementList .
ExprSwitchCase = "case" ExpressionList | "default" .
```

If the switch expression evaluates to an untyped constant, it is first implicitly [converted](https://golang.google.cn/ref/spec#Conversions) to its [default type](https://golang.google.cn/ref/spec#Constants); if it is an untyped boolean value, it is first implicitly converted to type `bool`. The predeclared untyped value `nil` cannot be used as a switch expression.

如果 switch 语句运算得到的是一个未定型的常量，它首先会被转换成它的默认类型；如果它是一个未定型的布尔值，它首先会被转换成为 `bool` 类型。Go 预定义的 `nil` 不能用在 switch 语句中。

If a case expression is untyped, it is first implicitly [converted](https://golang.google.cn/ref/spec#Conversions) to the type of the switch expression. For each (possibly converted) case expression `x` and the value `t` of the switch expression, `x == t` must be a valid [comparison](https://golang.google.cn/ref/spec#Comparison_operators).

如果 case 的表达式是未定型的，那么它首先会被转换成为 switch 语句对应的类型。对于每个 case 中的表达式 `x`，如果 switch 表达式的值是 `t`，则 `x == t` 必须是合法的。

In other words, the switch expression is treated as if it were used to declare and initialize a temporary variable `t` without explicit type; it is that value of `t` against which each case expression `x` is tested for equality.

换句话说，switch 的表达式就好像声明并初始化了一个临时变量 `t`（没有显式定义类型），然后这个 `t` 的值依次与每个 case 的表达式 `x` 比较可等性。

In a case or default clause, the last non-empty statement may be a (possibly [labeled](https://golang.google.cn/ref/spec#Labeled_statements)) ["fallthrough" statement](https://golang.google.cn/ref/spec#Fallthrough_statements) to indicate that control should flow from the end of this clause to the first statement of the next clause. Otherwise control flows to the end of the "switch" statement. A "fallthrough" statement may appear as the last statement of all but the last clause of an expression switch.

在一个 case 或者 default 语句中，最后一个非空的语句可能是（可能被标签化了）“fallthrough” 语句，这意味着控制语句从当前语句块中跳转到下一个 case 块的第一条语句。如果没有 fallthrough 语句，则一个 case 语句执行完成后整个 switch 语句就结束了。除了 switch 语句的最后位置，“fallthrough” 语句可以出现在任何位置。

The switch expression may be preceded by a simple statement, which executes before the expression is evaluated.

switch 语句可能是一个简单的语句，会在 switch 语句执行前首先执行得到这个简单语句的值。

```go
switch tag {
default: s3()
case 0, 1, 2, 3: s1()
case 4, 5, 6, 7: s2()
}

switch x := f(); {  // missing switch expression means "true"
case x < 0: return -x
default: return x
}

switch {
case x < y: f1()
case x < z: f2()
case x == 4: f3()
}
```

Implementation restriction: A compiler may disallow multiple case expressions evaluating to the same constant. For instance, the current compilers disallow duplicate integer, floating point, or string constants in case expressions.
实现上的限制：一个编译器可能不允许 case 语句中出现相同的常量。比如，当前的编译器不允重复的数字、浮点数或字符串常量出现在 case 语句中。

### Type switches （类型 switch）

A type switch compares types rather than values. It is otherwise similar to an expression switch. It is marked by a special switch expression that has the form of a [type assertion](https://golang.google.cn/ref/spec#Type_assertions) using the reserved word `type` rather than an actual type:

类型形式的 switch 比较的是类型而不是值，其他的都和表达式 switch 一样。这种形式的 switch 语句是一个 类型断言 的表达式，使用了 Go 预留的关键字 `type`。

```go
switch x.(type) {
// cases
}
```

Cases then match actual types `T` against the dynamic type of the expression `x`. As with type assertions, `x` must be of [interface type](https://golang.google.cn/ref/spec#Interface_types), and each non-interface type `T` listed in a case must implement the type of `x`. The types listed in the cases of a type switch must all be [different](https://golang.google.cn/ref/spec#Type_identity).

在这种类型 switch 语句中，case 会匹配动态类型 `x` 语句的真实类型 `T`。在类型 switch 中，`x` 必须是接口类型的值，同时 case 语句中的非接口类型 `T` 必须实现了 `x` 的接口类型。在 case 中列出来的类型必须都是不一样的（有一样的意味着两个 case 都匹配到了，这种是不允许的）。

```go
TypeSwitchStmt  = "switch" [ SimpleStmt ";" ] TypeSwitchGuard "{" { TypeCaseClause } "}" .
TypeSwitchGuard = [ identifier ":=" ] PrimaryExpr "." "(" "type" ")" .
TypeCaseClause  = TypeSwitchCase ":" StatementList .
TypeSwitchCase  = "case" TypeList | "default" .
TypeList        = Type { "," Type } .
```

The TypeSwitchGuard may include a [short variable declaration](https://golang.google.cn/ref/spec#Short_variable_declarations). When that form is used, the variable is declared at the end of the TypeSwitchCase in the [implicit block](https://golang.google.cn/ref/spec#Blocks) of each clause. In clauses with a case listing exactly one type, the variable has that type; otherwise, the variable has the type of the expression in the TypeSwitchGuard.

在 TypeSwitchGuard 中可能包含一个变量的短声明。当使用了短声明的时候，变量会在每个 case 隐式的块里进行声明并可在块内使用。如果一个 case 只包含一个类型，则短声明得到的变量的类型 就是 case 语句上指定的类型。否则，变量的类型和 TypeSwitchGuard 中的类型是一样的（大概率是一个接口）。

Instead of a type, a case may use the predeclared identifier [`nil`](https://golang.google.cn/ref/spec#Predeclared_identifiers); that case is selected when the expression in the TypeSwitchGuard is a `nil` interface value. There may be at most one `nil` case.

除了具体的类型，case 还可以使用预定义的标识符 nil；这种 case 会在 switch 语句中的值为 nil 时满足条件。在 switch 中至多有一个匹配 nil 的case。

Given an expression `x` of type `interface{}`, the following type switch:
给定一个类型为 `interface{}` 的表达式 `x`，有下面的类型 switch ：

```go
switch i := x.(type) {
case nil:
	printString("x is nil")                // type of i is type of x (interface{})
case int:
	printInt(i)                            // type of i is int
case float64:
	printFloat64(i)                        // type of i is float64
case func(int) float64:
	printFunction(i)                       // type of i is func(int) float64
case bool, string:
	printString("type is bool or string")  // type of i is type of x (interface{})
default:
	printString("don't know the type")     // type of i is type of x (interface{})
}
```

could be rewritten:

```go
v := x  // x is evaluated exactly once
if v == nil {
	i := v                                 // type of i is type of x (interface{})
	printString("x is nil")
} else if i, isInt := v.(int); isInt {
	printInt(i)                            // type of i is int
} else if i, isFloat64 := v.(float64); isFloat64 {
	printFloat64(i)                        // type of i is float64
} else if i, isFunc := v.(func(int) float64); isFunc {
	printFunction(i)                       // type of i is func(int) float64
} else {
	_, isBool := v.(bool)
	_, isString := v.(string)
	if isBool || isString {
		i := v                         // type of i is type of x (interface{})
		printString("type is bool or string")
	} else {
		i := v                         // type of i is type of x (interface{})
		printString("don't know the type")
	}
}
```

The type switch guard may be preceded by a simple statement, which executes before the guard is evaluated.

类型 switch 语句中可能是一个简单语句，这种情况下， switch 运行前会首先计算这个简单语句的值。

The "fallthrough" statement is not permitted in a type switch.
在类型 switch 中不允许使用 “fallthrough” 语句。

## For statements（for 语句）

A "for" statement specifies repeated execution of a block. There are three forms: The iteration may be controlled by a single condition, a "for" clause, or a "range" clause.

“for” 语句制定某个语句块重复执行。有三种形式：简单的条件语句、普通的“for”语法、“range”语法。

```go
ForStmt = "for" [ Condition | ForClause | RangeClause ] Block .
Condition = Expression .
```

### For statements with single condition（简单条件语句）

In its simplest form, a "for" statement specifies the repeated execution of a block as long as a boolean condition evaluates to true. The condition is evaluated before each iteration. If the condition is absent, it is equivalent to the boolean value `true`.

在简单语句方式中，只要布尔提交一直为真，“for” 语句就一直循环执行语句块。条件会在每次执行块之前进行计算。如果条件语句不存在，等效于传入了一个 true 值。

```go
for a < b {
	a *= 2
}
```

### For statements with `for` clause （简单“for" 语句）

A "for" statement with a ForClause is also controlled by its condition, but additionally it may specify an *init* and a *post* statement, such as an assignment, an increment or decrement statement. The init statement may be a [short variable declaration](https://golang.google.cn/ref/spec#Short_variable_declarations), but the post statement must not. Variables declared by the init statement are re-used in each iteration.

此种模式和 C 语言中的 for 循环类似，除了条件语句外，还包含 **初始化语句** 和 **后置** 语句，比如一个赋值语句，一个增减语句等。初始化语句可能是一个变量声明的短声明方式，后置语句则不能设置为变量声明语句。在初始化语句中声明的变量会在每次循环时重复使用；也就是说初始化语句只在 for 语句运行前执行一次。

```go
ForClause = [ InitStmt ] ";" [ Condition ] ";" [ PostStmt ] .
InitStmt = SimpleStmt .
PostStmt = SimpleStmt .
for i := 0; i < 10; i++ {
	f(i)
}
```

If non-empty, the init statement is executed once before evaluating the condition for the first iteration; the post statement is executed after each execution of the block (and only if the block was executed). Any element of the ForClause may be empty but the [semicolons](https://golang.google.cn/ref/spec#Semicolons) are required unless there is only a condition. If the condition is absent, it is equivalent to the boolean value `true`.

如果初始化语句不为空，则会在第一次执行条件语句前执行一次，以后就再也不执行了；相对的，后置语句会在每次 for 的代码块执行完后都执行一次（只在代码块执行完以后才执行）。这种形式的 for 循环的任何一个部分（初始语句、条件语句、后置语句）都可以省略，但是除非只有一个条件语句，否则必须是两个分号。如果省略了条件语句，则等效传入了一个 true 值。

```go
for cond { S() }    is the same as    for ; cond ; { S() }
for      { S() }    is the same as    for true     { S() }
```

### For statements with `range` clause （”range“ 形式的 for 循环）

A "for" statement with a "range" clause iterates through all entries of an array, slice, string or map, or values received on a channel. For each entry it assigns *iteration values* to corresponding *iteration variables* if present and then executes the block.

”range” 形式的 “for” 循环可以迭代遍历 数组、切片、字符串、map 映射 或 信道中的所有元素。对每个元素来说，如果存在则会被赋值到迭代变量上，然后可以在代码块中使用这个变量。

```go
RangeClause = [ ExpressionList "=" | IdentifierList ":=" ] "range" Expression .
```

The expression on the right in the "range" clause is called the *range expression*, which may be an array, pointer to an array, slice, string, map, or channel permitting [receive operations](https://golang.google.cn/ref/spec#Receive_operator). As with an assignment, if present the operands on the left must be [addressable](https://golang.google.cn/ref/spec#Address_operators) or map index expressions; they denote the iteration variables. If the range expression is a channel, at most one iteration variable is permitted, otherwise there may be up to two. If the last iteration variable is the [blank identifier](https://golang.google.cn/ref/spec#Blank_identifier), the range clause is equivalent to the same clause without that identifier.

在 “range” 语句中，右边的表达式被称为 “range 表达式”，它可能是一个数组，一个数组指针，一个切片，一个字符串，一个映射 map 或一个允许接收的信道。如果 range 语句中是一个赋值语句，则左边的操作数必须可以取址，或者是一个 map 的索引表达式，它们表示迭代的变量。如果 range 表达式是一个信道，至少可以允许赋值一个变量，最多可以赋值两个变量。如果最后一个迭代变量是空标识符，此时 range 语句等效于只有一个标识符的语法。

The range expression `x` is evaluated once before beginning the loop, with one exception: if at most one iteration variable is present and `len(x)` is [constant](https://golang.google.cn/ref/spec#Length_and_capacity), the range expression is not evaluated.

range 语句中的 `x` 会在循环开始前进行一次计算，除非下面这种情况：只有一个迭代变量且 `len(x)` 是一个常量，则 range 表达式不会执行。（不会对 数组、数组指针、切片中的元素进行获取）

Function calls on the left are evaluated once per iteration. For each iteration, iteration values are produced as follows if the respective iteration variables are present:

左侧表达式中的函数调用会在每一次循环开始前进行调用。对于每次迭代，如果相应的迭代变量存在，那么产生的值如下：

```go
Range expression                          1st value          2nd value

array or slice  a  [n]E, *[n]E, or []E    index    i  int    a[i]       E
string          s  string type            index    i  int    see below  rune
map             m  map[K]V                key      k  K      m[k]       V
channel         c  chan E, <-chan E       element  e  E
```

1. For an array, pointer to array, or slice value `a`, the index iteration values are produced in increasing order, starting at element index 0. If at most one iteration variable is present, the range loop produces iteration values from 0 up to `len(a)-1` and does not index into the array or slice itself. For a `nil` slice, the number of iterations is 0. 【对于数组、数组指针或切片的值 `a`，索引迭代值从索引 0 开始升序生成。如果至多只有一个迭代变量存在，此时 range 循环会产生一个从 0 到 `len(a)-1` 的值，不会生成索引对应的元素的值。对于值为 nil 的切片来说，迭代的数量是 0 次。
2. For a string value, the "range" clause iterates over the Unicode code points in the string starting at byte index 0. On successive iterations, the index value will be the index of the first byte of successive UTF-8-encoded code points in the string, and the second value, of type `rune`, will be the value of the corresponding code point. If the iteration encounters an invalid UTF-8 sequence, the second value will be `0xFFFD`, the Unicode replacement character, and the next iteration will advance a single byte in the string. 【对于字符串类型的值来说， range 语法会在 Unicode 的字节码上进行迭代，且从第 0 字节开始迭代。在连续的迭代过程中，索引的值会是 UTF-8 编码的字节码的第一个字节的索引，第二个值的类型为 `rune`，值是相对应的字节码。如果遇到了一个非 UTF-8 的序列，此时第二个值是 `0xFFFD`，也就是 Unicode 中的替代字符，这种情况下，然后前进一个字节进行下一次的循环。
3. The iteration order over maps is not specified and is not guaranteed to be the same from one iteration to the next. If a map entry that has not yet been reached is removed during iteration, the corresponding iteration value will not be produced. If a map entry is created during iteration, that entry may be produced during the iteration or may be skipped. The choice may vary for each entry created and from one iteration to the next. If the map is `nil`, the number of iterations is 0.【对于映射 map 的迭代顺序，可能这一次的迭代顺序和下一次的迭代顺序是不同的。如果一个未迭代到的 map 的键值在迭代的过程中被移除了，那么相关的迭代值就不会产生了。如果一个键值是在迭代的过程中增加的，那么这个键值可能被迭代到也可能被跳过。需要再次说明，从一个迭代的值到下一个迭代的值的选择是随机的。如果 map 的值是 nil，则迭代 0 次。
4. For channels, the iteration values produced are the successive values sent on the channel until the channel is [closed](https://golang.google.cn/ref/spec#Close). If the channel is `nil`, the range expression blocks forever. 【 对于信道来说，迭代的值是发送到信道中的值，且顺序不变。直到信道关闭，否则会一直尝试接收值并进行迭代。如果信道的值是 nil，range 表达式会一直阻塞。


The iteration values are assigned to the respective iteration variables as in an [assignment statement](https://golang.google.cn/ref/spec#Assignments).

迭代的值按照《赋值语句》里将的规则一次赋值给对应的迭代变量。

The iteration variables may be declared by the "range" clause using a form of [short variable declaration](https://golang.google.cn/ref/spec#Short_variable_declarations) (`:=`). In this case their types are set to the types of the respective iteration values and their [scope](https://golang.google.cn/ref/spec#Declarations_and_scope) is the block of the "for" statement; they are re-used in each iteration. If the iteration variables are declared outside the "for" statement, after execution their values will be those of the last iteration.

迭代变量可能通过变量的短声明方式在 range 的语句中声明产生。这种情况下，他们的类型会被设置为迭代得到的值的类型，且这些变量的作用域是整个 for 循环；需要注意的是，这些变量会在每次循环的过程中被重复利用（多线程时注意，有坑)。如果一个迭代变量在 for 循环之外声明，在 for 循环执行结束后整个变量会保留最后一次的迭代值。

```go
var testdata *struct {
	a *[7]int
}
for i, _ := range testdata.a {
	// testdata.a is never evaluated; len(testdata.a) is constant
	// i ranges from 0 to 6
	f(i)
}

var a [10]string
for i, s := range a {
	// type of i is int
	// type of s is string
	// s == a[i]
	g(i, s)
}

var key string
var val interface {}  // element type of m is assignable to val
m := map[string]int{"mon":0, "tue":1, "wed":2, "thu":3, "fri":4, "sat":5, "sun":6}
for key, val = range m {
	h(key, val)
}
// key == last map key encountered in iteration
// val == map[key]

var ch chan Work = producer()
for w := range ch {
	doWork(w)
}

// empty a channel
for range ch {}
```

## Go statements（go 语句）

A "go" statement starts the execution of a function call as an independent concurrent thread of control, or *goroutine*, within the same address space.

`go` 语句会使一个函数调用在一个独立的线程中进行调用，整个独立的线程和当前线程有相同的地址空间。

```go
GoStmt = "go" Expression .
```

The expression must be a function or method call; it cannot be parenthesized. Calls of built-in functions are restricted as for [expression statements](https://golang.google.cn/ref/spec#Expression_statements).

go 语句中的表达式必须是一个函数调用或者方法调用；它不能被括号括起来。调用内建的函数被限制只能是表达式语句。

The function value and parameters are [evaluated as usual](https://golang.google.cn/ref/spec#Calls) in the calling goroutine, but unlike with a regular call, program execution does not wait for the invoked function to complete. Instead, the function begins executing independently in a new goroutine. When the function terminates, its goroutine also terminates. If the function has any return values, they are discarded when the function completes.

在调用的 goroutine 协程中，函数值以及参数会像平时那样调用；和普通调用不同的是，主程序不会主动等待被调起的函数是否结束。相反的，函数在一个新的协程中独立地运行。当函数终止的时候，它的 goroutine 也终止了。如果函数有返回值，在函数结束后这个值会被丢弃。

```go
go Server()
go func(ch chan<- bool) { for { sleep(10); ch <- true }} (c)
```

## Select statements（选择语句）

A "select" statement chooses which of a set of possible [send](https://golang.google.cn/ref/spec#Send_statements) or [receive](https://golang.google.cn/ref/spec#Receive_operator) operations will proceed. It looks similar to a ["switch"](https://golang.google.cn/ref/spec#Switch_statements) statement but with the cases all referring to communication operations.

一个选择语句会监听任何一个可能的发送或接收操作，它和 switch 语句有点类似，不过 select 的所有 case 都是和信道的通信有关。

```go
SelectStmt = "select" "{" { CommClause } "}" .
CommClause = CommCase ":" StatementList .
CommCase   = "case" ( SendStmt | RecvStmt ) | "default" .
RecvStmt   = [ ExpressionList "=" | IdentifierList ":=" ] RecvExpr .
RecvExpr   = Expression .
```

A case with a RecvStmt may assign the result of a RecvExpr to one or two variables, which may be declared using a [short variable declaration](https://golang.google.cn/ref/spec#Short_variable_declarations). The RecvExpr must be a (possibly parenthesized) receive operation. There can be at most one default case and it may appear anywhere in the list of cases.

如果一个 case 是 RecvStmt 样式的，则可能会把 RecvExpr 返回的值赋值给 1 个或者 2 个变量，这两个变量可以由变量的短声明方式声明。 RecvExpr 必须是一个接收的操作（可能需要括号括起来）。像 switch 中一个，select 中可以存在一个 default 分支，且可以在任何位置声明。

Execution of a "select" statement proceeds in several steps:

select 语句的执行遵循几个步骤：

1. For all the cases in the statement, the channel operands of receive operations and the channel and right-hand-side expressions of send statements are evaluated exactly once, in source order, upon entering the "select" statement. The result is a set of channels to receive from or send to, and the corresponding values to send. Any side effects in that evaluation will occur irrespective of which (if any) communication operation is selected to proceed. Expressions on the left-hand side of a RecvStmt with a short variable declaration or assignment are not yet evaluated. 【对于语句中的所有case，在 select 语句正式执行前，信道的接收操作、信道、以及发送语句的右侧表达式都都会按照源码中的顺序依次运行一次。运算完得到的是将要接收信息或者将要发送信息的信道，当然还有需要发送的值。在这个运算过程中，无论选择哪种通信操作，如果有副作用的话都能感知到。此时，RecvStmt 中通过变量的短声明方式定义的左侧的变量尚未得到执行。
2. If one or more of the communications can proceed, a single one that can proceed is chosen via a uniform pseudo-random selection. Otherwise, if there is a default case, that case is chosen. If there is no default case, the "select" statement blocks until at least one of the communications can proceed.【如果有一个或者多个通信可以执行，此时会通过一个统一的伪随机的算法选择一个进行执行。如果没有通信可以执行，则会选择 default 语句块执行。如果没有 default 语句，此时 select 语句会阻塞，直到至少有一个通信可以发生。
3. Unless the selected case is the default case, the respective communication operation is executed.【除非是 default 的情况，否则相关的通信操作会得到执行。
4. If the selected case is a RecvStmt with a short variable declaration or an assignment, the left-hand side expressions are evaluated and the received value (or values) are assigned.【如果选中的情况是一个有遍历短声明方式的 RecvStmt， 或者是一个普通的赋值语句，此时左侧的表达式会被赋予接收到的值。
5. The statement list of the selected case is executed. 【选中的 case 的语句块得到执行。

Since communication on `nil` channels can never proceed, a select with only `nil` channels and no default case blocks forever.

由于在信道值为 nil 之上永远不可能有通信，如果一个 select 只有一个 nil 的信道且没有定义 default 的语句块，这个 select 会一直阻塞。

```go
var a []int
var c, c1, c2, c3, c4 chan int
var i1, i2 int
select {
case i1 = <-c1:
	print("received ", i1, " from c1\n")
case c2 <- i2:
	print("sent ", i2, " to c2\n")
case i3, ok := (<-c3):  // same as: i3, ok := <-c3
	if ok {
		print("received ", i3, " from c3\n")
	} else {
		print("c3 is closed\n")
	}
case a[f()] = <-c4:
	// same as:
	// case t := <-c4
	//	a[f()] = t
default:
	print("no communication\n")
}

for {  // send random sequence of bits to c
	select {
	case c <- 0:  // note: no statement, no fallthrough, no folding of cases
	case c <- 1:
	}
}

select {}  // block forever
```

## Return statements（返回语句）

A "return" statement in a function `F` terminates the execution of `F`, and optionally provides one or more result values. Any functions [deferred](https://golang.google.cn/ref/spec#Defer_statements) by `F` are executed before `F` returns to its caller.

函数 `F` 中的返回语句会终止 `F` 的执行，并且能可选地返回一个或多个返回值。在 `F` 中任何一个被 deferred 的函数都会在 `F` 返回给调用者之前进行执行。

```go
ReturnStmt = "return" [ ExpressionList ] .
```

In a function without a result type, a "return" statement must not specify any result values.

乳沟一个函数没有声明返回值，此时 return 语句则不能指定任何的返回值，否则会编译出错。

```go
func noResult() {
	return
}
```

There are three ways to return values from a function with a result type:

有三种方式可以给声明了返回值的函数返回对应的数值：

1. The return value or values may be explicitly listed in the "return" statement. Each expression must be single-valued and  assignable  to the corresponding element of the function's result type.【返回值或者返回值列表被显式地列在 return 语句的后面。每个表达式必须都是单值且可以赋给函数定义的返回值类型。

   ```go
   func simpleF() int {
   	return 2
   }
   
   func complexF1() (re float64, im float64) {
   	return -7.0, -4.0
   }
   ```

2. The expression list in the "return" statement may be a single call to a multi-valued function. The effect is as if each value returned from that function were assigned to a temporary variable with the type of the respective value, followed by a "return" statement listing these variables, at which point the rules of the previous case apply.【返回值语句的表达式可能是一个多值函数的调用。效果就变成了这样：每个从多值函数返回的值都被赋值给一个对应类型的临时变量，然后 return 语句会把这些变量放在表达式的地方，接下来就可以像第一种情况那样运行了。

   ```go
   func complexF2() (re float64, im float64) {
   	return complexF1()
   }
   ```

3. The expression list may be empty if the function's result type specifies names for its result parameters. The result parameters act as ordinary local variables and the function may assign values to them as necessary. The "return" statement returns the values of these variables.【如果函数返回值列表中预先定义了结果参数的名称，此时 return 表达式可以为空。这种情况下，返回参数会像普通的本地变量一样使用，函数可以给这些值赋值，最后 return 语句返回这些变量。

   ```go
   func complexF3() (re float64, im float64) {
   	re = 7.0
   	im = 4.0
   	return
   }
   
   func (devnull) Write(p []byte) (n int, _ error) {
   	n = len(p)
   	return
   }
   ```

Regardless of how they are declared, all the result values are initialized to the [zero values](https://golang.google.cn/ref/spec#The_zero_value) for their type upon entry to the function. A "return" statement that specifies results sets the result parameters before any deferred functions are executed.

不管返回列表如何声明，所有的返回值都会被初始化为他们的类型对应的零值。return 语句会在所有的 deferred 函数执行之前运行设置返回值。

Implementation restriction: A compiler may disallow an empty expression list in a "return" statement if a different entity (constant, type, or variable) with the same name as a result parameter is in [scope](https://golang.google.cn/ref/spec#Declarations_and_scope) at the place of the return.

实现限制：如果有一个不同的实体（常量、类型或者变量）在作用域内与定义的返回值名称一样，编译器可能不允许给 return 语句设置空表达式。否则会造成混淆。

```go
func f(n int) (res int, err error) {
	if _, err := f(n-1); err != nil {
		return  // invalid return statement: err is shadowed
	}
	return
}
```

## Break statements（break 语句）

A "break" statement terminates execution of the innermost ["for"](https://golang.google.cn/ref/spec#For_statements), ["switch"](https://golang.google.cn/ref/spec#Switch_statements), or ["select"](https://golang.google.cn/ref/spec#Select_statements) statement within the same function.

break 语句终止同一个函数调用中**最内部**的 ”for“、”switch“、”select“ 的执行。

```go
BreakStmt = "break" [ Label ] .
```

If there is a label, it must be that of an enclosing "for", "switch", or "select" statement, and that is the one whose execution terminates.

如果有标签，这个标签必须是包含 ”for“、”switch“、或”for“语句的标签，标签意味着对应语句的结束。

```go
OuterLoop:
	for i = 0; i < n; i++ {
		for j = 0; j < m; j++ {
			switch a[i][j] {
			case nil:
				state = Error
				break OuterLoop
			case item:
				state = Found
				break OuterLoop
			}
		}
	}
```

## Continue statements（Continue 语句）

A "continue" statement begins the next iteration of the innermost ["for" loop](https://golang.google.cn/ref/spec#For_statements) at its post statement. The "for" loop must be within the same function.

一个 continue 语句标明直接运行 for 循环的后置语句，然后马上开始最内层 for 循环的下一次迭代。当然，for 循环必须在同一个函数内，不能通过标签 continue 到另一个函数的 for 循环去。

```go
ContinueStmt = "continue" [ Label ] .
```

If there is a label, it must be that of an enclosing "for" statement, and that is the one whose execution advances.

如果有标签，标签必须是包含 for 语句，并且这个 for 语句就是当前正在执行的语句。

```go
RowLoop:
	for y, row := range rows {
		for x, data := range row {
			if data == endOfRow {
				continue RowLoop
			}
			row[x] = data + bias(x, y)
		}
	}
```

## Goto statements（goto 语句）

A "goto" statement transfers control to the statement with the corresponding label within the same function.

goto 语句可以在同一个函数中修改控制流到相应的标签。

```go
GotoStmt = "goto" Label .
```
```go
goto Error
```

Executing the "goto" statement must not cause any variables to come into [scope](https://golang.google.cn/ref/spec#Declarations_and_scope) that were not already in scope at the point of the goto. For instance, this example:

执行 goto 语句不允许引入之前未经声明的变量，比如下面的例子：

```go
	goto L  // BAD
	v := 3
L:
```

is erroneous because the jump to label `L` skips the creation of `v`.
因为跳转到标签 `L` 跳过了 `v` 的创建，因此会报错。

A "goto" statement outside a [block](https://golang.google.cn/ref/spec#Blocks) cannot jump to a label inside that block. For instance, this example:

某个代码块外面的 goto 语句不能跳转到它的内部，比如下面的例子：

```go
if n%2 == 1 {
	goto L1
}
for n > 0 {
	f()
	n--
L1:
	f()
	n--
}
```

is erroneous because the label `L1` is inside the "for" statement's block but the `goto` is not.

因为标签 `L1` 在 ”for“ 循环的内部，而 `goto` 不在，因此上面的代码会报错。

## Fallthrough statements（fallthrough 语句）

A "fallthrough" statement transfers control to the first statement of the next case clause in an [expression "switch" statement](https://golang.google.cn/ref/spec#Expression_switches). It may be used only as the final non-empty statement in such a clause.

fallthrough 语句用在 switch 语句汇总，会直接把控制流跳转到下一个 case 的代码块的第一句进行执行。fallthrough 只能放在对一个代码块的最后一句的位置（否则它后面的语句不会被执行，也就没有存在的意义）

```go
FallthroughStmt = "fallthrough" .
```

## Defer statements（defer 语句）

A "defer" statement invokes a function whose execution is deferred to the moment the surrounding function returns, either because the surrounding function executed a [return statement](https://golang.google.cn/ref/spec#Return_statements), reached the end of its [function body](https://golang.google.cn/ref/spec#Function_declarations), or because the corresponding goroutine is [panicking](https://golang.google.cn/ref/spec#Handling_panics).

defer 语句触发那些被推迟执行的函数的调用，直到函数返回；其中函数可能是因为 return 语句返回的，也可能是因为执行到了函数体的最后，甚至可能是因为 panic 退出的。

```go
DeferStmt = "defer" Expression .
```

The expression must be a function or method call; it cannot be parenthesized. Calls of built-in functions are restricted as for [expression statements](https://golang.google.cn/ref/spec#Expression_statements).

defer 的表达式必须是一个函数或者方法调用，它不能是被括起来的。如果调用的是一个内建函数，需要满足 表达式语句 中提到的限制。

Each time a "defer" statement executes, the function value and parameters to the call are [evaluated as usual](https://golang.google.cn/ref/spec#Calls) and saved anew but the actual function is not invoked. Instead, deferred functions are invoked immediately before the surrounding function returns, in the reverse order they were deferred. That is, if the surrounding function returns through an explicit [return statement](https://golang.google.cn/ref/spec#Return_statements), deferred functions are executed *after* any result parameters are set by that return statement but *before* the function returns to its caller. If a deferred function value evaluates to `nil`, execution [panics](https://golang.google.cn/ref/spec#Handling_panics) when the function is invoked, not when the "defer" statement is executed.

每当 defer 语句执行的时候，函数值及其参数都会像一般的情况那样运行且被保存，只是这个函数并不会触发调用。相反，被延迟的函数会在调用它的函数返回到调用者之前，以一种先进后执行的顺序依次执行。也就是说，如果周围函数通过一个显式的 return 语句返回，被推迟执行的函数被在所有结果参数被设置**之后**，在函数返回给其调用者**之前**进行调用。如果被推迟执行的函数的值是 nil，则会在它被触发运行的时候发生运行时错误，这种情况是不会再声明 defer 的时候报错的。

For instance, if the deferred function is a [function literal](https://golang.google.cn/ref/spec#Function_literals) and the surrounding function has [named result parameters](https://golang.google.cn/ref/spec#Function_types) that are in scope within the literal, the deferred function may access and modify the result parameters before they are returned. If the deferred function has any return values, they are discarded when the function completes. (See also the section on [handling panics](https://golang.google.cn/ref/spec#Handling_panics).)

比如，如果被推迟的函数是一个函数声明文本，且周围的函数有命名的结果列表，因为这些参数在被推迟执行的函数的作用域内，因此被推迟的函数是可以**在周围函数返回给调用者之前**访问并修改结果参数。如果被推迟的函数有任何返回值，他们会在函数返回前辈丢弃。（可以查看《panics 处理》一节的内容）

```go
lock(l)
defer unlock(l)  // unlocking happens before surrounding function returns

// prints 3 2 1 0 before surrounding function returns
for i := 0; i <= 3; i++ {
	defer fmt.Print(i)
}

// f returns 42
func f() (result int) {
	defer func() {
		// result is accessed after it was set to 6 by the return statement
		result *= 7
	}()
	return 6
}
```

# Built-in functions

Built-in functions are [predeclared](https://golang.google.cn/ref/spec#Predeclared_identifiers). They are called like any other function but some of them accept a type instead of an expression as the first argument.

内建函数是 Go 语言预先声明的函数，他们可以像其他函数那样调用，只不过一些内建函数会要求第一个参数必须是一个类型而不是一个表达式。

The built-in functions do not have standard Go types, so they can only appear in [call expressions](https://golang.google.cn/ref/spec#Calls); they cannot be used as function values.

内建函数没有标准的 Go 类型，因此他们可以在 call 中出现；另外需要注意的是，他们不能用来作为函数值。

## Close（关闭）

For a channel `c`, the built-in function `close(c)` records that no more values will be sent on the channel. It is an error if `c` is a receive-only channel. Sending to or closing a closed channel causes a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics). Closing the nil channel also causes a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics). After calling `close`, and after any previously sent values have been received, receive operations will return the zero value for the channel's type without blocking. The multi-valued [receive operation](https://golang.google.cn/ref/spec#Receive_operator) returns a received value along with an indication of whether the channel is closed.

对于信道 `c`，内建函数 `close` 用来标注当前信道上不会有其他值被发送了。如果 `c` 是一个只读的信道，关闭是会报错。向一个已经关闭的信道发送数据或者关闭一个已经关闭的信道都会导致一个运行时错误。调用了 close 以后，信道（带缓存）中已存的值依然可以继续被接收，这些值被接收完以后，接收操作会持续得到信道元素类型的零值，此时信道就不会阻塞了。如果接收操作左侧是两个值，其中一个用来接收信道中的值，另一个则用来表征信道是不是关闭了。

## Length and capacity（长度与容量）

The built-in functions `len` and `cap` take arguments of various types and return a result of type `int`. The implementation guarantees that the result always fits into an `int`.

内建的函数 `len` 和 `cap` 可以传入多个类型的值，然后返回一个整形的数值。Go 的实现保证了结果的返回总是整形的。

```go
Call      Argument type    Result

len(s)    string type      string length in bytes
          [n]T, *[n]T      array length (== n)
          []T              slice length
          map[K]T          map length (number of defined keys)
          chan T           number of elements queued in channel buffer

cap(s)    [n]T, *[n]T      array length (== n)
          []T              slice capacity
          chan T           channel buffer capacity
```

The capacity of a slice is the number of elements for which there is space allocated in the underlying array. At any time the following relationship holds:

切片的容量是其底层数组能放置的元素的个数，任何时候满足下面的关系：`0 <= len(s) <= cap(s)`。

```go
0 <= len(s) <= cap(s)
```

The length of a `nil` slice, map or channel is 0. The capacity of a `nil` slice or channel is 0.

值为 nil 的切片、映射 map 和信道的长度都是 0， 值为 nil 的切片和信道的容量是 0。

The expression `len(s)` is [constant](https://golang.google.cn/ref/spec#Constants) if `s` is a string constant. The expressions `len(s)` and `cap(s)` are constants if the type of `s` is an array or pointer to an array and the expression `s` does not contain [channel receives](https://golang.google.cn/ref/spec#Receive_operator) or (non-constant) [function calls](https://golang.google.cn/ref/spec#Calls); in this case `s` is not evaluated. Otherwise, invocations of `len` and `cap` are not constant and `s` is evaluated.

如果 `s` 是一个字符串，此时 `len(s)` 是一个常量。当 `s` 表达式的类型是一个数组或者数组指针，且表达式 `s` 不包含接收信道也不包含**非常量的函数调用**时，此时 `len(s)` 和 `cap(s)` 都是常量；这种情况下 `s` 是不会进行运算的。否则，`len` 和 `cap` 都不是常量且 `s` 会被运算。

```go
const (
	c1 = imag(2i)                    // imag(2i) = 2.0 is a constant
	c2 = len([10]float64{2})         // [10]float64{2} contains no function calls
	c3 = len([10]float64{c1})        // [10]float64{c1} contains no function calls
	c4 = len([10]float64{imag(2i)})  // imag(2i) is a constant and no function call is issued
	c5 = len([10]float64{imag(z)})   // invalid: imag(z) is a (non-constant) function call
)
var z complex128
```

## Allocation（内存分配）

The built-in function `new` takes a type `T`, allocates storage for a [variable](https://golang.google.cn/ref/spec#Variables) of that type at run time, and returns a value of type `*T` [pointing](https://golang.google.cn/ref/spec#Pointer_types) to it. The variable is initialized as described in the section on [initial values](https://golang.google.cn/ref/spec#The_zero_value).

内建函数 `new` 可以在运行时为类型为 `T` 的变量分配内存，并返回一个类型为 `*T` 的值。变量会被初始化为类型 `T` 的零值。

```go
new(T)
```

For instance

```go
type S struct { a int; b float64 }
new(S)
```

allocates storage for a variable of type `S`, initializes it (`a=0, b=0.0`), and returns a value of type `*S` containing the address of the location.

比如上面的语句给类型为 `S` 的变量分配了内存值，并把它初始化为 `a=0, b=0.0`，最后返回了一个类型为 `*S` 的地址。

## Making slices, maps and channels（创建切片、map 和信道）

The built-in function `make` takes a type `T`, which must be a slice, map or channel type, optionally followed by a type-specific list of expressions. It returns a value of type `T` (not `*T`). The memory is initialized as described in the section on [initial values](https://golang.google.cn/ref/spec#The_zero_value).

内建函数 `make` 可以传入类型 `T`，类型 `T` 可以为切片、map 或者信道，并且还可以根据各个不同的类型传入类型相关的参数列表。`make` 会返回类型为 `T` （区别于 `*T`）的值。内存会被初始化为对应类型的零值。

```go
Call             Type T     Result

make(T, n)       slice      slice of type T with length n and capacity n
make(T, n, m)    slice      slice of type T with length n and capacity m

make(T)          map        map of type T
make(T, n)       map        map of type T with initial space for approximately n elements

make(T)          channel    unbuffered channel of type T
make(T, n)       channel    buffered channel of type T, buffer size n
```

Each of the size arguments `n` and `m` must be of integer type or an untyped [constant](https://golang.google.cn/ref/spec#Constants). A constant size argument must be non-negative and [representable](https://golang.google.cn/ref/spec#Representability) by a value of type `int`; if it is an untyped constant it is given type `int`. If both `n` and `m` are provided and are constant, then `n` must be no larger than `m`. If `n` is negative or larger than `m` at run time, a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) occurs.

`make` 声明中的尺寸参数 `n` 和 `m` 必须是整形数字，或者是未定型的常量。如果是一个常量的话，必须是一个非负整形常量；如果传入的是一个未定型的常量，这个常量被默认是 `int` 类型。如果 `n` 和 `m` 都存在且都是常量，则 `n` 必须小于等于 `m`。如果 `n` 在运行时发现是负数或者比 `m` 大，则会产生运行时错误。

```go
s := make([]int, 10, 100)       // slice with len(s) == 10, cap(s) == 100
s := make([]int, 1e3)           // slice with len(s) == cap(s) == 1000
s := make([]int, 1<<63)         // illegal: len(s) is not representable by a value of type int
s := make([]int, 10, 0)         // illegal: len(s) > cap(s)
c := make(chan int, 10)         // channel with a buffer size of 10
m := make(map[string]int, 100)  // map with initial space for approximately 100 elements
```

Calling `make` with a map type and size hint `n` will create a map with initial space to hold `n` map elements. The precise behavior is implementation-dependent.

调用 `make` 方法创建一个 map 类型并传入 `n` 会创建一个能放 `n` 个映射元素值的 map。具体的行为会依赖具体的实现。

## Appending to and copying slices（增加、拷贝切片）

The built-in functions `append` and `copy` assist in common slice operations. For both functions, the result is independent of whether the memory referenced by the arguments overlaps.

内建函数 `append` 和 `copy` 协助一些基础的切片操作。这两个函数，其结果与参数引用的内存是否重叠没有关系。

The [variadic](https://golang.google.cn/ref/spec#Function_types) function `append` appends zero or more values `x` to `s` of type `S`, which must be a slice type, and returns the resulting slice, also of type `S`. The values `x` are passed to a parameter of type `...T` where `T` is the [element type](https://golang.google.cn/ref/spec#Slice_types) of `S` and the respective [parameter passing rules](https://golang.google.cn/ref/spec#Passing_arguments_to_..._parameters) apply. As a special case, `append` also accepts a first argument assignable to type `[]byte` with a second argument of string type followed by `...`. This form appends the bytes of the string.

不定参数函数 `append` 追加 0 个或者多个 `x` 到类型为 `S` 的 `s`，其中类型 `S` 必须是一个切片类型。`append` 返回一个类型同为 `S` 的切片。值 `x` 被传递给一个类型为 `...T` 的参数，其中 `T` 是 `S` 的元素类型，并且遵从参数传递规则。

作为一种特例，`append` 可以接受一个可以赋值给 `[]byte` 的参数，然后跟着一个有 `...` 后缀的字符串作为第二个参数。这种格式会把字符串的字节传给第一个参数。

```go
append(s S, x ...T) S  // T is the element type of S
```

If the capacity of `s` is not large enough to fit the additional values, `append` allocates a new, sufficiently large underlying array that fits both the existing slice elements and the additional values. Otherwise, `append` re-uses the underlying array.

如果 `s` 的容量不能够保存添加的值，`append` 会分配一个新的且足够大的底层数组，从而把已存的切片元素和新添加的元素保存起来。否则，`append` 会重复使用底层的数组。

```go
s0 := []int{0, 0}
s1 := append(s0, 2)                // append a single element     s1 == []int{0, 0, 2}
s2 := append(s1, 3, 5, 7)          // append multiple elements    s2 == []int{0, 0, 2, 3, 5, 7}
s3 := append(s2, s0...)            // append a slice              s3 == []int{0, 0, 2, 3, 5, 7, 0, 0}
s4 := append(s3[3:6], s3[2:]...)   // append overlapping slice    s4 == []int{3, 5, 7, 2, 3, 5, 7, 0, 0}

var t []interface{}
t = append(t, 42, 3.1415, "foo")   //                             t == []interface{}{42, 3.1415, "foo"}

var b []byte
b = append(b, "bar"...)            // append string contents      b == []byte{'b', 'a', 'r' }
```

The function `copy` copies slice elements from a source `src` to a destination `dst` and returns the number of elements copied. Both arguments must have [identical](https://golang.google.cn/ref/spec#Type_identity) element type `T` and must be [assignable](https://golang.google.cn/ref/spec#Assignability) to a slice of type `[]T`. The number of elements copied is the minimum of `len(src)` and `len(dst)`. As a special case, `copy` also accepts a destination argument assignable to type `[]byte` with a source argument of a string type. This form copies the bytes from the string into the byte slice.

函数 `copy` 会从 `src` 作为源把切片中的元素拷贝到目的切片 `dst` 中，并且会返回拷贝成功的元素个数。传入 `copy` 的两个参数必须有一样的元素类型 `T`，且必须都可以赋值给类型为 `[]T` 的切片。拷贝的元素的数目是 `len(src)` 和 `len(dst)` 的最小值。

作为一种特例，`copy` 可以接收类型为 `[]byte` 的目的参数，和类型为字符串的源参数。这种情况下，会把字符串的字节序列拷贝到目的切片中。

```go
copy(dst, src []T) int
copy(dst []byte, src string) int
```

Examples:

```go
var a = [...]int{0, 1, 2, 3, 4, 5, 6, 7}
var s = make([]int, 6)
var b = make([]byte, 5)
n1 := copy(s, a[0:])            // n1 == 6, s == []int{0, 1, 2, 3, 4, 5}
n2 := copy(s, s[2:])            // n2 == 4, s == []int{2, 3, 4, 5, 4, 5}
n3 := copy(b, "Hello, World!")  // n3 == 5, b == []byte("Hello")
```

## Deletion of map elements（删除 map 中的元素）

The built-in function `delete` removes the element with key `k` from a [map](https://golang.google.cn/ref/spec#Map_types) `m`. The type of `k` must be [assignable](https://golang.google.cn/ref/spec#Assignability) to the key type of `m`.

内建函数 `delete` 可以移除 map 中键为 `k` 的元素。其中 `k` 必须可以赋值给 `m` 的键类型。

```go
delete(m, k)  // remove element m[k] from map m
```

If the map `m` is `nil` or the element `m[k]` does not exist, `delete` is a no-op.

如果映射 `m` 的值是 nil，或者元素 `m[k]` 不存在，此时 `delete` 不会做任何操作。

## Manipulating complex numbers（操作复数）

Three functions assemble and disassemble complex numbers. The built-in function `complex` constructs a complex value from a floating-point real and imaginary part, while `real` and `imag` extract the real and imaginary parts of a complex value.

三个内建函数可以组装或者解构复数。内建函数 `complex` 可以从一个复数实部和虚部构造一个复数，内建函数 `real` 和 `imag` 则可以抽取复数的实部和虚部。

```go
complex(realPart, imaginaryPart floatT) complexT
real(complexT) floatT
imag(complexT) floatT
```

The type of the arguments and return value correspond. For `complex`, the two arguments must be of the same floating-point type and the return type is the complex type with the corresponding floating-point constituents: `complex64` for `float32` arguments, and `complex128` for `float64` arguments. If one of the arguments evaluates to an untyped constant, it is first implicitly [converted](https://golang.google.cn/ref/spec#Conversions) to the type of the other argument. If both arguments evaluate to untyped constants, they must be non-complex numbers or their imaginary parts must be zero, and the return value of the function is an untyped complex constant.

参数的类型和返回值的类型是相关的。对于 `complex` 来说，两个参数必须是一样的浮点类型，返回值和传入的浮点类型相关：如果传入的参数是 `float32` 类型则复数是 `complex64` 类型；如果传入的参数是 `float64` 类型则复数是 `complex128` 类型。如果其中一个参数运算后是一个未定型的常数，它首先会被显式转换成另一个参数的类型。如果两个参数运算后都是未定型的常数，他们必须都是非负数的数字，或者这两个参数的虚部都是 0，此时 `complex` 返回的是一个未定型的复数常量。

For `real` and `imag`, the argument must be of complex type, and the return type is the corresponding floating-point type: `float32` for a `complex64` argument, and `float64` for a `complex128` argument. If the argument evaluates to an untyped constant, it must be a number, and the return value of the function is an untyped floating-point constant.

对于 `real` 和 `imag` 来说，参数必须是一个复数类型，返回值是相对应的浮点类型：如果复数是 `complex64`，返回值是 `float32` 类型；如果复数是 `complex128` 类型，返回值是 `float64` 类型。如果参数运算后是一个未定型的常数，这个数必须是一个数值，`real` 和 `imag` 返回的是未定型的浮点数常量。

The `real` and `imag` functions together form the inverse of `complex`, so for a value `z` of a complex type `Z`, `z == Z(complex(real(z), imag(z)))`.

`real` 和 `imag` 两个函数一起组成了 `complex` 的反向功能，因此对于类型为 `Z` 的 `z` 值，有 `z == Z(complex(real(z), imag(z)))`。

If the operands of these functions are all constants, the return value is a constant.

这三个函数如果他们的操作数都是常量，那么他们的返回值也是常量。

```go
var a = complex(2, -2)             // complex128
const b = complex(1.0, -1.4)       // untyped complex constant 1 - 1.4i
x := float32(math.Cos(math.Pi/2))  // float32
var c64 = complex(5, -x)           // complex64
var s int = complex(1, 0)          // untyped complex constant 1 + 0i can be converted to int
_ = complex(1, 2<<s)               // illegal: 2 assumes floating-point type, cannot shift
var rl = real(c64)                 // float32
var im = imag(a)                   // float64
const c = imag(b)                  // untyped constant -1.4
_ = imag(3 << s)                   // illegal: 3 assumes complex type, cannot shift
```

## Handling panics（处理运行时错误）

Two built-in functions, `panic` and `recover`, assist in reporting and handling [run-time panics](https://golang.google.cn/ref/spec#Run_time_panics) and program-defined error conditions.

两个内建函数 `panic` 和 `recover`，二者互相辅助来抛出并捕获运行时错误和程序定义的错误条件。
```go
func panic(interface{})
func recover() interface{}
```

While executing a function `F`, an explicit call to `panic` or a [run-time panic](https://golang.google.cn/ref/spec#Run_time_panics) terminates the execution of `F`. Any functions [deferred](https://golang.google.cn/ref/spec#Defer_statements) by `F` are then executed as usual. Next, any deferred functions run by `F's` caller are run, and so on up to any deferred by the top-level function in the executing goroutine. At that point, the program is terminated and the error condition is reported, including the value of the argument to `panic`. This termination sequence is called *panicking*.

当运行函数 `F` 时，显式调用 `panic` 或者运行时错误都会终止 `F` 的执行。任何被 `F` 推迟执行的函数会像平时一样执行。然后，`F` 的调用方中被推迟执行的函数依次执行，如此往复，直到当前 goroutine 中最高层函数的推迟函数执行完为止。程序在那个特别的位置终止，并且错误条件被抛出，里面包含传递给 `panic` 的参数。上面讲到的这种终止顺序被称为 **Panicking**。

```go
panic(42)
panic("unreachable")
panic(Error("cannot parse"))
```

The `recover` function allows a program to manage behavior of a panicking goroutine. Suppose a function `G` defers a function `D` that calls `recover` and a panic occurs in a function on the same goroutine in which `G` is executing. When the running of deferred functions reaches `D`, the return value of `D`'s call to `recover` will be the value passed to the call of `panic`. If `D` returns normally, without starting a new `panic`, the panicking sequence stops. In that case, the state of functions called between `G` and the call to `panic` is discarded, and normal execution resumes. Any functions deferred by `G` before `D` are then run and `G`'s execution terminates by returning to its caller.

`recover` 函数允许程序处理 panicking 的协程。假设有一个函数 `G` 有个推迟执行的函数 `D`，在 `D` 中调用了 `recover` 函数，这个时候当前协程的函数 `G` 发生了 panic。当被推迟运行的函数到达 `D` 的时候，调用 `recover` 的返回值是传递给 `panic` 的参数。如果 `D` 像平时一样返回，期间并没有开始新的 `panic`，此时 panicking 就到此为止了。这种情况下，在 `G` 和 panic 之间的函数的状态被丢弃，程序开始正常运行。其他在 `D` 前面被 `G` 推迟执行的函数会依次执行，且`G`会正常返回到它的调用者。

The return value of `recover` is `nil` if any of the following conditions holds:

如果下面任何一个条件得到满足，`recover` 的返回值就是 `nil`：

- `panic`'s argument was `nil`;【panic 的参数是 nil；
- the goroutine is not panicking;【goroutine 没有发生 panicking；
- `recover` was not called directly by a deferred function.【 `recover` 没有被推迟的函数直接调用。

The `protect` function in the example below invokes the function argument `g` and protects callers from run-time panics raised by `g`.

下面例子中的 `protect` 函数触发调用了参数中的函数 `g`，并且保护由 `g` 引起的运行时的 panic。

```go
func protect(g func()) {
	defer func() {
		log.Println("done")  // Println executes normally even if there is a panic
		if x := recover(); x != nil {
			log.Printf("run time panic: %v", x)
		}
	}()
	log.Println("start")
	g()
}
```

## Bootstrapping（调试）

Current implementations provide several built-in functions useful during bootstrapping. These functions are documented for completeness but are not guaranteed to stay in the language. They do not return a result.

当前 Go 的实现中提供了几个内建的函数可以在调试时使用。记录这些文档时为了完整性，但是并不能保证它们会留在语言中。

```go
Function   Behavior

print      prints all arguments; formatting of arguments is implementation-specific
println    like print but prints spaces between arguments and a newline at the end
```

Implementation restriction: `print` and `println` need not accept arbitrary argument types, but printing of boolean, numeric, and string [types](https://golang.google.cn/ref/spec#Types) must be supported.

实现限制：`print` 和 `println` 不必接收任何的参数，但是最好能打印布尔、数值和字符串类型。


# Packages （包）

Go programs are constructed by linking together *packages*. A package in turn is constructed from one or more source files that together declare constants, types, variables and functions belonging to the package and which are accessible in all files of the same package. Those elements may be [exported](https://golang.google.cn/ref/spec#Exported_identifiers) and used in another package.

Go 语言通过链接包来构建程序。一个包又有一个或多个源文件组成，这些文件中声明属于这个包的常量、类型、变量和函数等，并在这个包的所有文件中共同使用。当然，元素是可导出的，也可以在其他的包里使用。

## Source file organization（源文件的结构）

Each source file consists of a package clause defining the package to which it belongs, followed by a possibly empty set of import declarations that declare packages whose contents it wishes to use, followed by a possibly empty set of declarations of functions, types, variables, and constants.

每个源文件都包含包语法从而定义它属于哪个包，然后是引入声明（可能为空结合）用来指明希望使用的第三方包，接着就是一些函数的声明、类型的声明、变量和常量的声明。

```go
SourceFile       = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .
```

## Package clause（包语法）

A package clause begins each source file and defines the package to which the file belongs.

每个源文件都必须由包语法开始，定义这个文件属于哪个包。

```go
PackageClause  = "package" PackageName .
PackageName    = identifier .
```

The PackageName must not be the [blank identifier](https://golang.google.cn/ref/spec#Blank_identifier).

需要强调一下，包名不能是空标识符。

```go
package math
```

A set of files sharing the same PackageName form the implementation of a package. An implementation may require that all source files for a package inhabit the same directory.

一组共享同样的包名的文件构建了整个包。当前 Go 语言的实现会要求同一个包的所有文件在同一个目录下面。不同的包不同的目录，而同一个目录里只能声明一个包。在实际开发过程中其实可以发现同一个目录里可以定义两个相关联的包，一个是 “package 包名”，另一个是“package 包名_test”。

## Import declarations（导入声明）

An import declaration states that the source file containing the declaration depends on functionality of the *imported* package ([§Program initialization and execution](https://golang.google.cn/ref/spec#Program_initialization_and_execution)) and enables access to [exported](https://golang.google.cn/ref/spec#Exported_identifiers) identifiers of that package. The import names an identifier (PackageName) to be used for access and an ImportPath that specifies the package to be imported.

导入语法声明当前文件依赖被导入的包的功能，并可以访问这些第三方包的导出变量。导入可以制定一个可选的包名用来在源文件中指代对应的包，然后把包路径写在 import 后面就可以了。

```go
ImportDecl       = "import" ( ImportSpec | "(" { ImportSpec ";" } ")" ) .
ImportSpec       = [ "." | PackageName ] ImportPath .
ImportPath       = string_lit .
```

The PackageName is used in [qualified identifiers](https://golang.google.cn/ref/spec#Qualified_identifiers) to access exported identifiers of the package within the importing source file. It is declared in the [file block](https://golang.google.cn/ref/spec#Blocks). If the PackageName is omitted, it defaults to the identifier specified in the [package clause](https://golang.google.cn/ref/spec#Package_clause) of the imported package. If an explicit period (`.`) appears instead of a name, all the package's exported identifiers declared in that package's [package block](https://golang.google.cn/ref/spec#Blocks) will be declared in the importing source file's file block and must be accessed without a qualifier.

包名作为“有效标识符”的一部分使用，通过它来访问相应包的导出标识符。这些导出的标识符是指在文件块范围内声明的那些元素。导入包的时候如果包名被省略了，默认采用这个包在 package 语法中定义的那个名称。如果 import 时包名的位置使用的是一个点（`.`）而不是一个有效的名称，相应被引入的包的所有导出的标识都能够且必须只能够直接被调用。

The interpretation of the ImportPath is implementation-dependent but it is typically a substring of the full file name of the compiled package and may be relative to a repository of installed packages.

导入路径的解析根据具体的实现有所不同，但一般是包的全路径名（带路径的那种）的一个子字符串，当然也可能是按照这个包的仓库的相对路径。（建议都以全路径名的形式来开发，现在很多开源库都是如此，引入方便也好维护）

Implementation restriction: A compiler may restrict ImportPaths to non-empty strings using only characters belonging to [Unicode's](https://www.unicode.org/versions/Unicode6.3.0/) L, M, N, P, and S general categories (the Graphic characters without spaces) and may also exclude the characters `!"#$%&'()*,:;<=>?[\]^`{|}` and the Unicode replacement character U+FFFD.

实践限制：编译器可能会限制导入路径是一个非空字符串，且只能是 Unicode 的 L、M、N、P、S 这些通用目录，且不允许包含 `!"#$%&'()*,:;<=>?[\]^`{|}` 以及 U+FFFD。我们只需要记住使用常规英文字符和数字就好了。


Assume we have compiled a package containing the package clause `package math`, which exports function `Sin`, and installed the compiled package in the file identified by `"lib/math"`. This table illustrates how `Sin` is accessed in files that import the package after the various types of import declaration.

如果我们编译了一个由 `package math` 定义的包，它有一个导出的函数 `Sin`，并且我们把编译的包安装全路径为 `"lib/math"`。下面解释了不同语法的 import 语句调用 `Sin` 方法的区别。

```go
Import declaration          Local name of Sin

import   "lib/math"         math.Sin
import m "lib/math"         m.Sin
import . "lib/math"         Sin
```

An import declaration declares a dependency relation between the importing and imported package. It is illegal for a package to import itself, directly or indirectly, or to directly import a package without referring to any of its exported identifiers. To import a package solely for its side-effects (initialization), use the [blank](https://golang.google.cn/ref/spec#Blank_identifier) identifier as explicit package name:

导入声明声明了导入和被导入包之间的依赖关系。禁止一个包直接或间接导入自己，也禁止导入一个包却不使用这个包里的任何导出元素。如果导入一个包，只是为了完成这个包的初始化，并不为使用这个包导出的变量，为了绕开错误可以在导入包时指定空标识符（下划线）作为这个包的包名。

```go
import _ "lib/math"
```

## An example package

Here is a complete Go package that implements a concurrent prime sieve.
下面的代码实现了并发的筛选功能。

```go
package main

import "fmt"

// Send the sequence 2, 3, 4, … to channel 'ch'.
func generate(ch chan<- int) {
	for i := 2; ; i++ {
		ch <- i  // Send 'i' to channel 'ch'.
	}
}

// Copy the values from channel 'src' to channel 'dst',
// removing those divisible by 'prime'.
func filter(src <-chan int, dst chan<- int, prime int) {
	for i := range src {  // Loop over values received from 'src'.
		if i%prime != 0 {
			dst <- i  // Send 'i' to channel 'dst'.
		}
	}
}

// The prime sieve: Daisy-chain filter processes together.
func sieve() {
	ch := make(chan int)  // Create a new channel.
	go generate(ch)       // Start generate() as a subprocess.
	for {
		prime := <-ch
		fmt.Print(prime, "\n")
		ch1 := make(chan int)
		go filter(ch, ch1, prime)
		ch = ch1
	}
}

func main() {
	sieve()
}
```

# Program initialization and execution（程序初始化及其执行）

## The zero value（零值）

When storage is allocated for a [variable](https://golang.google.cn/ref/spec#Variables), either through a declaration or a call of `new`, or when a new value is created, either through a composite literal or a call of `make`, and no explicit initialization is provided, the variable or value is given a default value. Each element of such a variable or value is set to the *zero value* for its type: `false` for booleans, `0` for numeric types, `""` for strings, and `nil` for pointers, functions, interfaces, slices, channels, and maps. This initialization is done recursively, so for instance each element of an array of structs will have its fields zeroed if no value is specified.

无论是通过普通的声明还是通过调用 `new` 方法**为变量分配到内存**，无论是通过复合类型语法还是调用`make` 方法创建一个新的值，如果没有显式地指定初始化值，这些变量或值都会有一个默认值。这些值或变量的每个元素都会被设置为他们对应类型的**零值**。比如布尔值设置为 false，数字设置为 0， 字符串设置为 `“”`，而指针、函数、接口、切片、信道和 map 设置为 nil。类似的初始化操作会一直迭代进行，因此如果没有显式指定，数组或者结构体的每一个元素都会有默认值。

These two simple declarations are equivalent:

```go
var i int
var i int = 0
```

根据上面的解释可以知道，`var i int` 和 `var i int=0` 具有相同的效果。

After

```go
type T struct { i int; f float64; next *T }
t := new(T)
```

the following holds:

```go
t.i == 0
t.f == 0.0
t.next == nil
```

The same would also be true after

```go
var t T
```

## Package initialization（包初始化）

Within a package, package-level variable initialization proceeds stepwise, with each step selecting the variable earliest in *declaration order* which has no dependencies on uninitialized variables.

在一个包内，包界别的变量会一步一步地进行初始化，每一步都会选择最早声明且不依赖其他变量的变量进行初始化。

More precisely, a package-level variable is considered *ready for initialization* if it is not yet initialized and either has no [initialization expression](https://golang.google.cn/ref/spec#Variable_declarations) or its initialization expression has no *dependencies* on uninitialized variables. Initialization proceeds by repeatedly initializing the next package-level variable that is earliest in declaration order and ready for initialization, until there are no variables ready for initialization.

进一步讲，对于一个没有初始化的包级别的变量，如果它没有显式的初始化表达式，或者有初始化表达式但是不依赖其他未初始化的变量，那么它就被认为**初始化就绪**了。初始化过程会循环地查找下一个最早声明且可以**初始化就绪**的变量进行初始化，直到没有变量需要初始化为止。

If any variables are still uninitialized when this process ends, those variables are part of one or more initialization cycles, and the program is not valid.

如果上面的过程结束的时候依然有变量没有初始化，说明代码存在问题，

Multiple variables on the left-hand side of a variable declaration initialized by single (multi-valued) expression on the right-hand side are initialized together: If any of the variables on the left-hand side is initialized, all those variables are initialized in the same step.

如果赋值符号左侧有多个变量需要初始化，此时右侧只有一个可以返回多个值的表达式，左侧的这些变量会一起被初始化：如果左侧的任何一个变量被初始化了，此时其他的变量也被同时初始化了。

```go
var x = a
var a, b = f() // a and b are initialized together, before x is initialized
```

For the purpose of package initialization, [blank](https://golang.google.cn/ref/spec#Blank_identifier) variables are treated like any other variables in declarations.

为了包的初始化，空变量和其他变量一样进行初始化。

The declaration order of variables declared in multiple files is determined by the order in which the files are presented to the compiler: Variables declared in the first file are declared before any of the variables declared in the second file, and so on.

不同文件中的变量的声明顺序与这些文件在编译器中的顺序有关：第一个文件中声明的变量会比第二个文件中的变量声明的早，依次类推。（这个规则决定了上面提到的初始化规则变量遍历的顺序？）

Dependency analysis does not rely on the actual values of the variables, only on lexical *references* to them in the source, analyzed transitively. For instance, if a variable `x`'s initialization expression refers to a function whose body refers to variable `y` then `x` depends on `y`. Specifically:

评估变量的依赖时不会看变量的真实值，而只会看他们在源文件中的词汇引用，且会进行引用的传递分析。比如，如果变量 `x` 的初始化表达式依赖一个函数，这个函数又依赖变量 `y`，此时可以确定 `x` 依赖 `y`。特别的有下面的规则：

- A reference to a variable or function is an identifier denoting that variable or function.【变化和函数的引用是指表示这个变量和函数的标识符。
- A reference to a method `m` is a [method value](https://golang.google.cn/ref/spec#Method_values) or [method expression](https://golang.google.cn/ref/spec#Method_expressions) of the form `t.m`, where the (static) type of `t` is not an interface type, and the method `m` is in the [method set](https://golang.google.cn/ref/spec#Method_sets) of `t`. It is immaterial whether the resulting function value `t.m` is invoked.【方法 `m` 的引用是指方法值或者类似 `t.m` 的方法表达式，其中静态类型 `t` 不能是一个接口类型，方法 `m` 在类型 `t` 的方法集合中。方法 `t.m` 是否被调用不影响这里的结论。
- A variable, function, or method `x` depends on a variable `y` if `x`'s initialization expression or body (for functions and methods) contains a reference to `y` or to a function or method that depends on `y`.【如果变量、函数或者方法 `x` 的初始化表达式有引用到 `y`，或者函数体和方法体中依赖了 `y`，那么变量 `x` 依赖变量 `y`。

For example, given the declarations

```go
var (
	a = c + b  // == 9
	b = f()    // == 4
	c = f()    // == 5
	d = 3      // == 5 after initialization has finished
)

func f() int {
	d++
	return d
}
```

the initialization order is `d`, `b`, `c`, `a`. Note that the order of subexpressions in initialization expressions is irrelevant: `a = c + b` and `a = b + c` result in the same initialization order in this example.

上面示例中初始化顺序是 `d, b, c, a`。注意在初始化表达式中子表达式的顺序不影响结果，换句话说，对于上面的例子，`a = c + b` 和 `a = b + c` 有相同的初始化顺序。

Dependency analysis is performed per package; only references referring to variables, functions, and (non-interface) methods declared in the current package are considered. If other, hidden, data dependencies exists between variables, the initialization order between those variables is unspecified.

依赖分析每个包都会有；当前包中只有变量、函数和非接口方法会被考虑进行依赖分析。如果变量之间存在其他隐藏的数据依赖性，这些变量之间的初始化顺序是没有指定的。

For instance, given the declarations【比如下面的例子：

```go
var x = I(T{}).ab()   // x has an undetected, hidden dependency on a and b
var _ = sideEffect()  // unrelated to x, a, or b
var a = b
var b = 42

type I interface      { ab() []int }
type T struct{}
func (T) ab() []int   { return []int{a, b} }
```

the variable `a` will be initialized after `b` but whether `x` is initialized before `b`, between `b` and `a`, or after `a`, and thus also the moment at which `sideEffect()` is called (before or after `x` is initialized) is not specified.

上面的例子中可以确定 `a` 在 `b` 之后进行初始化，但是否 `x` 是在 `b` 初始化之前，还是在 `b` 和 `a` 初始化之间，还是在 `a` 初始化之后，这些是不能确定的。同样的道理，`sideEffect()` 什么时候被调用也是不能确定的。

Variables may also be initialized using functions named `init` declared in the package block, with no arguments and no result parameters.

变量也可以在函数名为 `init` 的函数中进行初始化，这个函数没有参数也没有返回值。

```go
func init() { … }
```

Multiple such functions may be defined per package, even within a single source file. In the package block, the `init` identifier can be used only to declare `init` functions, yet the identifier itself is not [declared](https://golang.google.cn/ref/spec#Declarations_and_scope). Thus `init` functions cannot be referred to from anywhere in a program.

每个包中可以定义多个 `init()` 函数，甚至同一个文件中可以定义多个类似的函数。在包范围内，`init` 标识符只能用来定义 `init` 函数，即使这个标识符还没被使用也不能用来定义变量。因此 `init` 函数不能再程序中任何地方调用。

A package with no imports is initialized by assigning initial values to all its package-level variables followed by calling all `init` functions in the order they appear in the source, possibly in multiple files, as presented to the compiler. If a package has imports, the imported packages are initialized before initializing the package itself. If multiple packages import a package, the imported package will be initialized only once. The importing of packages, by construction, guarantees that there can be no cyclic initialization dependencies.

如果一个包没有引入任何包，所有的包级别的变量会在调用了 `init` 函数以后进行初始化，而这些 `init` 函数会按照它们在所有源文件中出现的顺序进行初始化。如果包有导入其他包，这些导入的包会在当前包初始化之前先进行初始化。如果多个包导入了同一个包，被导入的包只会被初始化一次。导入其他包的包需要保证各个包之间没有循环依赖。

Package initialization—variable initialization and the invocation of `init` functions—happens in a single goroutine, sequentially, one package at a time. An `init` function may launch other goroutines, which can run concurrently with the initialization code. However, initialization always sequences the `init` functions: it will not invoke the next one until the previous one has returned.

包初始化、变量初始化、以及 `init` 函数的执行，他们都是在一个协程中按顺序依次进行的，且每次只处理一个包。 `init` 函数可能会启动另一个协程与当前的初始化代码同时运行。不过初始化过程总是顺序执行 `init` 函数：除非当前的 `init` 函数结束并退出了，否则不会调用下一个。

To ensure reproducible initialization behavior, build systems are encouraged to present multiple files belonging to the same package in lexical file name order to a compiler.

为了复现初始化的每一次行为，构建系统建议按照字母顺序排列同一个包里的所有文件然后让编译器进行处理。

## Program execution（代码执行）

A complete program is created by linking a single, unimported package called the *main package* with all the packages it imports, transitively. The main package must have package name `main` and declare a function `main` that takes no arguments and returns no value.

一个完整的程序通过一个独立且非导出的 `main` 包来构建的，当然，这个包导入了所有依赖的包。main 的包名必须是 `main` 且必须声明一个没有参数也没有返回值的  `main` 函数。

```go
func main() { … }
```

Program execution begins by initializing the main package and then invoking the function `main`. When that function invocation returns, the program exits. It does not wait for other (non-`main`) goroutines to complete.

程序从初始化 main 包开始，然后调用 `main` 函数。当 `main` 函数退出以后程序也就退出了，而且它不会等待其他非主的协程结束就直接退出。

# Errors（错误）

The predeclared type `error` is defined as

预定义的类型 `error` 被定义为：

```go
type error interface {
	Error() string
}
```

It is the conventional interface for representing an error condition, with the nil value representing no error. For instance, a function to read data from a file might be defined:

`error` 是一个常规的接口，用来表示错误条件，如果没有 error，则其值为 nil。比如，从文件中读取数据的函数可以定义为 `func Read(f *File, b []byte) (n int, err error)`。

```go
func Read(f *File, b []byte) (n int, err error)
```

# Run-time panics（运行时错误）

Execution errors such as attempting to index an array out of bounds trigger a *run-time panic* equivalent to a call of the built-in function [`panic`](https://golang.google.cn/ref/spec#Handling_panics) with a value of the implementation-defined interface type `runtime.Error`. That type satisfies the predeclared interface type [`error`](https://golang.google.cn/ref/spec#Errors). The exact error values that represent distinct run-time error conditions are unspecified.

运行时错误，比如通过溢出的索引尝试访问数组，此时会触发 运行时 panic，其效果等效于调用内建函数 `panic`并传入了一个类型为 `runtime.Error` 的参数。`runtime.Error` 包含了预定义的 `error` 接口，运行时错误条件的具体值是无法确定的。

```go
package runtime

type Error interface {
	error
	// and perhaps other methods
}
```

# System considerations （系统考量）

## Package `unsafe`（unsafe 包）

The built-in package `unsafe`, known to the compiler and accessible through the [import path](https://golang.google.cn/ref/spec#Import_declarations) `"unsafe"`, provides facilities for low-level programming including operations that violate the type system. A package using `unsafe` must be vetted manually for type safety and may not be portable. The package provides the following interface:

内建包 `unsafe`，编译器知道它的存在，编码时也可以通过导入的方式引入它。`unsafe` 提供了一种底层的编程接口，包括对违反类型系统的一些操作。使用了 `unsafe` 的包必须要手动做类型安全检查且使用起来不再那么方便（没有了垃圾回收、类型检查等）。这个包提供下面的一些接口：

```go
package unsafe

type ArbitraryType int  // shorthand for an arbitrary Go type; it is not a real type
type Pointer *ArbitraryType

func Alignof(variable ArbitraryType) uintptr
func Offsetof(selector ArbitraryType) uintptr
func Sizeof(variable ArbitraryType) uintptr
```

A `Pointer` is a [pointer type](https://golang.google.cn/ref/spec#Pointer_types) but a `Pointer` value may not be [dereferenced](https://golang.google.cn/ref/spec#Address_operators). Any pointer or value of [underlying type](https://golang.google.cn/ref/spec#Types) `uintptr` can be converted to a type of underlying type `Pointer` and vice versa. The effect of converting between `Pointer` and `uintptr` is implementation-defined.

`Pointer` 是指针类型，但是 `Pointer` 不可以被**解引用**。任何底层类型是 `uintptr` 的指针或数值都可以被转换成底层类型为 `Pointer` 的类型，反之亦然。在 `Pointer` 和 `uintptr` 之间转换的影响取决于具体的实现。

```go
var f float64
bits = *(*uint64)(unsafe.Pointer(&f))

type ptr unsafe.Pointer
bits = *(*uint64)(ptr(&f))

var p ptr = nil
```

The functions `Alignof` and `Sizeof` take an expression `x` of any type and return the alignment or size, respectively, of a hypothetical variable `v` as if `v` was declared via `var v = x`.

函数 `Alignof` 和 `SizeOf` 取任意类型的表达式 `x` 并返回假设变量 `v` 的对齐值和字节大小，其中可以认为 `var v = x`。

The function `Offsetof` takes a (possibly parenthesized) [selector](https://golang.google.cn/ref/spec#Selectors) `s.f`, denoting a field `f` of the struct denoted by `s` or `*s`, and returns the field offset in bytes relative to the struct's address. If `f` is an [embedded field](https://golang.google.cn/ref/spec#Struct_types), it must be reachable without pointer indirections through fields of the struct. For a struct `s` with field `f`:

函数 `Offsetof` 取一个可能括起来的选择符 `s.f`（表示结构体 `s` 或 `*s` 的字段 `f`） 作为参数，返回字段相对于结构体地址的偏移的字节数。如果 `f` 是一个嵌套的字段，它必须可以不通过结构体字段的间接指针就可以访问到。对于一个结构体 `s`，如果有字段 `f`：

```go
uintptr(unsafe.Pointer(&s)) + unsafe.Offsetof(s.f) == uintptr(unsafe.Pointer(&s.f))
```

Computer architectures may require memory addresses to be *aligned*; that is, for addresses of a variable to be a multiple of a factor, the variable's type's *alignment*. The function `Alignof` takes an expression denoting a variable of any type and returns the alignment of the (type of the) variable in bytes. For a variable `x`:

计算机架构上可能会要求内存地址是对齐的；也就是说，为了保证变量的地址是某个值的整数倍，需要变量的类型对齐。函数 `Alignof` 取一个标识任意类型的变量的表达式作为参数，然后返回这个变量（其实是这个变量的类型）的对齐字节。比如对于变量 `x`：

```go
uintptr(unsafe.Pointer(&x)) % unsafe.Alignof(x) == 0
```

Calls to `Alignof`, `Offsetof`, and `Sizeof` are compile-time constant expressions of type `uintptr`.

`Alignof`, `Offsetof`, 和 `Sizeof` 的调用都是编译时类型 `uintptr` 的常量表达式。

## Size and alignment guarantees （位数以及对齐保障）

For the [numeric types](https://golang.google.cn/ref/spec#Numeric_types), the following sizes are guaranteed:
对于数字类型来说，下面的大小是需要保障的：

```go
type                                 size in bytes

byte, uint8, int8                     1
uint16, int16                         2
uint32, int32, float32                4
uint64, int64, float64, complex64     8
complex128                           16
```

The following minimal alignment properties are guaranteed:
下面最小的对齐属性是需要保障的：

1. For a variable `x` of any type: `unsafe.Alignof(x)` is at least 1.【对于任意类型的变量 `x`，`unsafe.Alignof(x)` 的至少为 1。
2. For a variable `x` of struct type: `unsafe.Alignof(x)` is the largest of all the values `unsafe.Alignof(x.f)` for each field `f` of `x`, but at least 1.【对于结构体类型的变量 `x`，`unsafe.Alignof(x)` 是 `x` 所有字段 `f` 的 `unsafe.Alignof(x.f)`的最大值，最小值是 1。
3. For a variable `x` of array type: `unsafe.Alignof(x)` is the same as the alignment of a variable of the array's element type.【对数组类型的变量 `x` ，`unsafe.Alignof(x)` 和每个元素的类型的对齐数值相同。

A struct or array type has size zero if it contains no fields (or elements, respectively) that have a size greater than zero. Two distinct zero-size variables may have the same address in memory.

对结构体和数组类型来说，如果它们不包含任意尺寸大于 0 的字段（或元素），他们的尺寸就是 0。两个不同的零尺寸的变量在内存中可能有相同的地址。