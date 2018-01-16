# 解决 VS 编译 C4819 警告

今天在编译一个第三方库的时候出现了 

```
warning C4819: The file contains a character that cannot be represented in the current code page (936). Save the file in Unicode format to prevent data loss
```

由于第三方库默认配置了 treat warning as error 导致报错编译不成功。这个问题是因为 VS 默认配置下不支持 BOM 头的 UTF-8 格式的文件。

VS 默认情况下读取没有 BOM 头的非 unicode 的文件时，会按照当前系统的 Code Page 读取文件内容，如果发现文件内容中包含了无法解析的 unicode 字符串时，就会报出这个错误。一般这个错误在默认语言设置为非英文的 Windows 系统中出现。

在一些跨平台的 C++ 开源库中，文件默认的编码方式是 UTF-8 没有 BOM 头，因为这种文件编码方式在非 Windows 平台上是首选。所以导致一些跨平台的 C++ 开源库在编译时，容易遇到 C4819 警告。

> 不过 VS 支持 UTF-16 编码方式的文件。此编码方式下如果大小端为 little endian ，那可以没有 BOM 头。

通常有几种方法解决这个问题；

## 1. 忽略

在编译选项中加入忽略 4819 的选项，大部分情况下可以编译过。

不过这种方式遇到一些特殊的 unicode 字符时，会导致源代码文件解析失败，出现语法错误的情况。

所以这种方法最不推荐。

## 2. 将源代码文件保存成 unicode 格式

如果源代码都是你自己的，或者涉及到的源代码文件数量很少，那么这个方法是最容易的。

不过对于第三方库，这种方法不适用。

## 3. 编译选项设置

对于我这种情况，设置编译选项 `/utf-8` ，让 VS 编译器默认认为没有 BOM 的文件内容为 UTF-8 编码，而不是默认的 Code Page 决定的。

参考编译选项文档 [/source-charset](https://msdn.microsoft.com/en-us/library/mt708819.aspx) ，其中 `/utf-8` 与 `/source-charset:utf-8 /execution-charset:utf-8` 含义相同。

> 此编译选项从 VS 2015 Update 2 （具体版本待考证） 开始支持。