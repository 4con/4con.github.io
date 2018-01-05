# 配合 C++ 语言的脚本语言研究

## Lua

官网： http://www.lua.org

特点： 功能强大，非常稳定，支持 coroutine ，MIT License

Lua 不用说，在很多实际工程项目及产品中已经用到这种语言， Lua 在游戏领域上也用的比较多，本人已知的有魔兽世界、饥荒等。

Lua 是纯 C 语言实现的，编译后的整体体积一般不超过 1 MB 大小。

Lua 语言内置库功能比较丰富，涵盖了数学计算、文件操作， coroutine 库等等。所以 Lua 语言作为独立使用也可以满足很多需求。

Lua 语言有 GC 。

（待写）

## ChaiScript

官网： http://chaiscript.com

GitHub： https://github.com/ChaiScript/ChaiScript

特点：非常小巧， Header Only ， 稳定， BSD License



ChaiScript 特别小巧，简单浏览了一下 Github ，有例子，有很完善的单元测试，开发时间也比较长， License 也很开放，适合商业使用。 ChaiScript 的本身语法与 ECMAScript 类似，所以脚本的语法上会给人感觉比较熟悉。

ChaiScript 要求使用支持 C++ 14 的编译器。

GC？

C++ 提供函数给 ChaiScript 比较简单，需要调用 `chai.add(chaiscript::fun(&your_func), "function_name")` 函数注册即可，官网首页的例子如下。

```C++
#include <chaiscript/chaiscript.hpp>

std::string helloWorld(const std::string &t_name) {
  return "Hello " + t_name + "!";
}

int main() {
  chaiscript::ChaiScript chai;
  chai.add(chaiscript::fun(&helloWorld), "helloWorld");

  chai.eval(R"(
    puts(helloWorld("Bob"));
  )");
}
```

`chai.add` 除了可以添加函数外，可以添加类型，添加类型的继承关系。

此外也可以添加对象或对象的引用。

从上面例子里可以看出 `std::string` 可以很好的被 ChaiScript 识别，实际上 ChaiScript 能够识别大多数的 C++ STL 中的类型。不过有些类型需要提前说明。

```C++
typedef std::vector<std::pair<int, std::string>> data_list;
data_list my_list{ make_pair(0, "Hello"), make_pair(1, "World") };
chai.add(chaiscript::bootstrap::standard_library::vector_type<data_list>("DataList"));
chai.add(chaiscript::bootstrap::standard_library::pair_type<data_list::value_type>("DataElement"));
chai.add(chaiscript::var(&my_list), "data_list");
chai.eval(R"_(
    for(var i=0; i<data_list.size(); ++i)
    {
      print(to_string(data_list[i].first) + " " + data_list[i].second)
    }
  )_");
```

ChaiScript 中的对象是引用计数的，所以对象的生命周期与 C++ 对象生命周期类似，超出 scope 会自动销毁，所以对于熟悉 C++ 的人来说是非常友好的。

ChaiScript 本身 Engine 部分是能够保证在多线程环境下正常运行的，不过如果用户创建的一个对象并在多线程环境下操作时，不保证此对象线程安全。

> 参考来自 http://discourse.chaiscript.com/t/introduction-and-questions/23/2

ChaiScript 的主要功能可以看这里 https://github.com/ChaiScript/ChaiScript/blob/develop/cheatsheet.md

