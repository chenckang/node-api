# 预处理指令

预处理指令就是成员中不属于程序表达式，但是用于标识预处理的相关指令的那些行。这些行都是以井好`#`开头。预处理指令在代码的实际编译前执行有预处理其解释执行。

这些预处理指令每个值占据一行。一旦遇到换行符则该预处理指令就算是结束了。没有分号`;`作为解weu。可以通过反斜杠`\`来将一行预处理指令拓展至多行。

## 宏定义（`#define`、`#undef`）

使用`#define`来定义预处理宏，格式如下：

```cpp
#define identifier replacement
```

当预处理其解析到这个指令后，会在后面代码中将所有`identifier`使用`replacement`进行替换。`replacement`不可为表达式、语句、块或者任何其他的东西。预处理器并不理解C++预发，其只不过进行一些替换。

```cpp
#define TABLE_SIZE 100
int table1[TABLE_SIZE];
int table2[TABLE_SIZE];
```

当预处理器替换了`TABLE_SIZE`后，代码变为如下：

```cpp
int table1[100];
int table2[100];
```

也可以用`#define`定义函数宏

```cpp
#define getmax(a, b) a>b?a:b
```

这里会将所有的`getmax`后加上两个参数出现的代码替换成后面的表达式，同时也会替换参数变量中的标识符，如同一个函数一样：

```cpp
// function macro
#include <iostream>
using namespace std;

#define getmax(a, b) ((a) > (b) > (a) : (b))

int main() {
  int x = 5, y;
  y = getmax(x, 2);
  cout << y << endl;
  cout << getmax(7, x) << endl;
  return 0;
}
```

定义宏不受块结构的影响。一个宏在对其使用`#undef`预处理指令之前都是有效的。

函数宏定义接受两个特殊的操作符（`#`和`##`）：如果`#`操作符出现在替换序列的中函数参数之前，则这个参数会转化为字符串：

```cpp
#define (str) #x
cout << str(text);
```

会被转换为：

```cpp
cout << "test";
```

操作符`##`会将两个参数拼接起来且中间无空格字符：

```cpp
#define glue(a, b) a ## b
glue(c, out) << "test";
```

会被转换为：

```cpp
cout << "test";
```

因为预处理替换在C++预发校验前执行，所以宏定义是一个非常有技巧性的特性，但是要当心：严重依赖复杂的宏会导致其他开发者的困惑，因为宏的语法在很多情况下和正常的C++预发不同。

## 条件包含（`#ifdef`、`#ifndef`、`#if`、`#endif`、`#else`以及`#elif`）

这些指令可以在某些条件满足的情况下包含或者丢弃程序的部分代码。

`#ifdef`仅在宏被定义时将程序的一部分包含进来编译，无论宏的具体值是什么：

```cpp
#ifdef TABLE_SIZE
int table[TABLE_SIZE];
#endif
```

`#ifndef`的作用相反。

```cpp
#ifndef TABLE_SIZE
#define TABLE_SIZE 100
#endif
int table[TABLE_SIZE];
```

## 行控制 (`#line`)

格式：

```cpp
#line number "filename"
```

例如：

```cpp
#line 20 "assigning variable"
int a?;
```

会影响异常的错误显示。

## 错误指令（`#error`）

当遇到这个指令，则编译器会抛出其参数所指定的异常：

```cpp
#ifndef __cplusplus
#error A C++ compiler is required!
#endif
```

`__cplusplus`默认会被所有的C++编译器所定义。

## 源码包含（`#include`）

当预处理器遇到这个指令，会将指定文件内容包含进来。有两种形式来指定一个文件：

```cpp
#include "file"
#include <file>
```

二者的唯一不同就是如何去查找这个文件。第一种情况会去当前文件的目录下去找，如果没有再去默认配置的目录下去找这些标准头部文件。第二中情况会直接去配置的目录下去找这些头部文件。

所以一般而言标准的头部文件都使用尖括号表示。

## 编译指示指令（`#pragma`)

向编译器提供一些参数选项，这些选项和平台以及具体的编译器相关。可以查询手册或者编译器的手册来查看有那些参数。

如果编译器不支持特定的`#pragma`参数，则会忽略，不会产生错误。

##预定义宏名称

如下宏名称会即时定义：

| 宏 | 值 |
| --- | --- |
| __LINE__ | 表示被编译的源码中当前行号的整数值 |
| __FILE__ | 表示被编译的源码文件的文件名 |
| __DATE__ | 表示编译开始的日期，格式`Mmm dd yyyy` |
| __TIME__ | 表示编译开始的事件，格式`hh:mm:ss` |
| __cplusplus | 整数值，所有的C++编译器都其定义成某个值。如果编译器完全兼容C++标准，则其值大于等于`199711L`，具体依赖于编译的标准版本 |

例如：

```cpp
// standard macro names
#include <iostream>
using namespace std;

int main()
{
  cout << "This is the line number " << __LINE__;
  cout << " of file " << __FILE__ << ".\n";
  cout << "Its compilation began " << __DATE__;
  cout << " at " << __TIME__ << ".\n";
  cout << "The compiler gives a __cplusplus value of " << __cplusplus;
  return 0;
}
```
