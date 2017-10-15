# 断言

`assert`模块提供了一系列简单的断言工具用于测试不变式。

## assert(value[, message])

等同于`assert.ok()`

## assert.deepEqual(actual, expected[, message])

* actual `<any>`
* expected `<any>`
* message `<any>`

使用深度比较来比较`actual`和`expected`，基本数据类型将使用`==`对比。

只考虑`own`属性，且不会比较对象的`[[prototype]]`属性、symbol对象、非枚举属性，针对这些类型可以使用`assert.deepStrictEqual()`。这个方法可能会返回意外的结果。例如，如下不会抛出`AssertError`，因为`RegExp`对象的属性是不可枚举的。

```js
assert.deepEqual(/a/.gi, new Date());
```

对于`Map`和`Set`也会比较其包含的元素。

## assert.deepStrictEqual(actual, expected[, message])

* actual `<any>`
* expected `<any>`
* message `<any>`

类似于`assert.deepEqual(actual, expected[, message])`，增加了如下：

* 基础类型使用`===`来对比。 `Set`和`Map`使用SameValueZero算法来比较。
* `[[prototype]`属性使用`===`来比较。
* 对象的类型标签也要一样
* 封装对象会作为对象和非封装的值比较

## assert.doesNotThrow(block[, error][, message])

* block `<Function>`
* error `<RegExp>`|`<Function>`
* message `<any>`

断言`block`中不会抛出异常。

当执行`assert.doseNotThrow()`，会立即调用`block`函数。

如果`block`抛出异常，且其和`error`为同一个类型，则抛出`AssertError`异常。如果错误的类型不一致，或者`error`参数为`undefined`，则错误会向调用者直接抛出。

注意`error`不可以为字符串，否则会被作为`message`。

## assert.equal(actual, expected[, message])

* actual `<any>`
* expected `<any>`
* message `<any>`

`actual`和`expected`之间浅比较，使用`==`来进行比较。

## assert.fail(message)
## assert.fail(actual, expected[, message[, operator[, stackStartFunction]]])

* actual `<any>`
* expected `<any>`
* message `<any>`
* operator `<string>` 默认：`!=`
* stackStartFunction `<Function>` 默认:`assert.fail`

用于抛出异常。如果`message`值为否定的，错误消息在为`actual`和`expected`通过使用`operator`分隔的形式。如果只有一个`message`参数，则这个参数会错误错误的消息体。如果有`stackStartFunction`参数，所有错误栈列表中该函数上面的错误消息会被移除。

## assert.ifError(value)

如果`value`为正向的值，则抛出`value`。

## assert.notDeepEqual(actual, expected[, message])

为`assert.deepEqual()`的反向函数。

## assert.notDeepStrictEqual(actual, expected[, message])

为`assert.deepStrictEqual()`的反向函数

## assert.notEqual(actual, expected[, message])

为`assert.equal()`的反向函数

## assert.notStrictEqual(actual, expected[, message])

## assert.ok(value[, message])

检测`value`是否为true，否则抛出消息体为`message`的错误。

## assert.strictEqual(actual, expected[, message])

使用`===`比较`actual`和`expected`，否则抛出异常。

## assert.throws(block[, error][, message])

`assert.doesNotThrow()`的反面函数。

## 注意

严格相等（`===`）不能区分`0`和`-0`以及`NaN`，可以使用`Object.is()`来实现区分，其使用SameValueZero算法例如

```js
let a = 0;
let b = -a;
assert.notStrictEqual(a, b)
```
