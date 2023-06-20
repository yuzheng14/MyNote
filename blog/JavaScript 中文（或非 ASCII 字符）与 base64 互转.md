# JavaScript 中文（或非 ASCII 字符）与 base64 互转

[TOC]

由于浏览器和 Nodejs 中的接口等不一致，所以需要分类讨论

## 浏览器

此方案基于 `utf-8` 编码实现编/解码。经过个人探索，除了 `utf-8` 外，JavaScript 字符串仅可以使用 `utf-16` 来编解码，原理类似，仅在取二进制值时不同，不在此赘述，留作思考题。

### 实现

如果仅仅只是快速采用轮子，可以直接 cv 下面的代码。

javascript 版：

```javascript
function encode64(text) {
  return btoa(String.fromCharCode(...new TextEncoder().encode(text)))
}

function decode64(text) {
  return new TextDecoder().decode(Uint8Array.from(atob(text), (c) => c.charCodeAt(0)))
}

```

typescript 版：

```typescript
function encode64(text: string): string {
  return btoa(String.fromCharCode(...new TextEncoder().encode(text)))
}

function decode64(text: string): string {
  return new TextDecoder().decode(Uint8Array.from(atob(text), (c) => c.charCodeAt(0)))
}
```

### 原理讲解

浏览器中用于将字符串和 base64 互转的 api 为 [`atob`](https://developer.mozilla.org/zh-CN/docs/Web/API/atob) 和 [`btoa`](https://developer.mozilla.org/zh-CN/docs/Web/API/btoa) ，但是这两个 API 只支持 Latin-1 字符集。如果需要对中文进行编码，`btoa` 则会出现如下错误：

```
Uncaught DOMException: Failed to execute 'btoa' on 'Window': The string to be encoded contains characters outside of the Latin1 range.
```

`atob` 则无法解码出正确的字符串

```typescript
// '5Lit5paH' 为 '中文' 的 utf-8 的 base64 编码
console.log(atob('5Lit5paH'))
// 输出: 'ä¸­æ\x96\x87'
```

mdn 中的[解释](https://developer.mozilla.org/zh-CN/docs/Web/API/btoa#unicode_%E5%AD%97%E7%AC%A6%E4%B8%B2)如下

> 根据设计，Base64 仅将二进制数据作为其输入。而在 JavaScript 字符串中，这意味着每个字符只能使用一个字节表示。所以，如果你将一个字符串传递给 `btoa()`，而其中包含了需要使用超过一个字节才能表示的字符，你就会得到一个错误，因为这个字符串不能被看作是二进制数据

所以对于中文编码成 base64，我们的思路就很简单：取字符串的二进制（`utf-8` 也好 `utf-16` 也好），取每一字节当做一个字符的码点，重新生成一个字符串，再拿这个字符串去调用 `btoa`，就可以成功转出来 base64 了。

那么第一步就是拿到这个字符串的 `utf-8` 的二进制流。虽然 JavaScript 默认使用 `utf-16`，但是其提供了 [`TextEncoder.encode()`](https://developer.mozilla.org/zh-CN/docs/Web/API/TextEncoder/encode) 和 [`TextDecoder.decode()`](https://developer.mozilla.org/zh-CN/docs/Web/API/TextDecoder/decode) 用于字符串和 `utf-8` 字节流转换。接下来以`中文`这个字符串为例，分**编码**和**解码**进行讲解。

#### 编码

对于编码，我们可以通过下面的代码拿到一个字符串的 `utf-8` 字节流

```typescript
new TextEncoder().encode('中文')
// 此字节流的值为 Uint8Array: [228, 184, 173, 230, 150, 135]
```

现在我们就拿到了`中文`的二进制，为 228, 184, 173, 230, 150, 135，一共6位。

接下来就可以拿这6个字节来构造6个字符，形成一个字符串。这些二进制的值又被称为一个字符的码点，所以我们可以使用 [`String.fromCharCode()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/fromCharCode) 这个静态方法将这 6 个值转换成 6 个字符

> 静态 **`String.fromCharCode()`** 方法返回由指定的 UTF-16 代码单元序列创建的字符串。
>
> ```
> String.fromCharCode(num1[, ...[, numN]])
> ```
>
> —— mdn

由 `String.fromCharCode()` 的函数签名和 `Uint8Array` 的数组特性可知，我们可以直接使用下面的代码将得到的二进制字节流转换成字符串

```typescript
String.fromCharCode(...new TextEncoder().encode('中文'))
// 值: 'ä¸­æ\x96\x87'
```

现在我们将一个 `utf-16` 的字符串成功转成了 `utf-8` 字节流对应的字符串，现在我们就可以使用 `btoa()` 将这个字符串转换成 base64 编码了。

```typescript
btoa(String.fromCharCode(...new TextEncoder().encode('中文')))
// 值: '5Lit5paH'
```

#### 解码

对于解码，首先我们使用 `atob()` 将上面得到的 base64 编码转换成字符串。

```typescript
atob('5Lit5paH')
// 值: 'ä¸­æ\x96\x87'
```

接下来我们需要将这个字符串转换成一个 `Uint8Array` 二进制字节流，这里我们可以使用 [`Uint8Array.from()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray/from) 这个 api 来将字符串转换成 `Uint8Array` 字节流。

> `TypedArray.from()` 方法 从一个类数组或者可迭代对象中创建一个新[类型数组](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray#typedarray_objects)。这个方法和 [`Array.from()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/from) 类似。
>
> ```javascript
> TypedArray.from(arrayLike, mapFn)
> TypedArray.from(arrayLike, mapFn, thisArg)
> ```
>
> —— mdn

但是如果我们如下代码直接传递给 `Uint8Array` 的话，会发现这个二进制字节流的每一位都是 `0`。

```typescript
Uint8Array.from(atob('5Lit5paH'))
// 值: Uint8Array: [0, 0, 0, 0, 0, 0]
```

此处明显为 `Uint8Array.from` 没有正确取到该字符串每一位的码点（即使是使用 `Uint16Array` 也不行），但是观察他的函数签名，我们可以发现有 `mapFn` 这个参数，所以我们可以通过这个参数取到每一个字符的码点。

对于取字符的码点，有 [`String.prototype.charCodeAt()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/charCodeAt) 这个 api。所以转换成二进制字节流的代码如下：

```typescript
Uint8Array.from(atob('5Lit5paH'), (c) => c.charCodeAt(0))
// 值: Uint8Array: [228, 184, 173, 230, 150, 135]
```

现在我们就可以将其传递给 `TextDecoder.decode()` 转换成字符串了！

```typescript
new TextDecoder().decode(Uint8Array.from(atob(text), (c) => c.charCodeAt(0)))
```

### 思考题

上面的实现为通过 `utf-8` 编码的二进制流与 base64 互转的实现，那么如果通过 JavaScript 默认的 `utf-16` 又该怎么实现呢？

> 提示：可以参考 mdn 中的[此部分](https://developer.mozilla.org/zh-CN/docs/Web/API/btoa#unicode_%E5%AD%97%E7%AC%A6%E4%B8%B2)进行实现，并优化到一行代码。

## Nodejs

Nodejs 中由于有原生的 api 用于转换 base64，在 vscode 中如果把鼠标放到 `atob()` 或者 `btoa()` 上，可以得到如下说明

> This function is only provided for compatibility with legacy web platform APIs and should never be used in new code, because they use strings to represent binary data and predate the introduction of typed arrays in JavaScript. For code running using Node.js APIs, converting between base64-encoded strings and binary data should be performed using `Buffer.from(str, 'base64')` and`buf.toString('base64')`.

据此实现就很简单了。

### 实现

JavaScript 版：

```javascript
function encode64(text) {
  return Buffer.from(text).toString('base64')
}

function decode64(text) {
  return Buffer.from(text, 'base64').toString()
}
```

typescript 版：

```typescript
function encode64(text: string): string {
  return Buffer.from(text).toString('base64')
}

function decode64(text: string): string {
  return Buffer.from(text, 'base64').toString()
}
```
