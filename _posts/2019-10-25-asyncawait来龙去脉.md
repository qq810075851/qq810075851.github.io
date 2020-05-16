---
layout: post
title: async/await的来龙去脉
date: 2019-10-25 
tags: ES6
---

### babel是如何来实现的

注：对于generator不了解的，可以先去看一下generator，顺带可以把iterator看了。

ex代码：

```
async function t() {
    const x = await getResult();
  	const y = await getResult2();
  	return x + y;
}
复制代码
```

<details>
<summary>babel转化代码</summary>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-meta">"use strict"</span>;

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">asyncGeneratorStep</span>(<span class="hljs-params">gen, resolve, reject, _next, _throw, key, arg</span>) </span>{
    <span class="hljs-keyword">try</span> {
        <span class="hljs-keyword">var</span> info = gen[key](arg);
        <span class="hljs-keyword">var</span> value = info.value;
    } <span class="hljs-keyword">catch</span> (error) {
        reject(error);
        <span class="hljs-keyword">return</span>;
    }
    <span class="hljs-keyword">if</span> (info.done) {
        resolve(value);
    } <span class="hljs-keyword">else</span> {
        <span class="hljs-built_in">Promise</span>.resolve(value).then(_next, _throw);
    }
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">_asyncToGenerator</span>(<span class="hljs-params">fn</span>) </span>{
    <span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">var</span> self = <span class="hljs-keyword">this</span>, args = <span class="hljs-built_in">arguments</span>;
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">resolve, reject</span>) </span>{
            <span class="hljs-keyword">var</span> gen = fn.apply(self, args);
            <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">_next</span>(<span class="hljs-params">value</span>) </span>{
                asyncGeneratorStep(gen, resolve, reject, _next, _throw, <span class="hljs-string">"next"</span>, value);
            }
            <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">_throw</span>(<span class="hljs-params">err</span>) </span>{
                asyncGeneratorStep(gen, resolve, reject, _next, _throw, <span class="hljs-string">"throw"</span>, err);
            }
            _next(<span class="hljs-literal">undefined</span>);
        });
    };
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">t</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">return</span> _t.apply(<span class="hljs-keyword">this</span>, <span class="hljs-built_in">arguments</span>);
}

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">_t</span>(<span class="hljs-params"></span>) </span>{
  _t = _asyncToGenerator(<span class="hljs-function"><span class="hljs-keyword">function</span>* (<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">const</span> x = <span class="hljs-keyword">yield</span> getResult();
    <span class="hljs-keyword">const</span> y = <span class="hljs-keyword">yield</span> getResult2();
    <span class="hljs-keyword">return</span> x + y;
  });
  <span class="hljs-keyword">return</span> _t.apply(<span class="hljs-keyword">this</span>, <span class="hljs-built_in">arguments</span>);
}
<span class="copy-code-btn">复制代码</span></code></pre></details>

从代码中可以看出，babel将一个generator转化为async用了两步`_asyncToGenerator`和`asyncGeneratorStep`。

#### `_asyncToGenerator`干了什么

1、调用`_asyncToGenerator`返回了一个promise，刚好符合async函数可以接then的特性。

2、定义了一个成功的方法`_next`，定义了一个失败的方法`_throw`。两个函数中是调用`asyncGeneratorStep`。看完`asyncGeneratorStep`就知道这其实是一个递归。

3、执行`_next`。也就是上面说的自执行的generator。

#### `asyncGeneratorStep`干了什么

1、try-catch去捕获generator执行过程中的错误。如果有报错，async函数直接是reject状态。

2、判断info中的done值，是否为true，为true就代表迭代器已经执行完毕了，可以将value值resolve出去。反之，则继续调用`_next`将值传递到下一个去。

> 这里我唯一没有看明白的是`_throw`，这个看代码像是执行不到的。promise.resolve状态值应该是fulfilled。看懂的可以在评论中和我说一下，感谢。

### async/await的优势

每当一种新的语法糖出现，必定是弥补上一代解决方案的缺陷。

**ex：**

promise的出现，是为了去避免`callback hell`，避免的方式就是链式调用。

**那async/await为了去解决什么呢？**

------

### 用async/await去替换掉promise的几点必要性

#### 同步的方式处理异步

async/await更贴近于同步性的风格，而promise则是用then的方式，于async/await相比，代码会变多，而且async/await和同步函数差别不大，promise则写法上还是有差距的。

<details>
<summary>promise和async/await代码对比</summary>
<p>promise版</p>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getData</span>(<span class="hljs-params"></span>) </span>{
    getRes().then(<span class="hljs-function">(<span class="hljs-params">res</span>) =&gt;</span> {
        <span class="hljs-built_in">console</span>.log(res);
    })
}
<span class="copy-code-btn">复制代码</span></code></pre><p>async/await版</p>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-keyword">const</span> getData = <span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">const</span> res = <span class="hljs-keyword">await</span> getRes();
    <span class="hljs-built_in">console</span>.log(res);
}
<span class="copy-code-btn">复制代码</span></code></pre></details>

#### 中间值

用promise的时候会发现，多个promise串行的时候，后面的promise需要去获取前面promise的值是非常困难的。而async恰好解决了这个点。

<details>
<summary>promise获取中间值的例子</summary>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-keyword">const</span> morePromise = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
	<span class="hljs-keyword">return</span> promiseFun1().then(<span class="hljs-function">(<span class="hljs-params">value1</span>) =&gt;</span> {
		<span class="hljs-keyword">return</span> promiseFun2(value1).then(<span class="hljs-function">(<span class="hljs-params">value2</span>) =&gt;</span> {
			<span class="hljs-keyword">return</span> promiseFun3(value1, value2).then(<span class="hljs-function">(<span class="hljs-params">res</span>) =&gt;</span> {
				<span class="hljs-built_in">console</span>.log(res);
			})
		}) 
	})
}
<span class="copy-code-btn">复制代码</span></code></pre><p>上面是嵌套版本的，可能根据不同的需求可以不嵌套的。</p>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-keyword">const</span> morePromise = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
	<span class="hljs-keyword">return</span> promiseFun1().then(<span class="hljs-function">(<span class="hljs-params">value1</span>) =&gt;</span> {
		<span class="hljs-keyword">return</span> promiseAll([value1, promiseFun2(value1)])
	}).then(<span class="hljs-function">(<span class="hljs-params">[value1, value2]</span>) =&gt;</span> {
		<span class="hljs-keyword">return</span> promiseFun3(value1, value2).then(<span class="hljs-function">(<span class="hljs-params">res</span>) =&gt;</span> {
			<span class="hljs-built_in">console</span>.log(res);
		})
	})
}
<span class="copy-code-btn">复制代码</span></code></pre><p>少了嵌套层级，但是还是不尽如人意。</p>
</details>

<details>
<summary>用async/await优化例子</summary>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-keyword">const</span> morePromise = <span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
	<span class="hljs-keyword">const</span> value1 = <span class="hljs-keyword">await</span> promiseFun1();
	<span class="hljs-keyword">const</span> value2 = <span class="hljs-keyword">await</span> promiseFun2(value1);
	<span class="hljs-keyword">const</span> res = <span class="hljs-keyword">await</span> promiseFun3(value1, valuw2);
	<span class="hljs-keyword">return</span> res;
}
<span class="copy-code-btn">复制代码</span></code></pre></details>

串行的异步流程，必定会有中间值的牵扯，所以async/await的优势就很明显了。

#### 条件语句的情况

比如，目前有个需求，你请求完一个数据之后，再去判断是否还需要请求更多数据。用promise去实现还是会出现嵌套层级。

<details>
<summary>例子</summary>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-keyword">const</span> a = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">return</span> getResult().then(<span class="hljs-function">(<span class="hljs-params">data</span>) =&gt;</span> {
        <span class="hljs-keyword">if</span>(data.hasMore) {
            <span class="hljs-keyword">return</span> getMoreRes(data).then(<span class="hljs-function">(<span class="hljs-params">dataMore</span>) =&gt;</span> {
                <span class="hljs-keyword">return</span> dataMore;
            })
        } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">return</span> data;
        }
    })
}
<span class="copy-code-btn">复制代码</span></code></pre></details>

但是用async去优化这个例子的话，能使代码更加优美。

<details>
<summary>async/await优化例子</summary>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-keyword">const</span> a = <span class="hljs-keyword">async</span>() =&gt; {
    <span class="hljs-keyword">const</span> data = <span class="hljs-keyword">await</span> getResult();
    <span class="hljs-keyword">if</span>(data.hasMore) {
        <span class="hljs-keyword">const</span> dataMore = <span class="hljs-keyword">await</span> getMoreRes(data);
        <span class="hljs-keyword">return</span> dataMore;
    } <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">return</span> data;
    }
}
<span class="copy-code-btn">复制代码</span></code></pre></details>

### async/await的劣势

上面我们讲了几点async/await的优势所在，但是async/await也不是万能的。上面清一色的是讲串联异步的场景。当我们变成并联异步场景时。**还是需要借助于promise.all来实现**

<details>
<summary>并联异步的场景</summary>
<pre><code class="hljs js copyable" lang="js"><span class="hljs-keyword">const</span> a = <span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">const</span> res = <span class="hljs-keyword">await</span> <span class="hljs-built_in">Promise</span>.all[getRes1(), getRes2()];
    <span class="hljs-keyword">return</span> res;
}
<span class="copy-code-btn">复制代码</span></code></pre></details>

### async/await的错误处理

async/await在错误捕获方面主要使用的是try-catch。

#### try-catch

```
const a = async () => {
    try{
        const res = await Promise.reject(1);
    } catch(err) {
        console.log(err);
    }
}
复制代码
```

#### promise的catch

可以抽离一个公共函数来做这件事情。因为每个promise后面都去做catch的处理，代码会写的很冗长。

```
const a = async function() {
    const res = await Promise.reject(1).catch((err) => {
        console.log(err);
    })
}
复制代码
// 公共函数

function errWrap(promise) {
    return promise().then((data) => {
        return [null, data];
    }).catch((err) => {
        return [err, null];
    })
}
```

