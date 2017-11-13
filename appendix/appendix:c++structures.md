# 结构体

我们已经学习了C++如何将序列数据进行分组。但是这存在限制，因为很多场景下我们希望存储的是一个不同类型的元素的集合，而不是纯粹的是同元素类型的元素序列。

## 结构体

结构体就是一系列数据元素归在一个名字下面的组合。这些元素，被称做成员、可以有不同的类型和长度。结构体使用如下的预发在C++中声明：

```cpp
struct struct_name {
  member_type1 member_name1;
  member_type2 member_name2;
  member_type3 member_name3;
  ...
} object_names;
```

这里`structure_name`为结构体类型的名字，`object_name`为一些列类型为这个结构体的有效标识符。在大括号`{}`中，为数据成员的列表，每个都由一个同类和一个标识符指定。

首先，我们要知道的是，结构体创建一个新的类型：一旦结构体被声明，一个`structure_name`标识的新类型就产生了，之后就在程序中就可以向其他类型一样使用。例如：

```cpp
struct product {
  int weight;
  float price;
};

product table;
product banana, melon;
```

首先，我们声明了一个结构体类型，叫做`product`，其含有两个成员：`weight`和`price`，每个都为不同的基本类型。然后使用这个结构体类型（`product`）来声明三个对象：`apple`、`banana`、`melon`，如同其他的基本类型的使用方式一样。

一旦声明了，`product`就为一个有效的类型名，如同基础类型的`int`、`char`或者`short`，至此之后，在程序中就可以声明这种复合新类型的对象（变量），如同`apple`、`banana`以及`melon`。

在`struct`的结尾和封号之前，可以使用可选的`object_name`来直接声明该结构体类型的对象。例如，可以在定义结构体类型时声明结构体对象`apple`、`banana`以及`melon`：

```cpp
struct product {
  int weight;
  float price;
} apple, banana, melon;
```

明确区分什么是结构体类型名和什么是这个结构体类型的对象（变量）是很重要的。可以在单个结果体类型（`product`）中初始化很多对象（即，诸如`apple`、`banana`、`melon`这些变量）。

一旦声明了指定结构体类型的三个对象（`apple`、`banana`、`melon`），就可以直接和它们的成员进行交互。使用点号（`.`)插入在对象名和成员名之间来完成这个操作。例如，我们可以像变量所代表的具体的类型那样直接操作这些元素：

```cpp
apple.weight;
apple.price;
banana.weight;
banana.price;
melon.weight;
melon.price;
```

其中每个成员都对应一个特定的数据类型：`apple.weight`、`banana.weight`以及`melon.weight`为`int`类型，而`apple.price`、`banana.price`和`melon.price`为`float`类型。

来看一个实际的例子，观察结构体类型如何像基本的数据类型那样使用：

```cpp
// example about structures
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

struct movies_t {
  string title;
  int year;
} mine, yours;

void printmovie(movies_t movie);

int main()
{
  string mystr;

  mine.title = "2001 A Apace Odyssey";
  mine.year = 1968;

  cout << "Enter title: ";
  getline(cin, yours.title);
  cout << "Enter year: ";
  getline(cin, mystr);
  stringsstream(mystr) >> yours.year;

  cout << "My favorite movie is: \n";
  printmovie(mine);
  cout << "And yours is:\n";
  printmovie(yours);
  return 0;
}

void printmovie(movies_t movie)
{
  cout << movie.title;
  count << " (" << movie.year << ") ";
}
```

这个例子展示了如何使用一个对象的成员作为普通变量来使用。例如，成员`yours.year`是一个有效的`int`类型，`mine.title`是一个有效的`string`类型。

`mine`和`yours`对象可以作为类型`movies_t`的有效变量，例如我们将其传递给函数`printmovie`，就如同对普通变量使用的一样。因此，结构体最大的一个优势就是既可以指代其各个成员也可以直到整个结构体。

结构体可以作为数据库的表示，特别是当考虑对齐创建数组时：

```cpp
// array of structures
#include <iostream>
#include <string>
#include <ssstream>
using namespace std;

#define N_MOVIES 3;

struct movies_t {
  string title;
  int year;
} films [N_MOVIES];

void printmovie(movies_t movie);

int main()
{
  string mystr;
  int n;

  for (n = 0; n < N_MOVIES; n++) {
    cout << "Enter title: ";
    getline(cin, films[n].title);
    count << "Enter year: ";
    getline(cin, mystr);
    stringstream(mystr) >> films[n].year;
  }

  cout << "\nYou have entered these movies:\n";
  for (n = 0; n < N_MOVIES; n++)
    printmovie(films[i]);
  return 0;
}

void printmovie(movies_t movie)
{
  cout << movie.title;
  cout << " ( " << movie.year << " )";
}
```

## 执行结构体的指针

如同其他类型，结构体可以通过其类型指针指向。

```cpp
struct movies_t {
  string title;
  int year;
}

movies_t amovie;
movies_t * pmovie;
```

这里，`amovie`为结构体类型`movie_t`的对象，`pmovie`为指向结构体类型`movies_t`的指针。如下代码也可适用：

```cpp
pmovie = &amovie;
```

指针`pmovie`的值为对象`amovie`的引用（内存地址）。

现在再来看一个例子，其中会设计到新的操作符：箭头操作符（`->`)：

```cpp
// pointers to structures
#include <iostream>
#include <string>
#include <sstream>
using namespace std;

struct movies_t {
  string title;
  string year;
};

int main()
{
  string mystr;

  movies_t amovie;
  movies_t * pmovie;
  pmovie = &amovie;

  cout << "Enter title: ";
  getline(cin, pmove->title);
  cout << "Enter year: ";
  getline(cin, mystr);
  (stringstream) mystr >> pmovie->year;
  cout << "\nYou have entered:\n";
  cout << pmovie->title;
  cout << " (" << pmovie->year << ")\n";

  return 0;
}
```

在这段代码中引入了一个非常重要的概念：箭头符号（`->`）。这个为专门用于指向指针成员的解引用符号。这个符号用于通过对象的应用访问成员。例子中，我们使用：

```cpp
pmovie->title;
```

等同于：

```cpp
(*pmovie).title;
```

两个表达式`pmovie->title`和`(*pmive).title`都是有效的且都是指针`pmovie`指向的结构体的`title`成员。必须要和如下区分一下：

```cpp
*pmovie.title;
```

等同于：
```cpp
*(pmovie.title)
```

这里`pmovie`假设不为指针。如下表格总结了指针和结构体成员之间的可能组合：

|Expression|What is evaluated|Equivalent|
|---|---|---|
| a.b | a的成员b | |
| a->b | a指针指向的对象的成员吧 | (*a).b |
| *a.b | a的成员b指向的值 | *(a.b) |

## 结构体嵌套

结构体也可以嵌套，所以结构体的有效成员也可以是其他结构体。

```cpp
struct movies_t {
  string title;
  int year;
};

struct friends_t {
  string name;
  string email;
  movies_t favorite_movie;
} charlie, maria;

friends_t * pfriends = &charlie;
```

这个什么之后，可以使用如下的表达式：

```cpp
charlie.name;
maria.favorite_movie.title;
charlie.favorite_movie.year;
pfriends->favorite_movie.year;
```

(后两个表达式指代同一个成员)。