---
layout: post
title: 前端进阶笔记(一)
date: 2019-10-14 
tags: 其他   
---

### 　一、apply/call/bind 一网打尽

首先，这三个方法是用来改变 this 指向的，接下来我们看一下它们的异同。

## 1. apply

- 调用一个对象的一个方法，用另一个对象替换当前对象。例如：`B.apply(A, arguments)`; 即 A 对象应用 B 对象的方法。
- 要注意的是第一个参数，如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对象，而其他原始值则会被相应的包装对象（wrapper object）所替代。

### 1.1 如何实现一个apply

回顾一下 apply 的效果，我们可以大致按以下思路走

1. 实现第一个参数的功能，改变 this 指向
2. 实现第二个参数的功能。第二个参数是作为调用函数的参数
3. 返回值：使用调用者提供的 this 值和参数调用该函数的返回值。若该方法没有返回值，则返回 undefined。

接下来，我们按以上思路来实现一下。

#### 1.1.1 第一步，绑定 this

```
微信公众号：世界上有意思的事

f.apply(o);

// 与下面代码的功能类似（假设对象o中预先不存在名为m的属性）。
o.m=f; //将f存储为o的临时方法
o.m(); //调用它，不传入参数
delete o.m;//将临时方法删除
复制代码
```

(以上代码摘录自犀牛书)
 依样画葫芦，我们可以这么写：

```
微信公众号：世界上有意思的事

Function.prototype.apply = function (context) {
    // context 就是需要绑定的对象，相当于上面的 o
    // this 就是调用了 apply 的函数，相当于 f
    context.__fn = this // 假设原先没有__fn
    context.__fn()
    delete context.__fn
}
复制代码
```

#### 1.1.2 第二步，给函数传递参数

接下来我们想办法实现一下 apply 的第二个参数。其实我最快想到的是 ES6 的方法。用`...` 直接展开就行了。不过 apply 才 ES3😂，还是再想想老的办法吧。

难点是这个数组的长度是不确定的，也就是说我们没办法很准确地给函数一个个传参。我们所能做的处理也就是把`arguments`转成字符串形式`'arguments[1], arguments[2], ...'`。那么如何让字符串能运行起来呢？？答案就是 `eval`！

稍稍总结一下, 目前想到的 2 种方法

> 1. es6。`context.__fn(...arguments)`
> 2. 把 arguments 转换成string，放到 eval 里面运行 `eval('context.__fn('+ 'arguments[1], arguments[2]' +')')`

以下是第二种思路的代码：

```
微信公众号：世界上有意思的事

Function.prototype.apply = function (context, others) {
    // context 就是需要绑定的对象，相当于上面的 o
    // this 就是调用了 apply 的函数，相当于 f
    context.__fn = this // 假设原先没有__fn

    var args = [];
    // args: 'others[0], others[1], others[2], ...'
    for (var i = 0, len = others.length; i < len; i++) {
        args.push('others[' + i + ']');
    }

    eval('context.__fn(' + args.toString() + ')')

    delete context.__fn
}
复制代码
```

#### 1.1.3 第三步，返回值

返回函数调用后的结果就行：

```
微信公众号：世界上有意思的事

Function.prototype.apply = function (context, others) {
    // context 就是需要绑定的对象，相当于上面的 o
    // this 就是调用了 apply 的函数，相当于 f
    context.__fn = this // 假设原先没有__fn

    var result;

    var args = [];
    // args: 'others[0], others[1], others[2], ...'
    for (var i = 0, len = others.length; i < len; i++) {
        args.push('others[' + i + ']');
    }

    result = eval('context.__fn(' + args.toString() + ')')

    delete context.__fn
}
复制代码
```

#### 1.1.4 更进一步，严格模式下的 this

我们之前有提到：第一个参数，如果这个函数处于非严格模式下，则指定为 null 或 undefined 时会自动替换为指向全局对象，而其他原始值则会被相应的包装对象（wrapper object）所替代

```
微信公众号：世界上有意思的事

Function.prototype.apply = function (context, others) {

    if (typeof argsArray === 'undefined' || argsArray === null) {
        context = window
    }

    // context 是一个 object
    context = new Object(context)

    // context 就是需要绑定的对象，相当于上面的 o
    // this 就是调用了 apply 的函数，相当于 f
    context.__fn = this // 假设原先没有__fn

    var result;

    var args = [];
    for (var i = 0, len = others.length; i < len; i++) {
        args.push('others[' + i + ']');
    }

    result = eval('context.__fn(' + args.toString() + ')')

    delete context.__fn
}
复制代码
```

#### 1.1.5 再进一步，确保 __fn 不存在

我们之前的代码都是建立在 `__fn` 不存在的情况下，那么万一存在呢？因此我们接下来就要找一个 `context` 中没有存在过的属性。
 🤔我们很快可以想到 ES6 的 symbol。

```
// 像这样
var __fn = new Symbol()
context[__fn] = this
复制代码
```

🤔如果不用 ES6，那么另一种方法，是根据 [这篇文章](https://juejin.im/post/5bf6c79bf265da6142738b29#heading-4)中提到的，自己用 Math.random() 模拟实现独一无二的 key。面试时可以直接用生成时间戳即可。

```
微信公众号：世界上有意思的事

// 生成 UUID 通用唯一识别码
// 大概生成 这样一串 '18efca2d-6e25-42bf-a636-30b8f9f2de09'
function generateUUID(){
    var i, random;
    var uuid = '';
    for (i = 0; i < 32; i++) {
        random = Math.random() * 16 | 0;
        if (i === 8 || i === 12 || i === 16 || i === 20) {
            uuid += '-';
        }
        uuid += (i === 12 ? 4 : (i === 16 ? (random & 3 | 8) : random))
            .toString(16);
    }
    return uuid;
}
// 简单实现
// '__' + new Date().getTime();
复制代码
```

如果这个key万一这对象中还是有，为了保险起见，可以做一次缓存操作(就是先把之前的值保存起来)

```
// 像这样
var originalvalue = context.__fn
var hasOriginalValue = context.hasOwnProperty('__fn')
context.__fn = this

if(hasOriginalValue){
    context.__fn = originalvalue;
}
复制代码
```

## 2. call

- 和 apply 的作用是一样的，只是 `call()` 方法接受的是一个参数列表，而 `apply()` 方法接受的是一个包含多个参数的数组。
- 例如 `func.apply(obj, [1,2])` 相当于 `func.call(obj, 1, 2)`

思路和 apply 一样。唯一区别就在于参数形式。我们按照 call 的要求来处理参数就可以了：

```
微信公众号：世界上有意思的事

Function.prototype.apply = function (context) {
    // context 就是需要绑定的对象，相当于上面的 o
    // this 就是调用了 apply 的函数，相当于 f
    context.__fn = this // 假设原先没有__fn

    var result;

    var args = [];
    // 我们从 arguments[1] 开始拼就好了
    for (var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }

    result = eval('context.__fn(' + args.toString() + ')')

    delete context.__fn
}
复制代码
```

## 3. bind

我们常将 bind 和以上两个方法区分开，是因为 bind 是 ECMAScript 5 中的方法，且除了将函数绑定至一个对象外还多了一些特点。

- **bind() 方法创建一个新的函数**，在 bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数，而其余参数将作为新函数的初始参数，供调用时使用。

  ```
  func.apply(obj, [1,2])
  // 相当于
  func.call(obj, 1, 2)
  // 相当于
  var boundFun = func.bind(obj, 1, 2)
  boundFun()
  // 也可以这样
  var boundFun = func.bind(obj, 1)
  boundFun(2)
  复制代码
  ```

- **绑定函数也可以使用 new 运算符构造**，它会表现为目标函数已经被构建完毕了似的。提供的 this 值会被忽略，但前置参数仍会提供给模拟函数。

我们还是先大致思考一下该怎么做：

1. 实现第一个参数的功能，改变 this 指向。这个和 apply/call 是一样的。
2. 返回值：返回一个新的函数。
3. 实现其它参数。其它参数将作为新函数的初始参数，供调用时使用。这个和 call 有些相似。
4. 使用 new 操作符时，应该忽略第一个参数

后续的步骤我会用 apply/call 来实现bind。如果不想直接用 apply/call，也可以按照上文先实现一个 apply/call。

#### 3.1.1 第一步，返回一个绑定了 this 的新函数

```
Function.prototype.bind = function (context) {
    var self = this;
    return function () {
        return self.apply(context);
    }
}
复制代码
```

#### 3.1.2 第二步，给新函数设定初始参数

```
微信公众号：世界上有意思的事

Function.prototype.bind = function (context) {
    var self = this;

    // 获取 bind 函数从第二个参数到最后一个参数
    var initialArgs = Array.prototype.slice.call(arguments, 1);
    
    // 返回一个绑定好 this 的新函数
    return function () {
        // 这个是调用新函数时传入的参数
        var boundArgs = Array.prototype.slice.call(arguments);
        // 最终的参数应该是初始参数+新函数的参数
        return self.apply(context, args.concat(bindArgs));
    }
}
复制代码
```

#### 3.1.3 第三步，作为构造函数调用时，忽略要绑定的 this

这里的难点是怎么知道是由 new 调用的。
 先说一下答案吧

```
// 假如有以下函数
function Person () {
    console.log(this)
}
复制代码
```

> 对于 `var gioia = new Person()` 来说
>  使用 new 时，this 会指向 gioia，并且 gioia 是 Person 的实例。 因此，如果 `this instance Person`，就说明是 new 调用的

new 这一部分这里先不展开讲，有兴趣的可以看一下 [JavaScript深入之new的模拟实现](https://github.com/mqyqingfeng/Blog/issues/13)
 接下来我们可以写代码了：

```
微信公众号：世界上有意思的事

Function.prototype.bind = function (context) {
    var self = this;

    // 获取 bind 函数从第二个参数到最后一个参数
    var initialArgs = Array.prototype.slice.call(arguments, 1);
    
    // 返回一个绑定好 this 的新函数
    function Bound() {
        // 这个是调用新函数时传入的参数
        var boundArgs = Array.prototype.slice.call(arguments);
        // 最终的参数应该是初始参数+新函数的参数
        return self.apply(this instance Bound ? this : context, args.concat(bindArgs));
    }

    Bound.prototype = this.prototype
    return Bound
}
复制代码
```

# 二、如何实现一个深拷贝

这部分我是看了 [lodash](https://github.com/lodash/lodash/blob/4.17.15/lodash.js#L11087) 的相关源码，它真的实现得非常完整！
 总结成一句话，就是需要考虑很多数据类型，然后针对这些数据类型拷贝就行了😏

对于引用类型来说，我们基本可以按照以下思路走：

1. 初始化。即调用相应的构造函数
2. 递归地赋值
3. 有循环引用的话需要处理一下

## 1. 拷贝基本类型

基本类型直接赋值就可以。

```
function deepClone (value) {
    // 基本类型
    if (!isObject(value)) {
        return value
    }
}

// 判断是不是对象
function isObject(value) {
    const type = typeof value
    return value != null && (type === 'object' || type === 'function')
}
复制代码
```

接下来是怎么拷贝引用类型。我会按照以下顺序来介绍：

1. 数组
2. 函数
3. 对象
4. 特殊类型。Boolean、Date、Map、Number 等等

另外，lodash 还实现了 Buffer（node.js）等拷贝，但我实际用得不多，就不展开了，有兴趣的可以去看看源码。

## 2. 拷贝数组

### 2.1 初始化

先初始化一个长度为原数组长度的数组

```
微信公众号：世界上有意思的事

export function deepClone(value) {
    let result

    // 基本类型
    if (!isObject(value)) {
      return value
    }

    const isArr = Array.isArray(value)
    
    // 数组
    if (isArr) {
        result = initCloneArray(value)
    }
    // 待续
}

const hasOwnProperty = Object.prototype.hasOwnProperty

// 数组初始化
function initCloneArray(array) {
    const { length } = array
    const result = new array.constructor(length)

    // 因为 RegExp.prototype.exec() 会返回一个数组或 null，这个数组里有两个特殊的属性：input、index
    // 类似 ["foo", index: 6, input: "table football, foosball", groups: undefined]
    // 所以需要进行特殊处理
    if (length && typeof array[0] === 'string' && hasOwnProperty.call(array, 'index')) {
        result.index = array.index
        result.input = array.input
    }
    return result
}
复制代码
```

### 2.2 赋值

```
微信公众号：世界上有意思的事

export function deepClone(value) {
    let result

    // 基本类型
    if (!isObject(value)) {
      return value
    }

    const isArr = Array.isArray(value)
    
    // 数组
    if (isArr) {
        result = initCloneArray(value)
    }

    // 赋值
    if (isArr) {
        for (let i = 0; i< value.length; i++) {
            result[i] = deepClone(value[i])
        }
    }

    return result
}
复制代码
```

## 3. 拷贝函数

函数的拷贝的话，我们还是返回之前的引用。

```
微信公众号：世界上有意思的事

export function deepClone(value) {
    let result

    // 基本类型
    if (!isObject(value)) {
      return value
    }

    const isArr = Array.isArray(value)
    
    // 数组
    if (isArr) {
        result = initCloneArray(value)
    } else {
        const isFunc = typeof value === 'function'
        // 函数
        if (isFunc) {
            return value
        }
    }

    // 赋值
    if (isArr) {
        for (let i = 0; i< value.length; i++) {
            result[i] = deepClone(value[i])
        }
    }

    return result
}
复制代码
```

## 4. 拷贝对象

初始化一个对象，然后赋值。
 要注意的是这个拷贝后的对象和原对象的原型链是一样的

```
微信公众号：世界上有意思的事

function deepClone(value) {
    let result
    
    // 基本类型
    if (!isObject(value)) {
        return value
    }
    
    const isArr = Array.isArray(value)
    const tag = getTag(value)

    // 数组
    if (isArr) {
        result = initCloneArray(value)
    } else {
        const isFunc = typeof value === 'function'
        // 函数
        if (isFunc) {
            return value
        }

        // 对象或 arguments
        if (tag == '[object Object]' || tag == '[object Arguments]') {
            result = initCloneObject(value)
        }
    }

    if (isArr) {
        // 数组赋值
        for (let i = 0; i< value.length; i++) {
            result[i] = deepClone(value[i])
        }
    } else {
        // 对象赋值
        Object.keys(Object(value)).forEach(k => {
            result[k] = deepClone(value[k])
        })
    }

    return result

}

// 能更细致地判断是什么类型
function getTag(value) {
    if (value == null) {
        return value === undefined ? '[object Undefined]' : '[object Null]'
    }
    return toString.call(value)
}

const objectProto = Object.prototype
function isPrototype(value) {
    const Ctor = value && value.constructor
    const proto = (typeof Ctor === 'function' && Ctor.prototype) || objectProto
  
    return value === proto
}

// 初始化对象
function initCloneObject(object) {
    return (typeof object.constructor === 'function' && !isPrototype(object))
      ? Object.create(Object.getPrototypeOf(object))
      : {}
}
复制代码
```

## 5. 拷贝特殊对象

包括 `Boolean`, `Date`, `Map`, `Number`, `RegExp`, `Set`,  `String`, `Symbol`

接下来的思路也是一样的，先调用对应的构造函数。然后赋值就行了。稍微麻烦一点的可能是 Regexp 正则对象和 Symbol 对象

### 5.1 初始化

```
微信公众号：世界上有意思的事

function deepClone(value) {
    let result
    
    // 基本类型
    if (!isObject(value)) {
        return value
    }

    const isArr = Array.isArray(value)
    const tag = getTag(value)

    // 数组
    if (isArr) {
        result = initCloneArray(value)
    } else {
        const isFunc = typeof value === 'function'
        // 函数
        if (isFunc) {
            return value
        }
      
        // 对象或 arguments
        if (tag == '[object Object]' || tag == '[object Arguments]') {
            result = initCloneObject(value)
        } else {
            // 特殊对象的初始化
            result = initCloneByTag(value, tag)
        }
    }

    if (isArr) {
        // 数组赋值
        for (let i = 0; i< value.length; i++) {
            result[i] = deepClone(value[i])
        }
    } else {
        // 对象赋值
        Object.keys(Object(value)).forEach(k => {
            result[k] = deepClone(value[k])
        })
    }

    return result

}

const toString = Object.prototype.toString
// 能更细致地判断是什么类型
function getTag(value) {
    if (value == null) {
        return value === undefined ? '[object Undefined]' : '[object Null]'
    }
    return toString.call(value)
}

const objectProto = Object.prototype
function isPrototype(value) {
    const Ctor = value && value.constructor
    const proto = (typeof Ctor === 'function' && Ctor.prototype) || objectProto
  
    return value === proto
}

// 特殊对象的初始化
function initCloneByTag(object, tag, isDeep) {
    const Ctor = object.constructor
    switch (tag) {
  
      case '[object Boolean]':
      case '[object Date]':
        return new Ctor(+object)
  
      case '[object Set]':
      case '[object Map]':
        return new Ctor
  
      case '[object Number]':
      case '[object String]':
        return new Ctor(object)
  
      case '[object RegExp]':
        return cloneRegExp(object)
  
      case '[object Symbol]':
        return cloneSymbol(object)
    }
}

const reFlags = /\w*$/
// 拷贝一个正则
function cloneRegExp(regexp) {
    // RegExp 构造函数有两个参数。pattern（正则表达式的文本。），flags（标志）
    // source 属性返回一个值为当前正则表达式对象的模式文本的字符串，该字符串不会包含正则字面量两边的斜杠以及任何的标志字符。
    // reFlags.exec(regexp) 实际上是 reFlags.exec(regexp.toString())。提取出了标志字符
    const result = new regexp.constructor(regexp.source, reFlags.exec(regexp))
    result.lastIndex = regexp.lastIndex
    return result
}

const symbolValueOf = Symbol.prototype.valueOf
// 拷贝一个 Symbol
function cloneSymbol(symbol) {
    return Object(symbolValueOf.call(symbol))
}
复制代码
```

### 5.2 赋值

虽然是特殊对象，但也是对象，所以我们的思路还是获取该对象的所有属性，然后赋值就可以了。
 需要注意的是

1. `Object.keys` 不能获取 Symbol 属性，可以再加上 `Object.getOwnPropertySymbols()`来获取所有 Symbol 属性名
2. Set 和 Map 的赋值是通过 add 和 set 来的

```
微信公众号：世界上有意思的事

function deepClone(value) {
    let result
    
    // 基本类型
    if (!isObject(value)) {
        return value
    }

    const isArr = Array.isArray(value)
    const tag = getTag(value)

    // 数组
    if (isArr) {
        result = initCloneArray(value)
    } else {
        const isFunc = typeof value === 'function'
        // 函数
        if (isFunc) {
            return value
        }
      
        // 对象或 arguments
        if (tag == '[object Object]' || tag == '[object Arguments]') {
            result = initCloneObject(value)
        } else {
            // 特殊对象的初始化
            result = initCloneByTag(value, tag)
        }
    }

    // Map 赋值
    if (tag === mapTag) {
        value.forEach((subValue, key) => {
            result.set(key, deepClone(subValue))
        })
        return result
    }

    // Set 赋值
    if (tag === setTag) {
        value.forEach((subValue) => {
            result.add(deepClone(subValue))
        })
      return result
    }

    if (isArr) {
        // 数组赋值
        for (let i = 0; i< value.length; i++) {
            result[i] = deepClone(value[i])
        }
    } else {
        // 对象赋值
        Object.keys(Object(value)).forEach(k => {
            result[k] = deepClone(value[k])
        })

        const propertyIsEnumerable = Object.prototype.propertyIsEnumerable
        
        // 过滤掉不可枚举的 Symbol 属性并赋值
        Object.getOwnPropertySymbols(value)
            .filter((symbol) => propertyIsEnumerable.call(value, symbol))
            .forEach(k => {
                result[k] = deepClone(value[k])
            })
    }

    return result

}
复制代码
```

## 6. 测试

```
微信公众号：世界上有意思的事

const set = new Set()
set.add('gioia')
set.add('me')

const map = new Map()
map.set(0, 'zero')
map.set(1, 'one')

const original = {
  name: 'gioia',
  getName: function () {
    return this.name
  },
  [Symbol()]: 'symbol prop',
  sym: Symbol('symbol value'),
  friends: [{
    name: 'xkld',
  }, 'cln'],
  dress: {
    pants: {
      color: 'black'
    },
    shirts: {
      colors: ['blue', 'white']
    }
  },
  map: map,
  set: set
}

const copy = deepClone(original)
console.log(copy)
console.log(copy.sym === original.sym)
console.log(copy.friends === original.friends)
console.log(copy.map === original.map)
复制代码
{
  name: 'gioia',
  getName: [Function: getName],
  sym: Symbol(symbol value),
  friends: [ { name: 'xkld' }, 'cln' ],
  dress: { pants: { color: 'black' }, shirts: { colors: [ 'blue', 'white' ] } },
  map: Map { 0 => 'zero', 1 => 'one' },
  set: Set { 'gioia', 'me' },
  [Symbol()]: 'symbol prop'
}

true
false
false
复制代码
```

## 7. 解决循环引用

以上我们已经初步实现了一个深拷贝了。但是在循环引用的场景下，会出现栈溢出的现象。 例如 `original.circle = original` 这种情况，我们要是还递归地赋值的话，就永远也没有尽头🥱
 解决办法就是，看看我们要拷贝的对象之前有没有处理过，有的话就直接引用就行了；没有的话再进行赋值并记录在案。你可以选择很多存储方案，像 Map，只要能记录键值就可以了。

```
微信公众号：世界上有意思的事

function deepClone(value) {
    let result
    
    // 基本类型
    if (!isObject(value)) {
        return value
    }

    const isArr = Array.isArray(value)
    const tag = getTag(value)

    // 数组
    if (isArr) {
        result = initCloneArray(value)
    } else {
        const isFunc = typeof value === 'function'
        // 函数
        if (isFunc) {
            return value
        }
      
        // 对象或 arguments
        if (tag == '[object Object]' || tag == '[object Arguments]') {
            result = initCloneObject(value)
        } else {
            // 特殊对象的初始化
            result = initCloneByTag(value, tag)
        }
    }

    // 检查循环引用并返回其对应的拷贝
    cache || (cache = new Map())
    const cached = cache.get(value)
    if (cached) {
        return cached
    }
    cache.set(value, result)

    // Map 赋值
    if (tag === mapTag) {
        value.forEach((subValue, key) => {
            result.set(key, deepClone(subValue))
        })
        return result
    }

    // Set 赋值
    if (tag === setTag) {
        value.forEach((subValue) => {
            result.add(deepClone(subValue))
        })
      return result
    }

    if (isArr) {
        // 数组赋值
        for (let i = 0; i< value.length; i++) {
            result[i] = deepClone(value[i])
        }
    } else {
        // 对象赋值
        Object.keys(Object(value)).forEach(k => {
            result[k] = deepClone(value[k])
        })

        const propertyIsEnumerable = Object.prototype.propertyIsEnumerable
        
        // 过滤掉不可枚举的 Symbol 属性并赋值
        Object.getOwnPropertySymbols(value)
            .filter((symbol) => propertyIsEnumerable.call(value, symbol))
            .forEach(k => {
                result[k] = deepClone(value[k])
            })
    }

    return result

}
复制代码
```

# 三、写一个简易的打包工具

## 1. 前言

不知道有没有人和我一样，不管看了几遍文档，还是不会自己写 webpack，只能在别人写的配置上修修补补，更别提什么优化了。
 于是我痛定思痛，决定从源头上解决这个问题！为了更好地应用 webpack，我们应该了解它背后的工作原理。
 因此，我阅读了 [miniwebpack](https://github.com/ronami/minipack/blob/master/src/minipack.js) 这个仓库。这个仓库实现了一个最简单的打包工具。接下来我会按照我的理解来解释一下怎么实现一个简单的打包工具

## 2. 主要思路

1. 代码处理。我们平常写代码的时候，用的可能是ES6、ES7等高版本的语法，我们需要将它们转换成浏览器能运行的语法
2. 打包。需要根据一个 entry 来输出一个 output，我们通过维护一个依赖关系图来解决这个问题



![图1：流程图.png](https://user-gold-cdn.xitu.io/2020/5/11/172044237fa8f4bc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 3. 代码处理

1. 解析（parse）。将源代码变成AST。
2. 转换（transform）。操作AST，这也是我们可以操作的部分，去改变代码。
3. 生成（generate）。将更改后的AST，再变回代码。

> 参考：[Babel用户手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-stages-of-babel)

下面我将介绍一些这个过程中需要用到的工具。

### 3.1 解析器 babylon

用来将源代码转换为 AST。
 （不了解 AST 的，可以先看看[在线AST转换器](https://astexplorer.net/)。）

#### 3.1.1 安装

```
npm install --save babylon
复制代码
```

#### 3.1.2  使用

```
import * as babylon from "babylon";

babylon.parse(code, [options])
复制代码
```

### 3.2 转换器 babel-traverse

用来操作 AST

#### 3.2.1 安装

```
npm install --save babel-traverse
复制代码
```

#### 3.2.2 使用

该模块仅暴露出一个 traverse 方法。traverse 方法是一个遍历方法， path 封装了每一个节点，并且还提供容器 container ，作用域 scope 这样的字段。提供个更多关于节点的相关的信息，让我们更好的操作节点。
 示例：

```
// 
import traverse from "babel-traverse";

traverse(ast, {
  enter(path) {
    if (path.node.type === "Identifier"
      && path.node.name === 'text') {
      path.node.name = 'alteredText';
    }
  }
})
复制代码
```

### 3.3 生成器 babel-generator

可以根据 AST 生成代码

#### 3.3.1 安装

```
npm install --save babel-generator
复制代码
```

#### 3.3.2 使用

```
import generate from "babel-generator";

const genCode = generate(ast, {}, code);
复制代码
```

## 4. 实现细节

### 4.1 第一步，提取某文件的依赖

最开始我们提到，需要构建一个依赖关系图。那么我们先从第一步开始，实现根据某个文件（输入绝对路径）提取依赖。大致可以分成以下几步：

1. 读取文件内容
2. 生成 AST
3. 遍历 AST 来理解这个模块依赖哪些模块
4. 为该模块分配唯一标识符
5. 使代码支持所有浏览器

#### 4.1.1 读取文件内容

我们用 node.js 的 `fs` 模块就可以

```
const fs = require('fs');
const content = fs.readFileSync(filename, 'utf-8');
复制代码
```

#### 4.1.2 生成 AST

用到我们之前提到的 babylon

```
const ast = babylon.parse(content, {
  sourceType: 'module',
});
复制代码
```

#### 4.1.3 遍历 AST 来试着理解这个模块依赖哪些模块

这里我们需要操作 AST，所以用到 babel-traverse

```
const dependencies = [];
// 要做到这一点,我们检查`ast`中的每个 `import` 声明.
traverse(ast, {
// `Ecmascript`模块相当简单,因为它们是静态的. 这意味着你不能`import`一个变量,
// 或者有条件地`import`另一个模块. 
// 每次我们看到`import`声明时,我们都可以将其数值视为`依赖性`.
  ImportDeclaration: ({node}) => {
    dependencies.push(node.source.value);
  },
});

复制代码
```

#### 4.1.4 为模块分配唯一标识符

我们简单地用 id 表示

```
// 递增简单计数器
const id = ID++;
复制代码
```

#### 4.1.5 使代码支持所有浏览器

使用 [babel](https://www.babeljs.cn/docs/)

```
const {transformFromAst} = require('babel-core');

// 该`presets`选项是一组规则,告诉`babel`如何传输我们的代码. 
// 我们用`babel-preset-env``将我们的代码转换为浏览器可以运行的东西. 
const {code} = transformFromAst(ast, null, {
  presets: ['env'],
});
复制代码
```

那么 code 到底长什么样呢

> 1. 首先，babel 能将 es6 等更新的代码转成浏览器能执行的低版本代码，这个之前一直在强调的
> 2. 其次，对于模块的转换。Babel 对 ES6 模块转码就是转换成 CommonJS 规范
>     Babel 对于模块输出的转换，就是把所有输出都赋值到 exports 对象的属性上，并加上 ESModule: true 的标识。表示这个模块是由 ESModule 转换来的 CommonJS 输出 输入就是 require

例如，对于以下文件

```
// entry.js
import message from './message.js';

console.log(message);
复制代码
// message.js
import {name} from './name.js';

export default `hello ${name}!`;
复制代码
```

按照上面的规范，转换后的代码大概是这样大概是这样：

```
// entry.js
"use strict";
var _message = require("./message.js");
var _message2 = _interopRequireDefault(_message);
function _interopRequireDefault(obj) { 
  return obj && obj.__esModule ? obj : { default: obj }; 
}
console.log(_message2.default);
复制代码
// message.js
"use strict";

// 加上 ESModule: true 的标识
Object.defineProperty(exports, "__esModule", {
  value: true
});
var _name = require("./name.js");

// 把所有输出都赋值到 exports 对象的属性上
exports.default = "hello " + _name.name + "!"; 
复制代码
```

#### 4.1.6 返回模块信息

```
return {
  id,
  filename,
  dependencies,
  code,
};
复制代码
```

以上，我们就处理好了一个模块。包含着以下 4 项信息

- 模块 id
- 文件的绝对路径
- 该模块的依赖。保存着的是依赖们的相对路径
- 该模块内部代码（浏览器可运行）

### 4.2 第二步，生成依赖图

通过第一步，我们已经能生成某个模块的依赖了。接下来，我们就可以顺藤摸瓜，从入口文件开始，生成入口文件的依赖，再生成入口文件的依赖的依赖，再生成入口文件的依赖的依赖依...（禁止套娃），直到所有模块处理完毕

```
微信公众号：世界上有意思的事

const path = require('path');

// entry 为入口文件的路径
function createGraph(entry) {

  // createAsset 是我们在【第一步，提取某文件的依赖】中实现的函数
  // mainAsset 就是入口模块的信息了
  const mainAsset = createAsset(entry);

  // 使用一个队列，刚开始只有入口模块
  const queue = [mainAsset];

  for (const asset of queue) {
    
    // mapping 用来将【依赖的相对路径】映射到【该依赖的模块 id】
    asset.mapping = {};

    // 这个模块所在的目录. 
    const dirname = path.dirname(asset.filename);

    // 遍历每一个依赖。
    asset.dependencies.forEach(relativePath => {

      // 得到依赖的绝对路径
      const absolutePath = path.join(dirname, relativePath);

      // 得到 child 的模块信息
      const child = createAsset(absolutePath);

      // 将【依赖的相对路径】映射到【该依赖的模块 id】
      // 因为如果不做映射。最终打包到一个文件后，编码时的相对路径就不管用了。我们就没法知道像 require('./child') 这种代码到底应该加载哪一个模块
      asset.mapping[relativePath] = child.id;

      // 把这个子模块也放进队列里面
      queue.push(child);
    });
  }

// 到这一步,队列 就是一个包含目标应用中 每个模块 的数组
// 实际上这个就是我们最终的依赖关系图了
  return queue;
}
复制代码
```

对于以下文件

```
// ./example/entry.js
import message from './message.js';

console.log(message);
复制代码
// ./example/message.js
import {name} from './name.js';

export default `hello ${name}!`;
复制代码
// ./example/name.js
export const name = 'world';
复制代码
```

我们处理后的依赖关系图应该是这样的

```
微信公众号：世界上有意思的事

[{
    id: 0,
    filename: './example/entry.js',
    dependencies: ['./message.js'],
    code: ,// 略
    mapping: {
        './message.js': 1
    }
}, {
    id: 1,
    filename: './example/message.js',
    dependencies: ['./name.js'],
    code: ,// 略
    mapping: {
        './name.js': 2
    }
}, {
    id: 2,
    filename: './example/name.js',
    dependencies: [],
    code: ,// 略
    mapping: {}
}]
复制代码
```

### 4.3 第三步，根据依赖图生成代码

目前，我们已经有了依赖图

```
graph: Module[]

interface Module {
  id: number // 模块id；在【提取某文件的依赖】这一步中我们使用的是一个递增的 id
  filename: string 
  dependencies: Module[]
  code: string // 该模块的代码（经过转换的，能在浏览器中运行） 
  mapping: Record<string, number> // 将依赖的相对路径转换成id。是我们在【生成依赖图】这一步所做的工作
}
复制代码
```

既然已经到了这一步了，就说明我们得处理一下 `code` 了。在【使代码支持所有浏览器】这一步中，我们已经知道了，`code` 是符合 CommonJS 规范的。但CommonJS 中有以下几个东西，是浏览器中没有的：

- require
- module
- exports

那么接下来就是我们自己实现这3个东西！

首先把咱目前的模块信息整合一下：

- mapping 是肯定要的。因为我们模块的被转换后会通过相对路径来调用 require() ，而我们需要知道对应去加载哪个模块

- code 需要稍微改一下。每个模块的作用域应该是独立的。所以我们改成这样：

  ```
  function (require, module, exports) { 
    {code}
  }
  复制代码
  ```

最终把所有这样的模块放在 modules 中，大概是这样：

```
/*
  {0: [
    function (require, module, exports) { 
      {code}
    },
    mapping: {
      './message.js': 1
    }
  ]}
*/
modules: Record<number, [(require, module, exports) => any, Record<string, number>]>
复制代码
```

接下来我们写主程序，我们主程序要做的工作有

1. 实现 `require`, `module`, `exports`
2. 默认调用入口文件
3. 自执行

```
微信公众号：世界上有意思的事

(function(modules) {
  
  function require(id) { 
    // 从 modules 拿到 【执行函数】和【mapping】
    const [fn, mapping] = modules[id];

    // 自己实现的 require，可以根据相对路径加载依赖
    function localRequire(name) { 
      return require(mapping[name]); 
    }
     
    // // 自己实现的 module 和 exports
    const module = { exports : {} };

    fn(localRequire, module, module.exports); 

    return module.exports;
  }

  // 调用入口文件
  require(0);
  
})(modules)
```

