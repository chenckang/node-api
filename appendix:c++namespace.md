# 命名空间

命名空间允许将诸如类、对象以及函数归纳为一个命名的分组下。这样，全局作用域可以划分为多个自作用域，每个都有其自己的名字。

命名空间的格式如下：

```c++
namespace identifier
{
  entities
}
```

这里`identifier`为任何有效的变量，`entities`为包含在命名空间内的类、对象以及函数的集合。例如：

```cpp
namespace myNamespace
{
  int a, b;
}
```

这个示例中，变量`a`和`b`为声明在`myNamespace`命名空间下的一般变量。要在`myNamespace`之外访问这些变量，必须使用作用于操作符`::`。例如，从`myNamespace`之外访问之前的那些变量，可以这样写：

```cpp
myNamespace::a;
myNamespace::b;
```

在全局对象或者函数有可能含有同一个命名导致重复定义错误时，命名空间就显得格外有用。例如：

```cpp
// namespace
#include <iostream>
using namespace std;

namespace first
{
  int var = 5;
}

namespace second
{
  double var = 3.1416;
}

int main() {
  cout << first::var << endl;
  cout << second::var << endl;
  return 0;
}
```

这个示例中，存在两个全局变量具有同一个名字`var`。一个定义在命名空间`first`，另一个定义在`second`中。因为命名空间，所以不存在重复定义错误。

## `using`

关键字`using`用于向当前的声明区域引入一个命名空间。例如：

```cpp
// using
#include <iostream>
using namespace std;

namespace first
{
  int x = 5;
  int y = 10;
}

namespace second
{
  double x = 3.1416;
  double y = 2.7183;
}

int main () {
  using first::x;
  using second::y;
  cout << x << endl;
  cout << y << endl;
  cout << first::y << endl;
  cout << second::x << endl;
  return 0;
}
```

注意在这个代码中，`x`（不含有任何表示名）指向`first::x`而`y`执行`second::y`，如`using`声明中所指定的。仍然可以使用完整的表示名`first::x`和`second::y`来访问相关变量。

关键字`using`作为引入整个命名空间的指令：

```cpp
// using
#include <iostream>
using namespace std;

namespace first
{
  int x = 5;
  int y = 10;
}

namespace second
{
  double x = 3.1416;
  double y = 2.7183;
}

int main () {
  using namespace first;
  cout << x << endl;
  cout << y << endl;
  cout << second::x << endl;
  cout << second::y << endl;
  return 0;
}
```

在这个例子中，因为我们已经使用`using namespace first`声明，所以所有直接使用`x`和`y`而不加名称标识的都指向命名空间`first`。

`using`和`using namepsace`则只会在他们所声明的代码块内有效，或者如果在全局作用域里直接使用则对整个代码都有效。例如，如果我们首先想要使用一个命名空间的对象，其次是另一个，我们可以像这样做：

```cpp
// using namespace example
# include <iostream>
using namespace std;

namespace first
{
  int x = 5;
}

namespace second
{
  double x = 3.1416;
}

int main() {
  {
    using namespace first;
    cout << x << endl;
  }
  {
    using namespace second;
    cout << x << endl;
  }
  return 0;
}
```

## 命名空间别名

可以针对已存在的命名空间声明别名，如下：

```cpp
namespace new_name = current_name;
```

## 命名空间`std`

C++标准库中声明的所有实体都在`std`命名空间下。这也是为什么我们一般会使用`using namespace std;`语句来包含`iostream`中定义的各种实体。