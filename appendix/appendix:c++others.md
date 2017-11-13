# C++的其他特性

## 内联函数

`inline`标识符向编译器表明对于一个特定的函数优先使用内联替换取代一般的函数调用。这不会更改函数本身的行为，但是会让编译器在函数调用的地方直接插入生成的函数体代码，而不是执行一般性质的调用，这回在运行是提升一定的性能。

相关声明格式如下：

```cpp
inline type name (arguments ...) { instructions ...}
```

其调用和其他一般函数的调用完全一样。在调用的时候不必加上`inline`关键字，仅仅需要在声明的时候使用就好。

大多数编译器已经在适当的情况下将代码优化成内联的方式。这个标识符仅仅在于告知编译器针对这个函数优先使用内联的方式。

## 函数重载

C++中不同函数可以有相同的函数名，只要其参数类型或者个数不同即可。

例如：

```cpp
#include <iostream>
using namespace std;

int operate(int a, int b)
{
  return ( a * b );
}

float operate(float a, float b)
{
  return (a / b);
}

int main()
{
  int x = 5, y = 2;
  float n = 5.0, m = 2.0;
  cout << operate(x, y);
  cout << "\n";
  cout << operate(n, m);
  cout << "\n";
  return 0;
}
```

注意，函数不能通过返回类型来重载，必须要有参数声明的差异才可以。

## 其他

### 常量

#### 字符串常量

注意字符常量表示方法为：'a'

字符串常量表示方法为："abc"

`wchar_t`类型的字符串，可以存储非英文字符，使用`L`作为前缀：L"abc中文"。

#### `#define`常量

其为预处理指令，在编译的预处理阶段直接将相关的变量替换成对应的值。

```cpp
#define PI 3.14159
```

注意因为其为预处理指令，所以其后面没有分号`;`，如果写上封号，则预处理替换时也会带上这个符号。

#### `const`常量

为语言的表达时，定义了一个运行时不能更改的变量。

### 数组

#### 初始化

```cpp
int billy [5] = {1, 2, 3, 4, 5};
int billy [] = {1, 2, 3, 4}; // 编译器会识别其长度为4
```

#### 作为函数参数

在C++中，不能够向函数传递一整块内存作为参数，但是可以传递内存的指针，这样执行效率会比较高。在声明函数时可以如下：

```cpp
void procedure (int arg[]);
```

### getline

`cin`在接受到空格字符时则会停止读取，这个使得用户无法输入完整的句子。

使用函数`getline`则可以从`cin`中获取完整的用户输入：

```cpp
// cin with strings
#include <iostream>
#include <string>
using namespace std;

int main()
{
  string mystr;
  cout << "What's your name? ";
  getline(cin, mystr);
  cout << "Hello " << mystr << ".\n";
  cout << "What is your favorite team? ";
  getline (cin, mystr);
  cout << "I like " << mystr << " too!\n";
  return 0;
}
```

注意`endl`可以作为换行符，但是当用于缓存流是，则触发缓存flush。

### stringstream

在标准头文件`<sstream>`中定义了`stringstream`，让基于字符串的对象成为流。

```cpp
string mystr ("1024");
int myint;
stringstream(mysty) >> myint;
```

注意：这里可以实现类型转换。

### 函数原型声明

* 在`.h`文件中声明函数主要是为了编译顺利进行。因为在调用函数之前必须先声明函数，这样编译器就能直到该函数的参数类型和返回类型等信息，来完成编译动作（所以函数声明中不必非要给出具体的参数名，只需要给出类型就好）。一般在`.cc`文件中，引入了相关的`.h`文件，确保可以顺利完成编译。在连接构建时，在再将各干`.cc`文件编译的中间文件结合到一起，生成最终的可执行文件。
* 另外对于相互调用的函数，即A函数调用B函数，同时B函数也要调用A函数，则函数声明能够让编译顺利进行。

### 类型转换

#### 隐式转换

基本类型：

```cpp
short a = 2000;
int b;
b = a;
```

勾在其或者符号转换，其中要求类包含特定的构造函数或者符号函数：

例如：

```cpp
class A() {};
class B() { public: B (A a) {} };

A a;
B b = a;
```

因为这里`B`的构造函数中存在`A`类型的参数，所以运行隐式转换。

#### 显示转换

```cpp
short a = 2000;
int b;
b = (int) a;
b = int (a);
```

但是类的指针之间的类型转换却比较特殊，提供四种特使的类型转换操作符：`dynamic_cast`、`reinterpret_cast`、`static_cast`以及`const_cast`。

* `dynamic_cast`会对类和待转换的变量的有效性做检查，如果要转换的是非完全的对象（比如从基类转换到派生类）则会返回空指针，如果转换完全不兼容则抛出`bad_cast`异常。
* `static_cast`如`static_cast<int>(d)`在相关类直接通过指针相互转换，且不会做类型检查，开发这需要保证类型转换是可行的，也可以进行类似于隐式转换的功能。
* `reinterpret_cast`可以在任意类型之间相互转换，即使类型之间无任何关联，这种转换和平台关联较大且不能移植。
* `const_cast`如`const_cast<char *> c`将变量转换为const常量。