# Glossary（词汇表）

* [原文](https://docs.python.org/3/glossary.html)


## bytecode（字节码）
在 CPython 的解释器中，Python 源码会被编译成**字节码**，也就是说在解释器中它主要以字节码的形式存在。字节码可以被缓存在 .pyc 后缀文件中，因此如果多次执行同一个文件，只需要编译一次就可以了，因此第一次以后的执行相对都会快一些。

字节码作为一种“中间语言”在虚拟机上运行，虚拟机每次执行一个字节码。

## CPython
主要用来区别于 Jython（基于JVM） 或者 IronPython，CPython 指的就是官方实现的 python 版本。

【注解】一般情况下，如果我们不提实现版本，默认都是指的 CPython。

## GIL
GIL 是解释器全局锁（Global interpreter lock） 的英文简称。

## Global interpreter lock（解释器全局锁）


## virtual machine（虚拟机）

指完全软件实现的计算机。python 的虚拟机执行的是由源码编译生成的字节码。