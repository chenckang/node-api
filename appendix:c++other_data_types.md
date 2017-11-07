# 其他数据类型

## 定义数据类型（typedef)

C++允许基于现有的数据类型定义其他的类型。使用`typedef`关键字来完成，格式如下：

```cpp
typedef existing_type new_type_name;
```

这里`existing_type`为C++基本类型或者复合类型，`new_type_name`为要定义的新类型。例如：

```cpp
typedef char C;
typedef unsigned int WORD;
typedef char * pchar;
typedef char field [50];
```

在这个例子中，我们定义了四个数据类型：`C`、`WORD`、`pChar`以及`field`，分别做为`char`、无符号`int`、`char*`以及`char[50]`，之后我们可以将他么作为有效的类型进行变量声明：

```cpp
C mychar, anotherchar, *ptc1;
WORD myword;
pChar ptc2;
field name;
```

`typedef`并不会创建新的不同数据类型。其只会创建已有类型的同义类型。这意味着`myword`的类型可以为无符号`int`或者WORD，因为其都为同一个类型。

`typedef`可以用于等义`type`的别名，这在程序中会经常使用。当我们在之后的程序版本中可能会改变相应的变量类型时也会定义一些数据类型，或者如果你要使用的类型名字太长或者容易引起混淆时。

## 联合体

联合体允许内存中同一部分作为不同的数据类型被访问，因为他们都在内存中的同一个位置。联合体的声明和使用方式和结构体很像，但是这二者的功能完全不同：

```cpp
union union_name {
  member_type1 member_name1;
  member_type2 member_name2;
  member_type3 member_name3;
  ...
} object_names;
```

`union`中的所有元素都占用内存中的同一个物理空间。其具体大小是声明中占用空间最大的那个元素。例如：

```cpp
union mytypes_t {
  char c;
  int i;
  float f;
} mytypes;
```

定义了三个元素：

```cpp
mytypes.c;
mytypes.i;
mytypes.f;
```

每个的类型都不同。因为他么都执行同一个内存地址，对其中一个元素的修改都会影响到所有其他的元素的值。不能将不同的元素相互独立的存储进去。

联合体的一个作用就是将元素类型和数组或者具备更小元素的结构体关联起来。例如：

```cpp
union mix_t {
  long l;
  struct {
    short hi;
    short lo;
  } s;
  char c[4];
} mix;
```

定义了三个名字来访问4字节的组：`mix.l`、`mix.s`以及`mix.c`，依据使用访问方式的不同则会使用不同的名字，如同它们都为`long`类型数据一样、或者两个`short`类型或者`char`的数组。在`little-endian`系统中（大多数PC平台），这个联合体大致如下图：

```
    +-------+-------+-------+-------+
mix |       |       |       |       |
    +-------+-------+-------+-------+

    +-------------------------------+
                 mix.l
    +--------------+ +--------------+
        mix.s.hi          mix.s.lo
    +-----+ +------+ +-----+ +------+
    mix.c[0]   ..........    mix.c[4]
```

在内存中联合体中具体的对齐和元素顺序是和平台相关的。所以要注意这类类型的可移植性问题。

## 匿名结构体

在C++中可以创建匿名的结构体。如果不适用名字声明一个结构体，这个结构体就是匿名的，直接通过成员名来访问成员。例如，查看下这两个结构体声明中的不同：

```cpp
//structure with regular union
struct {
  char title[50];
  char author[50];
  union {
    float dollars;
    int yen;
  } price;
} book;
//---
// structure with anonymous union
struct {
  char title[50];
  char author[50];
  union {
    float dollars;
    int yen;
  };
} book;

```

二者中的唯一不同就是，第一个联合体中给了名字（`price`)，第二个中却没有。当访问这类对象的`dollars`和`yen`成员是就可以看出差异。对于第一个类型，可以是：

```cpp
book.price.dollars;
book.price.yen;
```

而对于第二个类型，可以是：

```cpp
book.dollars;
book.yen;
```

这里注意的其是联合体而不是结构体，成员`dollars`和`yen`占据相同的物理空间，所以不能同时存储两个不同的值。你可以给`price`设置一个值美元或者日元，但是不能同时设置。

## 枚举（enum）

枚举会创建新的数据类型，而不限定于基础的数据类型。其形式如下：

```cpp
enum enumeration_name {
  value1,
  value2,
  value3,
  ...
} object_names;
```

例如，我们可以创建变量的新类型，称做`colors_t`来存储颜色：

```cpp
enum colors_t {black, blue, green, cyan, red, purple, yellow, white};
```

注意，在声明中我们不会包含基础的数据类型。换句话说，我们从新创建了一个新的数据类型，而不基于任何已有的数据类型。`color_t`这个新类型的可能值为花括号内的常量值。例如，一旦声明了`colors_t`枚举，则如下表达式就生效：

```cpp
colors_t mycolor;

mycolor = blue;
if (mycolor == green) mycolor = red;
```

枚举和数值变量之间是兼容的，所以其内部总是会赋予一个整数数值。如果没有特殊指定，第一个元素的整数值等于0，接下来的元素遵循`a+1`模式。这样，在`colors_t`数据类型中，`black`等于0，`blue`等于1，`green`等于2以此类推。

我们也可以明确指定一个值给这些常量。如果下一个常量没有给出具体的整数值，则会自动假设为之前的值加1。例如：

```cpp
enum months_t {
  january = 1, februray, march, aprill, may, june, july, august, september, october, november, december
} y2k;
```

这个例子中，变量`y2k`属于枚举类型`months_t`，可以包含从`january`到`december`的12个可能的值，对应的数值为1到12（不是0到11，因为这里设置`january`为1）。