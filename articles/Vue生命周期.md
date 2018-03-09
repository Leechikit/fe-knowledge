# Vue生命周期

![vue](https://cn.vuejs.org/images/lifecycle.png)

![vue](https://sfault-image.b0.upaiyun.com/229/095/2290954140-59361cf4c4837)

## created 和 mounted 相关

* beforecreated：el 和 data 并未初始化 
* created：完成了 data 数据的初始化，el没有
* beforeMount：完成了 el 和 data 初始化 
* mounted：完成挂载

在标红处，我们能发现el还是 {{message}}，这里就是应用的 Virtual DOM（虚拟Dom）技术，先把坑占住了。到后面mounted挂载的时候再把值渲染进去。

![](https://sfault-image.b0.upaiyun.com/130/248/1302481017-586cddaf83195_articlex)

## update 相关

```
setTimeout(()=>{
  app.message= 'yes !! I do';
},3000)
```

![](https://user-images.githubusercontent.com/9698086/37195577-684d78c0-23ae-11e8-83e1-5e72f78cf94a.jpg)

## destroy 相关

执行了destroy操作，后续就不再受vue控制了。

```
app.$destroy();
```

![](https://sfault-image.b0.upaiyun.com/396/155/3961550519-586ce0cb13de2_articlex)

## 用途

* beforecreate : 举个栗子：可以在这加个loading事件 
* created ：在这结束loading，还做一些初始化，实现函数自执行，不依赖dom的请求放在created() 
* mounted ： 在这发起后端请求，拿回数据，配合路由钩子做一些事情，有依赖dom做一些其他的操作就放到mounted()
* beforeDestory： 你确认删除XX吗？
* destoryed ：当前组件已被删除，清空相关内容