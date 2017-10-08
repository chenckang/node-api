# Cluster

Cluster模块用于创建共享服务端口的子进程。

## 工作原理

工作进程是使用`child_process.fork()`创建的，所以在父子进程之间可以通过IPC通道传递消息，且可以传递server对象。

cluster模块支持两种分发请求连接的方式。

第一种，为round-robin方法，主进程接受到请求连接，并将其以round-robin方法分发给子进程，并且增加一点内部机制来避免过度负载。

第二种，主进程侦听端口，并将连接发送给特定的工作进程，工作进程则直接接受连接。

第二种方法表明上性能较好，但是由于操作系统的调度问题会导致非常不均匀的负载分配。经过事件运行，结果在8个进程中，有70%的请求会被其中特定的两个进程处理。

## 类：Worker

包含所有工作进程的公开属性和方法。在工作进程中通过`cluster.worker`获得引用，在主进程中通过`cluster.workers`来引用。

### 事件：disconnect

### 事件：error

### 事件：exit

### 事件：listening

### 事件：message

### 事件：online

### worker.disconnect()

* 返回：`<Worker>` 返回`worker`的引用

在工作进程中，会关闭所有的server，等待这些server对象上的`close`事件触发，随后关闭IPC通道。

在主进程中，会向所有的工作进程发布内部消息，触发他们各自执行`disconnect()`方法。

会设置`.exitedAfterDisconnect`属性。

注意，在server关闭之后，其将不会接受新的请求连接，但是这些请求连接仍然可以被其他工作进程所接受。正在退出的请求连接是可以正常关闭的。当没有更多的请求连接要处理时，会开始关闭IPC通道，这样会让工作进程优雅的退出。

注意，其不等同于工作进程中的`process.disconnect()`方法（因为其会等待已有的连接关闭，然后才关闭IPC通道）。

由于长连接可能会阻止工作进程从主进程中断开，所以为了让程序执行特定的程序来关闭这些连接，需要发送一些消息来通知工作进程执行一些操作。也可以指定一个`timeout`参数，在一定事件内如果还没触发`disconnect`，则直接杀掉进程。

### worker.existedAfterDisconnect

通过执行`kill()`或者`disconnect`才会触发其值的设置。

这个布尔变量用于标识进程是否是自行退出或者意外退出。主进程可以依据这个值选择是否重启相应的进程。

### worker.id

在主进程创建工作进程是分配的唯一ID（一般就是从0开始的递增整数）。

### worker.isConnected()

如果工作进程和主进程间有IPC通道，则为true，否则为false。当工作进程创建时，和主进程之间会有IPC通道，当`disconnect`事件触发后则返回false。

### worker.isDead()

当工作进程终结时（可能由于退出或者接受到相应的信号），返回true，否则返回false。

### worker.kill([signal='SIGTEMR'])

* signal `<string>` 发送给工作进程的信号名

这个函数会杀死进程，在主进程中，通过和工作进程断开连接，在断开后发送`signal`信号。在工作进程中，会关闭和主进程间的通道，并以状态码0退出。

会设置`.existedAfterDisconnect`的值。

为了向后兼容系，这个方法和`worker.destroy()`等同。

注意，这个方法不等同于`process.kill()`

### worker.process

即为创建的进程，在工作进程中为全局变量`process`。

### worker.send(message[, sendHandle][, callback])

在主进程中等同于`ChildProcess.send()`。

在工作进程中等同于`process.send()`

### worker.suicide

> deprecacted 

等同于`worker.exitedAfterDisconnect`

## 事件：disconnect

工作进程的IPC通道关闭时触发

## 事件：exit

工作进程退出是触发，可以在这里重启进程。

## 事件：fork

但有新的工作进程被创建是时触发。

## 事件：listening

* worker `<cluster.Worker>`
* address `<object>`

当在工作进程中调用`listen()`方法之后，server对象上触发了`listening`事件，同时也会在`cluster`上触发`listening`事件。

其中`address`包含`address`、`port`和`addressType`，当监听多个地址时这个属性会有用。

`addressType`值如下：

* 4(TCPv4)
* 6(TCPv6)
* -1(unix domain socket)
* "udp4"或者"udp6"

## 事件：message

* worker `<cluster.Worker>`
* message `<Object>`
* handle `<undefined>`|`Object`

## 事件：online

当创建一个进程后会触发`online`事件，其相对于`fork`方法的不同是：`fork`在创建进程时触发，而`online`在创建的进程实际运行时触发。

## 事件：setup

* settings `<Object>`

每次试着`setupMaster()`时触发。

`settings`值即为调用`setupMaster()`时，`cluster.settings`的值。

如果要求精准性，建议使用`cluster.settings`。

## cluster.disconnect([callback])

## cluster.fork([env])

## cluster.isMaster

取决于`process.env.NODE_UNIQUE_ID`，如果这个值为undefined，则`isMaster`为true。

## cluster.isWorker

为cluster.isMaster的反面。

## cluster.schedulingPolicy

要么为`cluster.SCHED_RR`表示round-robin方法，或者为`cluster.SCHED_NONE`则由操作系统来分配。这在第一次创建进程或者`cluster.setupMaster`执行时设置其值，并在之后不能被更改。

`cluster.SCHED_RR`为除Windows平台外的默认值。

也可以通过环境变量`NODE_CLUSTER_SCHED_POLICY`来设置，可能的值为`"rr"`或者`"none"`。

## cluster.settings

* `<Object>`
	- execArgv `<Array>` 传递给Node.JS执行程序的参数，默认为`process.execArgv`。
	- exec `<string>` 要执行的文件地址，默认为`process.execArgv[1]`
	- args `<Array>` 传递给工作进程的参数，默认为`process.argv.slice(2)`
	- silent `<boolean>` 是否需要向父进程的stdio输出。
	- stdio `<Array>` 配置创建进程的管道配置，因为cluster模块依赖于IPC通道，所以其中必须含有`"ipc"`元素，且这个配置会覆盖`silent`
	- uid `<number>` 设置进程的用户id
	- gid `<number>` 设置进程的组id
	- inspectPort `<number>`|`<function>` 设置工作进程的监听端口，可以为数字或者返回数字的函数。默认情况下每个进程会在主进程的`process.debugPort`之上，按增1的方式获取其自己的端口。

这个值需要使用`cluster.setupMaster()`来设置，不能直接修改。

## cluster.setupMaster([settings])

用于更改默认的进程创建配置，一旦执行其值会体现在`cluster.settings`上。

注意：

* 仅影响在其被调用之后的`.fork()`方法，对已经运行的进程无影响。
* 无法设置`env`的值，`.fork`方法本身的参数中就有可选项`env`
* `cluster.settings`的默认值只在第一次执行本方法时有效，之后都会使用上一次调用本方法产生的值。

## cluster.worker

## cluster.workers


