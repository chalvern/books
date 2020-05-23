# Python 的 hello world

基础决定最终可达到的高度。

本节内容首先带着大家利用 ubuntu 原生的 python 进行简易版 helloworld 的编写与执行；然后带着大家按需安装特定版本的 python，并配置对应的开发环境。

## 简易版的 helloworld

在前面安装 vscode 的那一节我们已经编写过 `python` 的 helloworld。

由于 ubuntu18.04 系统已经默认安装了 python3 解释器，因此可以很方便写出并运行 helloworld。简单讲分成几步：

1. 创建一个项目目录，比如我们在家目录下面使用命令 `mkdir` 创建对应的目录；然后通过 `code .` 命令打开对应的目录作为工作目录。
1. 检查 vscode 是否安装 Python 插件；
1. 创建 helloworld.py 文件并编写内容 `print("hello world")`;
1. 配置对应的解释器，可以直接执行输出结果。
1. 也可以在命令行使用 `python3 helloworld.py` 来执行对应的文件得出结果。

上面简易版的 helloworld 直接使用了 ubuntu 系统默认带的 python3 环境，具体的是 `python 3.6.9` 版本，版本比较老。另一方面，在我们实际项目的开发过程中，大部分会指定特定的 python 版本进行开发，这个时候就需要我们在我们的本地安装对应的版本，搭建匹配的开发环境。接下来就让我们看一下如何从源码安装特定版本的 python。

## 高级版 helloworld

为什么提出“高阶版的 helloworld”呢？这是一个“基础决定最终可达到的高度”的问题；**“要想楼层盖的高，就看地基打多牢”**，话糙理不糙。作为一名开发者，如果只会扎架子（比如动口只是架构、方案、设计模式），优雅而丰满的项目大厦是建不成的。不过很多人把扎好架子作为目标，不去关注基础上细节，因此丢失了很多乐趣，也限制了自己可达的高度，还是有点惋惜的。

我们高阶版的 helloworld 希望可以给大家开一个好头，尝试从一个简单的 helloworld 里挖掘一些其他有意思的东西。

接下来让我们安装特定版本的 python，比如选定 3.8.3 进行安装 。

### 源码下载

首先让我们下载 python3.8.3 的源码。无论是 python 还是 go，它们都是开源的语言，因此我们都是到它们的官网找到对应的源码的下载链接，相对其他一些地方的下载链接要靠谱很多。这里顺便建议大家平日里可以多逛官网，作为我们最重要的的一手资料进行学习。

为了适应终端命令行，我们采用命令行的方式下载并安装。

1. 首先选择一个合适的目录，然后从官网找到对应的链接后复制，通过 `wget https://www.python.org/ftp/python/3.8.3/Python-3.8.3.tgz` 进行下载。

1. 下载完成后得到的是一个为方便传输的压缩包，需要我们把它解压缩，通过在当前目录执行 `tar -xzf Python-3.8.3.tgz` 即可完成，在当前目录生成一个 `Python-3.8.3` 的目录。

1. 我们可以通过 `code Python-3.8.3` 来打开对应的目录，简单预览一下 python 的源码项目（源码阅读一般流程）。

    1. 大概浏览一下，咦，发现 python 源码库里有好多 c 语言文件，可以知道 python 是基于 c 语言进行开发的，是不是很神奇 ^_^；

    1. 查看对应的 README，可以看到 源码库中有一个 README.rst 文件，查看一遍说了什么（英文学习多么重要啊）。我们目前只关注安装相关的内容，因此找到一段命令做参考，并看到有 `./configure --help`，那我们有必要探索一番。（在命令行环境多打 `--help` 是一个好习惯）

    1. 其他的我们暂时不关注，先放着不管，不过可以看到原来一个牛x的项目是这个样子的。

1. 通过 `./configure --help` 查看配置项，然后通过 `./configure --prefix=$HOME/software/mypy` 进行配置；也可以有选择地添加建议的参数，比如添加优化参数保证使用所有稳定的优化方案 `./configure --prefix=$HOME/software/mypy --enable-optimizations`。

1. 通过 `make` 和 `make install` 来编译并安装源码。在整个过程中可能会有一些警告的提示或者错误的提示，静待看最后的总结报告。

1. 安装依赖（整个需要我们通过比较久的经验做积累才可以慢慢理解为什么需要这些依赖）。这里我积累了几个必要的，大家先安装一下。如果未来在项目开发过程中使用到了其他的依赖，则可以按照类似的步骤进行安装。

    1. 通过 `sudo apt install zlib1g.dev` 安装 zlib 依赖（整个是在安装 python 的时候就会需要）
    
    1. 通过 `sudo apt install libssl-dev` 安装 ssl 依赖（编写 web 相关的服务的时候会用到）

    1. 通过 `sudo apt install libffi-dev` 安装 libffi 依赖（当 pip install 的时候会用到）

1. 重新通过 `./configure --prefix=$HOME/software/mypy` 来配置生成对应的 make 文件。并通过 `make` 和 `make install` 编译安装。

1. 安装完成以后，就可以通过 `~/software/mypy/bin/python3` 和 `~/software/mypy/bin/pip3`把玩了。

1. 为了便利性，我们可以把新安装的 python 的 bin 目录添加到 PATH 中去，这样就可以直接使用 `python3` 和 `pip3` 命令了。


## 小结

本小节介绍了 Python 的 helloworld，首先介绍利用 ubuntu 原带的 python 作为解释器，然后重点带大家安装自己的 python 解释器。并使用自己安装的解释器执行了 hellworld 脚本。