---
layout: post
title: Vue中使用provide和inject
date: 2020-05-16 
tags: VUE.js   
---

# 

相信大家在工作中一定遇到过多层嵌套组件，而vue 的组件数据通信方式又有很多种。

比如**vuex、$parent与$children、prop、$emit与$on、$attrs与$lisenters、eventBus、ref**。

今天主要为大家分享的是`provide`和`inject`。

很多人会问，那我直接使用vuex不就行了吗？

vuex固然是好！

但是，有可能项目本身并没有使用vuex的必要，这个时候`provide`和`inject`就闪亮登场啦～



**使我们开发的时候，如有神助～**



## 官方解释

### provide

选项应该是一个对象或返回一个对象的函数。该对象包含可注入其子孙的`property`。

### inject

可以是一个字符串数组、也可以是一个对象

说白了，就是`provide`在祖先组件中注入，`inject` 在需要使用的地方引入即可。

我们可以把依赖注入看做一部分大范围的`prop`，只不过它以下特点：

- 祖先组件不需要知道哪些后代组件使用它提供的属性
- 后代组件不需要知道被注入的属性是来自那里

### 注意：provide 和 inject 绑定并不是可响应的。这是刻意为之的。然而，如果你传入了一个可监听的对象，那么其对象的 property 还是可响应的。

## 实例

### 目录结构

![img](https://user-gold-cdn.xitu.io/2020/5/13/1720d33252dc91c8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 祖先

index.vue

```
<template>
<div class="grandPa">
  爷爷级别 : <strong>{{ nameObj.name }} 今年 <i class="blue">{{ age }}</i>岁， 城市<i class="yellow">{{ city }}</i></strong>
  <child />
  <br>
  <br>
  <el-button type="primary" plain @click="changeName">改变名称</el-button>

</div>
</template>

<script>
import child from '@/components/ProvideText/parent'
export default {
name: 'ProvideGrandPa',
components: { child },
data: function() {
  return {
    nameObj: {
      name: '小布'
    },
    age: 12,
    city: '北京'
  }
},
provide() {
  return {
    nameObj: this.nameObj,   //传入一个可监听的对象
    cityFn: () => this.city,  //通过computed来计算注入的值
    age: this.age  //直接传值

  }
},
methods: {
  changeName() {
    if (this.nameObj.name === '小布') {
      this.nameObj.name = '貂蝉'
      this.city = '香港'
      this.age = 24
    } else {
      this.nameObj.name = '小布'
      this.city = '北京'
      this.age = 12
    }
  }
}
}
</script>

<style lang="scss" scoped>
.grandPa{
width: 600px;
height:100px;
line-height: 100px;
border: 2px solid  #7fffd4;
padding:0 10px;
text-align: center;
margin:50px auto;
strong{
  font-size: 20px;
  text-decoration: underline;;
}
.blue{
    color: blue;
}
}
</style>

复制代码
```

### 中间组件

parent.vue

```
<template>
<div class="parent">
  父亲级别 : <strong>只用作中转</strong>
  <son />
</div>
</template>

<script>
import Son from './son'
export default {
name: 'ProvideParent',
components: { Son }
}
</script>

<style lang="scss" scoped>
.parent{
height:100px;
line-height: 100px;
border: 2px solid  #feafef;
padding:0 10px;
margin-top: 20px;
strong{
  font-size: 20px;
  text-decoration: underline;;
}

}
</style>

复制代码
```

### 后代组件

son.vue

```
<template>
<div class="son">
  孙子级别 : <strong>{{ nameObj.name }} 今年 <i class="blue">{{ age }}</i>岁， 城市<i class="yellow">{{ city }}</i></strong>
</div>
</template>

<script>
export default {
name: 'ProvideSon',
//inject 来获取的值
inject: ['nameObj', 'age', 'cityFn'],
computed: {
  city() {
    return this.cityFn()
  }
}
}
</script>

<style lang="scss" scoped>
.son{
  height:100px;
  line-height: 100px;
  padding:0 10px;
  margin: 20px;
  border: 1px solid #49e2af;
strong{
  font-size: 20px;
  text-decoration: underline;;
}
.blue{
    color: blue;
}
}
</style>


复制代码
```

我们来看一下运行结果。

图一：未点击【改变名称】按钮，原有状态

![img](https://user-gold-cdn.xitu.io/2020/5/13/1720d33252c227fd?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

图二：已经点击【改变名称】按钮，更新后状态 ![img](https://user-gold-cdn.xitu.io/2020/5/13/1720d33253387994?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

大家可以对比一下前后差异。

会发现一个小细节。

无论我点击多少次，孙子组件的年龄`age`字段永远都是`12`并不会发生变化。

正是官网所提到的`provide` 和 `inject` **绑定并不是可响应的。这是刻意为之的。**

所以大家使用的时候，一定要注意注入的方式，不然很可能无法实现数据响应。