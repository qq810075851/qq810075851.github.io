---
layout: post
title: "JavaScript设计模式！"
date: 2019-07-15 
tags: JavaScript  
---

## 　　构造函数模式

### 简介

在JavaScript里，构造函数通常是认为用来实现实例的特殊的构造函数。通过new关键字来调用定义的构造函数，你可以告诉JavaScript你要创建一个新对象并且新对象的成员声明都是构造函数里定义的。在构造函数内部，this关键字引用的是新创建的对象。

作为一个老联盟fans，一定要亲手实现一下设计模式也可以融会贯通。

现在打算创建一个英雄联盟对象，需要地图，英雄，士兵，野怪，还有开始游戏的按钮。



![img](https://user-gold-cdn.xitu.io/2020/5/15/17213ef849cbebc2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



```
function LOL(maps, heros, soldier, monster) {
    this.maps = maps
    this.heros = heros
    this.soldier = soldier
    this.monster = monster
    this.start = function() {
        return '地图:' + this.maps + '\n对战英雄:' + this.heros.join() + '\n小兵类型:' + this.soldier + '\n野怪:' + this.monster + '\n'
    }
}


var game1 = new LOL('召唤师峡谷', ['影流之主', '诡术妖姬'], '超级兵', '红buff')
var game2 = new LOL('大乱斗', ['影流之主', '诡术妖姬'], '超级兵', '红buff')
console.log(game1.start())
console.log(game2.start())

复制代码
```

这样写代码，每局游戏需要重新创建一个英雄联盟实例，

这样使用构造器，有多少个game就需要多少个start函数方法，如果共用一个start方法，可以节约很多内存

```
function LOL(maps, heros, soldier, monster) {
    this.maps = maps
    this.heros = heros
    this.soldier = soldier
    this.monster = monster
}
LOL.prototype.start = function() {
	 	return '地图:' + this.maps + '\n对战英雄:' + this.heros.join() + '\n小兵类型:' + this.soldier + '\n野怪:' + this.monster + '\n'
}


var game1 = new LOL('召唤师峡谷', ['影流之主', '诡术妖姬'], '超级兵', '红buff')
var game2 = new LOL('大乱斗', ['影流之主', '诡术妖姬'], '超级兵', '红buff')

console.log(game1.start())
console.log(game2.start())
复制代码
```

如果让`start`方法变成大家通用的就好了，因此把`LOL.prototype.start`改写，这样所以的LOL实例就可以共用一个方法，从原型链上继承即可

上面的方式可以节省内存，`start`实例函数可以在所有LOL对象的实例中使用

如果不使用new 也可以有其他方式创建对象

```
function LOL(maps, heros, soldier, monster) {
    this.maps = maps
    this.heros = heros
    this.soldier = soldier
    this.monster = monster
    this.start = function() {
        return '地图:' + this.maps + '\n对战英雄:' + this.heros.join() + '\n小兵类型:' + this.soldier + '\n野怪:' + this.monster + '\n'
    }
}


var game3 = new Object();
LOL.call(game3, "扭曲丛林", ['影流之主', '剑圣'], '远程兵', '大龙');
console.log(game3.start())

//也可以不使用new ，通过call方法在game3的作用域调用LOL

复制代码
```

这种方式虽然可以创建新的构造函数，但却不能继承LOL原型上的函数

如果直接运行LOL()函数（不使用new的情况下），由于this指向的是window对象，因此start方法会变成`window.start()`

如果强制要求函数使用new 方法也可以如下创建:

```
function LOL(maps, heros, soldier, monster) {
    if (!(this instanceof LOL)) {
        return new LOL(maps, heros, soldier, monster);
    }
    this.maps = maps
    this.heros = heros
    this.soldier = soldier
    this.monster = monster
}
复制代码
```

通过判断this的`instanceof`，就可以知道究竟是来自`new`方法，还是说是直接调用。如果是直接调用的话，判断条件为`true`，还是会return一个新的实例。

e.g:

```
var s = new String("lol");
var n = new Number(101);
var b = new Boolean(true);
// s n b返回的是实例对象 
//String{
//	0: 'l',
//	1: 'o',
//	2: 'l'
//	}

//可以直接给变量赋值，或者不加new 关键词
复制代码
```

## 外观模式

### 简介

外观模式最大的体现其实就是入口，比如`init()`函数，把一些内部的函数都放在这个门面之下，只需要调用这个门面函数，其他乱七八糟的功能都可以实现。

现在有一个英雄，叫做亚索，我希望给他一些配置，比如技能，衣服等



![img](https://user-gold-cdn.xitu.io/2020/5/15/17213f1399afdf68?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



```
function YaSuo() {
}
function Qskill (hero) {
    hero.prototype.Qskill = function() {
        console.log('hasaki!!')
    }
}
function Wskill (hero) {
    hero.prototype.Wskill = function() {
        console.log('风墙')
    }
}
function Eskill (hero) {
    hero.prototype.Eskill = function() {
        console.log('快乐')
    }
}
function Rskill (hero) {
    hero.prototype.Rskill = function() {
        console.log('痛里唉该痛')
    }
}
function Skin (hero, skin) {
    hero.prototype.skin = skin
}

function CreateYasuo () {
    Qskill(YaSuo)
    Wskill(YaSuo)
    Eskill(YaSuo)
    Rskill(YaSuo)
    Skin(YaSuo, 'originSkin')
    return new YaSuo()
}
CreateYasuo()
// 创建成功 外观模式启动
复制代码
```

通过上面的代码，成功创建了一个美丽的亚索。我们最后只需要了解，**外观模式不仅简化类中的接口，而且对接口与调用者也进行了解耦。外观模式经常被认为开发者必备，它可以将一些复杂操作封装起来，并创建一个简单的接口用于调用**。

说白了就是用一个接口封装其它的接口。

外观模式优点就是易使用。缺点则是，当连续使用外观模式创建的接口时，可能会产生性能问题。

e.g.

```
var addMyEvent = function (el, ev, fn) {
    if (el.addEventListener) {
        el.addEventListener(ev, fn, false);
    } else if (el.attachEvent) {
        el.attachEvent('on' + ev, fn);
    } else {
        el['on' + ev] = fn;
    }
}; 
复制代码
```

这是最常见对监听事件的处理，前端必会。其中的`addMyEvent`就是对其他三个接口的封装，产生了一个门面，也就是外观模式。

## 代理模式

### 简介

其实代理模式我们生活中接触的很多了。比如es6中的`proxy`对象，还有我们平时上网用的VPN。那其实代理模式，就是让一个对象帮助其他的对象来做事。

比如我现在想创建一个英雄，名字叫做卡莉斯塔，俗称滑板鞋。这个英雄有个特点，当她放R技能的时候，会把一个对象拉过来到自己身边几秒，代理这个对象的走路行为，禁止他释放技能等等，那这就要用到代理模式了。

```
// 声明走路动作
function Walk (hero) { // 代理期间执行的操作
    return function() {console.log(hero + ' is walk')}
}
function Kalisita () { // proxy
    this.walk = Walk('Kalisita')
    this.Rskill = function(hero) { // 传入要拉取的英雄
        this.walk = function() {
            Walk('Kalisita')() // 既需要自己走
            hero.walk() // 还需要带着人一起走
        }
    }
}
function HeroA () { // 被代理走路的英雄
    this.walk = Walk('heroA')
}

var k = new Kalisita()
var a = new HeroA()

k.walk() // Kalisita is walk
a.walk() // heroA is walk

k.Rskill(a) // k把a的walk事件代理了, 现在k触发walk的同时，也会带着a一起walk哦
k.walk()
// Kalisita is walk 
// heroA is walk

复制代码
```

代理模式主要用于几点，

- 要获取本来没有的对对象操作的权限
- 要获取远程文件，需要代理模式作为跳板
- 根据需要创建开销很大的对象，通过它来存放实例化需要很长时间的真实对象，比如浏览器的渲染的时候先显示问题，而图片可以慢慢显示（就是通过虚拟代理代替了真实的图片，此时虚拟代理保存了真实图片的路径和尺寸。
- 只当调用真实的对象时，代理处理另外一些事情。例如C#里的垃圾回收，使用对象的时候会有引用次数，如果对象没有引用了，GC就可以回收它了。

> ```
> 详情可以参考《大话设计模式》
> ```

而我们前端代码中用的比较多的，应该就是`vue.js`中对data中数据响应式的代理。`vue3`中也将使用大量`ES6支持的Proxy对象`来改写。

e.g.

通过代理，尝试设置私有属性

```
function getPrivateProps(obj, filterFunc) {
  return new Proxy(obj, {
    get(obj, prop) {
      if (!filterFunc(prop)) {
        let value = Reflect.get(obj, prop);
        // 如果是方法, 将this指向修改原对象
        if (typeof value === 'function') {
          value = value.bind(obj);
        }
        return value;
      }
    },
    set(obj, prop, value) {
      if (filterFunc(prop)) {
        throw new TypeError(`Cant set property ${prop}`);
      }
      return Reflect.set(obj, prop, value);
    },
    has(obj, prop) {
      return filterFunc(prop) ? false : Reflect.has(obj, prop);
    },
    ownKeys(obj) {
      return Reflect.ownKeys(obj).filter(prop => !filterFunc(prop));
    },
    getOwnPropertyDescriptor(obj, prop) {
      return filterFunc(prop) ? undefined : Reflect.getOwnPropertyDescriptor(obj, prop);
    }
  });
}

function propFilter(prop) {
  return prop.indexOf('_') === 0;
}
复制代码
```

## 总结

本次分享了三种设计模式，分别为构造函数模式，外观模式，代理模式；都是日常开发很常用的设计模式。

其中[es6proxy](https://es6.ruanyifeng.com/?search=proxy&x=0&y=0#docs/proxy)可以访问阮老师博客查看详细api，具体使用也可以借鉴vue源码。代理模式虽然很好用，但也不是任何时候都建议使用，如果你的代码需要获得某些对象的权限，不妨可以使用一下代理模式。在比较简单的场景，可能就没有必要了。

外观模式是最常用的，毕竟每一个js文件总是需要一个入口的。无论是`main`函数还是`init`函数，都是起到一个外观包装的作用。当然外观模式并不是必须作为一个文件入口存在，只要能把重复的代码提炼出来，就是一个合理的外观模式。

构造函数模式就不多说了，简单好用。

> - 复制代码是危险的。如果有两段相同的代码，几乎可以说一定是有问题的，因为每次改动，要维护两段代码
> - 尽量减少IO操作，如操作数据库，网络发送，甚至printf ，这些操作比直接操作内存，慢很多倍、
> - 修改Bug时，一定要从最简单的基本的地方开始检查，不要检查到最底层没问题，发现是传入的某个参数是错的。先不要怀疑系统的部分。
> - 设计架构，同时了解细节，有些Bug,调起来可能费时费力，甚至花个二三天，其实当时写的时候，只要稍微注意，就可以轻松避免。避免Bug的代价与找出并修改Bug的代价，实在是差太多了。
> - 把一段长代码，分成很多小函数，便于维护，连自己都不愿看，不愿改的代码，百分百有问题。
> - 写程序时，先把流程搞清楚。把各个流程用的函数写清楚，函数可以留空，这样编程就变成了填空题。
> - 做新功能时，把数据结构的设计，放在较重要的位置
