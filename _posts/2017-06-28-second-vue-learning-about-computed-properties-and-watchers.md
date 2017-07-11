---
layout: post
title: Vue学习笔记（二）：计算属性和watchers
date: 2017-06-28
tags: vue
comments: true
---

在Vue模板中，是可以使用表达式的，也就是说我们在显示一个数据的时候，可以对数据进行一定的处理再输出，这有时候会很方便，我们可以直接在模板内处理一些简单的逻辑或者计算。但是他们实际上只用于简单的计算。模板内放入过多的逻辑会让模板看起来不再简单清晰。同时当我们想要在模板中多次显示这个数据的时候就有点麻烦了，你需要重复的书写一串表达式。如下：

```javascript
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

上面的模板用于反向显示message。模板内表达式有些时候能给我们带来方便，但其实，我们还有其他的办法，更加绅士的做这些简单的甚至更加复杂的计算/逻辑处理，比如，

* **计算属性**
* Methods
* Watcher

然后我们对这三种方式进行一个比较。

## 计算属性

先看个例子：

```HTML
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

```javascript
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // a computed getter
    reversedMessage: function () {
      // `this` points to the vm instance
      return this.message.split('').reverse().join('')
    }
  }
})
```

这里我们通过`computed`声明了一个计算属性`reversedMessage`。现在我们可以像绑定普通属性一样在模板中绑定计算属性。Vue知道`vm.reversedMessage`依赖于`vm.message`，因此当`vm.message`发生改变时，所有依赖于`vm.reversedMessage`的绑定也会更新。看起来好像很神奇，实际上，我们在`computed`中声明计算属性`reversedMessage`的时候，同时会提供一个`function`，而Vue会把这个函数用作计算属性`reversedMessage`的`getter`，这样就很容易理解为什么计算属性会随着依赖的更新而变化。

## 计算属性 VS Methods

如前面的引入计算属性一样，我们也可以通过引入Methods来处理模板的逻辑计算。比如：

```HTML
<p>Reversed message: "{{ reversedMessage() }}"</p>
```

```javascript
// in component
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

这次，我们通过引入Methods，然后直接在模板中使用这个方法。相比于计算属性，从结果上来看的话，他们是相同的。然而，不同的是**计算属性是基于它们的依赖进行缓存的**。计算属性只有在它的相关依赖发生改变的时候才会重新求值。这意味着，只要`message`还没有发生改变，多次访问`reversedMessage`计算属性会立即返回之前的计算结果，而不必再次执行函数。

...未完
