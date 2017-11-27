# Windows 1709 修改了 ReplaceFile 的实现

> 本文只是描述了这个情况，并没有给出对应问题的解决方案。

## 问题情况

最近升级了一下公司电脑，从 Windows 10 1703 升级到了 1709 版（具体版本为 10.0.16299 pro 版）后，发现公司信息安全产品运行不正常。 Office Word 2016 在保存文件时，弹出对话框要求选择路径，此时文件名是一个类似 ~WRD0000.tmp 的。其他 Office 产品都有这个问题，不过其它部分软件能够正常运行。

## 一些前提知识

本文中很多知识是与 Windows 文件系统底层相关的，不做一一解释。

Office 系列的产品在保存文件时，不会直接在原文件上做操作，而是先创建临时文件写入新内容，然后重命名成原文件名的方式完成保存工作。在很多软件（Notepad例外，会直接写）中这种保存操作非常普遍，主要是为了防止写入新内容时，计算机宕机等导致原文件损坏。

## 分析过程

发现这样的问题时，首先想到的是在 1709 新版本中，文件重命名操作出现了什么问题，所以拿着 FileSpy 工具看了一下。

> ![FileSpy](http://www.zezula.net/en/fstools/filespy.html) 是个跟踪文件系统 IRP 的工具，文件系统、文件系统过滤驱动开发者会经常使用此工具排查问题

之前主要在 Windows 7 平台上调试的比较多， Windows 10 1709 跟踪保存操作的 IRP 中发现了较多的不认识的 IRP ，也没有找到 IRP_MJ_SET_INFORMATION 的 FileRenameInformation 。但是发现了一些 FileSpy 工具没有解析成功的内容 IRP_MJ_SET_INFORMATION 00000041 ， 41 对应的十进制是 65 。

> 几乎所有重命名操作，会与 IRP_MJ_SET_INFORMATION FileRenameInformation 这个 IRP 有关。

后来打开 WDK 的 C 头文件找到 FILE_INFORMATION_CLASS 的定义，发现 65 对应的是 FileRenameInformationEx ，打开 MSDN 的 FILE_INFORMATION_CLASS 对应的 ![文档](https://msdn.microsoft.com/en-us/library/windows/hardware/ff728840(v=vs.85).aspx) 只是简单了说了 A FILE_RENAME_INFORMATION structure which contains additional flags. This value is available starting with Windows 10, version 1709. 并没有说这个 additional flags 是什么，具体与 FileRenameInformation 的区别是什么。

Office 16 之前一直用过没有问题，这次出问题肯定是内核里的实现变化了，之前没有仔细跟过 Office 的应用层调用的 API ，这次拿 ![API Monitor](http://www.rohitab.com/apimonitor) 工具看了一下所调用的应用层 API ，发现使用了 `ReplaceFile` API 。

## 结论

Windows 1709 内核层 `ReplaceFile` 的实现使用了新的 FileRenameInformationEx ，这可能会影响一些文件系统相关的安全软件，请各大厂商熟知。

***

Windows 1709 handles `ReplaceFile` API in kernel use FileRenameInformationEx value for FILE_INFORMATION_CLASS in IRP_MJ_SET_INFORMATION IRP. It will cause some security products fail.
