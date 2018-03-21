# 事件循环 EVENT LOOP

* 一个线程中，事件循环是唯一的，但是任务队列可以拥有多个。
* 任务队列又分为macro-task（宏任务）与micro-task（微任务），在最新标准中，它们被分别称为task与jobs。
* macro-task大概包括：script(整体代码), setTimeout, setInterval, setImmediate, I/O, UI rendering。
* micro-task大概包括: process.nextTick, Promise, Object.observe(已废弃), MutationObserver(html5新特性)
* 当当前执行栈执行完毕时会立刻先处理所有微任务队列中的事件，然后再去宏任务队列中取出一个事件。同一次事件循环中，微任务永远在宏任务之前执行。

![599584-eb6742e93ff577cd](https://user-images.githubusercontent.com/9698086/37706930-2e54d5c2-2d3c-11e8-889f-29b71e4ad605.png)

```
Promise.resolve().then(function promise1 () {
       console.log('promise1');
    })
setTimeout(function setTimeout1 (){
    console.log('setTimeout1')
    Promise.resolve().then(function  promise2 () {
       console.log('promise2');
    })
}, 0)

setTimeout(function setTimeout2 (){
   console.log('setTimeout2')
}, 0)
```

运行过程：

script里的代码被列为一个task，放入task队列。

**循环1：**

* 【task队列：script ；microtask队列：】
1. 从task队列中取出script任务，推入栈中执行。
2. promise1列为microtask，setTimeout1列为task，setTimeout2列为task。

* 【task队列：setTimeout1 setTimeout2；microtask队列：promise1】
3. script任务执行完毕，执行microtask checkpoint，取出microtask队列的promise1执行。

**循环2：**

* 【task队列：setTimeout1 setTimeout2；microtask队列：】
4. 从task队列中取出setTimeout1，推入栈中执行，将promise2列为microtask。

* 【task队列：setTimeout2；microtask队列：promise2】
5. 执行microtask checkpoint，取出microtask队列的promise2执行。

**循环3：**

* 【task队列：setTimeout2；microtask队列：】
6. 从task队列中取出setTimeout2，推入栈中执行。
7. setTimeout2任务执行完毕，执行microtask checkpoint。

* 【task队列：；microtask队列：】


> * [https://github.com/aooy/blog/issues/5](https://github.com/aooy/blog/issues/5)