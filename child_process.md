## child_process

使用类似于`popen`的方式创建进程。主要通过`child_process.spawn()`来完成，其返回被创建的进程。

同时Node.JS提供了一系列的替代函数来实现相同的功能，注意这些替代函数都是基于`child_process.spawn()`或者`child_process.spawnSync()`：

* child_process.exec() 创建一个shell并在那个shell中运行命令。
* child_process.execFile() 类似于`child_process.exec()`，直接执行一个命令而不用创建一个shell。
* child_process.fork() 创建一个Node.JS进程，并执行一个特定的模块，包含一个可以在父子进程通信的IPC通道。
* child_process.execSync() 对应的方法的同步版，会阻塞事件循环
* child_process.execFileSync() 同

使用异步进程创建的方式对性能有益。

## 异步进程创建

`child_process.spawn()`、`child_process.fork()`、`child_process.exec()`、`child_process.execFile()`方法为遵循通用的异步模式。

每个方法返回一个`ChildProcess`实例。这些方法继承了`EventEmitter`，可以在父进程注册事件来监听子进程的变化。

`child_process.exec()`、`child_process.execFile()`方法存在可选的`callback`函数参数，在子进程结束的时候执行。

### 在Windows平台来依据`.bat`和`.cmd`文件来创建进程

`child_process.exec()`和`child_process.execFile()`在不同的平台上会表现出很大的不同。在类Unix平台上，`child_process.execFile()`因为不创建shell，所以会更高效。在Windows平台上，`.bat`和`.cmd`文件必须在命令行终端执行，所以不能使用`child_process.execFile()`执行。当在Windows上执行，`.bat`和`.cmd`文件可以使用`child_process.spawn()`通过shell模式来执行。

### child_process.exec(command[, options][, callback])

* command `<string>` 要执行的命令，可以包含空格分隔的参数
* options `<Object>` 
	- cmd `<string>` 子进程执行的工作目录
	- env `<object>` 环境变量的键值对
	- encoding `<string>` 默认为`utf-8`
	- shell `<string>` 执行命令行的shell（默认UNIX上为`'/bin/sh'`，在Windows上为`process.env.ComSpec`）
	- timeout `<number>` 默认为0
	- maxBuffer `<number>` stdout和stderr上最多数据量，单位为字节。（默认为`200*1024`），如果超出限制则命令行终止。
	- killSignal `<string>`|`<integer>`默认为`SIGTERM`
	- uid `<number>` 设置进程的uid
	- gid `<number>` 设置进程的gid
* callback `<Function>` 当子进程结束的时候调用
	- error `<Error>`
	- stdout `<string>`|`<Buffer>`
	- stderr `<string>`|`<Buffer>`
* Returns：`ChildProcess`

创建一个shell然后执行`command`，缓冲产生的输出。`command`字符串参数会直接被shell处理，所以相应的特殊字符要特殊处理。

```
exec('"/path/to/test file/test.sh" arg1 arg2');
exec('echo "The \\$HOME variable is $HOME"');
```

注意：必须传递安全的字符参数。任何的shell特殊字符都有可能触发一些执行逻辑。

```
const { exec } = require('child_process');
exec('cat *.js bad_file | wc -l', (error, stdout, stderr) => {
	if (error) {
		console.error(`exec error: ${error}`);
		return;
	}
	
	console.log(`stdout: ${stdout}`);
	console.log(`stderr: ${stderr}`);
});
```

如果有`callback`参数，其执行参数为`error`、`stdout`、`stderr`。如果执行成，则`error`为`null`。如果执行错误则`error`为`Error`的实例。`error.code`为子进程的退出状态码，`error.signal`为进程结束时进程信号。任何非0的退出状态码都被看作是错误情况。

`stdout`和`stderr`为则包含子进程的输出内容，默认情况下会被按UTF-8方式解码为字符串，如果`encoding`值为`buffer`，则这两个值为`Buffer`。

`options`的默认值为：

```
{
	encoding: 'utf8',
	timeout: 0,
	maxBuffer: 200 * 1024,
	killSignal: 'SIGTERM',
	cmd: null,
	env: null
}
```

如果`timeout`大于0，此时如果子进程的运行事件大于`timeout`，则父进程会发送`killSignal`信号（默认是`SIGTERM`）给子进程。

如果使用`util.promisify()`处理，则会返回一个`Promise`对象，其包含`stdout`和`stderr`属性。如果出现错误，则返回一个被拒绝的`Promise`，但是其包含`stdout`和`stderr`属性。

例如：

```
const util = require('util');
const exec = util.promisify(require('child_process').exec);

async function lsExample() {
	const { stdout, stderr } = await exec('ls');
	console.log('stdout:', stdout);
	console.log('stderr:', stderr);
}

lsExample();
```

### child_process.execFile(file[, args][, options][, callback])

类似于`child_process.exec()`，但是不会产生一个shell，所以特定的文件会直接用来产生进程，所以会比`child_process.exec()`性能更好。

同时由于其没有产生shell，所以也不支持I/O重定向以及文件通配符。

### child_process.fork(modulePath[, args][, options])

* modulePath `<string>` 子进程运行的模块
* args `<Array>` 参数列表
* options `<Object>` 
	- cmd `<string>` 子进程执行的工作目录
	- env `<object>` 环境变量的键值对
	- execPath `<string>` 用于创建子进程的执行程序
	- execArgv `<Array>` 传递给可执行程序的参数列表
	- silent `<boolean>` 如果为true，则子进程的`stdin`、`stdout`、`stderr`会传输给父进程，否则会从父进程继承，默认为false。参考`child_process.spawn()`的`stdio`参数获取`pipe`和`inherit`详细信息。
	- stdio `<Array>`|`<string>` 参考`child_process.spawn()`的`stdio`。当提供这个参数，会覆盖这个参数。如果是参数，则必须包含一个值为`ipc`的元素。
	- uid `<number>` 设置进程的uid
	- gid `<number>` 设置进程的gid
* Return: `ChildProcess`

`child_process.fork()`为`child_process.spawn()`的特殊情况，用于创建一个Node.JS进程。其返回的`ChildProcess`实例包含内置的IPC通道，运行父子进程之间通信。

注意，创建的子进程和父进程是相互独立的，各自使用独立的内存空间，以及底层使用不同的v8实例等。

一般`child_process.fork()`会使用`process.execPath`来创建独立的Node.JS实例，可以在`options.execPath`上设置其他的Node.JS执行文件。

### child_process.spawn(command[, args][, options])

* command `<string>` 要运行的命令
* args `<Array>` 运行参数
* options `<Object>` 
	- cmd `<string>` 子进程执行的工作目录
	- env `<object>` 环境变量的键值对
	- argv0 `<string>` 设置子进程的`argv[0]`值。如果不指定则为`command`的值
	- stdio `<Array>`|`<string>` 子进程的`stdio`配置
	- detached `<boolean>` 让子进程独立于父进程运行。具体行为和平台有关。
	- uid `<number>` 设置进程的uid
	- gid `<number>` 设置进程的gid
	- shell `<boolean>`|`<string>` 如果为true，则在shell中运行命令。可以通过字符传的方式具体指定shell的类型。默认为false。
* Return: `ChildProcess`

```
const { spawn } = require('child_process');
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
	console.log(`stdout: ${data}`);
}

ls.stderr.on('data', (data) => {
	console.log(`stdout: ${data}`);
}

ls.on('close', (code) => {
	console.log(`child process exit with code ${code}`);
}
```

#### options.detached

在Windows平台上，如果设置其为true，可以让子进程在父进程退出之后继续运行。子进程会有独立的命令行工作台。

在非Windows平台上，如果设置其为true，子进程会创建一个新的进程组和会话。注意，在父进程退出后，子进程无论是否`detached`都可能继续运行。

默认情况下，父进程会等待`detached`的子进程退出。要取消这个默认行为，可以使用`subprocess.unref()`方法。使用这个方法会导致父进程的事件循环中取消将子进程作为引用计数，从而导致父进程不用等待子进程的退出，除非在父子进程之间建立IPC通道。

当使用`detached`选项来创建长时间运行的进程，进程在父进程退出之后不会保持在后台运行，除非提供`stdio`配置中不连接父进程。如果继承父进程的`stdio`，子进程会和控制终端保持连接。

例如，对于长时间运行的进程，可以通过分离和忽略父进程`stdio`，可以达到在子进程中忽略父进程的终止。

```
const { spawn } = require('child_process');

const subprocess = spawn(process.argv[0], ['child_program.js'], {
	detached: true,
	stdio: 'ignore'
};

subprocess.unref();
```

#### options.stdio

用于配置父子进程之间的管道。默认情况下，子进程的`stdin`、`stdout`、`stderr`直接相应的重定向到`subprocess.stdin`、`subprocess.stdout`、`subprocess.stderr`的`Stream`对象上。这等同于`options.stdio`的设置为`['pipe', 'pipe', 'pipe']`。

`options.stdio`可以为如下几个字符串之一：

* 'pipe' - 等同于`['pipe', 'pipe', 'pipe']`
* 'ignore' - 等同于`['ignore', 'ignore', 'ignore']`
* 'inherit' - 等同于`[process.stdin, process.stdout, process.stderr]`或者`[0, 1, 2]`

否则，`options.stdio`的值为数组，其中每一个元素对应于子进程中的设备描述符（fd）。fds为0、1和2对应于`stdin`、`stdout`、`stderr`。另外可以用其他的值来在父子进程中创建额外管道。其值如下：

* 'pipe' - 在父子进程之间创建管道，管道的父进程端通过`child_process`对象的属性来获取——`subprocess.stdio[fd]`。0-2的设备也可以通过`subprocess.stdin`、`subprocess.stdout`、`subprocess.stderr`来获取。
* 'ipc' - 在父子进程之间创建IPC管道，在父子进程传输信息。`ChildProcess`的stdio最多只能有一个IPC设置。设置这个选项或会开启`subprocess.send()`方法。如果子进程往设备文件中写入JSON信息，那么父进程中`subprocess.on('message')`会被触发。如果子进程是Node.JS进程，IPC通道会开启子进程中`process.send()`、`process.disconnect()`、`process.on('disconnect')`和`process.on('message')`方法。
* 'ignore' - 让子进程忽略文件设备。Node.JS会为其创建的子进程打开设备0-2，设置文件设备为'ignore'导致Node.JS打开`/dev/null`并将其关联到子进程的设备文件。
* `<Stream>`对象 - 在父子进程之间共享一个Readable或者Writable流，指向tty、文件、socket或管道。注意流对象必须存在潜在的文件设备（文件流必须在`open`事件触发才有）。
* 正整数 - 整数会被解析成父文件中打开的文件描述符，和`<Stream>`类似其会和子进程共享。
* null、undefined - 使用默认值。对于设备文件0、1、2会创建管道，对于大于等于3的值，默认值为'ignore'

## 同步进程创建

和相应的异步方法类似，但是在子进程退出之前，函数不会返回，也就是会阻塞事件循环。

注意即使当前进程被中断，或者接受到`SIGTERM`事件，仍然会等待子进程结束后才退出。

### child_process.execFileSync(file[, args][, options])
### child_process.execSync(command[, options])
### child_process.spawnSync(command[, args][, options])

## 类：ChildProcess

只能通过进程创建的方法去创建`ChildProcess`的实例。

### 事件：close

### 事件：disconnect

### 事件：error

### 事件：exit

### 事件：message

### channel

### connected

### disconnected

### kill([signal])

### killed

### pid

### send(message[, sendHandle[, options]][, callbcak])

注意发送`{cmd: 'NODE_foo'}`这种消息，所有以'Node'开头的值都被认为是Node.JS内核保留使用，且不会触发子进程的'message'事件。这种消息可以通过'internalMessage'事件来监听，但是应用开发这不应该依赖于次，因为和可能会在后续的版本中有变化。

`sendHandle`用于传递TCP服务或者套接字到子进程。子进程会在'message'回调的第二个参数中接受中接受这个值。

`callback`参数在发送消息之后执行，但是不保证在子进程接受到消息才执行。这个函数只有一个参数，对于执行成功为`null`，否则为`Error`。如果没有`callback`参数，且执行失败，则`ChildProcess`会触发`'error'`事件。

如果通道关闭或者未发送消息个数超过阈值，则返回false，否则返回true。

例如，发送servers对象

```
const subprocess = require('child_process').fork('subprocess.js');
const server = require('net').createServer();

server.on('connection', (socket) => {
	socket.end('handled by parent');
});

server.listen(1337, () => {
	subprocess.send('server', server);
});
```

```
process.on('message', (m, server) => {
	if ( m === 'server' ) {
		server.on('connection', (socket) => {
			socket.end('handled by child');
		};
	}
});
```

则在父子进程之间共享server对象，从而分布完成不同的操作。

### stderr

### stdin

### stdio

### stdout

## `maxBuffer`和Unicode

`maxBuffer`设置`stdout`和`stderr`上运行的最大字节数。如果超出这个数，则子进程会退出。对于Unicode则实际的字节个数会大于字符个数。






