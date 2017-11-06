# Buffer

在ES2015（ES6）引入`TypedArray`前，JavaScript语言语言没有机制来处理二进制数据流。`Buffer`作为Node.JS的一部分用于在TCP流和文件系统中操作二进制数据流。

现在ES6中加入了`TypedArray`，`Buffer`类继承自`Uint8Array`但是增加了更多的优化从而适用于在Node.JS使用。

`Buffer`实例类似于整数数组，但是其长度不可以变更，其在V8堆之外进行原始的内存分配。`Buffer`的大小在创建时建立，且之后不能更改。

`Buffer`类在Node.JS中是全局变量，所以不用通过`require('buffer').Buffer`来获取。

示例：

```js
// Creates a zero-filled Buffer of length 10.
const buf1 = Buffer.alloc(10);

// Creates a Buffer of length 10, filled with 0x1.
const buf2 = Buffer.alloc(10, 1);

// Creates an uninitialized buffer of length 10.
// This is faster than calling Buffer.alloc() but the returned
// Buffer instance might contain old data that needs to be
// overwritten using either fill() or write().
const buf3 = Buffer.allocUnsafe(10);

// Creates a Buffer containing [0x1, 0x2, 0x3].
const buf4 = Buffer.from([1, 2, 3]);

// Creates a Buffer containing UTF-8 bytes [0x74, 0xc3, 0xa9, 0x73, 0x74].
const buf5 = Buffer.from('tést');

// Creates a Buffer containing Latin-1 bytes [0x74, 0xe9, 0x73, 0x74].
const buf6 = Buffer.from('tést', 'latin1');
```

## Buffer.from()，Buffer.alloc()和Buffer.allocUnSafe()

在Node.JS的v6版本之前，`Buffer`实例通过`Buffer`构造器来创建，通过不同的参数返回创建的不同的`Buffer`实例。

* 以数值作为`Buffer`的第一个参数（例如：`new Buffer(0)`)，将会用指定的大小分配一个新的`Buffer`对象。在Node.JS的8.0.0之前，创建的`Buffer`实例不会被初始化所以可能敏感数据。这样的`Buffer`实例必须相应的使用`buf.fill(0)`或者完全写入覆盖`Buffer`的数据。尽管这个行为主要是用于提示性能，实际的开发实践却表明应该在创建快“但未初始化”和“慢但是安全”的`Buffer`之间作出明确的区分。在Node.JS8.0.0之后，`Buffer(num)`和`new Buffer(num)`会返回经过初始化内存的`Buffer`对象。
* 传递字符串或者`Buffer`作为第一个参数，会将改参数对象的内容拷贝到返回的`Buffer`对象中。
* 传递`ArrayBuffer`作为第一个参数会返回共享`ArrayBuffer`内存的`Buffer`对象。

`new Buffer()`的行为会因为第一个参数的不同会有明显的差异，在应用程序中未进行恰当的参数校验或者没有恰当的初始化分配的`Buffer`内容，会导致安全和可靠性方面的问题。

所以为了让创建`Buffer`对象更加可靠和不易出错，不同形式的`new Buffer()`都被标记为过时的，由`Buffer.from()`，`Buffer.alloc()`和`Buffer.allocUnsafe()`取代。

开发这应该将其程序中使用的`new Buffer()`迁移到如下的新API：

* `Buffer.from(array)`返回包含指定二进制数据的`Buffer`对象
* `Buffer.from(arrayBuffer[, byteOffset[, length]])` 返回和参数中的`ArrayBuffer`共享内存的`Buffer`对象
* `Buffer.from(buffer)`返回含有给定的`Buffer`对象内容拷贝的`Buffer`对象
* `Buffer.from(string[, encoding])`返回包含给定字符串拷贝的`Buffer`对象
* `Buffer.alloc(size[, fill[, encoding]])`返回一个大小指定的被填充的`Buffer`对象。这个方法会比`Buffer.allocUnsafe(size)`慢很多，但是可以确保新创建的`Buffer`实例不包含旧数据和潜在的敏感数据。
* `Buffer.allocUnsafe(size)`和`Buffer.allocUnsafeSlow(size)`返回执行大小的`Buffer`对象，这些对象必须通过`buf.fill(0)`或者完全内容写入来初始化。

如果分配的类似小于或等于`Buffer.poolSize`二分之一，通过`Buffer.allocUnsafe`创建的`Buffer`实例从共享的内部内存池中分配空间。`Buffer.allocUnsafeSlow()`则不会使用共享的内部内存池。

## ｀--zero-fill-buffers｀命令行选项

可以使用`--zero-fill-buffers`来强制所有创建的`Buffer`实例（使用`new Buffer(size)`，`Buffer.allocUnsafe()`，`Buffer.allocUnsafeSlow`或者`new SlowBuffer(size)`）在创建的时候自行**zero-filled**。使用这个标志会改变这些方法的默认行为并会在性能上会有重大的影响。仅在有必要要确保所有的新建`Buffer`实例不能包含敏感数据时，使用`--zero-fill-buffers`才是必要的。

例如：

```shell
$ node --zero-fill-buffers
> Buffer.allocUnsafe(5);
<Buffer 00 00 00 00 00>
```

## 什么导致`Buffer.allocUnsafe()`和`Buffer.allocUnsafeSlow()`不安全？

当执行`Buffer.allocUnsafe()`和`Buffer.allocUnsafeSlow()`时，分配的内存片段是非初始化的（不是归零的）。然而这个设计使得内存分配特别快，分类的内存片段可能含有敏感的历史数据。使用`Buffer.allocUnsafe()`创建的`Buffer`，如果不完全的重写分配的内存，会导致在读取`Buffer`内存时泄漏历史数据。

尽管使用`Buffer.allocUnsafe()`具备明确的性能优势，但是也要额外关注避免引入安全漏洞。

## `Buffer`和字符编码

`Buffer`实例一般用于表示诸如UTF-8、UCS2或者十六进制的数据。可以让`Buffer`和原始的JavaScript字符传之间使用指定的字符编码方式来进行互相转换。

例如：

```js
const buf = Buffer.from('hello world', 'ascii');

// Prints: 68656c6c6f20776f726c64
console.log(buf.toString('hex'));

// Prints: aGVsbG8gd29ybGQ=
console.log(buf.toString('base64'));
```

Node.JS当前支持如下字符编码：

* `ascii` - 对于7比特的数据，这个编码方式会很快且会去除可能存在的高位字节。
* `utf8` - 多字节Unicode编码，很多web页面和其他的文档格式使用UTF-8。
* `utf16le` - 2字节或者4字节，“little-endian”模式的Unicode字符。支持代理快（U+10000到U+10FFFF）。
* `ucs2` - `utf16le`的别名
* `base64` - Base64编码。当从字符串创建`Buffer`，这个编码会正确接受“URL和文件名安全字符”，如在`RFC4648，section5`中描述。
* `latin1` - 一种将`Buffer`编码成一个字节编码的字符串
* `binary` - `latin1`的别名
* `hex` - 将每个字节编码成两个16进制的字符

注意：现代的浏览器遵从`WHATWG Encoding Standard`规范，其中`latin1`以及`ISO-8859-1`归为`win-1252`的别名。这意味着当执行诸如`http.get()`时，如果返回的字符集为WHATWG规范中列举的字符集之一，实际上则可能返回`win-1252-encoded`数据，此时使用`latin1`编码可能导致解码字符存在问题。

## `Buffer`和`TypedArray`

`Buffer`实例也是`Uint8Array`实例。然而，会和ECMAScript2015中`TypedArray`存在一点不兼容的特性。例如，尽管`ArrayBuffer#slice()`返回内容的拷贝，`Buffer#slice()`的实现是针对又有的`Buffer`创建一个视图而不是拷贝，这样效率也会高一点。

也可以通过如下方式来从`Buffer`创建`TypedArray`：

1. `Buffer`的对象内存拷贝到`TypedArray`中，而不是共享的方式
2. `Buffer`对象的内存作为独立的数组元素解析，而不是作为目标字节的字节数据。`new Uint32Array(Buffer.from([1,2,3,4]))`创建一个4元素的`Uint32Array`，元素分别为`[1,2,3,4]`，而不是包含但元素的`[0x1020304]`或者`[0x4030201]`的`Uint32Array`。

可以通过`TypedArray`对象的`.buffer`属性来创建和`TypedArray`实例共享内存的`Buffer`实例。

示例：

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Copies the contents of `arr`
const buf1 = Buffer.from(arr);

// Shares memory with `arr`
const buf2 = Buffer.from(arr.buffer);

// Prints: <Buffer 88 a0>
console.log(buf1);

// Prints: <Buffer 88 13 a0 0f>
console.log(buf2);

arr[1] = 6000;

// Prints: <Buffer 88 a0>
console.log(buf1);

// Prints: <Buffer 88 13 70 17>
console.log(buf2);
```

注意当使用`TypedArray`的`.buffer`属性来创建`Buffer`时，可以通过传递参数`byteOffset`和`length`来使用`ArrayBuffer`的一部分数据。

例如：

```js
const arr = new Uint16Array(20);
const buf = Buffer.from(arr.buffer, 0, 16);

// Prints: 16
console.log(buf.length);
```

`Buffer.from()`和`TypedArray.from()`有着不同的函数签名和实现。特别是`TypedArray`的变体接受映射函数作为第二个参数，会在数组的所有元素上执行。

* `TypedArray.from(source[, mapFn[, thisArg]])`

然而`Buffer.from()`方法不支持使用映射函数：

* `Buffer.from(array)`
* `Buffer.from(buffer)`
* `Buffer.from(arrayBuffer[, byteOffset[, length]])`
* `Buffer.from(string[, encoding])`

## `Buffer`和ES6迭代

`Buffer`实例可以使用ES6的`for..of`语法迭代。

例如：

```js
const buf = Buffer.from([1, 2, 3]);

// Prints:
//   1
//   2
//   3
for (const b of buf) {
  console.log(b);
}
```

另外使用`buf.values()`、`buf.keys()`和`buf.entries()`方法可以创建迭代器。

## 类：`Buffer`

`Buffer`类为用于直接处理二进制数据的全局变量。可以使用多种方法来创建其实例。

### `new Buffer(array)`

> 稳定性：0，过时：使用`Buffer.from(array)`取代之。

* `array` `<integer[]>` 用于拷贝的字节数据

使用`array`的二进制数据分配新的`Buffer`实例。

例如：

```js
// Creates a new Buffer containing the UTF-8 bytes of the string 'buffer'
const buf = new Buffer([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);

```

### `new Buffer(arrayBuffer[, byteOffset[, length]])`

> 稳定性：0，过时：使用`Buffer.from(arrayBuffer[, byteOffset[, length]])`取代之。

* `arrayBuffer` `<ArrayBuffer>` `ArrayBuffer`实例或者`TypedArray`的`.buffer`属性。
* `byteOffset` `<integer>` 获取的字节偏移量。默认为0。
* `length` `<integer>` 获取的字节长度。默认为：arrayBuffer.length - byteOffset。

其创建`ArrayBuffer`的视图而不拷贝潜在的内存数据。例如，当传递一个`TypedArray`的`.buffer`属性的引用，新创建的`Buffer`和`TypedArray`将共享分配的内存。

可选参数`byteOffset`和`length`指定`Buffer`和`TypedArray`共享的内存范围。

例如：

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Shares memory with `arr`
const buf = new Buffer(arr.buffer);

// Prints: <Buffer 88 13 a0 0f>
console.log(buf);

// Changing the original Uint16Array changes the Buffer also
arr[1] = 6000;

// Prints: <Buffer 88 13 70 17>
console.log(buf);
```

### `new Buffer(buffer)`

> 稳定性：0，过时：使用`Buffer.from(buffer)`取代之。

* `buffer` `<Buffer>` 待拷贝的`Buffer`实例。

从传递的`buffer`中拷贝数据到新的`Buffer`实例中。

例如：

```js
const buf1 = new Buffer('buffer');
const buf2 = new Buffer(buf1);

buf1[0] = 0x61;

// Prints: auffer
console.log(buf1.toString());

// Prints: buffer
console.log(buf2.toString())
```

### `new Buffer(size)`

> 稳定性：0，过时：使用`Buffer.alloc()`取代之(参考`Buffer.allocUnsafe()`)。

* `size` `<integer>` 新Buffer实例的期望大小

使用创建一个`size`字节的`Buffer`实例。如果`size`大于`buffer.constants.max_length`或者小于0，会抛出`RangeError`异常。如果`size`为0，则会创建长度为0的`Buffer`对象。

在Node.JS8.0.0之前，使用这种方式创建的`Buffer`对象其潜在的内存不会被初始化。新创建的`Buffer`对象的内容是未知的，可能含有敏感数据。使用`Buffer.alloc(size)`来初始化`Buffer`中的数据为0。

例如：

```js

const buf = new Buffer(10);

// Prints: <Buffer 00 00 00 00 00 00 00 00 00 00>
console.log(buf);

```
### `new Buffer(string[, encoding])`

> 稳定性：0，过时：使用`Buffer.from(string[, encoding])`取代之。

* `string` `<string>` 要编码的字符串。
* `encoding` `<string>` `string`的编码方式。默认：`utf8`

创建一个包含给定Javascript字符串`string`的`Buffer`实例。如果提供，`encoding`参数则表示`string`的字符编码格式。

例如：

```js
const buf1 = new Buffer('this is a tést');

// Prints: this is a tést
console.log(buf1.toString());

// Prints: this is a tC)st
console.log(buf1.toString('ascii'));


const buf2 = new Buffer('7468697320697320612074c3a97374', 'hex');

// Prints: this is a tést
console.log(buf2.toString());
```

### 类方法：`Buffer.alloc(size[, fill[, encoding]])`

* `size` `<integer>` 新建的`Buffer`对象的期望尺寸。
* `fill` `<string>`|`<Buffer>`|`<integer>` 新建的`Buffer`对象预填内容。默认：0
* `encoding` `<string>` 如果`fill`为字符串，则这个参数表示对应的编码方式。默认：`utf8`

分配一个`size`字节的新`Buffer`对象。如果`fill`为`undefined`，`Buffer`会使用零填充。

例如：

```js
const buf = Buffer.alloc(5);

// Prints: <Buffer 00 00 00 00 00>
console.log(buf);
```

使用创建一个`size`字节的`Buffer`实例。如果`size`大于`buffer.constants.max_length`或者小于0，会抛出`RangeError`异常。如果`size`为0，则会创建长度为0的`Buffer`对象。

如果指定了`fill`参数，则分配的`Buffer`对象会使用`buf.fill(fill)`初始化。

例如：

```js
const buf = Buffer.alloc(5, 'a');

// Prints: <Buffer 61 61 61 61 61>
console.log(buf);
```

如果，同时指定了`fill`和`encoding`参数，则分配的`Buffer`会使用`buf.fill(fill, encoding)`来初始化。

例如：

```js
const buf = Buffer.alloc(11, 'aGVsbG8gd29ybGQ=', 'base64');

// Prints: <Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
console.log(buf);
```

执行`Buffer.alloc()`会比执行其类似的方法`Buffer.allocUnsafe()`慢很多，但可以确保新建的`Buffer`实例不包含敏感数据。

如果`size`不为数值类型，则抛出`TypedError`。

### 类方法：`Buffer.allocUnsafe(size)`

* `size` `<integer>` 新建的`Buffer`对象的期望长度。

分配一个`size`字节的新`Buffer`对象。如果`size`大于`buffer.constants.MAX_LENGTH`或者小于0，则抛出`RangeError`异常。如果`size`为0，则会创建一个长度为0的`Buffer`对象。

使用这种方式创建的`Buffer`实例的内存区域没有被初始化。新创建的`Buffer`内容是未知的且可能还有敏感数据。使用`Buffer.alloc()`作为替代方案则会初始化实例。

例如：

```js
const buf = Buffer.allocUnsafe(10);

// Prints: (contents may vary): <Buffer a0 8b 28 3f 01 00 00 00 50 32>
console.log(buf);

buf.fill(0);

// Prints: <Buffer 00 00 00 00 00 00 00 00 00 00>
console.log(buf);
```

如果`size`不为数值，则抛出`TypeError`。

注意`Buffer`模块会预先分配一个内部`Buffer`对象，其尺寸为`Buffer.poolSize`作为使用`Buffer.allocUnsafe()`和过时的`new Buffer(size)`创建的`Buffer`实例快速分配的内存池，且`size`必须小于等于`Buffer.poolSize >> 1`(`Buffer.poolSize`除以2的整数部分)。

使用这个预分配的内存池是`Buffer.alloc(size, fill)`和`Buffer.allocUnsafe(size).fill(fill)`之间的关键区别。`Buffer.alloc(size, fill)`不会使用内置的`Buffer`池，而`Buffer.allocUnsafe(size).fill(fill)`在`size`小于`Buffer.poolSize`时会使用内置的`Buffer`内存池。当应用程序需要`Buffer.allocUnsafe()`所具备更好的性能时，这个微秒的差异就会很重要。

### 类方法：`Buffer.allocUnsafeSlow(size)`

* `size` `<integer>` 返回期望尺寸的`Buffer`实例

分配一个`size`字节的新`Buffer`对象。如果`size`大于`buffer.constants.MAX_LENGTH`或者小于0，则抛出`RangeError`异常。如果`size`为0，则会创建一个长度为0的`Buffer`对象。

使用这种方式创建的`Buffer`实例的内存区域没有被初始化。新创建的`Buffer`内容是未知的且可能还有敏感数据。使用`buf.fill(0)`可以初始化`Buffer`实例。

当使用`Buffer.allocUnsafe()`来分配常见新的`Buffer`实例时，分配大小小于4KB，默认会从一个预分配的`Buffer`中截取。这可以让程序在创建了大量的单个`Buffer`实例时避免垃圾回收。这样减少追踪和清理持久化对象，可以提升性能和内存使用。

然而，如果要避免不时的需要从内存池中保留维持使用一小块内存，使用`Buffer.allocUnsafeSlow()`方法可以避免在内存池中分配空间，会更加合适一点。

```js
// Need to keep around a few small chunks of memory
const store = [];

socket.on('readable', () => {
  const data = socket.read();

  // Allocate for retained data
  const sb = Buffer.allocUnsafeSlow(10);

  // Copy the data into the new allocation
  data.copy(sb, 0, 0, 10);

  store.push(sb);
});
```

`Buffer.allocUnsafeSlow()`在开发这发现其程序中出现意外的内存保留的情况下，可以作为最后的解决方案。

如果`size`不为数值，则抛出`TypeError`。

### 类方法：`Buffer.byteLength(string[, encoding])`

* string `<string>`|`<Buffer>`|`<TypedArray>`|`<DataView>`|`<ArrayBuffer>`
* encoding `<string>` 如果`string`为字符串，则其为编码方式。默认：`utf8`
* 返回：`<integer>` `string`中包含的字节数

返回字符串中实际的字节长度。这和`String.prototype.length`返回字符的个数并不同。

注意：对于`base64`和`hex`，这个函数假设输入是有效的。对于包含非Base64和十六进制编码的字符，返回的值可能大于从`string`创建的`Buffer`大小。

例如：

```js
const str = '\u00bd + \u00bc = \u00be';

// Prints: ½ + ¼ = ¾: 9 characters, 12 bytes
console.log(`${str}: ${str.length} characters, ` +
            `${Buffer.byteLength(str, 'utf8')} bytes`);
```

当`string`为`Buffer`/`DataView`/`TypedArray`/`ArrayBuffer`时，返回实际的字节长度。

### 类方法：`Buffer.compare(buf1, buf2)`

* `buf1` `<Buffer>`|`Uint8Array`
* `buf2` `<Buffer>`|`Unit8Array`
* 返回：`<integer>`

对比`buf1`和`buf2`，主要用于对`Buffer`实例的数组排序。和`buf1.compare(buf2)`等同。

例如：

```js
const buf1 = Buffer.from('1234');
const buf2 = Buffer.from('0123');
const arr = [buf1, buf2];

// Prints: [ <Buffer 30 31 32 33>, <Buffer 31 32 33 34> ]
// (This result is equal to: [buf2, buf1])
console.log(arr.sort(Buffer.compare));
```

### 类方法：`Buffer.concat(list[, totalLength])`

* `list` `<Array>` 待合并的`Buffer`或者`Unit8Array`列表。
* `totalLength` `<integer>` 返回`list`中所有的`Buffer`实例尺寸的和。
* 返回：`<Buffer>`

返回一个新的`Buffer`实例，为所有`list`中的`Buffer`实例的合并。

如果，`list`为空，或者`totalLength`为0，则返回一个零长度的`Buffer`。

如果未给出`totalLength`，则从`list`中的`Buffer`实例中计算。然而，这会导致需要额外的循环逻辑来计算`totalLength`，所以如果已知这个值则提供这个参数会加快速度。

如果给出了`totalLength`，会将其强制转换为无符号整型。如果`list`中的`Buffer`尺寸合并后超过`totalLength`，会将结果剪裁至`totalLength`这个尺寸。

例如：从三个`Buffer`实例的列表创建一个`Buffer`实例

```js
const buf1 = Buffer.alloc(10);
const buf2 = Buffer.alloc(14);
const buf3 = Buffer.alloc(18);
const totalLength = buf1.length + buf2.length + buf3.length;

// Prints: 42
console.log(totalLength);

const bufA = Buffer.concat([buf1, buf2, buf3], totalLength);

// Prints: <Buffer 00 00 00 00 ...>
console.log(bufA);

// Prints: 42
console.log(bufA.length)
```

### 类方法：`Buffer.from(array)`

* `array` `<Array>`

使用给定数组的二进制数据创建一个新的`Buffer`实例。

例如：

```js
// Creates a new Buffer containing UTF-8 bytes of the string 'buffer'
const buf = Buffer.from([0x62, 0x75, 0x66, 0x66, 0x65, 0x72]);
```

如果`array`不为数组，则抛出`TypeError`异常。

### 类方法：`Buffer.from(arrayBuffer[, byteOffset[, length]])`

* `arrayBuffer` `<ArrayBuffer>` `TypedArray`的`.buffer`属性或者`ArrayBuffer`对象。
* `byteOffset` `<integer>` 截取的第一个字节。默认为：0
* `length` `<integer>` 截取的字节长度。默认：`arrayBuffer.length - byteOffset`

针对`ArrayBuffer`创建一个新的视图，而无效拷贝其底层的内存数据。例如，当传递`TypedArray`的`.buffer`属性的引用时，新创建的`Buffer`会和`TypedArray`共享相同的内存空间。

例如：

```js
const arr = new Uint16Array(2);

arr[0] = 5000;
arr[1] = 4000;

// Shares memory with `arr`
const buf = Buffer.from(arr.buffer);

// Prints: <Buffer 88 13 a0 0f>
console.log(buf);

// Changing the original Uint16Array changes the Buffer also
arr[1] = 6000;

// Prints: <Buffer 88 13 70 17>
console.log(buf);
```

可选参数`byteOffset`和`length`指定了在`arrayBuffer`中和`Buffer`共享的内存空间的范围。

例如：

```js
const ab = new ArrayBuffer(10);
const buf = Buffer.from(ab, 0, 2);

// Prints: 2
console.log(buf.length);
```

如果`arrayBuffer`不是`ArrayBuffer`类型则抛出`TypeError`异常。

### 类方法：`Buffer.from(buffer)`

* `buffer` `<Buffer>` 待拷贝的`Buffer`对象。

将给定的`buffer`对象的数据拷贝到新的`Buffer`实例中。

例如：

```js
const buf1 = Buffer.from('buffer');
const buf2 = Buffer.from(buf1);

buf1[0] = 0x61;

// Prints: auffer
console.log(buf1.toString());

// Prints: buffer
console.log(buf2.toString());
```

如果`buffer`类型不为`Buffer`则抛出`TypeError`异常。

### 类方法：`Buffer.from(string[, encoding])`

* `string` `<string>` 要编码的字符串。
* `encoding` `<string>` `string`的编码方式。默认：`utf8`

创建一个包含指定JavaScript字符串`string`的`Buffer`。如果提供，则`encoding`蚕食表明`string`的编码格式。

例如：

```js
const buf1 = Buffer.from('this is a tést');

// Prints: this is a tést
console.log(buf1.toString());

// Prints: this is a tC)st
console.log(buf1.toString('ascii'));


const buf2 = Buffer.from('7468697320697320612074c3a97374', 'hex');

// Prints: this is a tést
console.log(buf2.toString());
```

如果`string`不为字符串类型，则抛出`TypeError`。

### 类方法：`Buffer.from(object[, offsetOrEncoding[, length]])`

* `object` `<Object>` 支持`Symbol.toPrimitive`或者`valueOf()`的对象。
* `offsetOrEncoding` `<number>`|`<string>` 字节偏移或者编码方式，依赖于`object.valueOf()`或者`object[Symbol.toPrimitive]`返回的值。
* `length` `<number>` 长度，依赖于`object.valueOf()`或者`object[Symbol.toPrimitive]`返回的值。

对于`valueOf()`返回的值不严格等于`object`的，返回`Buffer.from（object.valueOf(), offsetOrEncoding, length)`。

例如：

```js
const buf = Buffer.from(new String('this is a test'));
// <Buffer 74 68 69 73 20 69 73 20 61 20 74 65 73 74>
```

对于支持`Symbol.toPrimitive`的对象，返回`Buffer.from(object[Symbol.toPrimitive](), offsetOrEncoding, length)`。

例如：

```js
class Foo {
  [Symbol.toPrimitive]() {
    return 'this is a test';
  }
}

const buf = Buffer.from(new Foo(), 'utf8');
```

### 类方法：`Buffer.isBuffer(obj)`

* `obj` `<Object>`
* 返回：`<boolean>`

如果`object`为`Buffer`，返回`true`，否则返回`false`。

### 类方法：`Buffer.isEncoding(encoding)`

* `encoding` `<string>` 待检验的字符编码
* 返回：`<boolean>`

如果`encoding`包含支持的编码方式，则返回`true`，否则返回`false`。

### 类属性：`Buffer.poolSize`

* `<integer>` 默认：8192

定义预分配的内存的字节大小，用于提供内存池。可以更改这个值。

### buf[index]

索引查询操作`[index]`可以用于获取和设置`buf`中位置为`index`的二进制值。这个值指向不同的字节，所以合法的值范围为`0x00`到`0xFF`之间或者`0`到`255`之间`。

这个操作从`Uint8Array`中继承而来，所以其超出边界的行为和`Uint8Array`一样，也就是获取动作返回`undefined`以及赋值动作不做任何事情。

例如：将`ASCII`字符拷贝进`Buffer`，一次拷贝一个字节：

```js
const str = 'Node.js';
const buf = Buffer.allocUnsafe(str.length);

for (let i = 0; i < str.length; i++) {
  buf[i] = str.charCodeAt(i);
}

// Prints: Node.js
console.log(buf.toString('ascii'));
```

### buf.buffer

指向当前`Buffer`对象创建时提供的`ArrayBuffer`实例。

```js
const arrayBuffer = new ArrayBuffer(16);
const buffer = Buffer.from(arrayBuffer);

console.log(buffer.buffer === arrayBuffer);
// Prints: true
```

### buf.compare(target[, targetStart[, targetEnd[, sourceStart[, sourceEnd]]]])

* `target` `<Buffer>`|`<Uint8Array>` 待比较的`Buffer`或者`Uint8Array`对象。
* `targetSource` `<integer>` `target`中要比较的部分的偏移。默认：0。
* `targetEnd` `<integer>` `target`要比较的部分的偏移结尾。当`targetStart`为`undefined`时则忽略这个值。默认：`target.length`。
* `sourceTarget` `<integer>` `buf`中要比较的部分的偏移。当`targetStart`为`undefined`时忽略这个参数。默认：0。
* `sourceEnd` `<integer>` `buf`中要比较的部分的偏移结尾。当`targetStart`为`undefined`时忽略这个参数。默认：`buf.length`
* 返回：`<integet>`

比较`buf`和`target`，返回一个表示`buf`时否则在排序上位于前、后或者相等位置的数。比较方法时基于`Buffer`中实际的字节序列。

* 0如果`target`和`buf`在一个位置。
* 1如果`target`应该在`buf`之前
* -1如果`target`应该在`buf`之后

例如：

```js
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('BCD');
const buf3 = Buffer.from('ABCD');

// Prints: 0
console.log(buf1.compare(buf1));

// Prints: -1
console.log(buf1.compare(buf2));

// Prints: -1
console.log(buf1.compare(buf3));

// Prints: 1
console.log(buf2.compare(buf1));

// Prints: 1
console.log(buf2.compare(buf3));

// Prints: [ <Buffer 41 42 43>, <Buffer 41 42 43 44>, <Buffer 42 43 44> ]
// (This result is equal to: [buf1, buf3, buf2])
console.log([buf1, buf2, buf3].sort(Buffer.compare));
```

可选参数`targetStart`、`targetEnd`、`sourceStart`、`sourceEnd`参数可以用于将`target`和`buf`之间的比较限定在一个特定的字节范围内。

例如:

```js
const buf1 = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8, 9]);
const buf2 = Buffer.from([5, 6, 7, 8, 9, 1, 2, 3, 4]);

// Prints: 0
console.log(buf1.compare(buf2, 5, 9, 0, 4));

// Prints: -1
console.log(buf1.compare(buf2, 0, 6, 4));

// Prints: 1
console.log(buf1.compare(buf2, 5, 6, 5));
```

如果：`targetStart<0`、`sourceStart<0`、`targetEnd > target.byteLength`或者`sourceEnd>source.byteLength`，则抛出`RangeError`异常。

### buf.copy(target[, targetStart[, sourceStart[, sourceEnd]]])

* `target` `<Buffer>`|`<Uint8Array>` 一个用于拷贝的`Buffer`或者`Uint8Array`对象。
* `targetStart` `<integer>` `target`实例拷贝的起始偏移。默认：0
* `sourceStart` `<integer>` `buf`拷贝的起始偏移。如果`targetStart`为`undefined`则忽略这个参数。默认：0
* `sourceEnd` `<integer>` `buf`拷贝的结束偏移量（不包含该位置的字节）。如果`sourceStart`为`undefined`则忽略这个参数。默认：`buf.length`
* 返回：`<integer>` 拷贝成功的字节个数

将`buf`的一个部分拷贝到`target`中，即使`taget`和`buf`在内存中有重叠也仍然执行。

例如：创建两个`Buffer`实例，`buf1`和`buf2`，将`buf1`从16字节开始的19个字节拷贝给`buf2`，其中`buf2`从第8个字节开始。

```js
const buf1 = Buffer.allocUnsafe(26);
const buf2 = Buffer.allocUnsafe(26).fill('!');

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'
  buf1[i] = i + 97;
}

buf1.copy(buf2, 8, 16, 20);

// Prints: !!!!!!!!qrst!!!!!!!!!!!!!
console.log(buf2.toString('ascii', 0, 25));
```

例如：创建一个`Buffer`，将数据从一个区域拷贝到同一个`Buffer`的重叠区域。

```js
const buf = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'
  buf[i] = i + 97;
}

buf.copy(buf, 0, 4, 10);

// Prints: efghijghijklmnopqrstuvwxyz
console.log(buf.toString());
```

### buf.entries()

* 返回：`<Iterator>`

从`buf`的内容中创建和返回一个`[index, byte]`形式的迭代器。

例如：记录`Buffer`的全部内容

```js
const buf = Buffer.from('buffer');

// Prints:
//   [0, 98]
//   [1, 117]
//   [2, 102]
//   [3, 102]
//   [4, 101]
//   [5, 114]
for (const pair of buf.entries()) {
  console.log(pair);
}
```

### buf.equals(otherBuffer)

* `otherBuffer` `<Buffer>` 待比较的`Buffer`或者`Uint8Array`对象。
* 返回：`<boolean>`

如果`buf`和`otherBuffer`包含相同的字节，则返回true，否则返回`false`。

示例：

```js
const buf1 = Buffer.from('ABC');
const buf2 = Buffer.from('414243', 'hex');
const buf3 = Buffer.from('ABCD');

// Prints: true
console.log(buf1.equals(buf2));

// Prints: false
console.log(buf1.equals(buf3));
```

### buf.fill(value[, offset[, end[, encoding]]])

* `value` `<string>`|`<Buffer>`|`<integer>` 待填入`buf`的值。
* `offset` `<integer>` 开始填充前跳过的字节个数。默认：0
* `end` `<integer>` 停止填充`buf`的位置。默认：buf.length
* `encoding` `<string>` 如果`value`为字符串，则为其编码方式。默认：`utf8`
* 返回：`<Buffer>` 执向`buf`的引用。

使用给定的值`value`来填充`buf`。如果`offset`和`end`未给出，则会填充整个`buf`。这可以作为在一行中完成`Buffer`的创建和填充的简单途径。

例如：使用字符`h`填充一个`Buffer`：

```js
const b = Buffer.allocUnsafe(50).fill('h');

// Prints: hhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhhh
console.log(b.toString());
```

`value`如果不是字符传和整数，则会被强制转换为`uint32`。

如果`fill()`的最终写入操作针对的是多字节的字符，则只有该字符的第一个字节会被写入到`buf`中。

例如：使用两个自己的字符填充`Buffer`。

```js
// Prints: <Buffer c8 a2 c8>
console.log(Buffer.allocUnsafe(3).fill('\u0222'));
```

### buf.includes(value[, byteOffset[, encoding]])

* `value` `<string>`|`<Buffer>`|`<integer>` 检索的对象
* `byteOffset` `<integer>` `buf`中开始检索的位置。默认：0
* `encoding` `<string>` 如果`value`为字符串，则这为编码方式。默认：`utf8`
* 返回：`<boolean>` 如果`buf`中检索到`value`则返回`true`，否则返回`false`

和`buf.indexOf() !== -1`等同。

例如：

```js
const buf = Buffer.from('this is a buffer');

// Prints: true
console.log(buf.includes('this'));

// Prints: true
console.log(buf.includes('is'));

// Prints: true
console.log(buf.includes(Buffer.from('a buffer')));

// Prints: true
// (97 is the decimal ASCII value for 'a')
console.log(buf.includes(97));

// Prints: false
console.log(buf.includes(Buffer.from('a buffer example')));

// Prints: true
console.log(buf.includes(Buffer.from('a buffer example').slice(0, 8)));

// Prints: false
console.log(buf.includes('this', 4));
```

### buf.indexOf(value[, byteOffset[, encoding]])

* `value` `<string>`|`<Buffer>`|`<Uint8Array>`|`<integer>` 检索的内容
* `byteOffset` `<integer>` `buf`中检索的起始位置。默认：0
* `encoding` `<string>` 如果`value`为字符串，则这为其编码方式。默认：`utf8`
* 返回：`<integer>` `buf`中第一次出现`value`的索引值，或者如果`buf`不含有`value`返回-1

如果`value`为：

* 字符串，`value`会依据`encoding`解析成相应的字符。
* `Buffer`或者`Uint8Array`，`value`会作为整体使用。要比较`Buffer`的一部分，可以使用`buf.slice()`。
* 数值，`value`会被解析成无符号8比特整数，范围为0到255。

例如：

```js
const buf = Buffer.from('this is a buffer');

// Prints: 0
console.log(buf.indexOf('this'));

// Prints: 2
console.log(buf.indexOf('is'));

// Prints: 8
console.log(buf.indexOf(Buffer.from('a buffer')));

// Prints: 8
// (97 is the decimal ASCII value for 'a')
console.log(buf.indexOf(97));

// Prints: -1
console.log(buf.indexOf(Buffer.from('a buffer example')));

// Prints: 8
console.log(buf.indexOf(Buffer.from('a buffer example').slice(0, 8)));


const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'ucs2');

// Prints: 4
console.log(utf16Buffer.indexOf('\u03a3', 0, 'ucs2'));

// Prints: 6
console.log(utf16Buffer.indexOf('\u03a3', -4, 'ucs2'));
```

如果`value`不为数值、字符串或者`Buffer`对象，则抛出`TypeError`。如果`value`为数值，会将其转换为有效的字节数值，介于0到255的整数。

如果`byteOffset`不为数值类型，则会将其强制转换为数值。任何转换成`NaN`或者0的参数，如`{}`、`[]`、`null`、`undefined`，将会检索整个`Buffer`对象。这个行为和`string.indexOf()`类似。

```js
const b = Buffer.from('abcdef');

// Passing a value that's a number, but not a valid byte
// Prints: 2, equivalent to searching for 99 or 'c'
console.log(b.indexOf(99.9));
console.log(b.indexOf(256 + 99));

// Passing a byteOffset that coerces to NaN or 0
// Prints: 1, searching the whole buffer
console.log(b.indexOf('b', undefined));
console.log(b.indexOf('b', {}));
console.log(b.indexOf('b', null));
console.log(b.indexOf('b', []));
```

如果`value`为空字符传或者空`Buffer`对象且`byteOffset`小于`buf.length`，则返回`byteOffset`。如果`value`为空且`byteOffset`大于等于`buf.length`，则返回`buf.length`。

### buf.keys()

* 返回：`<Iterator>`

创建并返回由`buf`的键值构成的迭代器对象。

例如：

```js
const buf = Buffer.from('buffer');

// Prints:
//   0
//   1
//   2
//   3
//   4
//   5
for (const key of buf.keys()) {
  console.log(key);
}
```

### buf.lastIndexOf(value[, byteOffset[, encoding]])

* `value` `<string>`|`<Buffer>`|`<Uint8Array>`|`<integer>` 要检索的对象。
* `byteOffset` `<integer>` `buf`检索的起始位置。默认：`buf.length - 1`
* `encoding` `<string>` 如果`encoding`为字符串，则为对于的字符编码方式。默认：`utf8`
* 返回：`<integer>` `buf`中最后出现`value`的对应的锁应，如果`buf`不包含`value`则返回-1

除了从后向前检索之外，其他的等同于`buf.indexOf()`。

例如：

```js
const buf = Buffer.from('this buffer is a buffer');

// Prints: 0
console.log(buf.lastIndexOf('this'));

// Prints: 17
console.log(buf.lastIndexOf('buffer'));

// Prints: 17
console.log(buf.lastIndexOf(Buffer.from('buffer')));

// Prints: 15
// (97 is the decimal ASCII value for 'a')
console.log(buf.lastIndexOf(97));

// Prints: -1
console.log(buf.lastIndexOf(Buffer.from('yolo')));

// Prints: 5
console.log(buf.lastIndexOf('buffer', 5));

// Prints: -1
console.log(buf.lastIndexOf('buffer', 4));


const utf16Buffer = Buffer.from('\u039a\u0391\u03a3\u03a3\u0395', 'ucs2');

// Prints: 6
console.log(utf16Buffer.lastIndexOf('\u03a3', undefined, 'ucs2'));

// Prints: 4
console.log(utf16Buffer.lastIndexOf('\u03a3', -5, 'ucs2'));
```

如果`value`不为数值、字符串或者`Buffer`对象，则抛出`TypeError`。如果`value`为数值，会将其转换为有效的字节数值，介于0到255的整数。

如果`byteOffset`不为数值类型，则会将其强制转换为数值。任何转换成`NaN`或者0的参数，如`{}`、`undefined`，将会检索整个`Buffer`对象。这个行为和`string.lastIndexOf()`类似。

```js
const b = Buffer.from('abcdef');

// Passing a value that's a number, but not a valid byte
// Prints: 2, equivalent to searching for 99 or 'c'
console.log(b.lastIndexOf(99.9));
console.log(b.lastIndexOf(256 + 99));

// Passing a byteOffset that coerces to NaN
// Prints: 1, searching the whole buffer
console.log(b.lastIndexOf('b', undefined));
console.log(b.lastIndexOf('b', {}));

// Passing a byteOffset that coerces to 0
// Prints: -1, equivalent to passing 0
console.log(b.lastIndexOf('b', null));
console.log(b.lastIndexOf('b', []));
```

如果`value`为空字符串或者空`Buffer`对象，则返回`byteOffset`。

### buf.length

* `<integer>`

返回`buf`分配的内存字节数。注意这个并不表示`buf`中实际的数据量。

例如：创建`Buffer`以及写入短ASCII字符。

```js
const buf = Buffer.alloc(1234);

// Prints: 1234
console.log(buf.length);

buf.write('some string', 0, 'ascii');

// Prints: 1234
console.log(buf.length);
```

但是`lenght`属性不可更改，改变`length`的值可以导致意外的行为。应用程序中应该将`length`作为只读属性并使用`buf.slice()`创建新的`Buffer`实例。

例如：

```js
let buf = Buffer.allocUnsafe(10);

buf.write('abcdefghj', 0, 'ascii');

// Prints: 10
console.log(buf.length);

buf = buf.slice(0, 5);

// Prints: 5
console.log(buf.length);
```

### buf.parent

> 稳定性：0，过时：使用`buf.buffer`取代之。

过时属性，作为`buf.buffer`的别名。

### buf.readDoubleBE(offset[, noAssert])
### buf.readDoubleLE(offset[, noAssert])

* `offset` `<integer>` 在开始读取时跳过的字节个数。必须满足：`0 <= offset <= buf.length - 8`。
* `noAssert` `<boolean>` 是否跳过`offset`的校验。默认：`false`
* 返回：`<number>`

从`buf`中指定的`offset`偏移上使用指定的字节顺序（readDoubleBE()返回big-endian，readDoubeLE()返回'little-endian'）读取64比特。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

例如：

```js
const buf = Buffer.from([1, 2, 3, 4, 5, 6, 7, 8]);

// Prints: 8.20788039913184e-304
console.log(buf.readDoubleBE());

// Prints: 5.447603722011605e-270
console.log(buf.readDoubleLE());

// Throws an exception: RangeError: Index out of range
console.log(buf.readDoubleLE(1));

// Warning: reads passed end of buffer!
// This will result in a segmentation fault! Don't do this!
console.log(buf.readDoubleLE(1, true));
```

### buf.readFloatBE(offset[, noAssert])
### buf.readFloatLE(offset[, noAssert])

* `offset` `<integer>` 在开始读取时跳过的字节个数。必须满足：`0 <= offset <= buf.length - 4`。
* `noAssert` `<boolean>` 是否跳过`offset`的校验。默认：`false`
* 返回：`<number>`

从`buf`中指定的`offset`偏移上使用指定的字节顺序（readFloatBE()返回big-endian，readFloatLE()返回'little-endian'）读取32比特。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

例如：

```js
const buf = Buffer.from([1, 2, 3, 4]);

// Prints: 2.387939260590663e-38
console.log(buf.readFloatBE());

// Prints: 1.539989614439558e-36
console.log(buf.readFloatLE());

// Throws an exception: RangeError: Index out of range
console.log(buf.readFloatLE(1));

// Warning: reads passed end of buffer!
// This will result in a segmentation fault! Don't do this!
console.log(buf.readFloatLE(1, true));
```

### buf.readInt8(offset[, noAssert])

* `offset` `<integer>` 在开始读取时跳过的字节个数。必须满足：`0 <= offset <= buf.length - 1`。
* `noAssert` `<boolean>` 是否跳过`offset`的校验。默认：`false`
* 返回：`<integer>`

在`buf`中指定的`offset`位置读取8比特的整数。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

从`Buffer`中读取的整数解析成二进制补码。

例如：

```js
const buf = Buffer.from([-1, 5]);

// Prints: -1
console.log(buf.readInt8(0));

// Prints: 5
console.log(buf.readInt8(1));

// Throws an exception: RangeError: Index out of range
console.log(buf.readInt8(2));
```

### buf.readInt16BE(offset[, noAssert])
### buf.readInt16LE(offset[, noAssert])

* `offset` `<integer>` 在开始读取时跳过的字节个数。必须满足：`0 <= offset <= buf.length - 4`。
* `noAssert` `<boolean>` 是否跳过`offset`的校验。默认：`false`
* 返回：`<integer>`

从`buf`中的特定`offset`的位置以指定的字符序列读取32比特整数（`readInt32BE()`返回'big-endian'，`readInt32LE()`返回'little-endian'）。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

从`Buffer`中读取的整数解析成二进制补码。

例如：

```js
const buf = Buffer.from([0, 0, 0, 5]);

// Prints: 5
console.log(buf.readInt32BE());

// Prints: 83886080
console.log(buf.readInt32LE());

// Throws an exception: RangeError: Index out of range
console.log(buf.readInt32LE(1));
```

### buf.readIntBE(offset, byteLength[, noAssert])
### buf.readIntLE(offset, byteLength[, noAssert])

* `offset` `<integer>` 在开始读取时跳过的字节个数。必须满足：`0 <= offset <= buf.length - byteLength`
* `byteLength` `<integer>` 要读取的字节树。必须满足：`0 < byteLength <= 6`
* `noAssert` `<boolean>` 是否跳过`offset`和`byteLength`的校验。默认：`false`
* 返回：`<integer>`

从`buf`中的特定`offset`的位置以指定的字符序列读取`byteLength`字节（`readInt32BE()`返回'big-endian'，`readInt32LE()`返回'little-endian'）。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

从`Buffer`中读取的整数解析成二进制补码。

例如：

```js
const buf = Buffer.from([0, 0, 0, 5]);

// Prints: 5
console.log(buf.readInt32BE());

// Prints: 83886080
console.log(buf.readInt32LE());

// Throws an exception: RangeError: Index out of range
console.log(buf.readInt32LE(1));
```

### buf.readIntBE(offset, byteLength[, noAssert])
### buf.readIntLE(offset, byteLength[, noAssert])

* `offset` `<integer>` 开始读取前跳过的字节数。必须满足`0 <= offset <= buf.length - byteLength`。
* `byteLength` `<integer>` 读取的字节个数。必须满足`0 < byteLength < 6`
* `noAssert` `<boolean>` 是否跳过`offset`和`byteLength`的校验。默认：`false`
* 返回：`<integer>`

从`buf`中的特定`offset`的位置读取`byteLength`字节并将结果解析成二进制补码的形式。支持48比特精度。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

例如：

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

// Prints: -546f87a9cbee
console.log(buf.readIntLE(0, 6).toString(16));

// Prints: 1234567890ab
console.log(buf.readIntBE(0, 6).toString(16));

// Throws an exception: RangeError: Index out of range
console.log(buf.readIntBE(1, 6).toString(16));
```

### buf.readUInt8(offset[, noAssert])

* `offset` `<integer>` 开始读取前跳过的字节数。必须满足`0 <= offset <= buf.length - 1`。
* `noAssert` `<boolean>` 是否跳过`offset`校验。默认：`false`
* 返回：`<integer>`

从`buf`中的特定`offset`位置读取8比特整数。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

例如：

```js
const buf = Buffer.from([1, -2]);

// Prints: 1
console.log(buf.readUInt8(0));

// Prints: 254
console.log(buf.readUInt8(1));

// Throws an exception: RangeError: Index out of range
console.log(buf.readUInt8(2));
```

### buf.readUInt16BE(offset[, noAssert])
### buf.readUInt16LE(offset[, noAssert])

* `offset` `<integer>` 开始读取前跳过的字节数。必须满足`0 <= offset <= buf.length - 2`。
* `noAssert` `<boolean>` 是否跳过`offset`校验。默认：`false`
* 返回：`<integer>`

从`buf`中的特定`offset`的位置以指定的字符序列读取无符号16比特整数（`readInt32BE()`返回'big-endian'，`readInt32LE()`返回'little-endian'）。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

例如：

```js
const buf = Buffer.from([0x12, 0x34, 0x56]);

// Prints: 1234
console.log(buf.readUInt16BE(0).toString(16));

// Prints: 3412
console.log(buf.readUInt16LE(0).toString(16));

// Prints: 3456
console.log(buf.readUInt16BE(1).toString(16));

// Prints: 5634
console.log(buf.readUInt16LE(1).toString(16));

// Throws an exception: RangeError: Index out of range
console.log(buf.readUInt16LE(2).toString(16));
```

### buf.readUInt32BE(offset[, noAssert])
### buf.readUInt32LE(offset[, noAssert])

* `offset` `<integer>` 开始读取前跳过的字节数。必须满足`0 <= offset <= buf.length - 4`。
* `noAssert` `<boolean>` 是否跳过`offset`校验。默认：`false`
* 返回：`<integer>`

从`buf`中特定那个的`offset`位置以特定的字节序列读取一个无符号的32字节整数（`readUInt32BE()`返回`big-endian`，`readUInt32LE()`返回`little-endian`）。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

例如：

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78]);

// Prints: 12345678
console.log(buf.readUInt32BE(0).toString(16));

// Prints: 78563412
console.log(buf.readUInt32LE(0).toString(16));

// Throws an exception: RangeError: Index out of range
console.log(buf.readUInt32LE(1).toString(16));
```

### buf.readUIntBE(offset, byteLength[, noAssert])
### buf.readUIntLE(offset, byteLength[, noAssert])

* `offset` `<integer>` 开始读取前跳过的字节数。必须满足`0 <= offset <= buf.length - byteLength`。
* `byteLength` `<integer>` 读取的字节个数。必须满足`0 < byteLength < 6`
* `noAssert` `<boolean>` 是否跳过`offset`校验。默认：`false`
* 返回：`<integer>`

从`buf`中指定的`offset`位置读取`byteLength`个字节，并将其解释成无符号整型。支持48比特精度。

设置`noAssert`为`true`则允许`offset`在`buf`尾部之后，但是结果可以认为是未定义的。

例如：

```js
const buf = Buffer.from([0x12, 0x34, 0x56, 0x78, 0x90, 0xab]);

// Prints: 1234567890ab
console.log(buf.readUIntBE(0, 6).toString(16));

// Prints: ab9078563412
console.log(buf.readUIntLE(0, 6).toString(16));

// Throws an exception: RangeError: Index out of range
console.log(buf.readUIntBE(1, 6).toString(16));
```

### buf.slice([start[, end]])

* `start` `<integer>` 返回新`Buffer`实例截取的开始位置。默认：0。
* `end` `<integer>` 返回新`Buffer`实例截取的结束位置（不包含该位置）。默认：`buf.length`
* 返回：`<Buffer>`

返回和`buf`中共享相同内存的新`Buffer`实例，`start`和`end`表示字节偏移和剪裁位置。

如果指定的`end`大于`buf.length`则会返回`end`等于`buf.length`一样的结果。

注意：更改新的截取`Buffer`对象会更改原始的`Buffer`对象内存，因为两个对象的分配内存存在重叠。

例如：使用ASCII字符创建一个`Buffer`对象，进行截取，然后在原始的`Buffer`对象上更改一个字节。

```js
const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'
  buf1[i] = i + 97;
}

const buf2 = buf1.slice(0, 3);

// Prints: abc
console.log(buf2.toString('ascii', 0, buf2.length));

buf1[0] = 33;

// Prints: !bc
console.log(buf2.toString('ascii', 0, buf2.length));
```

指定负数的参数则会相对于`buf`的尾端来裁剪而不是头部。

例如：

```js
const buf = Buffer.from('buffer');

// Prints: buffe
// (Equivalent to buf.slice(0, 5))
console.log(buf.slice(-6, -1).toString());

// Prints: buff
// (Equivalent to buf.slice(0, 4))
console.log(buf.slice(-6, -2).toString());

// Prints: uff
// (Equivalent to buf.slice(1, 4))
console.log(buf.slice(-5, -2).toString());
```

### buf.swap16()

* 返回：`<Buffer>` `buf`的引用

将`buf`作为无符号16比特整型解析，并置换字节顺序。如果`buf.length`不为2的整数，则抛出`RangeError`。

例如：

```js
const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

// Prints: <Buffer 01 02 03 04 05 06 07 08>
console.log(buf1);

buf1.swap16();

// Prints: <Buffer 02 01 04 03 06 05 08 07>
console.log(buf1);


const buf2 = Buffer.from([0x1, 0x2, 0x3]);

// Throws an exception: RangeError: Buffer size must be a multiple of 16-bits
buf2.swap16();
```

### buf.swap32()

* 返回：`<Buffer>` `buf`的引用

将`buf`作为无符号32比特整型解析，并置换字节顺序。如果`buf.length`不为4的整数，则抛出`RangeError`。

例如：

```js
const buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

// Prints: <Buffer 01 02 03 04 05 06 07 08>
console.log(buf1);

buf1.swap32();

// Prints: <Buffer 04 03 02 01 08 07 06 05>
console.log(buf1);


const buf2 = Buffer.from([0x1, 0x2, 0x3]);

// Throws an exception: RangeError: Buffer size must be a multiple of 32-bits
buf2.swap32();
```

### buf.swap64()

* 返回：`<Buffer>` `buf`的引用

将`buf`作为无符号64比特整型解析，并置换字节顺序。如果`buf.length`不为8的整数，则抛出`RangeError`。

例如：

```js
onst buf1 = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5, 0x6, 0x7, 0x8]);

// Prints: <Buffer 01 02 03 04 05 06 07 08>
console.log(buf1);

buf1.swap64();

// Prints: <Buffer 08 07 06 05 04 03 02 01>
console.log(buf1);


const buf2 = Buffer.from([0x1, 0x2, 0x3]);

// Throws an exception: RangeError: Buffer size must be a multiple of 64-bits
buf2.swap64();
```

注意JavaScript无法编码64比特整数。该方法用于64比特的浮点型。

### buf.toJSON()

* 返回：`<Object>`

返回`buf`的JSON形式。`JSON.stringify()`在`Buffer`实例中调用会潜在的执行这个方法。

例如：

```js
const buf = Buffer.from([0x1, 0x2, 0x3, 0x4, 0x5]);
const json = JSON.stringify(buf);

// Prints: {"type":"Buffer","data":[1,2,3,4,5]}
console.log(json);

const copy = JSON.parse(json, (key, value) => {
  return value && value.type === 'Buffer' ?
    Buffer.from(value.data) :
    value;
});

// Prints: <Buffer 01 02 03 04 05>
console.log(copy);
```

### buf.toString([encoding[, start[, end]]])

* `encoding` `<string>` 解码的字符编码。默认：`utf8`
* `start` `<integer>` 解码的起始字节偏移。默认：0
* `end` `<integer>` 解码的结束字节位置（不包含）。默认：`buf.length`
* 返回：`<string>`

依据`encoding`指定的字符编码方式解码`buf`为字符串。`start`和`end`用于解码`buf`的一个子集。

字符串实例的最大长度作为`buffer.constants.MAX_STRING_LENGTH`。

例如：

```js
const buf1 = Buffer.allocUnsafe(26);

for (let i = 0; i < 26; i++) {
  // 97 is the decimal ASCII value for 'a'
  buf1[i] = i + 97;
}

// Prints: abcdefghijklmnopqrstuvwxyz
console.log(buf1.toString('ascii'));

// Prints: abcde
console.log(buf1.toString('ascii', 0, 5));


const buf2 = Buffer.from('tést');

// Prints: 74c3a97374
console.log(buf2.toString('hex'));

// Prints: té
console.log(buf2.toString('utf8', 0, 3));

// Prints: té
console.log(buf2.toString(undefined, 0, 3));
```

### buf.values()

* 返回：`<Iterator>`

创建和返回`buf`值的迭代器。当对`Buffer`使用`for..of`，则会自动调用这个方法。

例如：

```js
const buf = Buffer.from('buffer');

// Prints:
//   98
//   117
//   102
//   102
//   101
//   114
for (const value of buf.values()) {
  console.log(value);
}

// Prints:
//   98
//   117
//   102
//   102
//   101
//   114
for (const value of buf) {
  console.log(value);
}
```

### `buf.write(string[, offset[, length]][, encoding])`

* `string` `<string>` 写入`buf`的字符串。
* `offset` `<integer>` 写入字符串前跳过的字节个数。默认：0
* `length` `<integer>` 写入的字节个数。默认：`buf.length - offset`
* `encoding` `<string>` 字符串的字符编码。默认：`utf8`
* 返回：`<integer>` 写入的字节个数

在依据字符编码方式在`offset`特定的位置向`buf`写入字符串。`length`参数为写入的字节个数。如果`buf`不包含足够的字符来匹配整个字符串，`string`仅部分会被写入。然而，部分编码的字符不会别写入。

例如：

```js
const buf = Buffer.allocUnsafe(256);

const len = buf.write('\u00bd + \u00bc = \u00be', 0);

// Prints: 12 bytes: ½ + ¼ = ¾
console.log(`${len} bytes: ${buf.toString('utf8', 0, len)}`);
```

### buf.writeDoubleBE(value, offset[, noAssert])
### buf.writeDoubleLE(value, offset[, noAssert])

* `value` `<number>` 写入`buf`的数值。
* `offset` `<integer>`在开始写入之前跳过的字节数。必须满足：`0 <= offset <= buf.length - 8`。
* `noAssert` `<boolean>` 是否跳过`value`和`offset`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在特定的`offset`使用特定的字节序列向`buf`写入`value`（`writeDoubleBE()`使用`big-endian`写入，`writeDoubleLE()`使用`little-endian`写入）。`value`应该为64比特的双精度值。当`value`不为64比特的双精度浮点型时，其行为是未定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

例如：

```js
const buf = Buffer.allocUnsafe(8);

buf.writeDoubleBE(0xdeadbeefcafebabe, 0);

// Prints: <Buffer 43 eb d5 b7 dd f9 5f d7>
console.log(buf);

buf.writeDoubleLE(0xdeadbeefcafebabe, 0);

// Prints: <Buffer d7 5f f9 dd b7 d5 eb 43>
console.log(buf);
```

### buf.writeFloatBE(value, offset[, noAssert])
### buf.writeFloatLE(value, offset[, noAssert])

* `value` `<number>` `buf`写入的数值。
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - 4`
* `noAssert` `<boolean>` 是否跳过`value`和`offset`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在特定的`offset`使用特定的字节序列向`buf`写入`value`（`writeFloatBE()`使用`big-endian`写入，`writeFloatLE()`使用`little-endian`写入）。`value`应该为32比特的浮点值。当`value`不为32比特的浮点型时，其行为是未定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

例如：

```js
const buf = Buffer.allocUnsafe(4);

buf.writeFloatBE(0xcafebabe, 0);

// Prints: <Buffer 4f 4a fe bb>
console.log(buf);

buf.writeFloatLE(0xcafebabe, 0);

// Prints: <Buffer bb fe 4a 4f>
console.log(buf);
```

### buf.writeInt8(value, offset[, noAssert])

* `value` `<integer>` `buf`写入的数值
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - 1`
* `noAssert` `<boolean>` 是否跳过`value`和`offset`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在指定的`offset`位置，向`buf`中写入`value`。`value`为有效的有符号8比特的整数。当`value`不为有符号8比特的整型，其行为为为定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

例如：

```js
const buf = Buffer.allocUnsafe(2);

buf.writeInt8(2, 0);
buf.writeInt8(-2, 1);

// Prints: <Buffer 02 fe>
console.log(buf);
```

### buf.writeInt16BE(value, offset[, noAssert])
### buf.writeInt16LE(value, offset[, noAssert])

* `value` `<integer>` `buf`写入的数值。
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - 2`
* `noAssert` `<boolean>` 是否跳过`value`和`offset`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。\

在特定的`offset`使用特定的字节序列向`buf`写入`value`（`writeInt16BE()`使用`big-endian`写入，`writeInt16LE()`使用`little-endian`写入）。`value`应该为有效的16比特浮点值。当`value`不为16比特的浮点型时，其行为是未定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

`value`会作为二进制补码的整数形式解析和写入。

例如：

```js
const buf = Buffer.allocUnsafe(4);

buf.writeInt16BE(0x0102, 0);
buf.writeInt16LE(0x0304, 2);

// Prints: <Buffer 01 02 04 03>
console.log(buf);
```

### buf.writeInt32BE(value, offset[, noAssert])
### buf.writeInt32LE(value, offset[, noAssert])

* `value` `<integer>` `buf`写入的数值。
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - 4`
* `noAssert` `<boolean>` 是否跳过`value`和`offset`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在特定的`offset`使用特定的字节序列向`buf`写入`value`（`writeInt32BE()`使用`big-endian`写入，`writeInt32LE()`使用`little-endian`写入）。`value`应该为有效的32比特浮点值。当`value`不为有符号的32比特的浮点型时，其行为是未定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

`value`会作为二进制补码的整数形式解析和写入。

```js
const buf = Buffer.allocUnsafe(8);

buf.writeInt32BE(0x01020304, 0);
buf.writeInt32LE(0x05060708, 4);

// Prints: <Buffer 01 02 03 04 08 07 06 05>
console.log(buf);
```

### buf.writeIntBE(value, offset, byteLength[, noAssert])
### buf.writeIntLE(value, offset, byteLength[, noAssert])

* `value` `<integer>` `buf`写入的数值。
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - byteLength`
* `byteLength` `<integer>` 写入的字节个数。必须满足：`0 < byteLength <= 6`
* `noAssert` `<boolean>` 是否跳过`value`、`offset`和`byteLength`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在指定的`offset`位置将`value`的`byteLength`长度的字节写入`buf`。支持48比特的精度。当`value`不为有符号的整数，其行为为未定义的。

设置`noAssert`为true，则允许`value`的编码形式超出`buf`的末端，但是其行为为未定义的。

例如：

```js
const buf = Buffer.allocUnsafe(6);

buf.writeIntBE(0x1234567890ab, 0, 6);

// Prints: <Buffer 12 34 56 78 90 ab>
console.log(buf);

buf.writeIntLE(0x1234567890ab, 0, 6);

// Prints: <Buffer ab 90 78 56 34 12>
console.log(buf);
```

### buf.writeUInt8(value, offset[, noAssert])

* `value` `<integer>` `buf`写入的数值。
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - 1`
* `noAssert` `<boolean>` 是否跳过`value`、`offset`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在指定的`offset`位置，向`buf`中写入`value`。`value`为有效的无符号8比特的整数。当`value`不为无符号8比特的整型，其行为为为定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

例如：

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt8(0x3, 0);
buf.writeUInt8(0x4, 1);
buf.writeUInt8(0x23, 2);
buf.writeUInt8(0x42, 3);

// Prints: <Buffer 03 04 23 42>
console.log(buf);
```

### buf.writeUInt16BE(value, offset[, noAssert])
### buf.writeUInt16LE(value, offset[, noAssert])

* `value` `<integer>` `buf`写入的数值。
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - 2`
* `noAssert` `<boolean>` 是否跳过`value`、`offset`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在特定的`offset`使用特定的字节序列向`buf`写入`value`（`writeUInt16BE()`使用`big-endian`写入，`writeUInt16LE()`使用`little-endian`写入）。`value`应该为有效无符号16比特整数。当`value`不为无符号16字节整数，其行为是未定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

例如：

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt16BE(0xdead, 0);
buf.writeUInt16BE(0xbeef, 2);

// Prints: <Buffer de ad be ef>
console.log(buf);

buf.writeUInt16LE(0xdead, 0);
buf.writeUInt16LE(0xbeef, 2);

// Prints: <Buffer ad de ef be>
console.log(buf);
```

### buf.writeUInt32BE(value, offset[, noAssert])
### buf.writeUInt32LE(value, offset[, noAssert])

* `value` `<integer>` `buf`写入的数值。
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - 4`
* `noAssert` `<boolean>` 是否跳过`value`、`offset`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在特定的`offset`使用特定的字节序列向`buf`写入`value`（`writeUInt16BE()`使用`big-endian`写入，`writeUInt16LE()`使用`little-endian`写入）。`value`应该为有效无符号32比特整数。当`value`不为无符号32字节整数，其行为是未定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

例如：

```js
const buf = Buffer.allocUnsafe(4);

buf.writeUInt32BE(0xfeedface, 0);

// Prints: <Buffer fe ed fa ce>
console.log(buf);

buf.writeUInt32LE(0xfeedface, 0);

// Prints: <Buffer ce fa ed fe>
console.log(buf);
```

### buf.writeUIntBE(value, offset, byteLength[, noAssert])
### buf.writeUIntLE(value, offset, byteLength[, noAssert])

* `value` `<integer>` `buf`写入的数值。
* `offset` `<integer>` 开始写入之前跳过的字节个数。必须满足：`0 <= offset <= buf.length - byteLength`
* `byteLength` `<integer>` 写入的字节个数。必须满足：`0 < byteLength <= 6`。
* `noAssert` `<boolean>` 是否跳过`value`、`offset`和`byteLength`的校验。默认：false
* 返回：`<integer>` `offset`加上写入的字节个数。

在特定的`offset`位置写入`value`的`byteLength`字节写入。支持48比特的精度。当`value`不为无符号整数，其行为为未定义的。

设置`noAssert`为`true`可以让`value`的编码结果超过`buf`的末端，但是其结果为未定义的行为。

例如：

```js
const buf = Buffer.allocUnsafe(6);

buf.writeUIntBE(0x1234567890ab, 0, 6);

// Prints: <Buffer 12 34 56 78 90 ab>
console.log(buf);

buf.writeUIntLE(0x1234567890ab, 0, 6);

// Prints: <Buffer ab 90 78 56 34 12>
console.log(buf);
```

### `buffer.INSPECT_MAX_BYTES`

* `<integer>` 默认：50

返回当`buf.inspect()`执行时能够返回的最大字节个数。可以使用用户的模块来覆盖。

注意这为`require('buffer')`返回的`buffer`模块的属性，而不是`Buffer`全局对象或者`Buffer`实例的属性。

### buffer.kMaxLength

* `<integer>` 单个`Buffer`实例的最大尺寸。

`buffer.constants.MAX_LENGTH`的别名。

注意这为`require('buffer')`返回的`buffer`模块的属性，而不是`Buffer`全局对象或者`Buffer`实例的属性。

### buffer.transcode(source, fromEnc, toEnc)

* `source` `<Buffer>`|`<Uint8Array>` `Buffer`或者`UInt8Array`实例。
* `fromEnc` `<string>` 当前的编码方式。
* `toEnc` `<string>` 目标编码方式

对给定的`Buffer`或者`Uint8Array`实例从一个字符编码重新编码成另一个编码方式。返回新的`Buffer`实例。

如果`fromEnc`或者`toEnc`指定无效的字符编码或者从`fromEnc`向`toEnc`转换不合法，则抛出异常。

如果给定的字节序列不能使用目标编码来相应的表示，则会使用替代字符来表示。例如：

```js
const buffer = require('buffer');

const newBuf = buffer.transcode(Buffer.from('€'), 'utf8', 'ascii');
console.log(newBuf.toString('ascii'));
// Prints: '?'
```

因为欧元符号（€）不能使用US-ASCII来表示，在转换编码的`Buffer`中使用`?`替代。

注意这为`require('buffer')`返回的`buffer`模块的属性，而不是`Buffer`全局对象或者`Buffer`实例的属性。


## 类：SlowBuffer

> 稳定性：0 - 过时：使用`Buffer.allocUnsafeSlow()`取代之

返回非从内存池分配的`Buffer`对象。

为了避免创建大量独立分配的`Buffer`实例的垃圾回收，默认对小于4KB的内存分配会从预先分配的大对象上分配。这个方法会提升性能和性能使用，因为v8不会使用追踪和清理大量的`Persistent`对象。

为了避免开发者在不确定的时间从内存池中持有一小块的内存，这样使用`SlowBuffer`创建非内存池分配的`Buffer`实例就很适合。

例如：

```js
// Need to keep around a few small chunks of memory
const store = [];

socket.on('readable', () => {
  const data = socket.read();

  // Allocate for retained data
  const sb = SlowBuffer(10);

  // Copy the data into the new allocation
  data.copy(sb, 0, 0, 10);

  store.push(sb);
});
```

当用户程序中观察到不是到的内存持有，则使用`SlowBuffer`为左后的解决方法。

### new SlowBuffer(size)

> 稳定性：0 - 过时：使用`Buffer.allocUnsafeSlow()`取代之

* `size` `<integer>` 新建的`SlowBuffer`实例的期望尺寸。

分配`size`字节的`Buffer`实例。如果`size`大于`buffer.constants.MAX_LENGTH`或者掉小于0，会抛出`RangeError`。如果`size`为0，则会创建零尺寸的`Buffer`实例。

`SlowBuffer`实例的潜在内存为初始化。新建的`SlowBuffer`的内容为未知的且可能包含敏感数据。使用`buf.fill(0)`来初始化`SlowBuffer`的内存。

例如：

```js
const { SlowBuffer } = require('buffer');

const buf = new SlowBuffer(5);

// Prints: (contents may vary): <Buffer 78 e0 82 02 01>
console.log(buf);

buf.fill(0);

// Prints: <Buffer 00 00 00 00 00>
console.log(buf);
```

## Buffer常量

注意`require('buffer')`返回的`buffer`模块的`buffer.constants`属性，而不是`Buffer`全局对象或者`buffer`实例。

### buffer.constants.MAX_LENGTH

* `<integer>` 对单个`Buffer`实例允许分配的最大尺寸。

在32位机器上，这个值为`2^30 -1`(~1GB)。在64为机器上，这个值为`2^31 - 1`(~2GB)。

这个值也可以在`buffer.kMaxLength`获得。

### `buffer.constants.MAX_STRING_LENGTH`

* `<integer>` 允许单个`string`实例允许分配的最大尺寸。

`string`类型实例可以表示的最大的`length`，使用UTF-16代码单元计算。

这依赖于具体使用的JS引擎。
