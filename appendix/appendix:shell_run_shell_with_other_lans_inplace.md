# 在Shell中执行其他语言脚本的特殊技巧

## 问题场景描述

现在有一个shell执行文件，一般而言如果要在其中执行其他语言的脚本，比如python脚本，可以这样做：

```
#!/bin/sh

python test.py
```

此时我们需要一个shell脚本文件以及一个名为`test.py`的python文件，但是在有些场景下可能并并不期望使用两个文件来实现这个场景，比如用于编译Node.JS的`configure`文件。

## 运行示例

如下代码截取自Node.JS工程的`configure`文件，并做了适当的修改。

```bash
#!/bin/sh

_=[ 'exec' '/bin/sh' '-c' '''
which python2.7 >/dev/null && exec python2.7 "$0" "$@"
which python2 >/dev/null && exec python2 "$0" "$@"
exec python "$0" "$@"
''' "$0" "$@"
]
del _

import sys
print sys.argv
```

假设其文件名为`test`，在命令行中运行`sh test`，其运行结果如下：

```
[test]
```

非常神奇的其居然执行了python的代码，下面来解释下其原因。

## 解释

其中秘诀就是文件开头的那一块复杂的代码，在运行`sh test`时，当前文件会以shell脚本的方式解释和执行文件，那么：

1. `_`为一个合法的shell变量；
2. 中括号`[]`原是shell中的条件测试，但是在这里以shell的角度来看并不表示任何含义，原因在于在第一个`[`后面的空格，所以这里实际上是两个语句`_=[`以及后面的那段执行脚本；
3. 最后`exec`是shell命令，系统调用`exec`是以新的进程去代替原来的进程而不创建新的进程，且其之后的代码**不再**继续执行，可以看成其将原进程窃取来完成自己的调用。

问题在于为什么用这么复杂的写法来执行shell的执行，原因在于当如下命令执行时：

```bash
which python2.7 >/dev/null && exec python2.7 "$0" "$@"
which python2 >/dev/null && exec python2 "$0" "$@"
exec python "$0" "$@"
```

只要其中一个执行了，那么当前文件（`$0`表示当前文件）就会再次以用python执行（同时由于`exec`的作用第一次的shell执行会终止），所以当前文件也必须符合python的语法。因此按如下方式编写的代码片段即是一段合法的shell脚本也是一段合法的python脚本，所以由`python`命令启动的脚本第二次解释执行如下代码会通过解释执行并不会抛出异常，最终会执行之后的所有`python`语句。

```
_=[ 'exec' '/bin/sh' '-c' '''
which python2.7 >/dev/null && exec python2.7 "$0" "$@"
which python2 >/dev/null && exec python2 "$0" "$@"
exec python "$0" "$@"
''' "$0" "$@"
]
```

另一个非常关键的点即使`shell`和`python`都可以用`#`作为注释符号，所以文件最开始的`#!/bin/sh`不会有任何问题。

这里总结一下，要实现如上的场景的语言所必须具备的功能：

1. 必须可以以`#`或者`#!`作为注释符号或者跳过shebang符号的语句
2. 语法上可以和shell完成兼容

## 在Node.JS上实现同样的能力

其关键在于如何写入兼容bash和js的语法，经过多次实验，如下代码能够实现上述同样的功能。

```bash
#!/bin/sh

arr=1// which node > /dev/null && exec node "$0" "$@"
arr=1// exec echo "Error: I can't find node anywhere"

arr=undefined
console.log(88888)
```

其中`1//`在shell中会被解释成路径的地址，而在Node.JS中会被解释成`arr=1`以及一个当行注释，且Node.JS会跳过所有的shebang，所以第一句`#!/bin/sh`也是合法的Node.JS。

但是这里需要一个变量`arr`，可以将其升级一下:


```bash
#!/bin/sh

':' // which node > /dev/null && exec node "$0" "$@"
':' // exec echo "Error: I can't find node anywhere"

console.log(88888)
```
