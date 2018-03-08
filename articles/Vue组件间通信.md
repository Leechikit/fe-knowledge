# Vue组件间通信

## 父组件下发数据到子组件
父组件的数据通过`prop`下发到子组件
```
Vue.component('child', {
  // 声明 props
  props: ['message'],
  template: '<span>{{ message }}</span>'
});

<child message="hello!"></child>
```

## 父组件动态下发数据到子组件
用`v-bind`来动态地将`prop`绑定到父组件的数据。每当父组件的数据变化时，该变化也会传导给子组件

<details>
<summary>CODE</summary>

```
<div>
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>

<child :my-message="parentMsg"></child>
```

OR

```
<template>
    <div class="parent">
        <child :pname="name"></child>
    </div>
</template>
<script>
import child from './child.vue';
export default {
    data() {
        return {
            name: 'parent'
        }
    },
    components: {
        child
    }
}
</script>
```

</details>

## 子组件与父组件通信

父组件可以在使用子组件的地方直接用`v-on`来监听子组件触发的事件。

子组件使用`$emit(eventName)`触发事件。

<details>
<summary>CODE</summary>

```
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```
```
Vue.component('button-counter', {
  template: '<button v-on:click="incrementCounter">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    incrementCounter: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

</details>

## 非父子组件的通信

```
var bus = new Vue()

// 触发组件 A 中的事件
bus.$emit('id-selected', 1)

// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
  // ...
})
```

## Vuex

![vuex](https://vuex.vuejs.org/zh-cn/images/vuex.png)