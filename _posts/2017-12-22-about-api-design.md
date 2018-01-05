# 漫谈 API 设计

> 本文主要是个人对 API 设计的想法，欢迎讨论。

## 概述

一个系统的 API 设计能反映出系统的架构，好的设计往往能让 API 使用者快速上手，接触不久就能够“猜”出想要使用的功能在哪里，“猜”出每个参数的目的与使用方法。

这里的“猜”并不是毫无根据的猜测，而是这个人的在技术领域的知识积累与经验起到作用。

## 抽象极端

极端例子：小明设计了一个通用性很高的宇宙无敌 API

```
  void* do(void*);
```

只要定义好参数与返回值，这个 API 等效于目前世界上存在的所有 API ，同时也能用这一个 API 作为多个 API 的整合，最终这一个 API 就能作为任何复杂系统的 API 设计了。

不过，你如果真这么设计，会被使用者打，或者根本没有使用者。

另一个例子：需求是需要实现被加数为 1-9 的加法函数，小明想到一个设计

```
  int add1(int a);
  int add2(int a);
  ...
  int add9(int a);
```

没错，这个设计完美的实现了需求。

不过，你如果真这么设计，还是会被使用者打，或者根本没有使用者。

这两个例子很明显，可能大家会觉得自己不会做出这样的事情，不过这类事情就会发生在你或你的身边。

一个是过度抽象，另一个是导致更具体。

“过度抽象”与“太具体”之间有很大的间隙，这个间隙间，有时候很难判定一个设计是好是坏，每个开发人员/ API 设计者都有自己的喜好倾向，不同人所看到的同一个设计会有不同的评价。

## 好的设计

API 设计是为使用者服务的，设计者是乙方，使用者是甲方；设计者是产品经理，使用者是用户；

产品经理要让产品解决用户某些需求的同时，要提高产品的易用性。

一个系统或者说一个设计，肯定是为了解决某些需求的，为了解决某些需求，使用方想通过使用 API 解决需求，这个 API 的正确使用方法是与需求有关系的。

### 定义

下列定义可能不严谨，但凑合着看吧。。。

```
Rsys: 系统需求：系统本身能够完成的功能需求
Rapi: 系统 API 需求：系统对外开放的接口需求
Learn(Rsys): 学习系统功能实现所需的努力程度
Learn(Rapi): 学习系统 API 使用方法所需的努力程度
```

假设对于所有满足 Rapi 的集合为 `S_Rapi = { api_x | x 取值 1-n，api_x 表示第 x 个元素 }` 总共有 n 种方法设计（实际上 n 可能为无限大）。

### 理论 1 接口比实现细节简单

```
  Learn(Rsys) > Learn(Rapi)
```

*一般来讲系统 API 学习会比实现系统要简单，如果不是，那就没有必要做 API 。*

### 理论 2 实际的接口设计会带来额外的复杂度

```
  Learn(api_x) > Learn(Rapi)  对 x 取值 1-n 有效
```

*无论系统 API 最终怎么设计，最终不会比系统 API 需求的复杂度等级低*

所以，复杂度有下限，无法设计出更简单（复杂度低）的 API ，如果能，肯定是砍了一些需求。

### 理论 3 如果某种 API 设计比自己实现起来还复杂，那就别用了

```
  如果 Learn(api_x) > Learn(Rsys) ，那么 api_x 很失败。
```

*如果自己实现一个系统的难度比学习这个系统的某个 API 实现 api_x ，那么这个 api_x 非常失败。*

这种情况下，使用者还不如自己实现系统需求。

### 理论 4 好的 API 设计的复杂度会小

```
  当 x = k 时，能让 Learn(api_x) 最小，那么这是最好的设计
```

因理论 2 可以得出

```
  当 x = k 时，能让 Learn(api_x) - Learn(api) 最小，那么这是最好的设计
```

即，能让 API 设计实现带来的额外复杂度越小越好。






