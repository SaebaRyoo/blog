### 异步更新队列

在vue中，更新DOM时是异步执行的，当watcher侦听到数据变化，vue将开启一个队列，并**缓冲在同一个事件循环中发生的所有数据变化**。如果同一个watcher被多次触发，则只会被推入到队列中一次。这种在**缓冲时**去除重复数据对于避免不必要的计算和 DOM 操作是非常重要的。然后，在下一个`Event Loop`时，执行并清空队列，进行DOM更新。

在vue中，是如何将以上的队列变成异步队列的呢？vue采用降级策略，vue的异步队列会按照顺序尝试原生的`Promise.then` -> `MutationObserver` -> `setImmediate`。如果以上都不支持，则会采用`setTimeout(fn, 0)`作为兜底方案。所以，异步队列利用的就是事件循环中的`micro-task(微任务)`，当不支持微任务时，采用`macro-task(宏任务)`，也就是setTimeout。


### $nextTick
虽然异步更新很好的避免了不必要的计算和DOM操作。但是，有时候我们想要基于更新后的状态来做一些操作。那么这个时候可以采用`Vue.nextTick(callback)`,组件内采用`this.$nextTick`。它的原理就如上的异步更新，它也采用这种降级策略。通过向队列的末尾添加一个`task`来实现**同步执行**的目的。看如下例子:

```html
<div id="app">
  <example />
</div>
```

```js

Vue.component('example', {
  template: '<span v-on:click=updateMessage>{{ message }}</span>',
  data: function () {
    return {
      message: '未更新'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = '已更新'
      console.log(this.$el.textContent) // => '未更新'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => '已更新'
      })
    }
  }
})
new Vue({
  el: '#app',
})
```