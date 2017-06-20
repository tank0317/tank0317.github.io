---
layout: post
title: Vue学习笔记（一）：组件之间的数据传递
date: 2017-06-19
tags: vue
comments: true
---

Vue开发中一个重要的特性就是组件的使用。在使用组件的过程中，一个经常遇到的问题是，组件之间可能存在某种关系，我们需要让两个或多个组件之间可以共享或传递数据。Vue支持这种组件之间的数据传递，并且有一定的规则，我们今天就来学习Vue中组件数据传递的一些知识。

首先不同的组件可以分为，父子组件关系和非父子组件关系。而我们需要数据传递的需求更多的发生在父子组件之间，所以这里重点总结（其实就是把文档抄一遍）父子组件的数据传递。

## 父子组件

重点！！！**父子组件的数据传递可以总结为 props down, events up。父组件通过props向下传递数据给自组件，自组件通过events给父组件发送消息。**

### 使用Prop传递数据
	
组件实例之间的作用域是**孤立**的。所以我们不能在子组件内直接引用父组件的数据。如果自组件需要获得父组件的数据，需要使用自组件内的props选项。

子组件要显示地用props选项声明它期待获得的数据，如下，自组件在内部通过props选项声明期待获得`message`数据。

```javascript
Vue.component('child', {
  // 声明 props
  props: ['message'],
  // 就像 data 一样，prop 可以用在模板内
  // 同样也可以在 vm 实例中像 “this.message” 这样使用
  template: '<span>{{ message }}</span>'
})
```

然后我们就可以在父组件里，通过`message`向自组件传递数据，比如一个简单的字符串：

```javascript
<child message="hello!"></child>
```

上面是给`message`传递了一个静态的字符串。我们也可以通过`v-bind`指令，动态的绑定父组件的数据到子模板的props。如下：

```javascript
<div>
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
```

这样当父组件的数据变化的时候，自组件的props也会跟着变化，从而实现父子组件之间的数据同步。

### 单向数据流

上面我们提到props可以实现父子组件之间的数据同步，但是这只是单向的同步，也就是父组件的数据变化时会传递到自组件，但是反过来，自组件内部prop的变化时不会传递给父组件的。也就是**prop只能实现单向的数据流。**这么做是为了防止自组件无意修改了父组件的状态--这会让应用的数据流难以理解。

另外，子组件不应该在内部改变prop的值，如果你这么做了，Vue会在控制台给出警告。

如果我们想要在自组件内部修改prop数据，如下面的两种需求：

* prop 作为初始值传入后，子组件想把它当作局部数据来用；
* prop 作为初始值传入，由子组件处理成其它数据输出。 

对于以上两种需求，正确的应对方式是：

* 定义一个局部变量，并用 prop 的值初始化它：

```javascript
props: ['initialCounter'],
data: function () {
  return { counter: this.initialCounter }
}
```

* 定义一个计算属性，处理 prop 的值并返回。

```javascript
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

> 注意在 JavaScript 中对象和数组是引用类型，指向同一个内存空间，如果 prop 是一个***对象或数组**，在子组件内部改变它会影响父组件的状态。

### Prop验证

我们可以为组件的props制定验证规格。果传入的数据不符合规格，Vue 会发出警告(如果使用的是开发版本）。当组件给其他人使用时，这很有用。

首先需要用对象的形式制定验证规格。另外，通常在验证中我们可以添加如下规则：

* 类型检测，同时可以设置多种类型。可以是下面原生构造器：
	* String
	* Number
	* Boolean
	* Function
	* Object
	* Array
	* type 也可以是一个自定义构造器函数，使用 instanceof 检测。 
* 是否必须
* 添加默认值，（如果是对象或数组，需要由工厂函数返回）
* 自定义验证函数

下面是一些例子：

```javascript
Vue.component('example', {
  props: {
    // 基础类型检测 （`null` 意思是任何类型都可以）
    propA: Number,
    // 多种类型
    propB: [String, Number],
    // 必传且是字符串
    propC: {
      type: String,
      required: true
    },
    // 数字，有默认值
    propD: {
      type: Number,
      default: 100
    },
    // 数组／对象的默认值应当由一个工厂函数返回
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

## Events up

父组件是使用 props 传递数据给子组件，但如果子组件要把数据传递回去，应该怎样做？那就是自定义事件！

### 使用v-on绑定自定义事件

每个Vue实例都实现了事件接口，即

* 使用$on(eventName)监听事件
* 使用$emit(eventName)触发事件

另外，父组件可以在使用子组件的地方直接用v-on监听组件触发的事件。

例子看vue文档[example](https://cn.vuejs.org/v2/guide/components.html#使用-v-on-绑定自定义事件)吧，我就不搬运了。

### 给组件绑定原生事件

如果需要在组件的根元素上监听一个原声事件。可以使用`.native`修饰`v-on`。例如：

```javascript
<my-component v-on:click.native="doTheThing"></my-component>
```

## .sync修饰符

前面我们说过，prop是【单向数据流】。但是有时候我们需要对一个prop进行【双向绑定】，所以vue引入了`.sync`修饰符。即当一个子组件改变了一个prop的值时，这个变化也会同步到父组件中绑定的值。在实际开发中，为了实现prop的双向绑定，并且让子组件在改变prop状态时，能够很容易的被区分开来，在vue 2.3中，`.sync`修饰符只是作为一个编译时的语法糖存在子它会被扩展为一个自动更新父组件属性的v-on侦听器。

如下代码：

```javascript
<comp :foo.sync="bar"></comp>
```

会被扩展为：

```javascript
<comp :foo="bar" @update:foo="val => bar = val"></comp> 
```

当子组件需要更新`foo`的值时，它需要显式地触发一个更新事件。

```javascript
this.$emit('update:foo', newValue);
```

### 使用自定义事件的表单输入组件

自定义事件可以用来创建自定义的表单输入组件，使用`v-model`来进行数据的双向绑定。例如：

```javascript
<input v-model="something">
```

这里`v-model`不过是一个语法糖，最终会被扩展为：

```javascript
<input v-bind:value="something" v-on:input="something = $event.target.value">
```

所以在组件中使用时，它相当于下面的简写：

```javascript
<custom-input v-bind:value="something" v-on:input="something = arguments[0]"></custom-input>
```

所以要让组件的`v-model`生效，它必须：

* 接受一个`value`属性
* 在有新的value时触发`input`事件

具体例子看vue文档[example](https://cn.vuejs.org/v2/guide/components.html#使用自定义事件的表单输入组件)吧。

事件接口不仅仅可以用来连接组件内部的表单输入，也很容易集成你自己创造的输入类型。想象一下：

```javascript
<voice-recognizer v-model="question"></voice-recognizer>
<webcam-gesture-reader v-model="gesture"></webcam-gesture-reader>
<webcam-retinal-scanner v-model="retinalImage"></webcam-retinal-scanner>
```

## 非父子组件通信

有时候两个组件也需要通信（非父子组件）。在简单的场景下，可以使用一个空的Vue实例作为中央事件总线：

```javascript
var bus = new Vue()
```

```javascript
// 触发组件 A 中的事件
bus.$emit('id-selected', 1)
```

```javascript
// 在组件 B 创建的钩子中监听事件
bus.$on('id-selected', function (id) {
  // ...
  })
```

在复杂情况下，我们应该考虑使用专门的[【状态管理模式】](https://cn.vuejs.org/v2/guide/state-management.html)。
