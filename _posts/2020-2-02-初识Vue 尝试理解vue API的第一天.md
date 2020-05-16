---
layout: post
title: 初识Vue 尝试理解vue API的第一天
date: 2020-02-02 
tags: VUE.js   
---

# vue选项/数据

## data

## props

- 类型: `Object | Function`
- 限制: 组件的定义只接收`function`
- 详细:
  - Vue实例的数据对象.Vue将会递归将data的property转换为getter/swetter,从而让data的property能够响应数据变化.**对象必须是纯粹的对象(含有零个或多个key/value对)**  浏览器API创建的原生对象, 原型上的property会被忽略.大概来说,data应该只能是数据- 不推荐观察拥有状态行为的对象.
  - 一旦观察过,你就无法在根数据对象上添加响应式property.因此推荐在创建实例之前,就声明所有的根级响应式property.
  - 创建实例之后,可以通过 `vm.$data` 访问原始数据对象. Vue实例也代理了data对象上所有的property,因此访问`vm.a`等价于访问`vm.$data.a`
  - 以_或 ![开头的property 不会被 Vue实例代理,因为他们可能和Vue内置的peoperty API方法冲突.你可以使用例如`vm.](https://juejin.im/equation?tex=%E5%BC%80%E5%A4%B4%E7%9A%84property%20%E4%B8%8D%E4%BC%9A%E8%A2%AB%20Vue%E5%AE%9E%E4%BE%8B%E4%BB%A3%E7%90%86%2C%E5%9B%A0%E4%B8%BA%E4%BB%96%E4%BB%AC%E5%8F%AF%E8%83%BD%E5%92%8CVue%E5%86%85%E7%BD%AE%E7%9A%84peoperty%20API%E6%96%B9%E6%B3%95%E5%86%B2%E7%AA%81.%E4%BD%A0%E5%8F%AF%E4%BB%A5%E4%BD%BF%E7%94%A8%E4%BE%8B%E5%A6%82%60vm.)data._property`的方式访问这些property.
  - 当一个组件被定义,`ata`必须声明为返回一个初始数据对象的函数,因为组件可能被用来创建多个实例.如果`data`仍然是一个纯粹的对象,则所有的实例将共享引用同一个数据对象! 通过提供`data` 函数, 每次创建一个新实例后, 我们能够调用`data`函数,从而返回初始数据的一个全新副本数据对象.
  - 如果需要,可以通过将`vm.$data`传入`JSON.parse(JSON.stringify(...))` 得到深拷贝的原始数据对象.
- 示例

```
var data = {
            a: 1
        };
        //直接创建一个实例
        var vm = new Vue({
            data: data
        })
        vm.a // =>1
        vm.$data === data;//=>true
        //Vue.extend()中 data必须是函数
        var Component = Vue.extended({
            data() {
                return {
                    a: 1
                }
            },
        })

复制代码
```

- 深入响应式原理[cn.vuejs.org/v2/guide/re…](https://cn.vuejs.org/v2/guide/reactivity.html)

## propsData

- 类型: `Array | Object`

- 详细 : props 可以是数组或对象,用于接受来自父组件的数据.props可以是简单的数组,或者使用对象作为替代,对象允许配置高级选项,如

  类型检测 自定义验证 和设置默认值

  你可以给予对象的语法使用以下选项 :

  - `type` : 可以是下列原生构造函数中的一种:`String Number Boolean Array Object Date Function Symbol` 任何自定义构造函数 或上述内容组成的数组.会检查一个prop是否是给定的
     类型,否则抛出警告.
  - `defalut: any` 为该prop指定一个默认值.如果改prop没有被传入,则换做用这个值.对象或数组的默认值必须从一个工厂函数返回
  - `required : Boolean` 定义该prop是否是必填项.在非生产环境中,如果这个值为truthy且该prop没有被传入的,则一个控制台警告将会被抛出.
  - `validator : Function` 自定义验证函数会将该prop值作为唯一参数带入.在非生产环境下,如果该函数返回一个falsy的值(验证失败),一个控制台警告将会被抛出.
  - 示例

```
    //简单语法
    Vue.component(`props-demo-simple`,{
        props:['size','myMessage' ]
    })
    
    //对象语法,提供验证
    Vue.component('props-demo-advanced',{
        props:{
            //检测类型
            height: Number
            //检测类型+其他验证
            age:{
                type: Number,
                default : 0,
                required : true,
                validator : function (value){
                    return  value =>0
                }
            }
         }
    })
复制代码
```

- 深入了解prop[cn.vuejs.org/v2/guide/co…](https://cn.vuejs.org/v2/guide/components-props.html)

## propsData

- 类型 : `{ [ key: string ] : any }`
- 限制 :  只用于`new` 创建的实例中.
- 详细:
  - 创建实例时传递props.主要作用是方便测试
- 示例:

```
var Comp  =Vue.extend({
    props:[ ' msg ' ],
    template: ' <div> {{ msg }} </div> ' 
})

var vm  = new Comp( { 
    propsData :｛
        msg : ' hello ' 
    ｝
})
复制代码
```

## computed

- 类型 : `{ [ key : string ] : Function ｜ { get : Function, set : Function }  }`

- 详细 :

  - 计算属性将被混入到Vue实例中. 所有getter和setter的this上下文自动地绑定为Vue实例.
  - 注意如果你为一个计算属性使用了箭头函数,则`this` 不会指向这个组件的实例, 不过你仍然可以将其实例在uowei函数的第一个参数来访问.

  ```
  computed: { 
  aDouble : vm => vm.a&emsp;*&emsp;２
  }  
  复制代码
  ```

  ＊计算属性的结果会被缓存，除非依赖的响应式property变化才会重新计算. 注意,如果某个依赖(比如响应式property)在该实例范畴之外,则计算属性不会被更新的

  - 示例

```
var vm = new Vue({
            data: {
                a: 1
            },
            computed: {
                //仅读取
                aDouble: function() {
                    return this.a * 2
                },
                // 读取和设置
                aPlus: {
                    get: function() {
                        return this.a + 1
                    },
                    set: function() {
                        this.a = v - 1
                    }
                }
            },
        })
        vm.aPlus //2
        vm.aPlus //3
        vm.a //2
        vm.avm.aDouble //2
```