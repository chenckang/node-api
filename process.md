# process

process是全局变量，所以在node中不需要通过require函数来获取。

## 事件

#### `beforeExit`

#### `exit`

#### `disconnect`

#### `message`

#### `rejectionHandled`

当`Promise`对象被`Reject`，且在下一次node的事件循环中给相应的`Promise`对象被注册了错误处理事件时触发。

其实际含义就是触发`unhandledRejection`的`Promise`对象在程序的运行过程中获得了相应的错误处理函数，意味着`Promise`的`reject`将被处理。

建立这个事件机制的一个原因是，node中并不存在一个顶层的机制来确保`Promise`链中的错误被捕获，因为`Promise`天然的异步特性，导致其错误处理可能在本次事件循环之后的某个时刻，也就是可能在`unhandledRejection`事件触发之后的某个时刻。

相对于`uncaughtException`这种针对同步程序的错误机制，`rejectedHandled`和`unhandledRejection`是针对异步程序的错误处理机制。

如下，可以获得一个随应用进行中的异步错误表，且其会自行增长和收缩：

```
let unhandledRejections = new Map();
process.on('unhandledRejection', (reason, p) => {
	unhandledRejections.set(p, reason);
});
process.on('rejectionHandled', (p) => {
	unhandledRejections.delete(p);
});
```

#### `uncaughtException`

当未捕获的异常，一直冒泡到事件循环的最顶端时触发。处理这个事件的默认行为就是在`stderr`中打印错误消息栈并退出。给`uncaughtException`注册事件处理函数，则会覆盖默认的行为。

注意，使用`uncaughtException`时，不能将其作为错误处理和恢复机制来使用。未处理异常内在含义就是程序处于非预定义的状态，此时尝试去恢复程序会导致难以预料的问题。这时应该需要一个额外的进程去检测程序的状态，应在异常状况时重启相应的进程。

在`uncaughtException`的回调函数中抛出的异常是不可以被捕获的。此时程序会以一个非0的退出码退出，并且会打印错误栈。这样的目的是避免无线递归错误触发和捕获。

#### `unhandledRejection`

当`Promise`被拒绝，且在当前事件循环内，未注册相应的错误处理函数时触发`unhandledReject`。

其回调函数存在两个参数`reason`和`p`，`reason`的类型为`Error`或其他任何类型。

其和`rejectionHandled`一起用于`Promise`的异步事件捕获和处理。

#### `warning`

当node发出进程警告时触发`warning`事件。其并不是Node.JS或者JavaScript的标准错误流，Node.JS可以在检测问题——譬如问题代码、bug或者安全问题时触发`warning`事件。

其事件回调函数接受一个Error类型的参数。

默认情况下，Node.js会向`stderr`中打印`warning`消息。通过`--no-warnings`命令行选项可以取消默认的命令行输出，但是`warning`事件仍然会通过`process`触发。

```
events.defaultMaxListeners = 1;
process.on('foo', () => {});
process.on('foo', () => {});
//(node:92843) Warning: Possible EventEmitter memory leak detected. 2 foo listeners added. Use emitter.setMaxListeners() to increase limit
```

命令行选项：

`--trace-warnings`用于在默认的命令行输出中打印`warnin`的栈信息。

`--throw-deprecation`用于将自定义的deprecation警告转换成异常抛出。

`--trace-deprecation`用于打印deprecation警告的栈信息。

`--no-deprecation`将取消deprecation的报警信息。

`*-deprecation`命令行选项仅仅影响使用`DeprecationWarning`名称的警告。

可以通过`process.emitWarning()`来触发自动已的警告信息。

## 进程信号事件

当Node.js进程收到进程型号时触发。

* SIGUSR1 用于Node.JS启动debugger的启动。可以给这个事件注册侦听函数，但是这不会阻止debugger的启动。
* SIGTERM和SIGINT 针对非Windows平台可以在退出之前重置终端模式，退出码为128+`singal numner`。如果给这两个事件之一注册侦听函数，则会移除默认的行为。
* SIGPIPE 默认被忽略，可以注册侦听函数。
* SIGHUP 当控制台关闭时在Windows平台上产生，在其他平台上在相似的情况下也会产生。可以为其注册侦听函数，然而在Windows平台上Node.JS程序会在10秒内无条件的终止。在非Windows平台上，`SIGHUP`的默认行为是结束Node.JS程序，如果注册了侦听函数，则会移除的默认的行为。
* SIGTERM Windows平台不支持，可以注册侦听函数。
* SIGINT 在所有平台上支持，一般通过`CTRL+C`（也可以具体设置）产生。当终端raw模式启动，则不会产生这个事件。
* SIGBREAK 在Windows平台上，当按下`CTRL+BREAK`时触发。在非Windows平台可以监听，但是没有方法来触发。
* SIGWINCH 当控制台大小被重置时会触发。在Windows平台上，只有在鼠标移动时触发，或者在raw模式下。
* SIGKILL 不能注册侦听函数，会在所有的平台上无条件的终止Node.JS程序。
* SIGSTOP 不可以注册侦听函数。
* SIGBUS、SIGFPE、SIGSEGV和SIGILL，如果不是使用kill(2)手动触发事件，让程序进程处于执行回调不安全状态。可能导致程序处于无限循环状态，同时由于事件回调通过`process.on()`来注册的，也就是本质上是异步的，导致无法纠正程序中产生的问题。

注意：Windows平台上不支持发送进程信号，但是Node.JS提供了模拟——`process.kill()`和`subprocess.kill()`。发送信号0可以用于测试进程的存在。发送SIGINT、SIGTERM、SIGKILL会导致目标进程无条件的终止。

## 属性和方法

### Node.JS相关

#### process.config

存储用于编译当前Node.JS程序的配置选项。其等同于运行`./configure`中的`config.gypi`。

#### process.execArgv

* `<Object>`

为存储命令行参数的对象。

不同于`process.argv`，其不包含node程序路径和JavaScript文件路径两个元素。

#### process.execPath

启动当前进程的Node.JS程序的绝对路径。


#### process.release

返回当前Node.JS发布版本的meta信息，包含源代码tar包的URL地址等

#### process.version

返回Node.JS版本。

#### process.versions

返回一个对象，包含Node.JS的版本信息以及其依赖的库的版本信息。

其中`process.versions.modules`表示当前的ABI版本，这随着C++的API变化而变化，Node.JS不接受不同ABI版本编译的模块。

### 系统相关

#### process.arch

* String

返回当前Node.JS运行的处理器上的架构。例如：`arm`、`ia32`、`x64`。

#### process.platform

* `<string>`

返回进程的运行平台，例如`darwin`、`freebsd`、`linux`、`sunos`、`win32`

#### process.argv

* Array

返回包含Node.JS程序启动时传入的命令行参数的数组。数组的第一位为`process.execPath`。第二个参数为JavaScript文件的地址。之后的为其他的命令行参数。

#### process.argv0

* String

只读属性，为`process.argv[0]`的拷贝。

#### process.env

* `<Object>`

返回存储环境变量的对象。

可以在Node.JS程序内部更改相关的环境变量的值，但这个改动不会影响Node.JS外部的环境。

例如：

```
node -e 'process.env.foo = "bar"' && echo $foo
```

不会影响shell内的变量值。

在`process.env`对象上赋的值会导致相应的值被转化为字符串。

其中在Windows平台下，环境变量的值不区分大小写。

#### process.cpuUsage([previousValue])

* previousValue `<Object>` 上一次调用process.cpuUsage()时返回的结果
* return: `<Object>`
	- user `<integer>`
	- system `<integer>`

返回当前的进程user和system的CPU时间，其单位为微秒

#### process.memoryUsage()

返回进程所使用的内存的描述，单位为字节。格式如下：

```
{
  rss: 4935680,
  heapTotal: 1826816,
  heapUsed: 650472,
  external: 49879
}
```

`heapTotal`和`heapUsed`为v8使用的内存，`external`指的是绑定到JavaScript对象的C++对象使用的内存，这些JavaScript对象由v8管理。

#### process.title

返回当前进程的标题，一般就是ps中的输出的程序命令，对这个值赋值则会导致ps命令中的值也发送变化。

#### process.hrtime([time])

返回'high-resolution'的高分辨时间，返回结果为数组。

注意：不同的平台在设置这个值的时候会有不同的长度限制，

#### process.emitWarning(warning[, options])

* warning `<string>`|`<Error>` 要触发的警告
* option `<Object>`
	- type `<string>` 当`warning`为`String`时，`type`为要触发的警告的名字，默认为`warning`
	- code `<string>` 警告的唯一标识符
	- ctor `<Function>` 当`warning`为`String`时，`ctor`为限制产生的堆栈的可选函数参数，默认`process.emitWarning`
	- detail `<string>` 关于错误的额外描述

`process.emitWarning()`用于触发自定义或者应用特定的进程警告。可以通过`warning`事件来侦听相应的警告。

如果`warning`为`Error`类型，则会忽略`option`参数。

#### process.emitWarning(warning[, type[, code]][, ctor])

同上

如果`warning`不为`String`或者`Error`，则抛出`TypeError`异常。

注意，当`warning`为`Error`时，不能作为错误处理的替代机制。

当`type`为`DeprecationWarning`时，会有如下处理：

* 如果使用`--throw-deprecation`命令行标志，会将警告当作异常抛出
* 如果使用`--no-deprecation`命令行标志，会压制相应的警告信息
* 如果使用`--trace-deprecation`命令行标志，相应的警告栈信息页会被打印

#### 避免重复的警告消息

一个进程应该只触发一次`warning`消息。推荐在当前进程触发过`warning`事件之后，增加相应的标识，避免后续再次触发。

### 进程控制相关

#### process.pid

* `<integer>`

返回进程的pid

#### process.abort()

立即结束Node.JS程序并产生core文件。

#### process.channel

在Node.JS使用IPC通道建立，则该属性为指向IPC通道的引用。否则为undefined。

#### process.chdir(directory)

* directory `<string>`

更改Node.JS进程当前的工作目录，如果失败则抛出异常。

#### process.connected

* `<boolean>`

如果使用IPC通道来创建Node.JS进程，此时只要IPC通道还链接着process.connected则为true，在调用`process.disconnect()`之后则返回false。

如果process.connected为false，则不能再通过IPC通道发送进程消息。

#### process.cwd()

* return: <string>

返回Node.JS的当前工作目录。

#### process.kill(pid[, signal])

* pid `<number>` 进程pid
* signal `<string>`|`<number>` 要发送的信号，要么为字符串要么为数字，默认为`SIGTERM`。

向进程`pid`发送`signal`信号。

信号名诸如`SIGINT`和`SIGHUP`。

如果目标进程不存在，则抛出异常。比较特殊的是，可以发送0来检测进程是否存在。

注意，尽管这个函数的名字为kill，但是其只不过是发送进程消息的函数。

```
process.on('SIGHUP', () => {
	console.log('Got SIGHUP signal');
});

process.kill(process.pid, 'SIGHUP');
```

注意，当Node.JS进程接受到了`SIGUSR1`消息，则Node.JS会启动debugger。

#### process.disconnect()

但进程使用IPC通道创建的，则调用`process.disconnect()`会关闭IPC通道，当进程的所有的IPC通道关闭后则会运行子进程优雅的退出。

在自进程中执行`process.disconnect()`和在父进程中调用`ChildProcess.disconnect()`。

#### process.exit([code])

让当前进程以`code`的退出状态以同步的方式退出。如果未指定具体的`code`，则默认会使用`success`退出状态——0状态退出，或者如果设置了`process.exitCode`则使用这个值。Node.JS会在所有的`exit`侦听回调执行完成后，才彻底退出。

`process.exit()`会让进程尽快的退出，即使仍然存在异步的操作未完成。

通常不需要特别的调用`process.exit()`，Node.JS程序会在系统事件循环中无额外执行任务时自行退出。可以通过设置`process.exitCode`来告知进程使用那个退出状态退出。

向`process.stdout`中写入消息可能是异步的，可能会跨越多个Node.JS的事件循环。执行`process.exit()`会强制在完成这些异步操作在完成之前退出，导致可能存在一定的问题。

如果需要因异常状况而退出，可以抛出未捕获的异常，可以让进程以相对安全的方式退出。

#### process.exitCode

* `<string>`

进程退出时的退出状态码，调用`process.exit([code])`会覆盖这个值。


#### process.nextTick(callback[,...args])

* callback `<Function>`
* ...args `<any>` 传递给callback的参数

用于将callback添加到`next-tick`队列中，当本次事件循环执行完成时，所有在这个队列中任务都将会被执行。

这和`setTimeout(fn, 0)`不一样，因为其更加高效，而且会在任何I/O操作之前触发。

可以用于在创建对象时，在任何I/O之前注册事件回调。

```
function MyThing (options) {
	this.setupOptions(options);
	process.nextTick(() => {
		this.startDoingStuff();
	});
}

let thing = new MyThing();
thing.getReadyForStruff();
```

另外也可以将同步的操作变成异步的，因为其足够高效，所以可以轻易将任务延迟到事件循环之后进行。

#### process.send(message[, sendHandle[, options][, callback])

用于进程间通过IPC通道发布消息。

### 权限相关方法

#### 用户权限

名词：

* real user id —— ruid
* real group id —— rgid
* effective user id —— euid
* effective group id —— egid
* saved set-user-id —— suid
* saved set-group-id —— sgid

其中，
第1、2条，由启动进程的用户决定，通常是当前登录的用户
第3、4条，在进程启动时一般从1、2复制而来，或者当对于的可执行文件的set-user-id或set-group-id（chomd u+s）标识位为true时，为该文件所属的用户/组的权限。
第5、6条，从euid/egid复制。

当进程启动时，

1. 设置当前登录用户的uid/gid为当前进程的ruid/rgid。
2. 获取文件的suid和sgid，若为1，则获取可执行文件的uid/gid，并设置进程的euid/egid，否则直接从ruid和rgid拷贝。

#### 附加组ID

也就是'Supplmentary Group ID'，在早期的系统中，一个用户只属于一个组，那么当这个用户同时想要进行其他组的操作，比如ftp，则必须更换组。后再引入了附加组概念，用户可以加入其他的附加组，最多可以加16个附加组。

#### process.getegid()

#### process.setegid(id)

#### process.geteuid()

#### process.seteuid(id)

#### process.getuid()

#### process.setuid(id)

#### process.getgid()

#### process.setgid(id)

#### process.setgroups(groups)

* group `<Array>` 

设置附加组ID，这个方法需要`root`权限或者`CAP_SETGID`权限。

`group`参数可以包含分组ID或者分组名。

#### process.getgroups()

返回附加组ID列表。

#### process.initgroups(user, extra_group)

* user `<string>`|`<number>` 用户名或者用户标识ID
* extra_group `<string>`|`<number>` 分组名或者分组ID

读取`/etc/group`文件并初始化组访问列表，获取当前用户所属的所有组。调用这个方法需要Node.JS具有`root`权限或者`CAP_SETGID`权限。

这个方法只在POSIX平台上支持。

#### process.umask([mask])

返回或者设置Node.JS进程文件创建的权限掩码。

### 控制台相关

#### 进程IO

`process.stdout`和`procoss.stdout`不同于Node.JS中的一般的流：

1. 其被`console.log()`和`console.error()`所使用
2. 不可以被关闭（调用`end()`会抛出异常）
3. 不会触发`finish`事件
4. 写入可能是同步的，这取决于流所连接的对象，系统是否是Windows或者POSIX：
	- 文件：在Windows和POSIX上是同步的
	- TTYs：在Windows上是异步的，在POSIX上是同步的
	- Pipes：在Windows是同步的，在POSIX上是异步的

这些特性部分是因为历史原因，因为改变他们会导致不兼容性。

同步的写入可以避免`console.log()`和`console.error()`的输出相互交叉，以及`process.exit()`丢弃异步写入的特征。

注意，同步写入会阻塞事件循环，直到写入完成。对于写入文件而言，可能是瞬间的事，但是在一个高系统负载的情况下，可能会经常导致事件队列阻塞并且阻塞事件较长，从而引起性能问题。

可以通过`process.stdin.isTTY`等来判断是否连接到`TTY`设备。

#### process.stderr

返回连接到`stderr`（fd为2）的流，为一个`net.Socket`对象，除非fd2指向一个文件，此时其为一个Writable流。

#### process.stdin

返回连接到`stdin`（fd为0）的流，为一个`net.Socket`对象，除非fd0指向一个文件，此时其为一个Readable流。

#### process.stdout

返回连接到`stdout`（fd为1）的流，为一个`net.Socket`对象，除非fd1指向一个文件，此时为一个Writable流。

#### process.mainModule

返回当前执行进程的入口文件。

同`require.main`，不同点在于会随着程序的主模块变更而变更。

#### process.uptime()

返回当前Node.JS进程已经运行的事件，单位为秒。

## 退出状态码

通常，进程会以0作为状态码退出。退出状态码具体列表如下：

* 1 - 未捕获异常，也就是`uncaughtException` 
* 2 - 未使用（为Bash保留作为内部使用）
* 3 - 内部JavaScript解析错误，一般在开发Node.JS本身的时候才会抛出
* 4 - 内部JavaScript执行错误，同上
* 5 - 致命错误，v8中存在不可恢复的错误，这会在控制台输出以`Fatal Error`开头的错误。
* 6 - 非函数内部异常句柄，存在未捕获异常，内部致命异常处理函数不知为何被设置成了非函数对象，所以无法执行
* 7 - 内部异常句柄运行时异常
* 8 - 未使用
* 9 - 不合法的参数，
* 10 - 内部JavaScript运行时异常，JavaScript源码在Node.JS启动是抛出异常
* 12 - 不合法的Debug参数
* > 128 - 如果Node.JS进程接受到了诸如`SIGKILL`或者`SIGHUP`，那么退出码将为128加上信号码的值。这是POSIX的标准动作，因为退出码为7比特的整型，信号会设置高位的字节，之后为具体的信号码的值。

