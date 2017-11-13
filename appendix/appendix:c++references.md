# 引用（或者别名）（`&`）

回忆一下C/C++在表达式中使用`&`做为取值操作符。在声明语句中C++赋予`&`更过的含义来声明变量引用。

`&`符号在表达式和声明中的含义不同。当在表达式中使用时，`&`表示取值操作符，返回变量的地址，例如如果`number`是整数变量，`&number`返回`number`变量的地址。

然而，当`&`使用在声明中时（包括函数的正式参数），则其为标识符的一部分，用于声明一个引用变量（或者引用或者别名或者可选名）。用于提供另一个名字、或者另一个引用、或者已知对象的别名。

语法如下：

```cpp
type &newname = existingname;
type& newname = existingname;
type & newname = existingname;
```

这应该读做“newname是existngname的一个引用”。现在你可以用`newname`或者`existingname`来访问变量。

例如：

```cpp
/* Test reference declaration and initialization (TestReferenceDeclaration.cpp) */
#include <iostream>
using namespace std;

int main() {
  int number = 88; // Declare an int variable called number
  int & refNumber = number; // Declare a reference (alias) to the variable number
                            // Both refNumber and number refer to the same valu

  cout << number << endl;    // Print value of variable number (88)
  cout << refNumber << endl;  // Print value of reference (88)

  refNumber = 99;             // Re-assign a new value to refNumber
  cout << refNumber << endl;
  cout << number << endl;     // Value of number also changes (99)

  number = 55;                // Re-assign a new value to number
  cout << number << endl;
  cout << refNumber << endl;  // Value of refNumber also changes (55)

  return 0;
}
```

```
number(int)
+
|
v
+------------------------+
|           88           |
+------------------------+
^
|
+
refNumber(int &)
(A reference or alias to an int variable)
```

## 引用如何工作

引用的工作方式类似于指针。引用是做为变量的别名声明。其存储变量的地址，如图：

```
Name: refNumber(int &)
Address: 0x??????
+
|
v
+-------------------------+              Name: number(int)
|    0x22ccec(&number)    |              Address: 0x22ccec(&number)
+-------------------------++             +
 A reference contain a     |             |
 memory address of         |             v
 a variable                +-----------> +-------------------------+
                                         |           88            |
                                         +-------------------------+
                                          An int variable contains
                                          an int value
```

## 引用和指针

引用和指针很相似，除了：

1. 引用是指向地址的常量。需要在声明时初始化引用。

```cpp
int & iRef; // Error: 'iRef' declared as reference but not initialized
```

一旦对一个变量建立一个引用，就不能将这个引用指向其他变量。

2.获取指针指向的值，需要使用解引用符号`*`（例如，如果`pNumber`为`int`指针，`*pNumber`返回`pNumber`指向的值。其称做解引用）。要将变量地址赋给指针，需要使用取地址符号`&`（例如，`pNumber = &number`）。另一方面，引用和解引用是通过引用内部实现的。例如，如果`refNumber`是指向另一个`int`变量的引用（别名），`refNumber`返回变量的值。无需明确的使用解引用符号`*`。更进一步而言，将变量的地址赋给引用变量中无需使用取值符号`&`。

例如：

```cpp
/* References vs. Pointers (TestReferenceVsPointer.cpp) */
#include <iostream>
using namespace std;

int main() {
   int number1 = 88, number2 = 22;

   // Create a pointer pointing to number1
   int * pNumber1 = &number1;  // Explicit referencing
   *pNumber1 = 99;             // Explicit dereferencing
   cout << *pNumber1 << endl;  // 99
   cout << &number1 << endl;   // 0x22ff18
   cout << pNumber1 << endl;   // 0x22ff18 (content of the pointer variable - same as above)
   cout << &pNumber1 << endl;  // 0x22ff10 (address of the pointer variable)
   pNumber1 = &number2;        // Pointer can be reassigned to store another address

   // Create a reference (alias) to number1
   int & refNumber1 = number1;  // Implicit referencing (NOT &number1)
   refNumber1 = 11;             // Implicit dereferencing (NOT *refNumber1)
   cout << refNumber1 << endl;  // 11
   cout << &number1 << endl;    // 0x22ff18
   cout << &refNumber1 << endl; // 0x22ff18
   //refNumber1 = &number2;     // Error! Reference cannot be re-assigned
                                // error: invalid conversion from 'int*' to 'int'
   refNumber1 = number2;        // refNumber1 is still an alias to number1.
                                // Assign value of number2 (22) to refNumber1 (and number1).
   number2++;
   cout << refNumber1 << endl;  // 22
   cout << number1 << endl;     // 22
   cout << number2 << endl;     // 23
}
```

引用变量对已存在的变量提供一个新的名称。其解引用过程是自动进行的，无需解引用操作符`*`来获取引用的值。另一方面，指针变量存储变量地址。可以改变存储在指针中的地址值。要获取指针指向的值，需要使用解引用符号`*`。引用可以作为常量指针。必须在声明的时候初始化，且其值不可以改变。

引用和指针关系密切。在很多情况下，可以作为指针的其他可选项。引用可以让你使用指针控制一个对象而无需使用引用和解引用的指针语法。

上面的例子展示了引用如何工作，但是没有展示其典型的使用方法——做为函数参数来传递应用。

## 在函数参数中通过应用传递

### 值传递

在C/C++中，函数参数默认是通过值传递的（数组除外，其作为指针传递）。也就是，会拷贝参数的值然后传递给函数。在函数内部对拷贝值进行修改不会影响调用时的传入的参数。也就是，被调用的函数无法访问调用时的变量。例如：

```cpp
/* Pass-by-value into function (TestPassByValue.cpp) */
#include <iostream>
using namespace std;

int square(int);

int main() {
   int number = 8;
   cout <<  "In main(): " << &number << endl;  // 0x22ff1c
   cout << number << endl;         // 8
   cout << square(number) << endl; // 64
   cout << number << endl;         // 8 - no change
}

int square(int n) {  // non-const
   cout <<  "In square(): " << &n << endl;  // 0x22ff00
   n *= n;           // clone modified inside the function
   return n;
}
```

输出表明了他们是两个不同的内存地址。

### 通过指针进行引用传递

很多情况下，我们需要直接修改原始的值（特别是传递大对象或者数据）且避免对值进行拷贝。可以通过给函数传递对象的指针来实现，称做引用传递。例如：

```cpp
/* Pass-by-reference using pointer (TestPassByPointer.cpp) */
#include <iostream>
using namespace std;

void square(int *);

int main()
{
  int number = 8;
  cout << "In main(): " << &number << endl; // 0x22ff1c
  cout << number << endl; // 8
  square(&number);        // Explicit referencing to pass an address
  cout << number << endl; // 64
  return 0;
}

void square(int * pnumber) // Function takes an int pointer (non-const)
{
  cout << "In square(): " << pNumber << endl; // 0x22ff1c
  *pNumber *= *pNumber;                       // Explicit de-referencing to get the value pointed-to
}
```

被调用的方法在同样的内存地址上操作，因此可以改变调用者的变量。

### 通过引用来进程引用传递

除了传递指针给函数外，也可以传递引用给函数，这样可以避免晦涩的引用和解引用语法。例如：

```cpp
/* Pass-by-reference using reference (TestPassByReference.cpp) */
#include <iostream>
using namespace std;

void square(int &);

int main()
{
  int number = 8;
  cout << "In main(): " << &number << endl; // 0x22ff1c
  cout << number << endl; // 8
  square(number);         // Implicit referencing (without '&')
  cout << number << endl; // 64
  return 0;
}

void square(int & rNumber) // Function takes an int reference (non-const)
{
  cout << "In square(): " << &rNumber << endl;  // 0x22ff1c
  rNumber *= rNumber;                           // Implicit de-referencing (without '*')
}
```

再次，初始显示了被执行的函数在同一个地址上进行操作，所以可以更改调用者变量。

注意引用（调用者中）和解引用（在函数中）都是隐含的执行的。和值传递唯一不同点就是函数的参数声明。

之前提到引用在声明的时候进行初始化。对于函数参数的情况下，其是在函数执行的时候初始化引用。

引用主要使用在向函数内或者外传递引用，从而让被调用的函数可以直接访问调用者中的变量。

### `const`函数引用或者指针参数

声明为常量的函数参数在函数内部不能被更改。尽可能使用`const`因为避免你不小心改变参数以及避免很多程序错误。

声明为常量的函数参数可以接受常量和非常量变量。而非常量的引用或者指针参数只能接受非常了参数。例如：

```cpp
/* Test Function const and non-const parameter (FuncationConstParameter.cpp) */
#include <iostream>
using namespace std;

int squareConst(const int);
int squareNonConst(int);
int squareConstRef(const int &);
int squareNonConstRef(int &);

int main() {
   int number = 8;
   const int constNumber = 9;
   cout << squareConst(number) << endl;
   cout << squareConst(constNumber) << endl;
   cout << squareNonConst(number) << endl;
   cout << squareNonConst(constNumber) << endl;

   cout << squareConstRef(number) << endl;
   cout << squareConstRef(constNumber) << endl;
   cout << squareNonConstRef(number) << endl;
   // cout << squareNonConstRef(constNumber) << endl;
       // error: invalid initialization of reference of
       //  type 'int&' from expression of type 'const int'
}

int squareConst(const int number) {
   // number *= number;  // error: assignment of read-only parameter
   return number * number;
}

int squareNonConst(int number) {  // non-const parameter
   number *= number;
   return number;
}

int squareConstRef(const int & number) {  // const reference
   return number * number;
}

int squareNonConstRef(int & number) {  // non-const reference
   return number * number;
}
```

## 传递函数的返回值

### 以引用的方式传递返回值

```cpp
/* Passing back return value using reference (TestPassByReferenceReturn.cpp) */
#include <iostream>
using namespace std;

int & squareRef(int &);
int * squarePtr(int *);

int main() {
   int number1 = 8;
   cout <<  "In main() &number1: " << &number1 << endl;  // 0x22ff14
   int & result = squareRef(number1);
   cout <<  "In main() &result: " << &result << endl;  // 0x22ff14
   cout << result << endl;   // 64
   cout << number1 << endl;  // 64

   int number2 = 9;
   cout <<  "In main() &number2: " << &number2 << endl;  // 0x22ff10
   int * pResult = squarePtr(&number2);
   cout <<  "In main() pResult: " << pResult << endl;  // 0x22ff10
   cout << *pResult << endl;   // 81
   cout << number2 << endl;    // 81
}

int & squareRef(int & rNumber) {
   cout <<  "In squareRef(): " << &rNumber << endl;  // 0x22ff14
   rNumber *= rNumber;
   return rNumber;
}

int * squarePtr(int * pNumber) {
   cout <<  "In squarePtr(): " << pNumber << endl;  // 0x22ff10
   *pNumber *= *pNumber;
   return pNumber;
}
```

在函数声明返回引用的条件下，要避免将函数的本地变量通过引用传递出去。

```cpp
/* Test passing the result (TestPassResultLocal.cpp) */
#include <iostream>
using namespace std;

int * squarePtr(int);
int & squareRef(int);

int main() {
   int number = 8;
   cout << number << endl;  // 8
   cout << *squarePtr(number) << endl;  // ??
   cout << squareRef(number) << endl;   // ??
}

int * squarePtr(int number) {
   int localResult = number * number;
   return &localResult;
      // warning: address of local variable 'localResult' returned
}

int & squareRef(int number) {
   int localResult = number * number;
   return localResult;
      // warning: reference of local variable 'localResult' returned
}
```

上面的程序中存在严重的逻辑问题，函数的本地变量通过引用的方式返回出去。本地变量在函数内存在本地作用域，且在函数退出时就被销毁了。GCC编译能够给出警告（但是不是错误）。

将参数中传入的引用返回回去是安全的。

## 总结

指针和引用是非常复杂的，并且难以掌握。但是他们可以很高的提示程序的效率。

对于新手，要避免在程序中使用指针。不适当的使用会导致严重的逻辑问题。但是你需要理解通过指针和引用这种值传递形式的语法，因为他们在很多函数中被使用了。

* 在值传递中，会将值拷贝一份传递给函数。调用者的变量不能被更改。
* 在引用传递中，将指针传递给函数。调用者的变量可能会在函数内部被改变。
* 在使用引用作为参数的引用传递中，使用变量名作为参数。
* 在使用指针作为参数的引用传递中，需要使用`&varName`（一个内存地址）作为参数。