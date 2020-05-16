---
layout: post
title: JavaScript版数据结构与算法——基础篇
date: 2019-11-16 
tags: JavaScript    
---

## 数组

**数组**——最简单的内存数据结构

数组存储一系列同一种数据类型的值。（ `Javascript` 中不存在这种限制）

对数据的随机访问，数组是更好的选择，否则几乎可以完全用 「链表」 来代替

> 在很多编程语言中，数组的长度是固定的，当数组被填满时，再要加入新元素就很困难。`Javascript` 中数组不存在这个问题。

> 但是 `Javascript` 中的数组被实现成了对象，与其他语言相比，效率低下。

**数组的一些核心方法**

| 方法        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| push        | 方法将一个或多个元素添加到数组的末尾，并`返回该数组的新长度`。(`改变原数组`) |
| pop         | 方法从数组中删除最后一个元素，并`返回该元素的值`。(`改变原数组`) |
| shift       | 方法从数组中删除第一个元素，并`返回该元素`的值，如果数组为空则返回 `undefined` 。(`改变原数组`) |
| unshift     | 将一个或多个元素添加到数组的开头，并返回该数组的`新长度`(`改变原数组`) |
| concat      | 连接两个或多个数组，并返回结果（`返回一个新数组，不影响原有的数组`。） |
| every       | 对数组中的每个元素运行给定函数，如果该函数对`每个元素`都返回 `true`，则返回 `true`。**若为一个空数组，，始终返回 true。** （`不会改变原数组`，`[].every(callback)`始终返回 `true`） |
| some        | 对数组中的每个元素运行给定函数，如果`任一元素`返回  true，则返回 true。**若为一个空数组，，始终返回 false。**（`不会改变原数组`，） |
| forEach     | 对数组中的每个元素运行给定函数。这个方法`没有返回值`，没有办法中止或者跳出 `forEach()` 循环，除了抛出一个异常（`foreach不直接改变原数组`，但原数组可能会被 callback 函数该改变。） |
| map         | 对数组中的每个元素运行给定函数，返回每次函数调用的结果组成的数组（`map不直接改变原数组`，但原数组可能会被 callback 函数该改变。） |
| sort        | 按照`Unicode位点`对数组排序，支持传入指定排序方法的函数作为参数（`改变原数组`） |
| reverse     | 方法将数组中元素的位置颠倒，并返回该数组（`改变原数组`）     |
| join        | 将所有的数组元素连接成一个字符串                             |
| indexOf     | 返回第一个与给定参数相等的数组元素的索引，没有找到则返回 -1  |
| lastIndexOf | 返回在数组中搜索到的与给定参数相等的元素的索引里最大的值，没有找到则返回 -1 |
| slice       | 传入索引值，将数组里对应索引范围内的元素（`浅复制原数组中的元素`）作为新数组返回（`原始数组不会被改变`） |
| splice      | 删除或替换现有元素或者原地添加新的元素来修改数组,并以数组形式返回被修改的内容(`改变原数组`) |
| toString    | 将数组作为字符串返回                                         |
| valueOf     | 和 `toString` 类似，将数组作为字符串返回                     |

## 栈

> **栈**是一种遵循**后进先出（LIFO）\**原则的有序集合，新添加或待删除的元素都保存在栈的同一端，称作\**栈顶**，另一端就叫**栈底**。在栈里，新元素都靠近栈顶，旧元素都接近栈底。

通俗来讲，就是你向一个桶里放书本或者盘子，你要想取出最下面的书或者盘子，你必须要先把上面的都先取出来。

栈也被用在编程语言的编译器和内存中保存变量、方法调用等，也被用于浏览器历史记录 (浏览器的返回按钮)。

代码实现

```
// 封装栈
    function Stack() {
        // 栈的属性
        this.items = []

        // 栈的操作
        // 1.将元素压入栈
        Stack.prototype.push = function (element) {
            this.items.push(element)
        }
        // 2.从栈中取出元素
        Stack.prototype.pop = function () {
            return this.items.pop()
        }
        // 3.查看栈顶元素
        Stack.prototype.peek = function () {
            return this.items[this.items.length - 1]
        }
        // 4.判断是否为空
        Stack.prototype.isEmpty = function () {
            return this.items.length === 0
        }
        // 5.获取栈中元素的个数
        Stack.prototype.size = function () {
            return this.items.length
        }
        // 6.toString()方法
        Stack.prototype.toString = function () {
            let str = ''
            for (let i = 0; i< this.items.length; i++) {
                str += this.items[i] + ' '
            }
            return str
        }

    }

    // 栈的使用
    let s = new Stack()
复制代码
```

## 队列

> **队列**是遵循**先进先出**(FIFO，也称为先来先服务)原则的一组有序的项。队列在尾部添加新 元素，并从顶部移除元素。最新添加的元素必须排在队列的末尾。

生活中常见的就是排队

代码实现

```
function Queue() {
        this.items = []
        // 1.将元素加入队列
        Queue.prototype.enqueue = function (element) {
            this.items.push(element)
        }
        // 2.从队列前端删除元素
        Queue.prototype.dequeue = function () {
            return this.items.shift()
        }
        // 3.查看队列前端元素
        Queue.prototype.front = function () {
            return this.items[0]
        }
        // 4.判断是否为空
        Queue.prototype.isEmpty = function () {
            return this.items.length === 0
        }
        // 5.获取队列中元素的个数
        Queue.prototype.size = function () {
            return this.items.length
        }
        // 6.toString()方法
        Queue.prototype.toString = function () {
            let str = ''
            for (let i = 0; i< this.items.length; i++) {
                str += this.items[i] + ' '
            }
            return str
        }
    }
    
    // 队列使用
    let Q = new Queue()
复制代码
```

## 优先级队列：

代码实现

```
function PriorityQueue() {
        function QueueElement(element, priority) {
            this.element = element
            this.priority = priority
        }
        this.items = []

        PriorityQueue.prototype.enqueue = function (element, priority) {
            let queueElement = new QueueElement(element,priority)

            // 判断队列是否为空
            if (this.isEmpty()) {
                this.items.push(queueElement)
            } else {
                let added = false // 如果在队列已有的元素中找到满足条件的，则设为true，否则为false，直接插入队列尾部
                for (let i = 0; i< this.items.length; i++) {
                    // 假设priority值越小，优先级越高，排序越靠前
                    if (queueElement.priority < this.items[i].priority) {
                        this.items.splice(i, 0, queueElement)
                        added = true
                        break
                    }
                }
                if (!added) {
                    this.items.push(queueElement)
                }
            }

        }
        
    }
    
复制代码
```

## 链表

**链表**——存储`有序的元素集合`，但在内存中`不是连续放置`的。

链表(单向链表)中的`元素`由存放元素`本身「data」` 的节点和一个`指向下一个「next」` 元素的指针组成。**`牢记这个特点`**

相比数组，链表添加或者移除元素不需要移动其他元素，但是需要使用`指针`。访问元素每次都需要从`表头`开始查找。

代码实现: 单向链表

```
function LinkedList() {
        function Node(data) {
            this.data = data
            this.next = null

        }
        this.head = null // 表头
        this.length = 0
        // 插入链表
        LinkedList.prototype.append = function (data) {
            // 判断是否是添加的第一个节点
            let newNode = new Node(data)
            if (this.length == 0) {
                this.head = newNode
            } else {
                let current = this.head
                while (current.next) { 
                // 如果next存在，
                // 则当前节点不是链表最后一个
                // 所以继续向后查找
                    current = current.next
                }
                // 如果next不存在
                 // 则当前节点是链表最后一个
                // 所以让next指向新节点即可
                current.next = newNode
            }
            this.length++
        }
        // toString方法
        LinkedList.prototype.toString = function () {
            let current = this.head
            let listString = ''
            while (current) {
                listString += current.data + ' '
                current = current.next
            }
            return listString
        }
         // insert 方法
        LinkedList.prototype.insert = function (position, data) {
            if (position < 0 || position > this.length) return false
            let newNode = new Node(data)
            if (position == 0) {
                newNode.next = this.head
                this.head = newNode
            } else {
                let index = 0
                let current = this.head
                let prev = null
                while (index++ < position) {
                    prev = current
                    current = current.next
                }
                newNode.next = current
                prev.next = newNode
            }
            this.length++
            return true
        }
        // get方法
        LinkedList.prototype.get = function (position) {
            if (position < 0 || position >= this.length) return null
            let index = 0
            let current = this.head
            while (index++ < position){
                current = current.next
            }
            return current.data
        }
        LinkedList.prototype.indexOf = function (data) {
            let index = 0
            let current = this.head
            while (current) {
                if (current.data == data) {
                    return index
                } else {
                    current = current.next
                    index++
                }
            }

            return  -1
        }
        LinkedList.prototype.update = function (position, data) {
            if (position < 0 || position >= this.length) return false
            let index = 0
            let current = this.head
            while (index++ < position) {
                current = current.next
            }
            current.data = data
            return  true
        }
        LinkedList.prototype.removeAt = function (position) {
            if (position < 0 || position >= this.length) return null
            if (position == 0) {
                this.head = this.head.next
            } else {
                let index = 0
                let current = this.head
                let prev = null
                while (index++ < position) {
                    prev = current
                    current = current.next
                }
                prev.next = current.next
            }
            this.length--
            return  true


        }
        LinkedList.prototype.remove = function (data) {
            let postions = this.indexOf(data)

            return this.removeAt(postions)
        }
        
    }

    let list = new LinkedList()
复制代码
```

双向链表：包含`表头`、`表尾` 和 存储数据的 `节点`，其中`节点`包含三部分：一个链向下一个元素的`next`， 另一个链向前一个元素的`prev` 和存储数据的 `data`。**`牢记这个特点`**

```
 function doublyLinkedList() {
        this.head = null // 表头：始终指向第一个节点，默认为 null
        this.tail = null // 表尾：始终指向最后一个节点，默认为 null
        this.length = 0 // 链表长度

        function Node(data) {
            this.data = data
            this.prev = null
            this.next = null
        }

        doublyLinkedList.prototype.append = function (data) {
            let newNode = new Node(data)

            if (this.length === 0) {
            // 当插入的节点为链表的第一个节点时
            // 表头和表尾都指向这个节点
                this.head = newNode
                this.tail = newNode
            } else {
            // 当链表中已经有节点存在时
            // 注意tail指向的始终是最后一个节点
            // 注意head指向的始终是第一个节点
            // 因为是双向链表，可以从头部插入新节点，也可以从尾部插入
            // 这里以从尾部插入为例，将新节点插入到链表最后
            // 首先将新节点的 prev 指向上一个节点，即之前tail指向的位置
                newNode.prev = this.tail
            // 然后前一个节点的next（及之前tail指向的节点）指向新的节点
            // 此时新的节点变成了链表的最后一个节点
                this.tail.next = newNode
            // 因为 tail 始终指向的是最后一个节点，所以最后修改tail的指向
                this.tail = newNode
            }
            this.length++
        }
        doublyLinkedList.prototype.toString = function () {
            return this.backwardString()
        }
        doublyLinkedList.prototype.forwardString = function () {
            let current = this.tail
            let str = ''

            while (current) {
                str += current.data + ''
                current = current.prev
            }

            return str
        }
        doublyLinkedList.prototype.backwardString = function () {
            let current = this.head
            let str = ''

            while (current) {
                str += current.data + ''
                current = current.next
            }

            return str
        }

        doublyLinkedList.prototype.insert = function (position, data) {
            if (position < 0 || position > this.length) return false
            let newNode = new Node(data)
            if (this.length === 0) {
                this.head = newNode
                this.tail = newNode
            } else {
                if (position == 0) {
                    this.head.prev = newNode
                    newNode.next = this.head
                    this.head = newNode
                } else if (position == this.length) {
                    newNode.prev = this.tail
                    this.tail.next = newNode
                    this.tail = newNode
                } else {
                    let current = this.head
                    let index = 0
                    while( index++ < position){
                        current = current.next
                    }
                    newNode.next = current
                    newNode.prev = current.prev
                    current.prev.next = newNode
                    current.prev = newNode

                }

            }

            this.length++
            return true
        }
        doublyLinkedList.prototype.get = function (position) {
            if (position < 0 || position >= this.length) return null
            let current = this.head
            let index = 0
            while (index++) {
                current = current.next
            }

            return current.data
        }
        doublyLinkedList.prototype.indexOf = function (data) {
            let current = this.head
            let index = 0
            while (current) {
                if (current.data === data) {
                    return index
                }
                current = current.next
                index++
            }
            return  -1
        }
        doublyLinkedList.prototype.update = function (position, newData) {
            if (position < 0 || position >= this.length) return false
            let current = this.head
            let index = 0
            while(index++ < position){
                current = current.next
            }
            current.data = newData
            return true
        }
        doublyLinkedList.prototype.removeAt = function (position) {
            if (position < 0 || position >= this.length) return null
            let current = this.head
            if (this.length === 1) {
                this.head = null
                this.tail = null
            } else {
                if (position === 0) { // 删除第一个节点
                    this.head.next.prev = null
                    this.head = this.head.next
                } else if (position === this.length - 1) { // 删除最后一个节点
                    this.tail.prev.next = null
                    this.tail = this.tail.prev
                } else {
                    let index = 0
                    while (index++ < position) {
                        current = current.next
                    }
                    current.prev.next = current.next
                    current.next.prev = current.prev
                }
            }
            this.length--
            return current.data
        }
        doublyLinkedList.prototype.remove = function (data) {
            let index = this.indexOf(data)
            return this.removeAt(index)
        }
    }
```