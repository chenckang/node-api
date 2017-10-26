# 异步钩子

> Stability：1  实验特性

`async_hooks`模块用于对异步资源注册回调从而完成异步资源的生命周期追踪。可以通过如下引入：

```js
const async_hooks = require('async_hooks');
```

## 术语

异步资源表示含有一些列关联回调的对象。回调会被多次执行，例如，对于`net.createSXerver`的`connection`事件，或者`fs.open`。异步资源也可以在回调执行前关闭。`AsyncHook`并不明确区分这些不同的不同的异步场景，会将它们统一抽象成资源。

## 公开API

### 概览

如下为公开API的一个简单预览：

```js
const async_hooks = require('async_hooks);

const eid = async_hooks.execuationAsyncId();

const tid = async_hooks.triggerAsyncId();

const asyncHook = async_hooks.createHook({init, before, after, destroy, promiseResolve});

asyncHook.enable();

asyncHook.disable();

function init(asyncId, type, triggerAsyncId, resource) {}

function before(asyncId) {}

function after(asyncId) {}

function destroy(asyncId) {}

function promiseResolve(asyncId) {}
```

###  `async_hook.createHook(callbacks)`

* callback `<Object>` 要注册的回调函数
  - init `<Function>` init回调
  - before `<Function>` before回调
  - after `<Function>` after回调
  - destroy `<Function>` destroy回调
* 返回：`{AsyncHook}`实例，用于开启和关闭回调

用于对异步操作生命周期的不同阶段注册回调函数。

回调函数`init()/before()/after()/destroy()`在资源的生命周期的相应阶段执行。

所有的回调函数都是可选参数。所以，例如只需追踪资源的清理，则只需要传递`destroy`回调函数。所有回调函数的具体传入参数在下面会具体介绍。

```js
const async_hooks = require('async_hooks');

const asyncHook = async_hooks.create({
  init(asyncId, type, triggerAsyncId, resource) {},
  destroy(asyncId) {},
});
```

注意回调可以通过原型链来继承：

```js
class MyAsyncCallbacks {
  init(asyncId, type, triggerAsyncId, resource) {}
  destroy(asyncId) {}
}

class MyAddedCallbacks extends MyAsyncCallbacks {
  before(asyncId) {}
  after(asyncId) {}
}

const asyncHook = async_hook.create(new MyAddedCallbacks());
```

#### 错误处理

如果`AsyncHook`的回调函数抛出异常，则应用程序会打印出错误栈并退出。退出的方式和`uncaughtException`类似，但是会移除所有的`uncaughtException`侦听函数，这样会强制进程退出。`exit`回调仍然会执行，除非应用程序使用`--abort-on-uncaught-exception`来启动，此时会打印错误栈且程序会退出，创建一个core文件。

这种错误处理方式的原因是这些回调函数运行在对象生命周期中的一个不确定的状态上，例如在类的实例化中或者销毁中。因此，有必要迅速终止进程来阻止后续的非意料中的中断。这部分在未来版本中，如果在可以执行全面的分析后确定异常不会导致非预料的副作用时会作出更改。

#### 在`AsyncHook`回调中打印消息

因为向控制台打印消息是异步的行为，`console.log()`会引起`AsyncHook`的回调被执行。在`AsyncHook`的回调函数中，使用`console.log()`或者类似的方法会导致无限递归。这个问题的一个简单的解决方法就是使用`fs.writeSync(1, msg)`来打印调试信息。这会往stdout中打印消息，因为`1`为stdout的文件描述符，且不会引起`AsyncHooks`的递归执行，因为其为同步的。

```js
const fs = require('fs');
const util = require('util');

function debug(...args) {
  fs.writeSync(1, `{utils.format(...args)}\n`);
}
```

如果需要使用异步操作来记录日志，则可以使用`AsyncHook`本身提供的信息来追踪异步操作。当记录日志本身引起了`AsyncHook`回调执行，则会跳过日志记录。通过这样可以打破无限递归。

### `asyncHook.enable()`

* 返回：`<AsyncHook>` `asyncHook`的引用

启动指定`AsyncHook`实例的回调函数。如果没有相应的回调函数，则启动动作不做任何事。

`AsyncHook`实例默认是关闭的。如果需要在创建完`AsyncHook`实例之后立即开启，则可以遵循如下模式：

```js
const async_hooks = require('async_hooks');

const hook = async_hooks.createHook(callbacks).enable();
```

### `asyncHook.disable()`

* 返回：`AsyncHook` `asyncHook`的引用

从全局的AsyncHook执行池中关闭指定`AsyncHook`实例的回调函数。一旦被关闭，则在再次开启之前不会被执行。

为了API的一致性，`disable()`同样返回`AsyncHook`实例。

#### 回调钩子

异步动作的生命周期中关键事件可以氛围四个阶段：初始化，回调执行之前或者之后，以及实例销毁。

`init(asyncId, type, triggerAsyncId, resource)`

* `asyncId` `<number>` 异步资源的唯一ID
* `type` `<string>` 异步资源的类型
* `triggerAsyncId` `<number>` 创建当前异步资源的执行上下文所属的异步资源唯一ID
* `resource` `<Object>` 指向表示异步操作资源的引用，需要在销毁时释放这个对象。

当构建的类能够触发异步事件时执行。但是在`destroy`执行之前不是一定要执行`before`/`after`。

这个行为可以通过注入打开资源随后在资源可以使用前关闭它来观察。例如如下：

```js
require('net').createServer.listen(function() {this.close()});

clearTimeout(setTimeout(() => {}, 10));
```

每个新建资源都会在当前进程的执行中分配一个唯一的ID。

##### `type`

`type`为标识触发`init`回调执行的资源类型。通常，其为资源的构造器名字。

```
FSEVENTWRAP, FSREQWRAP, GETADDRINFOREQWRAP, GETNAMEINFOREQWRAP, HTTPPARSER,
JSSTREAM, PIPECONNECTWRAP, PIPEWRAP, PROCESSWRAP, QUERYWRAP, SHUTDOWNWRAP,
SIGNALWRAP, STATWATCHER, TCPCONNECTWRAP, TCPWRAP, TIMEWRAP, TTYWRAP,
UDPSENDWRAP, UDPWRAP, WRITEWRAP, ZLIB, SSLCNNECTION, PBKDF2REQUEST,
RANDOMBYTESREQUEST, TLSWRAP, Timeout, Immediate, TickObject
```

也有`PROMISE`资源类型，用于追踪`Promise`实例和他们的异步工作。

用户可以通过嵌入API来自定义`type`。

注意：有可能存在名字冲突。嵌入方式应该使用独特的前缀，例如npm包的名字，防止监听时的执行中冲突。

##### `triggerId`

`triggerAsyncId`为引起新资源初始化和`init`执行的那个资源的`asyncId`。这和显示资源何时创建的`async_hooks.executionAsyncId()`不同，`triggerAsyncId`表示资源为什么被创建。

如下为`triggerAsyncId`的简单示例：

```js
async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId();

    fs.writeSync(
      1, `${type}(${asyncId}): trigger: ${triggerAsyncId} execution: ${eid}\n`
    );
  }
}).enable();

require('net').createServer((conn) => {}).listen(8080);
```

通过`nc localhost 8080`:

```
TCPWRAP(2): trigger: 1 execution: 1
TCPWRAP(4): trigger: 2 execution: 0
```

第一个`TCPWRAP`为接受连接的服务器。

第二个`TCPWRAP`为从客户端发起的新连接。当建立新连接时，会立即构建`TCPwrap`实例。这在JavaScript栈之外执行（注意：`executionAsyncId()`为0，则以为这其总C++中执行，而不是在JavaScript的堆栈中）。因为这个原因，就不太可能去追踪资源的创建关系，所以`triggerAsyncId`则可以标识哪个资源触发新资源的创建。

##### `resource`

`resource`表示实际初始化的实际异步资源。这可以包含基于`type`值信息。例如，对于`GETADDRINFOREQWRAP`资源类型，`resource`包含`net.Server.listenr()`中用于查询IP地址的域名。用于访问这个信息的API暂且还不公开，可以使用嵌入API，用户可以使用其自定义的资源对象。例如，诸如包含SQL查询渔区的资源对象。

对于`Promise`，`resource`对象可以包含`promise`属性指向正在初始化的`Promise`实例，`parentId`属性指向父`Promise`的`asyncId`，如果不存在则为`undefined`。例如，`b = a.then(handler)`，`a`可以被看作`b`的父`Promise`。

注意：在有些情况下，资源对象会重用来提示性能，所以在`WeakMap`使用其作为键值则不太安全，同样给其增加属性则也不安全。


###### 异步山下问例子

如下为`before`和`after`执行之间的`init`调用中搜集更多信息的例子，特别的是`listen()`方法的回调。

```js
let indent = 0;
async_hooks.createHook({
  init(asyncId, type, triggerAsyncId) {
    const eid = async_hooks.executionAsyncId();
    const indentStr = ' '.repeat(indent);
    fs.writeSync(
      1,
      `${indentStr}${type}(${asyncId}):` +
      ` trigger: ${triggerAsyncId} execution: ${eid}\n`);
  },
  before(asyncId) {
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}before:  ${asyncId}\n`);
    indent += 2;
  },
  after(asyncId) {
    indent -= 2;
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}after:   ${asyncId}\n`);
  },
  destroy(asyncId) {
    const indentStr = ' '.repeat(indent);
    fs.writeSync(1, `${indentStr}destroy: ${asyncId}\n`);
  },
}).enable();

require('net').createServer(() => {}).listen(8080, () => {
  // Let's wait 10ms before logging the server started.
  setTimeout(() => {
    console.log('>>>', async_hooks.executionAsyncId());
  }, 10);
});
```

启动服务其的输出如下：

```
TCPWRAP(2): trigger: 1 execution: 1
TickObject(3): trigger: 2 execution: 1
before:  3
  Timeout(4): trigger: 3 execution: 3
  TIMERWRAP(5): trigger: 3 execution: 3
after:   3
destroy: 3
before:  5
  before:  4
    TTYWRAP(6): trigger: 4 execution: 4
    SIGNALWRAP(7): trigger: 4 execution: 4
    TTYWRAP(8): trigger: 4 execution: 4
>>> 4
    TickObject(9): trigger: 4 execution: 4
  after:   4
after:   5
before:  9
after:   9
destroy: 4
destroy: 9
destroy: 5
```

注意：如例子中展示的，`executionAsyncId()`和`excution`都表示了当前执行上下文的值；如同`before`和`after`中描述的。

可以用执行过程来表示资源分配，如下：

```
TTYWRAP(6) -> Timeout(4) -> TIMERWRAP(5) -> TickObject(3) -> root(1)
```

`TCPWrap`不在本图中，尽管其为`console.log()`执行的触发者。这是因为不用域名来绑定一个端口实际是同步的动作，但是为了维护完整的异步API，会将用户的回调函数放在`process.nextTick()`中。

这几图只展示了资源何时被创建，而不是为什么被创建，要追踪这个可以使用`triggerAsyncId`。

### `before(asyncId)`

* asyncId `<number>`

当初始化异步操作（例如TCP服务其接受新的请求）或者执行回调来通知用于（例如向磁盘写入数据）。`before`回调会在目标回调执行前被执行。`asyncId`为要执行回调的对应资源的唯一标识。

`before`回调可以执行0到N次。如果异步操作被取消，则`before`回调会执行0次，例如，如果TCP服务没有接受到连接。持久的异步资源如TCP服务对象会多次执行`before`回调，而其他操作如`fs.open()`只会调用一次。

### `after(asyncId)`

* asyncId `<number>`

在`before`所指定的回调完成后立即执行。

注意：如果在回调执行时发生未捕获异常，则`after`会在`uncaughtException`触发后执行。

### `destroy(asyncId)`

* asyncId `<number>`

在`asyncId`对应的资源被销毁时执行。同时会在嵌入API的`emitDestroy`异步执行。

注意：一些资源依赖于垃圾回收的清理，所以，如果资源的应用被传入到`init`中，则`destroy`可能不会被调用，导致内存泄漏。如果资源不依赖于垃圾回收，则就不是个问题。

### `promiseResolve(asyncId)`

* asyncId `<number>`

当传递给`Promise`的`resolve`函数被执行时调用（无论直接或者通过promise的其他方法来完成）。

注意`resolve()`不会执行任何显著的同步动作。

注意：如果`Promise`的完成依赖于其他`Promise`的状态，则这并不意味着`Promise`在这个时间点上完成或者拒绝。

例如：

```
new Promise((resolve) => resolve(true)).then((a) => {});
```

执行如下回调：

```
init for PROMISE with id 5, trigger id: 1
  promise resolve 5      # corresponds to resolve(true)
init for PROMISE with id 6, trigger id: 5  # the Promise returned by then()
  before 6               # the then() callback is entered
  promise resolve 6      # the then() callback resolves the promise by returning
  after 6
```

### `async_hooks.executionAsyncId()`

* 返回：`<number>`当前执行上下文的`asyncId`。用于追踪一些执行时间点。

例如：

```js
const async_hooks = require('async_hooks');

console.log(async_hooks.executionAsyncId());  // 1 - bootstrap
fs.open(path, 'r', (err, fd) => {
  console.log(async_hooks.executionAsyncId());  // 6 - open()
});
```

要注意的是`executionAsyncId()`返回的ID和执行时间有关，而不是因果关系（这个由`triggerAsyncId()`来实现）。例如：

```js
const server = net.createServer(function onConnection(conn) {
  // Returns the ID of the server, not of the new connection, because the
  // onConnection callback runs in the execution scope of the server's
  // MakeCallback().
  async_hooks.executionAsyncId();

}).listen(port, function onListening() {
  // Returns the ID of a TickObject (i.e. process.nextTick()) because all
  // callbacks passed to .listen() are wrapped in a nextTick().
  async_hooks.executionAsyncId();
});
```

### `async_hooks.triggerAsyncId()`

* 返回：`<number>` 触发当前正在执行的回调的资源ID

例如：

```js
const server = net.createServer((conn) => {
  // The resource that caused (or triggered) this callback to be called
  // was that of the new connection. Thus the return value of triggerAsyncId()
  // is the asyncId of "conn".
  async_hooks.triggerAsyncId();

}).listen(port, () => {
  // Even though all callbacks passed to .listen() are wrapped in a nextTick()
  // the callback itself exists because the call to the server's .listen()
  // was made. So the return value would be the ID of the server.
  async_hooks.triggerAsyncId();
});
```

## JavaScript嵌入API

复用库的开发者可以定义其自身的异步资源来执行如I/O/、连接池、或者管理异步队列使用`AsyncWrap`来执行适当的回调。

### `class AsyncResource()`

`AsyncResource`通过嵌入者的异步资源进行扩展。使用这个，用户可以方便的触发他们自定义资源的声明周期事件。

`init`钩子在`AsyncResource`初始化时触发。

注意：`before`/`after`按照属性调用是必要的。否则，会发生不可恢复异常，且进程会退出。

如下为`AsyncResource`API的概览：

```js
const { AsyncResource } = require('async_hooks');

// AsyncResource() is meant to be extended. Instantiating a
// new AsyncResource() also triggers init. If triggerAsyncId is omitted then
// async_hook.executionAsyncId() is used.
const asyncResource = new AsyncResource(type, triggerAsyncId);

// Call AsyncHooks before callbacks.
asyncResource.emitBefore();

// Call AsyncHooks after callbacks.
asyncResource.emitAfter();

// Call AsyncHooks destroy callbacks.
asyncResource.emitDestroy();

// Return the unique ID assigned to the AsyncResource instance.
asyncResource.asyncId();

// Return the trigger ID for the AsyncResource instance.
asyncResource.triggerAsyncId();
```

#### `AsyncResource(type[, triggerAsyncId])`

* type `<string>` 异步事件的类型
* triggerAsyncId `<number>` 创建当前异步事件的执行上下文

例如：

```js
class DBQuery extends AsyncResource {
  constructor(db) {
    super('DBQuery');
    this.db = db;
  }

  getInfo(query, callback) {
    this.db.get(query, (err, data) => {
      this.emitBefore();
      callback(err, data);
      this.emitAfter();
    });
  }

  close() {
    this.db = null;
    this.emitDestroy();
  }
}
```

#### `asyncResource.emitBefore()`

执行所有的`before`回调通知正要进入一个新的异步执行上下文。如果存在嵌套执行`emitBefore`的情况，则可以最终`asyncIds`的栈并依据情况解开。

#### `asyncResource.emitAfter()`

执行所有的`after`回调执行。如果存在嵌套执行`emitBefore`的情况，则要确保栈被妥当解开。否则会抛出异常。

如果用户的回调抛出异常，`emitAfter()`会自动对所有的栈中的`asyncIds`执行。

#### `asyncResource.emitDestroy()`

执行所有的`destroy`钩子。只能执行一次。如果执行多次则抛出异常。必须手动执行。如果资源被GC收集从而不能回收，则`destroy`钩子不会被执行。

#### `asyncResource.asyncId()`

* 返回：`<number>` 资源的唯一`asyncId`

#### `asyncResource.triggerAsyncId()`

* 返回：`<number>` 同`AsyncResource`构造函数中的`triggerAsyncId`参数。
