# 模板
## 函数模板

函数模板是可以处理泛型的特殊函数。这使得我们可以创建适用于多个类型和类而无需重复对每个类型写个独立函数的函数模板。

在C++中，可以通过模板参数来实现。模板参数就是就是可以传递类型作为参数的特殊参数：就如同可以传递普通的值给一般的函数参数一样，模板参数可以给函数传递类型作为参数。这些函数模板可以想其他一般的类型那样使用这些参数。

使用类型参数定义函数模板的格式如下：

```cpp
template <class identifier> function_declaration;
template <typename identifier> function_declaration;
```

这两个原型之间的唯一的不同就是使用关键字`class`或者`typename`。这二者起始没啥区别，因为两个表达式含义一样且行为也完全一样。

例如，创建一个返回两个对象中较大的函数模板，可以使用：

```cpp
template <class myType>
myType GetMax(myType a, myType b) {
  return (a > b ? a : b);
}
```

这里我们创建了一个模板参数为`myType`的函数模板。模板参数代表了一个还不确定的类型，但是可以如同一个一般的类型一样使用在函数模板中。如各位所见，函数模板`GetMax`返回这个仍然不确定类型的参数中较大的那个。

要使用这个函数模板，可以使用如下的格式来执行函数：

```cpp
function_name <type> (parameters);
```

例如，执行`GetMax`比较`int`类型的两个整数，可以如下写：

```cpp
int x, y;
GetMax <int> (x, y);
```

当编译器解析到这个函数模板的调用，其会使用函数模板自动生成一个函数，其中使用实际传入的模板参数类型（这里是`int`）替换每个`myType`然后执行之。这个过程会被编译器自动执行并且对开发者不可见。

如下为完整的示例：

```cpp
// function template
#include <iostream>
using namespace std;

template <class T>
T GetMax(T a, T b) {
  T resulte;
  result = (a>b) ? a : b;
  return (result);
}

int main() {
  int t = 5, j = 6, k;
  long l = 10, m = 5, n;
  k = GetMax<int>(i, j);
  n = GetMax<long>(l, m);
  cout << k << endl;
  count << n << endl;
  return 0;
}
```

这个例子中，我们`T`作为模板参数名，而不是`myType`，因为其更加简短，并且实际上也是一个常用的模板参数名。但是你可以使用任何其他的变量名。

上门的例子中，我们使用了两次`GetMax()`函数模板。第一次使用`int`作为参数，第二次使用`long`作为参数。编译器会初始化他们并每次执行的时候会调用相应版本的函数。

如你所见，类型`T`在函数模板`GetMax()`中使用，甚至声明该类型的新实例：

```cpp
T result;
```

因此，当函数模板使用特定的类型初始化时，`result`会是和参数`a`和`b`具有一致的类型。

在这个特殊的例子中，泛型`T`作为`GetMax`的一个参数，编译器可以自动找出要初始化那种数据类型，而无需使用尖括号明确的标注类型（如之前使用`<int>`和`<long>`)。所以也可以写成如下：

```cpp
int i, j;
GetMax(i, j);
```

应为`i`和`j`都是`int`类型，且编译器会自动算出模板参数只能为`int`。这个隐含方法会输出同样的结果：

```cpp
// function tempalte II
using namespace std;

template <class T>
T GetMax(T a, T b) {
  return (a > ? a : b);
}

int main() {
  int i = 5, j = 6, k;
  long l = 10, m = 5, n;
  k = GetMax(i, j);
  n = GetMax(l, m);
  count << k << endl;
  count << n << endl;
  return 0;
}
```

注意在这个例子中，我们如何执行函数模板`GetMax()`而没有明切的指定尖括号之间的类型。编译器可以自动算出每个调用中具体需要哪个类型。

因为我们的函数模板中只有一个模板参数（`class T`），函数模板本身接受两个参数，且都是`T`类型，那么我们就不能使用不同的类型作为参数调用函数模板：

```cpp
int i;
long l;
k = GetMax(i, l);
```

因为`GetMax()`函数模板期望接受两个同类型的参数，而这里使用了两个不同类型的对象，所以会导致错误。

我们也可以定义一个接受不止一个类型参数的函数模板，只需要在尖括号之间定义多个模板参数。 例如：

```cpp
template <class T, class U>
T GetMin(T a, u b) {
  return (a < b ? a : b);
}
```

在这个例子中，函数模板`GetMin()`接受两个不同类型的参数，并返回第一个参数（`T`）同类型的结果对象。例如，在声明函数模板后，可以执行`GetMin()`：

```cpp
int i, j;
long l;
i = GetMin<int, long>(j, l);
```

或者简单的使用：

```cpp
i = GetMin(j, l);
```

即使，`j`和`i`类型不同，编译器仍然可以确定相应的初始化。

## 类模板

也可以编写类模板，所以类可以有使用模板参数的类型成员。例如：

```cpp
template <class T>
class mypair {
    T values [2];
  public:
    mypair (T first, T second)
    {
      values[0] = first;
      values[1] = second;
    }
}
```

这个类型用于存储两个优先类型的两个元素。例如，如果我们想要声明这个类的一个对象来存储两个类型为`int`的整数值115和36，可以这样写；

```cpp
mypair<int> myobject (115, 36);
```

同样的类也可以用于创建一个存储任何类型的对象：

```cpp
mypair<double> myfloats (3.0, 2.18)
```

上面的类模板的唯一成员函数是定义在类的内部。为了在类模板外定义函数成员，需要一致在定义语句前加上`template <...>`前缀：

```cpp
// class template
#include <iostream>
using namespace std;

template <class T>
class mypair {
    T a, b;
  public:
    mypair (T first, T second)
      { a = first; b = second; } // 构造函数
    T getmax();
};

template <class T>
T mypair<T>::getmax() {
  T retval;
  retval = a > b ? a : b;
  return retval;
}

int main() {
  mypair <int> myobject (100, 75);
  count << myobject.getmax();
  return 0;
}
```

注意定义成员函数getmax的语法：

```cpp
template <class T>
T mypair<T>::getmax()
```

对这么多`T`感到困惑？声明中有三个T：第一个是模板参数。第二个`T`指函数返回的数据类型。第三个`T`（尖括号中间的那个）也是必要的：其指出函数的模板参数也是类的模板参数。

## 模板特化

当使用特定的类型作为模板参数，如果要定义模板的不同实现，可以声明模板的特化。

例如，假设有一个简单类`mycontainer`可以存储任何类型的一个元素，其只有一个成员函数`increase`来增加值。但是发现如果存入`char`类型的元素，可以更加方便的实现一个完全不同的`uppercase`成员方法，所以我们决定对该类型声明一个类模板特化。

```cpp
// template specification
using namespace std;

template <class T>
class mycontainer {
    T element;
  public
    mycontainer (T arg) { element = arg; }
    T increase() { return ++element; }
};

// class template specification
template <>
class mycontainer <char> {
    char element;
  public:
    mycontainer (char arg) { element = arg; }
    char uppercase()
    {
      if ((element >= 'a' )&& (element <= 'z' ))
      element += 'A' - 'a';
      return element;
    }
};

int main() {
  mycontainer<int> myint(7);
  mycontainer<char> mychar('j');
  cout << myint.increase << endl;
  cout << mychar.uppercase << endl;
  return 0;
}
```

在类的模板特化中使用如下语法:

```cpp
template <> class mycontainer <char> { ... };
```

首先，注意在类模板之前是一个空的参数列表`template <>`。这就是将其明确定声明为模板特化。

但是比这个前缀更中博更要的是类模板名后面的`<char>`特化参数。这个特化参数本身表示类模板特化将要声明的具体类型。注意一般类模板和特化之间的差异：

```cpp
template <class T> class mycontainer { ... };
template <> class mycontainer <char> { ... };
```

第一行为普通模板，第二行为特化。

当为类模板声明特化时，也同样需要定义所有的成员，即使和一般类模板完全一致，因为在一般模板和特化之间的成员不存在“继承”关系。

## 模板的非类型参数

`class`或者`typename`关键字之前的表示类型的模板参数之外，模板也可以有一般的类型参数，类似于函数中的参数。作为示例，如下类模板包含一系列元素：

```cpp
// sequence template
#include <iostream>
using namespace std;

template <class T, int N>
class mysequence {
    T memblock [ N ];
  public:
    void setmember(int x, T value);
    T getmember( int x);
};

template <class T, int N>
void mysequence<T,N>::setmember(int x) {
  memblock[x]=value;
}

template <class T, int N>
T mysequence<T,N>::getmember (int x) {
  return memblock[x];
}

int main () {
  mysequence <int, 5> myints;
  mysequence <double, 5> myfloats;
  myints.setmember(0, 100);
  myfloats.setmember(3, 3.1416);
  cout << myints.getmember(0) << '\n';
  cout << myfloats.getmember(3) << '\n';
  return 0;
}
```

也可以对类模板参数设置默认值，例如，之前的类模板为：

```cpp
template <class T=char, int N=10> class mysequence { .. };
```

可以通过声明设置默认的模板参数创建对象：

```cpp
mysequence<> myseq;
```

等同于：

```cpp
mysequence<char, 10> myseq;
```

## 模板和多文件工程

从编译器的角度而言，模板不是普通的函数或者类。他们按需编译，意味着模板函数的代码直到使用特定的模板参数初始化之前是不会编译的。此时，当需要实例化时，编译器会针对模板中的参数生成一个函数。

当工程越来越大，通常会将程序的代码放在不同的源码我呢见中。这样，接口和事件一般都是分开的。以函数的库作为例子，接口一般由可以执行的函数的原型声明组成。这些一般都被声明在`.h`结尾的“头文件”中，具体实现（函数的定义）在一个独立的c++源代码文件中。

因为模板是按需编译的，这会限制多文件工程的编译：模板类或者函数的实现（定义）必须和声明在统一文件中。也就是不能将接口分开在不同的头文件中，且必须将任何使用模板的接口和实现文件包含进来。

所以在模板实例化进行前不会生成任何代码，编译器会允许同一个模板文件，无论是声明还是定义的，多次包含进来而不会产生链接错误。
