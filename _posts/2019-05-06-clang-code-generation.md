# 使用 Clang 与 Python 生成 C/C++ 代码

## 应用场景

在 Windows C++ 开发中，有时候需要使用第三方厂商提供的闭源 SDK 库，一般会提供 DLL 文件与 C/C++ 头文件，有些情况也会提供 lib 文件。

有静态加载与动态加载 DLL 的两种方式，静态加载方式就需要 lib 文件，如果没有提供 lib 文件时，还需要自己生成。静态加载及lib生成方法不是本文重点，就不再提及。静态加载方案有个缺点是，如果这个 DLL 加载过程就出错的话，你的程序也会启动不了。如果对于你的程序这个 DLL 只是个选项的话，那还是使用动态加载的方式比较好一些。

动态加载就是使用 LoadLibrary 加载 DLL 文件后，使用 GetProcAddress 方法获取导出函数入口地址，并调用。如果需要使用的导出函数过多的话，你会想到，如果能根据头文件中的每个函数声明，生成一个对应的封装函数，封装函数里调用实际的 DLL 的函数入口地址。同时这个封装函数里也可以做一些日志记录等工作，有效检查你的程序不被第三方库的错误而导致出问题。

不过问题是这个 DLL 提供了 80+ 个 API ，一一手工封装变的不太现实。

## 解决方法

决定通过脚本生成这部分代码。

首先想到的方法是用正则表达式，不过因为函数参数个数不同，以及部分参数类型还可能由多个词组成，使得这一方法实现起来比较困难。

然后还是决定用 clang 做语法解析，并根据解析结果生成代码。因为是代码生成用的代码，所以决定才用 Python 语言来编写脚本，这样会比直接使用 C/C++ 开发会快一些。

## 环境准备

1. 安装 clang

假设已经安装到默认路径 `C:\Program Files\LLVM`

2. 安装 python

理论上 py2 或 py3 都可以，我使用的是 3

3. 安装 clang package

```
pip install clang
```

4. 验证下 clang 是否工作正常

打开 python 命令行后，尝试运行 `import clang` ，如果没有错误提示，说明已装好 clang 包。

> 如果你安装了多个 python 版本，运行的 pip 对应的 python 版本可能与你的预期不符合，导致 clang 包没有装到此版本的 python 上。

## 参考资料

python clang 包的文档，这个好像是 prophy 里附带的

https://www.pydoc.io/pypi/prophy-1.0.1/autoapi/parsers/clang/cindex/index.html

也可以直接参考 clang 的文档

https://clang.llvm.org/doxygen

一开始文档可能比较晦涩，不过了解 clang 包跟 clang 里对应关系后，会好一些。

## 语法解析

python 的 clang 包使用起来比较简单，想解析一个 C/C++ 文件，只需要下面几行代码

```
import clang.cindex
clang.cindex.Config.set_library_path(r"C:\Program Files\LLVM\bin")
index = clang.cindex.Index.create()
tu = index.parse("header.h", args=["-std=c++17"])
```

第一行导入库，第二行设定了 llvm 可执行文件的路径。后面解析了 header.h 文件。可以看到 index.parse 函数的 args 参数里需要传入相应参数，此参数与编译 header.h 时所需要的参数一样，比如如果此头文件引用了某个库函数，需要告诉编译器路径时，加入 `-I` 参数。

遍历语法树节点的方式如下

```
def traversal(node):
  # Do some things here.
  do_something(node)
  for child in node.get_children():
    traversal(child)

traveral(tu.cursor)
```

可以看出 traversal 是一个递归函数，传入了 tu.cursor 为根结点，遍历语法树。

## 一些常用的 node 属性

判断一个节点（node）是否为函数声明

```
node.king == clang.cindex.CursorKind.FUNCTION_DECL
```

判断一个节点是否为参数

```
node.king == clang.cindex.CursorKind.PARM_DECL
```

节点名称：`node.spelling` 函数声明节点对应函数名称，参数节点对应参数名称。

节点对应的源码位置：`node.extent` 以下函数可以获取节点对应节点的所有源码：

```
def get_source(extent):
  with open(extent.start.file.name, 'rb') as f:
    content = f.read()
    return content[extent.start.offset:extent.end.offset].decode('gbk')
```

源码文档为 ANSI 编码（GBK）情况下有效，如果是其它编码，请修改最后 decode 传入的参数。

节点对应源码文件名称：`node.extent.start.file.name`

## 部分错误情况

首先你需要保证你的头文件是能够被 clang 正确解析的。这一点可以通过直接运行 clang.exe 来验证。

```
clang -Xclang -ast-dump -fsyntax-only header.h
```

如果上述命令能够没有出现错误，并能够正确显示语法树的话，就说明 header.h 能够正常解析了。

如果你的 header.h 有依赖第三方库，需要加入 `-I` 参数来指定 include 目录的路径。

## 结尾

用 python 解析 C/C++ 代码后可以做很多代码生成的事情，发挥你的想象力！

