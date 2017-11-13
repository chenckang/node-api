# 类和多态

## 类

`class`是数据结构体的拓展概念：其可以持有数据和函数。

对象为类的实例。从变量的角度而言，类就是数据类型，而对象为变量。

类一般通过`class`关键字来声明，格式如下：

```cpp
class class_name {
  access_specifier_1:
    member1;
  access_specifier_2:
    member2;
  ...
} object_names;
```
`class_name`为类的一个有效标识符，`object_name`为类的可选的名字列表。声明体中可以包含成员，其要么是数据或者函数声明，以及可选的访问说明符。

其类似于数据结构体的声明，除了其可以包含函数和成员，但是也包含访问说明符。访问说明符为如下三个关键字之一：`private`、`public`和`protected`。这些说明符会改变在其后的访问权限：

* `private`成员只能在同一个类或者这些类的`friends`实体中访问
* `protected`成员可以在同一个类或者这些类的`friends`实体中访问，也可以从其衍生类中来访问。
* 最后，`public`成员可以在所有对象存在的地方访问。

默认，类的所有成员是`private`成员。因此，所有在访问标识符之前声明的成员都为`private`。例如：

```cpp
class CRectangle {
  int x, y;
  public:
    void set_values(int, int);
    int area(void);
} rect;
```

声明一个`CRectangle`类，以及该类的一个对象`rect`。这个类有四个成员：两个`int`型的`private`数据成员（成员`x`和成员`y`)（因为`private`为默认的访问级别），以及两个`public`成员函数：`set_values`和`area()`，且其中只包含它们的声明，而不是定义。

注意，类名和对象名之间的差异：在之前的例子中，`CRectangle`为类名（也就是类型），而`rect`为类型`CRectangle`的对象。其和如下的`int`和`a`的关系是一致的：

```cpp
int a;
```

其中，`int`为类型名（对应类），而`a`为变量名（对应对象）。

在之前的`CRectangle`和`rect`声明之后，在程序的主体中，可以访问`rect`对象的共有成员，如同它们为普通函数和普通变量，只需在对象名和成员名之间加上点（`.`）符号。类似于对普通的数据结构体一样：

```cpp
rect.set_values(3,4);
myarea = rect.area();
```

`rect`中不能在类外部访问的成员为`x`和`y`，因为其为`private`成员所以只能在同一个类的其他成员中访问。

如下为类`CRectangle`的完整实例：

```cpp
// classes example
#include <iostream>
using namespace std;

class CRectangle {
    int x, y;
  public:
    void set_values(int, int);
    int area() { return (x * y);}
}

void CRectangle::set_values (int a, int b) {
  x = a;
  y = b;
}

int main() {
  CRectangle rect;
  rect.set_values(3, 4);
  cout << "area: " << rect.area();
  return 0
}
```

这段代码中，最重要的就是在`set_values()`定义中的域操作符（`::`，两个冒号）。其用于在类之外定义其类的成员。

注意调用`rect.area()`和`rectb.area()`的结果并不一样。这是因为每个类`CRectangle`的对象都有其自身的`x`和`y`，同样它们也都有其自身的函数成员`set_value()`和`area()`来操作对象本身拥有的成员变量。

如上为面向对象编程的基本概念：数据和函数都为对象的成员。无需使用全局变量在函数之间作为参数来回传递，取而代之的是对象有其自身的嵌入数据以及函数作为成员。注意，无需一定在调用`rect.area()`和`rectb.area()`时传入参数。这些成员函数可以直接使用相应实例`rect`和`rectb`的数据成员。

### 构造器和析构器

在进程创建是对象需要初始化和动态分配内存，来完成交互和避免执行时返回异常结果。例如，如果在之前的例子中，在调用`set_values()`之前执行`area()`时会怎样？很可能会得到不确定的结果，因为`x`和`y`成员未赋值有关。

要避免如此，类可以包含称为`constructor`的特殊函数，其在类中创建新对象时自动调用。构造函数的函数名必须和类的名称一样，不可以有返回类型，即使`void`也不行。

如下，通过构造器来实现`CRectangle`：

```cpp
// example: class constructor
#include <iostream>
using namespace std;

class CRectangle {
    int width, height;
  public:
    CRectangle(int, int);
    int area() { return (width * height);}
};

CRectangle::CRectangle(int a, int b) {
  width = a;
  height = b;
}

int main() {
  CRectangle rect (3, 4);
  CRectangle rectb (5, 6);
  cout << "rect area: " << rect.area() << endl;
  cout << "rectb area: " << rectb.area() << endl;
  return 0;
}
```

如你所见，这个例子中结果和之前的例子是一致的。但是现在移除了`set_values()`函数，而是使用构造器来取代之提供类似的功能：其传递给构造器的参数初始化`width`和`height`。

注意这些参数在对象创建时传递给构造器：

```cpp
CRectangle rect (3, 4);
CRectangle rectb (5, 6);
```

构造函数不能像普通成员函数那样被调用。只在类的新对象被创建时会执行。

可以看到构造器的原型声明（类中）和之后的勾在其实现中都不包含返回值；包括`void`。

析构器完成相反的功能。在对象被销毁时自动执行，在其作用域退出时（例如，如果在函数中作为本地对象，且函数执行结束）或者动态分配的对象并使用`delete`来释放。

析构器必须和类包含同样的名字，同时必须在其前面加上波浪符号`~`，且也必须不包含返回值。

析构函数在对象动态分配内存后销毁时用于将这些占用内存释放。

```cpp
// example on constructors and destructors
#include <iostream>
using namespace std;

class CRectangle {
    int *width, *height;
  public:
    CRectangle (int, int);
    ~CRectangle ();
    int area() {return (*width, *height);}
}

CRectangle::CRectangle (int a, int b) {
  width = new int;
  height = new int;
  *width = a;
  *height = b;
}

CRectangle::~CRectangle(int a, int b) {
  delete with;
  delete height;
}

int main() {
  CRectangle rect(3, 4), rectb (5, 6);
  cout << "rect area: " << rect.area() << endl;
  cout << "rectb area: " << rectb.area() << endl;
  return 0;
}
```

### 重载构造器

如同其他函数一样，构造器也可以被重载成多个形式，主要其参数类型或者数量不同即可。针对重载的函数，编译器会执行函数调用中参数匹配的函数。在构造器的上，当创建新对象调用构造器时也同样适用。

```cpp
// overload class constructors
#include <iostream>
using namespace std;

class CRectangle {
    int width, height;
  public:
    CRectangle();
    CRectangle(int, int);
    int area(void) { return (width* height); }
};

CRectangle::CRectangle() {
  width = 5;
  height = 5;
}

CRectangle::CRectangle(int a, int b) {
  width = a;
  height = b;
}

int main() {
  CRectangle rect (3, 4);
  CRectangle rectb;
  cout << "rect area: " << rect.area() << endl;
  cout << "rectb area: " << rectb.area() << endl;
  return 0;
}
```

在这个情况下，`rectb`在声明时未使用任何参数，所以其使用没有参数的构造器初始化，将`width`和`height`初始化置为5。

重要：注意如何使用默认的构造器来声明一个新对象（无参数的构造器），此时不要使用括号`()`：

```cpp
CRectangle rectb; // right
CRectangle rectb(); // wrong
```

### 默认构造器

如果在类的定义中不声明任何构造器，则编译器会假定类具有无参数的默认构造器。因此，如下来声明一个类：

```cpp
class CExample {
  public:
    int a, b, c;
    void multiply (int n, int m) {a = n; b = m; c = a * b; }
}
```

编译器假设`CExample`有个默认的构造器，所以可以定义这个类的一个实例而不用使用任何参数：

```cpp
CExample ex;
```

但是一旦对类声明了构造器，编译器则不再提供默认的构造函数。所以，要声明所有的类的对象时，必须针对构造函数的原型提供相应的参数。


```cpp
class CExample {
  public:
    int a, b, c;
    CExample (int n, int m) { a = n; b = m; };
    void multiply () { c = a * b; };
}
```

这里我们声明了一个含有两个`int`类型参数的构造器。因此，如下的对象声明则为有效的：

```cpp
CExamble ex (2, 3);
```

但是

```cpp
CExample ex;
```

则为无效的，因为类的声明中存在明确的构造器，因此会替换掉默认的构造器。

但是编译器不仅仅会在你没有明确定义构造器时创建一个默认的构造器，其也会针对你未声明相应特殊函数时创建三个特殊成员函数。它们时复制构造器、赋值操作符以及默认的构造器。

复制构造器和赋值操作符将一个对象内的所有数据拷到当前对象中。例如，编译器声明的复制构造器会类似于如下：

```cpp
CExample::CExample (const CExample& rv) {
  a = rv.a; b = rv.b; c = rv.c;
}
```

因此，如下对象声明为有效的：

```cpp
CExample ex (2, 3);
CExample ex2 (ex);
```

### 指向类的指针

可以创建一个指向类的指针。只需要考虑的是一旦声明，则类就为一个有效的类型，所以可以针对指针使用类名。例如：

```cpp
CRectangle * prect;
```

其为指向`CRectangle`类的对象的指针。

如同在结构体中的一样，为了直接引用一个指针指向的对象的成员，可以使用箭头操作符`->`，如下为一些可能的例子：

```cpp
// pointer to classes example
#include <iostream>
using namespace std;

class CRectangle {
    int width, height;
  public:
    void set_values(int, int);
    int area(void) {
      return (width * height);
    }
};

void CRectangle::set_values (int a, int b) {
  width = a;
  height = b;
}

int main() {
  CRectangle a, *b, *c;
  CRectangle * d = new Rectangle[2];
  b = new CRectangle;
  c = & a;
  a.set_values(1, 2);
  b->set_values(3, 4);
  d->set_values(5, 6);
  d[1].set_values(7, 8);
  cout << "a area: " << a.area() << endl;
  cout << "*b area: " << b->area() << endl;
  cout << "*c area: " << c->area() << endl;
  cout << "d[0] area: " << d[0].area() << endl;
  cout << "d[1] area: " << d[1].area() << endl;
  delete [] d;
  delete b;
  return 0;
}
```

### 使用`struct`和`union`定义类

不仅仅可以通过`class`关键字来定义类，也可以通过`struct`和`union`关键字来定义。

类和结构体的概念类似，所以两个关键词（`struct`和`class`）可以在C++中由来定义类（即，`struct`定义中可以含有函数成员，而不仅仅是数据成员）。唯一的不同是`class`声明中类的成员默认是`private`，而`struct`中默认是`public`的。其他方面，二者是等同的。

`union`的概念和使用`class`和`struct`定义的类不同，因为`union`只能含有一个数据成员，但它们也可以含有成员函数。默认的成员的访问属性为`public`。

### 重载操作符

C++引入了使用标准操作符来在基本类型之外执行类之间的操作。例如：

```cpp
int a, b, c;
a = b + c;
```

这很显然是有效的C++代码，因为加法运算的不同变量都为基本类型。然而，如下类似操作则不那么明显：

```cpp
struct {
  string product;
  float price;
} a, b, c;
a = b + c;
```

实际上，这会导致编译错误，因为我们没有定义类的加法操作行为。然而，多亏了C++引入了操作符重载的特征，我们可以使用基本的操作符来完成类之间的操作。如下为可以重载的操作符列表：

| 可重载操作符 |
| --- |
| `+ - * / = < > += -= *= /= << >> <<= =>> == != <= >= ++ -- % & ^ ! | - &= ^= |= && || %= [] () . ->* -> new delete new[] delete[]`|

要使用那个操作符重载，需要在列中定义操作符函数，其为普通的函数但是函数名由`operator`关键字加上操作符构成。格式为：

```cpp
type operator sign (parameters) { /* ... */ }
```

如下为重载加号操作符的例子。这里要创建一个存储二维向量的类，然后将两个实例相加：`a(3, 1)`和`b(1, 2)`。两个二维向量相加如同将`x`座标相加等到最终的`x`座标以及将`y`座标相加得到最终的`y`左边一样简单。这里结果为`(3 + 1, 1 + 2) = (4, 3)`。


```cpp
// vector: overloading operators example
#include <iostream>
using namespace std;

class CVector {
  public:
    int x, y;
    CVector () {};
    CVector (int, int);
    CVector operator + (CVector);
}

CVector::CVector (int a, int b) {
  x = a;
  y = b;
}

CVector CVector::operator+ (CVector param) {
  CVector temp;
  temp.x = x + param.x;
  temp.y = y + param.y;
  return (temp);
}

int main () {
  CVector a (3, 1);
  CVector b (1, 2);
  CVector c;
  c = a + b;
  cout << c.x << ", " << c.y;
  return 0;
}
```

函数`operator+`为类`CVector`中重载加号运算符的函数。这个函数要么通过操作符来调用，要么通过函数名来调用：

```cpp
c = a + b;
c = a.operator+(b);
```

两个表达式是等同的。

注意，这其中包含了空构造器（没有参数），并使用了空区块来定义之：

```cpp
CVector () {};
```

这是必要的，以为我们定义了另一个构造器：

```cpp
CVector (int, int);
```

当明确的定义了一个构造器，使用任意个数的参数，默认的有编译器定义的无参数构造器就不再被自动声明了，所以我们要手动定义从而可以在构建参数的时候不使用参数。否则，如下声明：

```cpp
CVector () {}:
```

在`main()`中则为无效的。

无论如何，必须要说明的是使用空语句块来实现构造函数是一种反模式，因为其并不能满足构造函数通常所期望的功能，来初始化类中所有的成员变量。在这个情况下，这个构造函数使得`x`和`y`变量均为未定义的。因此，如下的定义是比较可取的：

```cpp
CVector() {x = 0; y = 0;}
```

和类包含默认的构造器和复制构造器一样，其也包含默认的赋值操作符（`=`）重载函数，其参数类型为类本身。这个的行为默认行为就是将对象作为参数，从而将对象的全部数据成员拷贝到表达式左边：

```cpp
CVector d (2, 3);
CVector e;
e = d;
```

复制操作符函数是唯一被默认实现的操作符重载成员函数。当然你也可以重现定义其为自己想要的函数，例如，值拷贝类的部分成员或者执行额外的初始化动作。

操作符重载中不一定要操作符合数学或者通常意义上的操作符定义，尽管推荐这么做。例如，代码可以非常不直观的将操作符`+`号用来实现两个类的减法，使用`==`来实现类的初始填充。

尽管函数`operator+`的原型是比较直观的实现，因为其操作符的右边为操作符成员函数的参数，其左边为对象本身，其他符号可能不会这么直观。如下为展示不同的操作符函数的声明总结（每个例子中用`@`替换操作符）：

| 表达式 | 操作符 | 成员函数 | 全局函数 |
| --- | --- | --- | --- |
| @a | + - * & ! ~ ++ -- | A::operator@() | operator@(A) |
| a@ | ++ -- | A::operator@(int) | operator@(A, int) |
| a@b | + - * / % ^ & | < > == != <= >= << >> && || | A::operator@(B) | operator@(A, B) |
| a@b | = += -= *= /= %= ^= &= != <<= >>= [] | A::operator@(B) | - |
| a(b, c...) | () | A::operator() (B, C, ...) | - |
| a->x | -> | A::operator->() | - |

其中`a`是类`A`的对象，`b`是类`B`的对象，`c`是类`C`的对象。

从表格中可以看到有两种方式来重载操作符：作为成员函数以及作为全局函数。


### 关键词`this`

关键词`this`表示指向当前对象的指针，其成员函数正在被执行。

其中一个使用方式就是检查传入的参数是否是对象本身。例如：

```cpp
// this
#include <iostream>
using namespace std;

class Dummy {
  public:
    int isitme(CDummy& param);
}

int CDummy::isitme(CDummy& param)
{
  if (&param == this) return true;
  else return false;
}

int main() {
  CDummy a;
  CDummy* b = &a;
  if (b->isitme(a))
    cout << "yes. &a is b";
  return 0;
}
```

在`operator=`中也经常会使用，返回对象的引用（避免使用临时对象）。如下向量的实例可以使用如下的`operator=`函数来实现：

```cpp
CVector& CVector::operator=(const CVector& param)
{
  x = param.x;
  y = param.y;
  return *this;
}
```

实际上，这个函数类似于编译器在这个类没有定义这个函数时产生的默认函数的代码。

### 静态成员

类可以含有静态成员，可以为数据或者函数。

类的数据成员可以认为是“类变量”，因为对所有的该类的对象都是同一个值。

例如，其可以用于在类中存储变量来计数该类的当前分配的实例个数，如下面例子：

```cpp
// static members in classes
#include <iostream>
using namespace std;

class Dummy {
  public:
    static int n;
    CDummy() {n++}
    ~CDummy() {n--}
}

int CDummy::n = 0;

int main() {
  CDummy a;
  CDummy b[5];
  CDummy * c = new CDummy;
  cout << a.n << endl;
  delete c;
  cout << CDummy::n << endl;
  return 0;
}
```

实际上，静态成员和全局变量具有相似的属性，但是其在类的作用域中。对此，要避免多次声明静态变量，仅可以在类的声明中包含静态变量的声明，而不能在类的定义（初始化）中。为了初始化静态数据成员，必须将正式定义放在类之外，在全局作用域中，如之前例子：

```cpp
int CDummy::n = 0;
```

因为对于同一个类的所有对象，其具有唯一的变量值，可以被作为该类的任何对象的成员，甚至直接使用类名来访问（仅对静态成员有效）：

```cpp
cout << a.n;
cout << CDummy::n;
```

这两个调用指向的是统一个变量：类`CDummy`的静态变量`n`，被所有该类的对象共享。

也可以包含静态的方法。他们仿佛为全局函数，制式作为给定类的成员。他们仅能访问静态数据成员，而不能访问类的非静态成员，同样不能使用关键字`this`，因为其指向对象的指针，而这些函数实际上不是任何对象的成员而直接是类的成员。

## 关系和继承

### 关系函数

原则上，私有和保护成员函数不能在类的声明之外使用。但是这个原则不适用于关系者。

关系者为以关键字`friend`声明的函数或者类。

如果要将一个外部函数定义为类的关系函数，这样可以让这个函数访问到类的私有和保护成员，可以通过在类中声明外部函数的原型，并加上`friend`关键字来实现

```cpp
// friend functions
#include <iostream>
using namespace std;

class CRectangle {
    int width, height;
  public:
    void set_values(int, int);
    int area() {return (width * height);}
    friend CRectangle duplicate (CRectangle);
}

void CRectangle::set_values(int a, int b) {
  width = a;
  height = b;
}

CRectangle duplicate(CRectangle rectparam) {
  CRectangle rectres;
  rectres.width = rectparam.with * 2;
  rectres.height = rectparam.height * 2;
  return (rectres);
}

int main() {
  CRectangle rect, rectb;
  rect.set_values(2, 3);
  rectb = duplicate(rect);
  cout << rectb.areas();
  return 0;
}
```

`duplicate`函数为类的关系函数。所以在这个函数中可以访问到`CRectangle`类不同实例的`width`和`height`私有成员。注意，无论在`duplicate()`声明和之后在`main()`中的使用，`duplicate`都不做为类`CRectangle`的成员函数。其只不过是可以访问类的私有成员和保护成员而不作为类的成员的函数。

关系函数可以用于在不同的类之间执行操作。一般而言，使用关系函数并不符合面向对象方法，所以应该尽可能的避免这种方法，而是用类的成员函数来完成。例如如上的例子中，如果将`duplicate`作为类的成员函数则代码会更加简洁。

### 关系类

如同关系函数一样，可以定义关系类，从而可以使一个类可以访问另一个类的私有或者保护成员。

```cpp
// friend class
#include <iostream>
using namespace std;

Class CSquare;

Class CRectangle {
    int width, height;
  public:
    int area() { return (width * height) ;};
    void convert(CSquare a);
};

Class CSquare {
  private:
    int side;
  public:
    void set_side(int a) { size = a; }
    friend class CRectangle;
}

void CRectangle::convert(CSquare a) {
  width = a.side;
  height = a.side;
}

int main() {
  CSquare sqr;
  CRectangle rect;
  sqr.set_side(4);
  rect.convert(sqr);
  cout << rect.area();
  return 0;
}
```
这个例子中，我们将类`CRectangle`作为`CSquare`的关系函数，所以`CRectangle`的成员函数可以访问`CSquare`的保护和私有成员，也就是`CSquare::side`。

你可能注意到了，在成语的开始位置上有一个类`CSquare`的空声明。这是必要的，因为在类`CRectangle`的声明中，我们引用了`CSquare`（在`convert()`中作为参数）。`CSquare`类在之后进行了定义，所以如果不在对类`CSquare`进行空的声明，则在类`CRectangle`中就无法访问`CSquare`。

注意，关系类并不是对等的，除非明确指定关系。在这个例子中，`CRectangle`为类`CSqaure`的关系类，反之则不适用，所以`CRectangle`可以访问`CSquare`的私有和保护成员，但是`CSquare`却不能访问`CRectangle`的。当然也可以将`CSquare`作为`CRectangle`的关系类。

关系函数的另一个特性就是不能传递：一个类的关系类的关系类并不是该类的关系类，除非明确指定。

### 在类中继承

C++中类的关键的一个特性就是继承。继承允许创建其他类的衍生类。所以这些类自动的将父类的成员引入进来，在加上其自身的成员。例如，假设我们创建一系列形容形状的类，如`CRectangle`和`CTriangle`。他们有些共性，例如可以用两个指标表示：高长和底长。

在类中，可以用`CPolygon`来表示，并衍生出其他两个类`CRectangle`和`CTriangle`。

```
           +-----------------+
           |                 |
           |     CPolygon    |
           |                 |
        +--+-----------------+---+
        |                        |
+-------v--------+     +---------v--------+
|                |     |                  |
|   CRectangle   |     |     CTriangle    |
|                |     |                  |
+----------------+     +------------------+
```

类`CPolygon`包含两个形状类型的公共成员。在这个例子中为：`width`和`height`。`CRectangle`和`CTriangle`为其衍生类，其包含自身区别于其他形状类型的属性。

从其他类衍生而来的类会包含其基类的所有可访问成员。这意味着如果我们在基类中包含成员`A`并将其衍生为另一个包含`B`的类，衍生的类会包含两个成员`A`和`B`。

要从另一个类中衍生一个类，在衍生类的声明中使用冒号`:`来表示，如下格式：

```cpp
class derived_class_name: public base_class_name
{ /*...*/ }
```

这其中`derived_class_name`为衍生的类名，`base_class_name`为基类的类名。`public`访问控制符可以由其他访问控制符`protected`和`private`取代。访问控制符可以限制从基类继承的成员的最大访问级别：访问级别更加松散的成员继承中会使用这个访问控制级别，而访问等级更加严格的成员会在衍生类中保持其访问级别。

```cpp
// derived class
#include <iostream>
using namespace std;

class CPolygon {
  protected:
    int width, height;
  public:
    void set_values(int a, int b) { width = a; height = b; }
};

class CRectangle: public CPolygon {
  public:
    int area() { return (width * height); }
};

class CTriangle: public CPolygon {
  public:
    int area() { return (width * height / 2); }
};

int main() {
  CRectangle rect;
  CTriangle trgl;
  rect.set_values(4, 5);
  trgl.set_values(4, 5);
  cout << rect.area() << endl;
  cout << trgl.area() << endl;
  return 0;
}
```

类`CRectangle`和类`CTriangle`的实例包含从`CPolygon`继承而来的成员：`width`、`height`以及`set_values`。

`protected`和`private`访问级别类似。不同点在于继承中，但类从另一个类继承而来，其可以访问基类的`proctected`成员，但不能访问`private`成员。

因为我们向让`width`和`height`在派生类中也可以被访问，所以使用`protected`来取代`private`。

不同的访问类型和访问控制的对应关系如下：

| 访问者 | public | protected | private |
| --- | --- | --- | --- |
| 同一个类的其他成员 | yes | yes | yes |
| 派生类的成员 | yes | yes | no |
| 非成员 | yes | no | no |

“非成员”表示从类的外面访问，例如`main()`函数中、从其他的类或者从其他的函数。

在上面的例子中`CRectangle`和`CTriangle`对基类的成员的访问级别和基类对这些成员的访问级别一致。

```cpp
CPolygon::width // protected
CRectangle::width // protected

CPolygon::set_values() // public access
CRectangle::set_values() // public access
```

这是因为我们用关键字`public`类定义派生类和基类的继承关系：

```cpp
class CRectangle: public CPolygon { ... }
```

冒号`:`后面的关键字`public`表示将从其后面的类（`CPolygon`)的成员中继承最大的访问级别。

如果我们使用更加严格的访问级别`protected`，则基类中的所有的`public`成员会作为`protected`成员继承。如果使用最严格的`private`访问级别，则基类中的所有成员会作为`private`继承。

如果没有指定访问级别关键字，则编译器会对以`class`声明的类使用`private`关键词，对`struct`声明的类使用`public`关键词。

### 从基类继承那些内容

原则上，会从基类继承所有的成员，除了：

* 构造器和析构器
* `operator=`成员
* 关系（`friend`）实体

尽管基类的构造器和析构器不会自动继承，但是当派生类的创建新的实例或者销毁实例时，会调用基类的默认构造器（无参数的那个构造器）以及析构器。

如果基类没有默认的构造器，或者你想要执行重载的构造器，可以在每个派生类的构造器中指定：

```cpp
derived_constructor_name(parameters): base_constructor_name(parameters) {}
```

例如：

```cpp
// constructors and derived classes
#include <iostream>
using namespace std;

class mother {
  public:
    mother ()
      { cout << "mother: no parameters\n"; }
    mother (int a)
      { cout << "mother: int parameter\n"; }
};

class daughter : public mother {
  public:
    daughter (int a)
      { cout << "daughter: int parameter\n\n"; }
};

class son : public mother {
  public:
    son (int a) : mother (a)
      { cout << "son: int parameter\n\n"; }
};

int main () {
  daughter cynthia (0);
  son daniel(0);

  return 0;
}
```

注意当`daughter`对象和`son`对象被创建时，调用的`mother`的构造函数的不同。其不同主要是因为`daughter`和`son`的声明方式：

```cpp
daught(int a);
son(int a): mother(a);
```

### 多继承

在C++中，完全可以从多个类继承。可以简单的通过在类的声明中用逗号将多个基类分隔开来。例如，有一个特定的类用于在屏幕中打印（`COutput`），并且让`CRectangle`和`CTriangle`在`CPolygon`之外也继承这个类：

```cpp
class CRectangle: public CPloygon, public COutput;
class CTriangle: public CPolygon, public COutput;
```

如下为完整的例子：

```cpp
// multiple inheritance
#include <iostream>
using namespace std;

class CPolygon {
  protected:
    int width, height;
  public:
    void set_values(int a, int b) {
      width = a;
      height = b;
    }
};

class COutput {
  public:
    void output (int i);
}

void COutput::output(int i) {
  cout << i << endl;
}

class CRectangle: public CPolygon, public COutput {
  public:
    int area() {
      return (width * height);
    }
};

class CTriangle: public CPolygon, public COutput {
  public:
    int area() {
      return (width * height / 2);
    }
};

int main () {
  CRectangle rect;
  CTriangle trgl;

  rect.set_values(4, 5);
  trgl.set_values(4, 5);
  rect.output(rect.area());
  trgl.output(trgl.area());
  return 0;
}
```

## 多态

在进入本节之前，建议先对指针和类继承有适当的理解。如果如下表格内的内容让你觉得陌生，那你就需要复习相关的章节：

| 语句 | 章节 |
| --- | --- |
| int a::b(int c) {} | 类 |
| a->b | 数据结构体 |
| class a: public b{} | 关系和继承 |

### 指向基类的指针

派生类的重要特征是指向派生类的指针和执行基类的指针在类型上是兼容的。多态就是利用这个简单但是强大并常用的特性，将面向对象的方法潜力发挥出来。

下面通过将上面例子中长方形和三角形的例子引入来说明指针兼容这个特性。

```cpp
// pointer to base class
#including <iostream>
using namespace std;

class CPolygon {
  protected:
    int width, height;
  public:
    void set_values(int, int) {
      width = a;
      height = b;
    }
};

class CRectangle: public CPolygon {
  public:
    int area() {
      return (width * height);
    }
};

class CTriangle: public CPolygon {
  public:
    int area() {
      return (width * height / 2);
    }
};

int main() {
  CRectangle rect;
  CTriangle trgl;
  CPolygon * ppoly1 = &rect;
  CPolygon * ppoly2 = &trgl;
  ppoly1->set_values(4, 5);
  ppoly2->set_values(4, 5);
  cout << rect.area() << endl;
  cout << trgl.area() << endl;
  return 0;
}
```

在`main`函数中，创建了两个指向类`CPolygon`的对象的指针（`ppoly1`和`ppoly2`）。将后将`rect`和`trgl`的引用传递给这些指针，因为二者都是派生自`CPolygon`类的对象。

使用`*ppoly1`和`*ppoly2`取代`rect`和`trgl`的唯一限制就是`*ppoly1`和`*ppoly2`的类型为`CPolygon*`，因此只能使用这些指针访问`CRectangle`和`CTriangle`从`CPolygon`继承而来的成员。因此当调用`area()`成员时不得不直接使用`rect`和`trgl`而不是`*ppoly1`和`*ppoly2`。

要在`CPolygon`上使用`area()`成员，则也要在类`CPolygon`中声明这个成员，而不仅仅是其派生类中，这里的问题在于`CRectangle`和`CTriangle`使用了不同版本的`area()`，因此不能在基类中实现这个方法。此时虚成员就派上用场。

### 虚成员

类的成员可以在派生类中重新定义，这就是虚成员。要定义一个类的虚成员，必须在其声明中加上`virtual`关键字：

```cpp
// virtual members
#include <iostream>
using namespace std;

class CPolygon {
  protected:
    int width, height;
  public:
    void set_values(int a, int b) {
      width = a;
      height = b;
    }
    virtual int area() {
      return 0;
    }
};

class CRectangle: public CPolygon {
  public:
    int area() {
      return (width * height);
    }
};

class CTriangle: public CPolygon {
  public:
    int area() {
      return (width * height / 2);
    }
};

int main() {
  CRectangle rect;
  CTriangle trgl;
  CPolygon poly;
  CPolygon * ppoly1 = &rect;
  CPolygon * ppoly2 = &trgl;
  CPolygon * ppoly3 = &poly;
  ppoly1->set_values(4, 5);
  ppoly2->set_values(4, 5);
  ppoly3->set_values(4, 5);
  cout << ppoly1->area() << endl;
  cout << ppoly2->area() << endl;
  cout << ppoly3->area() << endl;
  return 0;
}
```

现在三个类（`CPolygon`、`CRectangle`和`CTriangle`）所具备的成员都一样：`width`、`height`、`set_values()`和`area()`。

成员函数`area()`在基类中作为虚拟函数声明，应为其在派生类中会被重新定义。如果你将`virtual`关键字从`area()`的声明中移除，则会看到三个多边形实例的输出结果都为0，而不是20、10和0。这是因为相对于执行每个实例的`area()`的函数（`CRectangle::area()`、`CTriangle::area()`和`CPolygon::area()`），而真正执行在所有的情况下会执行`CPolygon::area()`，因为这些指针指向的类型为`CPolygon*`。

因此，`virtual`关键字所做的就是派生类和基类的同名成员函数可以从指针调用中调用适当的函数，更明确的说就是当指针的类型为基类，但是其指向的是派生类的对象时会执行适当的函数。

声明或者实现了虚函数的类被称做多态类。

注意在其虚拟性之外，也可以声明`CPolygon`的实例，并执行其自身的`area()`函数，其返回0。

### 抽象基类

抽象基类类似于之前的例子中的`CPolygon`。不同的是在之前的例子中为类`CPolygon`的对象定义类一个有效的基础功能函数`area()`，而抽象基类可以完全不实现`area()`函数。这可以通过给函数声明增加`=0`来完成。

抽象基类`CPolygon`可以类似于如下：

```cpp
// abstrat class CPolygon
class CPolygon {
  protected:
    int width, height;
  public:
    void set_values (int a, int b){
      width = a;
      height = b;
    }
    virtual int area() =0;
};
```

注意这里个`virtual int area()`追加了`=0`而没有对函数进行实现。这中函数称做纯虚拟函数，所有包含一个或多个纯虚拟函数的类都称之为抽象基类。

抽象基类和一般的多态类的一个不同指出就是不能对抽象基类创建实例，这是因为抽象基类中至少有一个成员缺少具体实现。

但是一个不能初始化对象的类并不是无用的。可以对其声明指针，然后使用多态的特性。因此，如下声明：

```cpp
CPolygon poly;
```

对于声明的抽象基类是无效的，因为其试图创建一个对象。

然而，如下指针：

```cpp
CPolygon * ppoly1;
CPolygon * ppoly2;
```

则完全有效。

这是因为`CPolygon`中包含纯虚拟函数，因此是一个抽象基类。然而，抽象基类的指针则可以用于指向该类的派生类。

如下为完整示例：

```cpp
// abstract base class;
#include <iostream>
using namespace std;

class CPolygon {
  protected:
    int width, height;
  public:
    void set_values(int a, int b) {
      width = a;
      height = b;
    }
    virtual int area(void) =0;
};

class CRectangle: public Cpolygon {
  public:
    int area(void) {
      return (width * height);
    }
};

class CTriangle: public CPolygon {
  public:
    int area(void) {
      return (width * height / 2);
    }
};

int main () {
  CRectangle rect;
  CTriangle trgl;
  CPolygon * ppoly1 = &rect;
  CPolygon * ppoly2 = &trgl;
  ppoly1->set_values(4, 5);
  ppoly2->set_values(4, 5);
  cout << ppoly1->area() << endl;
  cout << ppoly2->area() << endl;
  return 0;
}
```

可以看到针对相关类的不同对象使用了同一个指针类型（`CPolygon*`)。这其中用处很多。例如，现在我们对抽象基类`CPolygon`创建一个函数成员来答应`area()`函数的结果，即使`CPolygon`本身没有实现这个方法也可以完成。

```cpp
// pure virtual members can be called
// from the abstract base class
#include <iostream>
using namespace std;

class CPolygon {
  protected:
    int width, height;
  public:
    void set_values(int a, int b) {
      width = a;
      height = b;
    }
    virtual int area(void) =0;
    void printarea(void) {
      cout << this->area() << endl;
    }
};

class CRectangle: public CPolygon {
  public:
    int area(void) {
      return (width * height);
    }
};

class CTriangle: public CPolygon {
  public:
    int area(void) {
      return (width * height / 2);
    }
};

int main() {
  CRectangle rect;
  CTriangle trgl;
  CPolygon * ppoly1 = &rect;
  CPolygon * ppoly2 = &trgl;
  ppoly1->set_values(4, 5);
  ppoly2->set_values(4, 5);
  ppoly1->printarea();
  ppoly2->printarea();
  return 0;
}
```

虚成员和抽象类让C++具备多态的特性，可以在大型项目中充分利用面向对象编程的方法，我们已经了解了这些特性的基本使用方法，但是这些也可以用在对象数组或者动态分配对象上。

下面使用动态分配对象来展示：

```cpp
// dynamic allocation and polymorphism
#include <iostream>
using namespace std;

class CPolygon {
  protected:
    int width, height;
  public:
    void set_values(int a, int b) {
      width = a;
      height = b;
    }
    virtual int area(void) =0;
    void printarea(void) {
      cout << this.area() << endl;
    }
};

class CRectangle: public CPolygon {
  public:
    int area(void) {
      return (width * height);
    }
};

class Triangle: public CPolygon {
  public:
    int area(void) {
      return (width * height / 2);
    }
};

int main() {
  CPolygon * ppoly1 = new CRectangle;
  CPolygon * ppoly2 = new Triangle;
  ppoly1->set_values(4, 5);
  ppoly2->set_values(4, 5);
  ppoly1->printarea();
  ppoly2->printarea();
  delete ppoly1;
  delete ppoly2;
  return 0;
}
```

注意`ppoly`指针：

```cpp
CPolygon * ppoly1 = new CRectangle;
CPolygon * ppoly2 = new CTriangle;
```

使用指向`CPolygon`的指针声明，不过动态分配的对象声明会直接具备扩展类的类型。
