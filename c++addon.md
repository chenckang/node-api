# C++插件

Node.JS插件为动态连接共享对象，使用C++编写，可以通过`require()`函数加载到Node.JS中，且可以像一般的Node.JS模块一样使用。这些主要用于提供Node.JS中JavaScript和C++库的关联接口。

现在，用于实现插件的方法相当复杂，涉及到一系列组件的知识和API：

* V8：NodeJS当前使用的用于提供JavaScript实现的C++库。V8提供了创建对象、执行函数等机制。V8的API大多数都定义在`v8.h`（`deps/v8/include/v8.h`）头文件中。
* libuv：实现Node.JS事件循环、工作进程以及所有的异步行为的C库。其也作为一个跨平台的抽象库，提供主流操作系统上易于使用的、类POSIX的众多基础系统任务，诸如和磁盘系统交互、套接字、计时器和系统事件。libuv同样提供类pthread的进程抽象，可以用在更加高级的异步插件从而越过标准事件循环。建议插件的作者思考如何在I/O或者其他时间密集型任务中通过libuv将任务加入到非阻塞的系统操作、工作进程或者自定义的libuv进程中。
* 内部Node.JS库。Node.JS本身导出了一系列插件可以使用的C++API——其中最重要的为`node::ObjectWrap`类。
* Node.JS包含一系列静态连接库包括OpenSSL。其他的库在Node.JS源码目录树中的`deps/`目录。只有V8和OpenSSL符号被Node.JS预先导出，可以用于插件的各种扩展。

所有如下的例子都可以下载，且可以用于插件的初始工程。

## Hello world

这个“Hello world”例子是简单的插件，使用C++编写，等同于如下的JavaScript代码：

```js
module.exports.hello = () => 'world';
```

首先创建`hello.cc`文件：

```c++
// hello.cc
#include <node.h>

namespace demo {
  using v8::FunctionCallbackInfo;
  using v8::Isolate;
  using v8::Local;
  using v8::Object;
  using v8::String;
  using v8::Value;

  void Method(const FunctionCallbackInfo<Value>& args) {
    Isolate * isolate = args.GetIsolate();
    args.GetReturnValue().Set(String::NewFromUtf8(isolate, "world));
  }

  void init(Local<Object> exports) {
    NODE_SET_METHOD(exports, "hello", Method);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, init)
} // namespace demo
```

注意所有的Node.JS插件必须导出一个初始化函数，按如下模式：

```c++
void Initialize(Local<Object> exports);
NODE_MODULE(NODE_GYP_MODULE_NAME, Initialize)
```

`NODE_MODULE`之后没有分号，因为其不为函数（见`node.h`)。

`module_name`必须匹配最终的二进制文件的文件名（包含`.node`后缀）。

在`hello.cc`例子中，初始化函数名为`init`，插件的模块名为`addon`。

## 构建

完成源码之后，必须要将其编译成二进制`addon.node`文件。要完成这件事，在工程的更目录创建一个名为`binding.gyp`的文件，使用JSON格式来定义构建配置。这个文件会被`node-gyp`——用于编译Node.JS插件的工具所使用。

```json
{
  "target": [
    {
      "target_name": "addon",
      "sources": ["hello.cc"]
    }
  ]
}
```

注意：`node-gyp`的一般作为`npm`的一部分随Node.JS一起绑定发布。这个版本不能被开发者直接使用，仅用于支持`npm install`命名编译和安装插件所使用。想要直接使用`node-gyp`的开发者可以通过`npm install -g node-gyp`命令来安装。查看`node-gyp`的安装指导来获取更多信息，包括平台相关的要求。

一旦创建了`binding.gyp`文件，使用`node-gyp configure`来针对当前的平台生成相应的工程文件。这要么会生成一个`Makefile`（在Unix平台上）或者`vcxproj`（在Windows平台上）文件，位于`build`目录下。，

接下来，执行`node-gyp build`命令来编译成`addon.node`文件。这会被放置在`build/Release/`目录下。

当使用`npm install`来安装`Node.js`插件时，npm会使用其帮定的`node-gyp`版本来执行同样的动作集合，在用户的平台上依据实际情况生成插件的编译版本。

一旦构建完成，二进制的插件在Node.JS中通过`require()`来加载`addon.node`模块：

```js
// hello.js

const addon = require('./build/Release/addon`);

console.log(addon.hello());
// prints: 'world'
```

查看如下实例获取更多信息或者访问`https://github.com/arturadib/node-qt`获取生成环境使用的示例。

因为依据不同的编译方法，编译完成的插件二进制的确切路径也不尽相同（也就是有时其可能在`./build/Debug/`目录中），插件可以使用`binding`包来加载编译的模块。

注意，尽管`binding`包在定位插件模块时的实现更加高级，其本质上是使用类似如下的`try-catch`模式：

```js
try {
  return require('./build/Release/addon.node');
}
catch(e) {
  return require('./build/Debug/addon.node');
}
```

## 链接到Node.JS本身的依赖

Node.JS使用一系列诸如V8、libuv和OpenSSL静态链接库。所有的插件要求链接到V8并且也可以链接到任何其他的依赖。一般而言，就如包含适当的`#include <...>`语句一样简单（例如`#include <v8.h>`)，`node-gyp`会自动定位相应的头部文件。然而需要注意如下事项：

* 当`node-gyp`运行时，其会检测Node.JS的发布版本，会要么下载整个源码的压缩包或者仅仅头部文件。如果下载了全部源码，插件可以访问Node.JS的全部依赖。然而，如果只下载了头部文件，那么只有Node.JS导出的符号可以被使用。
* `node-gyp`可以使用`--nodedir`指定本地Node.JS的源码镜像地址。使用这个选项，插件可以访问全部的依赖库。

## 使用`require()`来加载插件

编译的插件二进制文件的拓展名为`.node`（而不是`.dll`或者`.so`）。`require()`函数会去查找拓展名为`.node`的文件并将它们初始化为动态链接库。

当执行`require()`时，可以忽略`.node`扩展名，Node.JS仍然会去查找并初始化插件。然而，需要注意，Node.JS会首先尝试定位和加载同名的JavaScript文件。例如，假设在`addon.node`同目录下有文件`addon.js`，则`require('addon')`会优先加载`addon.js`。

## Node.JS的本地化抽象

当前文档中每个实例都直接使用`Node.JS`和V8的API来实现插件。V8的API在V8版本从一个版本发布到另一个版本发布（也就是Node.JS的一个主要版本到下一个版本）会有巨大的变化。对于每个更改，插件都可能要更新并且重新编译。Node.JS的版本发布计划会减少出现这种情况的频率以及影响，但是当前Node.JS无法保证V8API的稳定。

Node.JS的本地化抽象（后者`nan`）提供了一些列的工具推荐插件的开发者使用，从而在V8和Node.JS的先后不同版本中保持兼容性。查看`nan`例子观察如何使用。

## N-API

> 稳定性：1 - 实验性

N-API为构建本地插件的API。其独立于底层的JavaScript运行时环境（除了V8之外）并且作为Node.JS本身的一部分维护。这个API会在所有的Node.JS版本中作为ABI保持其不变性。其目标在于将插件从底层的JavaScript引擎中隔离开来，让插件只需在一个Node.JS版本中编译后就能在其他之后的版本中直接使用而无效重新编译。插件会以文档（node-gyp等）给出的方法构建后者打包。唯一的不同就是本地化代码中使用的API集合。使用N-API中的方法而不是V8或者Node.JS的本地化抽象API。

相关方法以及如何使用在`C/C++Addon - N-API`中有描述。

## 插件示例

如下为插件的实际示例来帮助开发者开始开发。这个例子使用V8API。访问在线`V8 Reference`获取各种V8调用，以及V8的`Embedder Guild`获取一些概念的解释，诸如`handles`、`scopes`、`function template`等。

每个示例使用如下`binding.gyp`文件：

```json
{
  "target": [
    {
      "target_name": "addon",
      "sources": ["addon.cc"]
    }
  ]
}
```

对于含有多个`.cc`文件的情况，直接往`sources`数组中其他文件名。例如：

```json
"sources": ["addon.cc", "filename.cc"]
```

一旦准备好`binding.gyp`文件，可以使用`node-gyp`来配置和构建示例插件。

## 函数参数

插件一般都会暴力对象或者函数供运行在Node.JS中的JavaScript调用。当函数在JavaScript中调用，输入参数和返回值必须能够映射到对于的C/C++代码中。

下面示例展示了如何从读取JavaScript中传入的函数参数以及如何返回结果：

```c++
// addon.cc

namespace demo {
  using v8::Exception;
  using v8::FunctionCallbackInfo;
  using v8::Isolate;
  using v8::Local;
  using v8::Number;
  using v8::Object;
  using v8::String;
  using v8::Value;

  // This is the implementation of the "add" method
  // Input arguments are passed using the
  // const FunctionCallbackInfo<Value>& args struct
  void Add(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    // Check the number of arguments passed.
    if (args.Length() < 2) {
      // Throw an Error that is passed back to JavaScript
      isolate->ThrowException(Exception::TypeError(
        String::NewFromUtf8(isolate, 'Wrong number of arguments')
      ));
      return;
    }

    // Check the argument types
    if (!args[0]->isNumber() || !args[1].isNumber()) {
      isolate->ThrowException(Exception::TypeError(
        String::NewFromUtf8(isolate, "Wrong numbers")
      ));
      return;
    }

    // Perform the operation
    double value = args[0]->NumberValue() + args[1]->NumberValue();
    Local<number> num = Number::New(isolate, value);

    // Set the return value (using the passed in
    // FunctionCallbackInfo<Value>&)

    args.GetReturnValue().Set(num);
  }

  void init(Local<Object> exports) {
    NODE_SET_METHOD(exports, 'add', Add);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, init);
}
```

一旦编译完成，可以在Node.JS中按如下方式加载和使用插件：

```js
// test.js
const addon = require('./build/Release/addon');

console.log('This should be eight:', addon.add(3, 5));
```

## 回调

一般也有在插件中传入JavaScript函数给C++函数，并在C++中执行传入函数。如下示例展示了如何执行这样的函数回调：

```c++
// addon.cc
#include <node.h>

namespace demo {

  using v8::Function;
  using v8::FunctionCallbackInfo;
  using v8::Isolate;
  using v8::Local;
  using v8::Null;
  using v8::Object;
  using v8::String;
  using v8::Value;

  void RunCallback(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();
    Local<Function> cb = Local<Function>::Cast(args[0]);
    const unsigned argc = 1;
    Local<Value> argv[argc] = { String::NewFromUtf8(isolate, "hello world") };
    cb->Call(Null(isolate), argc, argv);
  }

  void Init(Local<Object> exports, Local<Object> module) {
    NODE_SET_METHOD(module, "exports", RunCallback);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

注意这个例子的`Init`函数使用两个参数的形式，第二个参数接受完整的`module`对象。这可以让插件以一个函数完全重写`exports`，而不是将函数作为`exports`一个属性。

使用如下JavaScript代码运行：

```js
// test.js
const addon = require('./build/Release/addon');

addon((msg) => {
  console.log(msg);
// Prints: 'hello world'
});
```

注意，这个示例中回调函数是以同步的方式运行的。

## 对象工厂

如下示例中，C++函数可以创建和返回新建对象。创建和返回一个对象，且对象中有个`msg`的属性会反馈传入给`createObject()`的字符串。

```c++
// addon.cc
#include <node.h>

namespace demo {

  using v8::FunctionCallbackInfo;
  using v8::Isolate;
  using v8::Local;
  using v8::Object;
  using v8::String;
  using v8::Value;

  void CreateObject(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    Local<Object> obj = Object::New(isolate);
    obj->Set(String::NewFromUtf8(isolate, "msg"), args[0]->ToString());

    args.GetReturnValue().Set(obj);
  }

  void Init(Local<Object> exports, Local<Object> module) {
    NODE_SET_METHOD(module, "exports", CreateObject);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

在JavaScript中测试：

```js
// test.js
const addon = require('./build/Release/addon');

const obj1 = addon('hello');
const obj2 = addon('world');
console.log(obj1.msg, obj2.msg);
// Prints: 'hello world'
```

## 函数工厂

另一个常见使用方法是创建一个包裹C++函数的JavaScript函数，并将其返回到JavaScript中：

```c++
// addon.cc
#include <node.h>

namespace demo {

  using v8::Function;
  using v8::FunctionCallbackInfo;
  using v8::FunctionTemplate;
  using v8::Isolate;
  using v8::Local;
  using v8::Object;
  using v8::String;
  using v8::Value;

  void MyFunction(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();
    args.GetReturnValue().Set(String::NewFromUtf8(isolate, "hello world"));
  }

  void CreateFunction(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, MyFunction);
    Local<Function> fn = tpl->GetFunction();

    // omit this to make it anonymous
    fn->SetName(String::NewFromUtf8(isolate, "theFunction"));

    args.GetReturnValue().Set(fn);
  }

  void Init(Local<Object> exports, Local<Object> module) {
    NODE_SET_METHOD(module, "exports", CreateFunction);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, Init)

}  // namespace demo
```

测试：

```js
// test.js
const addon = require('./build/Release/addon');

const fn = addon();
console.log(fn());
// Prints: 'hello world'
```

## 包裹C++对象

也可以包括C++对象和类，从而可以通过`new`操作符来创建新实例：

```c++
// addon.cc
#include <node.h>
#include "myobject.h"

namespace demo {

  using v8::Local;
  using v8::Object;

  void InitAll(Local<Object> exports) {
    MyObject::Init(exports);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

在`myobject.h`中，包裹类继承自`node::ObjectWrap`:

```c++
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {
  class MyObject : public node::ObjectWrap {
    public:
      static void Init(v8::Local<v8::Object> exports);

    private:
      explicit MyObject(double value = 0);
      ~MyObject();

      static void New(const v8::FunctionCallbackInfo<v8::value>& args);
      static void PlusOne(const v8::FunctionCallbackInfo<v8::value>& args);
      static v8::persistent<v8::Function> constructor;
      double value_;
  };
} // namespace demo

#endif
```

在`myobject.cc`中，实现了要暴力的各种方法。如下所示，`plusOne`通过往构造函数的原型中增加属性来实现暴露：

```c++
#include "myobject.h"

namespace demo {
  using v8::Context;
  using v8::Function;
  using v8::FunctionCallbackInfo;
  using v8::FunctionTemplate;
  using v8::Isolate;
  using v8::Local;
  using v8::Number;
  using v8::Object；
  using v8::Persistent;
  using v8::String;
  using v8::Value;

  Persistent<Function> MyObject::constructor;

  MyObject::MyObject(double value): value_(value) {
  }

  MyObject::~MyObject() {}

  void MyObject::Init(Local<Object> exports) {
    Isolate* isolate = exports.GetIsolate();

    // Prepare constructor template
    Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
    tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
    tpl->InstanceTemplate()->SetInternalFieldCount(1);

    // Prototype
    NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

    constructor.Reset(isolate, tpl->GetFunction());
    exports->Set(String::NewFromUtf8(isolate, "MyObject"),
               tpl->GetFunction());
  }

  void MyObject::New(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    if (args.IsConstructCall()) {
      // Invoked as constructor: `new MyObject(...)`
      double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
      MyObject* obj = new MyObject(value);
      obj->Wrap(args.This());
      args.GetReturnValue().Set(args.This());
    } else {
      // Invoked as plain function `MyObject(...)`, turn into construct call.
      const int argc = 1;
      Local<Value> argv[argc] = { args[0] };
      Local<Context> context = isolate->GetCurrentContext();
      Local<Function> cons = Local<Function>::New(isolate, constructor);
      Local<Object> result =
          cons->NewInstance(context, argc, argv).ToLocalChecked();
      args.GetReturnValue().Set(result);
    }
  }

  void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
    obj->value_ += 1;

    args.GetReturnValue().Set(Number::New(isolate, obj->value_));
  }

}  // namespace demo
```

要构建这个示例，必须在`binding.gyp`中增加`myobject.cc`文件：

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [
        "addon.cc",
        "myobject.cc"
      ]
    }
  ]
}
```

使用如下代码检测：

```js
// test.js
const addon = require('./build/Release/addon');

const obj = new addon.MyObject(10);
console.log(obj.plusOne());
// Prints: 11
console.log(obj.plusOne());
// Prints: 12
console.log(obj.plusOne());
// Prints: 13
```

## 包裹对象的工厂

作为可选的另一种方案，可以通过工厂模式来避免显式的通过JavaScript的`new`操作符来创建对象实例。

```js
const obj = addon.createObject();

// instead of:
// const obj = new addon.Object();
```

首先在`addon.cc`中实现`createObject()`方法。

```c++
// addon.cc
#include <node.h>
#include "myobject.h"

namespace demo {

  using v8::FunctionCallbackInfo;
  using v8::Isolate;
  using v8::Local;
  using v8::Object;
  using v8::String;
  using v8::Value;

  void CreateObject(const FunctionCallbackInfo<Value>& args) {
    MyObject::NewInstance(args);
  }

  void InitAll(Local<Object> exports, Local<Object> module) {
    MyObject::Init(exports->GetIsolate());

    NODE_SET_METHOD(module, "exports", CreateObject);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

在`myobject.h`中，加入静态方法`NewInstance()`来处理对象的初始化。这个方法可以替代在JavaScript中使用`new`操作符：

```c++
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {

class MyObject : public node::ObjectWrap {
 public:
  static void Init(v8::Isolate* isolate);
  static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);

 private:
  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
  static void PlusOne(const v8::FunctionCallbackInfo<v8::Value>& args);
  static v8::Persistent<v8::Function> constructor;
  double value_;
};

}  // namespace demo

#endif
```

`myobject.cc`的实现类似于之前的示例：

```c++
// myobject.cc
#include <node.h>
#include "myobject.h"

namespace demo {

  using v8::Context;
  using v8::Function;
  using v8::FunctionCallbackInfo;
  using v8::FunctionTemplate;
  using v8::Isolate;
  using v8::Local;
  using v8::Number;
  using v8::Object;
  using v8::Persistent;
  using v8::String;
  using v8::Value;

  Persistent<Function> MyObject::constructor;

  MyObject::MyObject(double value) : value_(value) {
  }

  MyObject::~MyObject() {
  }

  void MyObject::Init(Isolate* isolate) {
    // Prepare constructor template
    Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
    tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
    tpl->InstanceTemplate()->SetInternalFieldCount(1);

    // Prototype
    NODE_SET_PROTOTYPE_METHOD(tpl, "plusOne", PlusOne);

    constructor.Reset(isolate, tpl->GetFunction());
  }

  void MyObject::New(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    if (args.IsConstructCall()) {
      // Invoked as constructor: `new MyObject(...)`
      double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
      MyObject* obj = new MyObject(value);
      obj->Wrap(args.This());
      args.GetReturnValue().Set(args.This());
    } else {
      // Invoked as plain function `MyObject(...)`, turn into construct call.
      const int argc = 1;
      Local<Value> argv[argc] = { args[0] };
      Local<Function> cons = Local<Function>::New(isolate, constructor);
      Local<Context> context = isolate->GetCurrentContext();
      Local<Object> instance =
          cons->NewInstance(context, argc, argv).ToLocalChecked();
      args.GetReturnValue().Set(instance);
    }
  }

  void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    const unsigned argc = 1;
    Local<Value> argv[argc] = { args[0] };
    Local<Function> cons = Local<Function>::New(isolate, constructor);
    Local<Context> context = isolate->GetCurrentContext();
    Local<Object> instance =
        cons->NewInstance(context, argc, argv).ToLocalChecked();

    args.GetReturnValue().Set(instance);
  }

  void MyObject::PlusOne(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.Holder());
    obj->value_ += 1;

    args.GetReturnValue().Set(Number::New(isolate, obj->value_));
  }

}  // namespace demo
```

再次，要构建这个示例，必须将`myobject.cc`文件加入到`binding.gyp`中：

```json
{
  "targets": [
    {
      "target_name": "addon",
      "sources": [
        "addon.cc",
        "myobject.cc"
      ]
    }
  ]
}
```

使用如下方法检测：

```js
// test.js
const createObject = require('./build/Release/addon');

const obj = createObject(10);
console.log(obj.plusOne());
// Prints: 11
console.log(obj.plusOne());
// Prints: 12
console.log(obj.plusOne());
// Prints: 13

const obj2 = createObject(20);
console.log(obj2.plusOne());
// Prints: 21
console.log(obj2.plusOne());
// Prints: 22
console.log(obj2.plusOne());
// Prints: 23
```

## 传递包裹对象

在包裹和返回C++对象之外，可以通过Node.JS的辅助函数`node::ObjectWrap::Unwrap`来解包对象并将其传入。如下示例展示了含有两个`MyObject`作为输入参数的函数`add()`：

```cpp
// addon.cc
#include <node.h>
#include <node_object_wrap.h>
#include "myobject.h"

namespace demo {

  using v8::FunctionCallbackInfo;
  using v8::Isolate;
  using v8::Local;
  using v8::Number;
  using v8::Object;
  using v8::String;
  using v8::Value;

  void CreateObject(const FunctionCallbackInfo<Value>& args) {
    MyObject::NewInstance(args);
  }

  void Add(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    MyObject* obj1 = node::ObjectWrap::Unwrap<MyObject>(
        args[0]->ToObject());
    MyObject* obj2 = node::ObjectWrap::Unwrap<MyObject>(
        args[1]->ToObject());

    double sum = obj1->value() + obj2->value();
    args.GetReturnValue().Set(Number::New(isolate, sum));
  }

  void InitAll(Local<Object> exports) {
    MyObject::Init(exports->GetIsolate());

    NODE_SET_METHOD(exports, "createObject", CreateObject);
    NODE_SET_METHOD(exports, "add", Add);
  }

  NODE_MODULE(NODE_GYP_MODULE_NAME, InitAll)

}  // namespace demo
```

在`myobject.h`中，增加了一个公开方法来访问解包后的对象的私有属性。

```cpp
// myobject.h
#ifndef MYOBJECT_H
#define MYOBJECT_H

#include <node.h>
#include <node_object_wrap.h>

namespace demo {

class MyObject : public node::ObjectWrap {
 public:
  static void Init(v8::Isolate* isolate);
  static void NewInstance(const v8::FunctionCallbackInfo<v8::Value>& args);
  inline double value() const { return value_; }

 private:
  explicit MyObject(double value = 0);
  ~MyObject();

  static void New(const v8::FunctionCallbackInfo<v8::Value>& args);
  static v8::Persistent<v8::Function> constructor;
  double value_;
};

}  // namespace demo

#endif
```

`myobject.cc`的实现和之前的类似

```cpp
// myobject.cc
#include <node.h>
#include "myobject.h"

namespace demo {

  using v8::Context;
  using v8::Function;
  using v8::FunctionCallbackInfo;
  using v8::FunctionTemplate;
  using v8::Isolate;
  using v8::Local;
  using v8::Object;
  using v8::Persistent;
  using v8::String;
  using v8::Value;

  Persistent<Function> MyObject::constructor;

  MyObject::MyObject(double value) : value_(value) {
  }

  MyObject::~MyObject() {
  }

  void MyObject::Init(Isolate* isolate) {
    // Prepare constructor template
    Local<FunctionTemplate> tpl = FunctionTemplate::New(isolate, New);
    tpl->SetClassName(String::NewFromUtf8(isolate, "MyObject"));
    tpl->InstanceTemplate()->SetInternalFieldCount(1);

    constructor.Reset(isolate, tpl->GetFunction());
  }

  void MyObject::New(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    if (args.IsConstructCall()) {
      // Invoked as constructor: `new MyObject(...)`
      double value = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
      MyObject* obj = new MyObject(value);
      obj->Wrap(args.This());
      args.GetReturnValue().Set(args.This());
    } else {
      // Invoked as plain function `MyObject(...)`, turn into construct call.
      const int argc = 1;
      Local<Value> argv[argc] = { args[0] };
      Local<Context> context = isolate->GetCurrentContext();
      Local<Function> cons = Local<Function>::New(isolate, constructor);
      Local<Object> instance =
          cons->NewInstance(context, argc, argv).ToLocalChecked();
      args.GetReturnValue().Set(instance);
    }
  }

  void MyObject::NewInstance(const FunctionCallbackInfo<Value>& args) {
    Isolate* isolate = args.GetIsolate();

    const unsigned argc = 1;
    Local<Value> argv[argc] = { args[0] };
    Local<Function> cons = Local<Function>::New(isolate, constructor);
    Local<Context> context = isolate->GetCurrentContext();
    Local<Object> instance =
        cons->NewInstance(context, argc, argv).ToLocalChecked();

    args.GetReturnValue().Set(instance);
  }

}  // namespace demo
```

验证：

```js
// test.js
const addon = require('./build/Release/addon');

const obj1 = addon.createObject(10);
const obj2 = addon.createObject(20);
const result = addon.add(obj1, obj2);

console.log(result);
// Prints: 30
```

## AtExit钩子

“AtExit“钩子是Node.JS事件结束之后，在JavaScriptVM结束和Node.JS关闭前执行的函数。“AtExit”钩子使用`node::AtExit`API注册。

### void AtExit(callback, args)

* `callback` `<void (*)(void*)>` 指向退出时调用的函数的指针
* `args` `<void*>` 执行退出时传递给回调的参数

在事件循环接受后VM退出之前注册退出钩子。

AtExit接受两个参数：指向退出时执行的回调函数的指针，以及相应会掉函数接受到的无符号上下文数据的指针。

回调函数按照后进先出的序列执行。

如下`addon.cc`实现了AtExit：

```cpp
// addon.cc
#include <assert.h>
#include <stdlib.h>
#include <node.h>

namespace demo {

using node::AtExit;
using v8::HandleScope;
using v8::Isolate;
using v8::Local;
using v8::Object;

static char cookie[] = "yum yum";
static int at_exit_cb1_called = 0;
static int at_exit_cb2_called = 0;

static void at_exit_cb1(void* arg) {
  Isolate* isolate = static_cast<Isolate*>(arg);
  HandleScope scope(isolate);
  Local<Object> obj = Object::New(isolate);
  assert(!obj.IsEmpty()); // assert VM is still alive
  assert(obj->IsObject());
  at_exit_cb1_called++;
}

static void at_exit_cb2(void* arg) {
  assert(arg == static_cast<void*>(cookie));
  at_exit_cb2_called++;
}

static void sanity_check(void*) {
  assert(at_exit_cb1_called == 1);
  assert(at_exit_cb2_called == 2);
}

void init(Local<Object> exports) {
  AtExit(at_exit_cb2, cookie);
  AtExit(at_exit_cb2, cookie);
  AtExit(at_exit_cb1, exports->GetIsolate());
  AtExit(sanity_check);
}

NODE_MODULE(NODE_GYP_MODULE_NAME, init)

}  // namespace demo
```

在JavaScript使用如下来验证：

```js
// test.js
require('./build/Release/addon');
```
