Vue 实例还暴露了一些有用的实例属性与方法。它们都有前缀 $，以便与用户定义的属性区分开来。例如：
~~~
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch 是一个实例方法
vm.$watch('a', function (newValue, oldValue) {
  // 这个回调将在 `vm.a` 改变后调用
})
~~~

## 模板语法
### 文本  v-text或者{{}}

### 原始HTML v-html
双大括号会将数据解释为普通文本，而非 HTML 代码。为了输出真正的 HTML，你需要使用 v-html 指令：
>你的站点上动态渲染的任意 HTML 可能会非常危险，因为它很容易导致 XSS 攻击。请只对可信内容使用 HTML 插值，绝不要对用户提供的内容使用插值。

### v-bind
~~~
<button v-bind:disabled="isButtonDisabled">Button</button>
//缩写
<button :disabled="isButtonDisabled">Button</button>
~~~
如果 isButtonDisabled 的值是 null、undefined 或 false，则 disabled 特性甚至不会被包含在渲染出来的 \<button> 元素中。

### v-on
~~~
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
~~~

## 计算属性和侦听器
对于任何复杂逻辑，都应当使用计算属性
~~~
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
~~~

**计算属性是基于它们的依赖进行缓存的**计算属性只有在它的相关依赖发生改变时才会重新求值。

#### 计算属性的setter
计算属性默认只有 getter ，不过在需要时你也可以提供一个 setter ：
~~~
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
~~~

### 用key管理可复用的元素
vue会尽可能高效地渲染元素，通常会复用已有元素而不是从头开始渲染。这么做除了使Vue变得非常快之外，还有其它一些好处。例如，如果你允许用户在不同的登录方式之间切换：
~~~
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
~~~
那么在上面的代码中切换 loginType 将不会清除用户已经输入的内容。因为两个模板使用了相同的元素，\<input> 不会被替换掉——仅仅是替换了它的 placeholder。

这样也不总是符合实际需求，所以 Vue 为你提供了一种方式来表达“这两个元素是完全独立的，不要复用它们”。只需添加一个具有唯一值的 key 属性即可：
~~~
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
~~~
现在，每次切换时，输入框都将被重新渲染。  
注意，\<label> 元素仍然会被高效地复用，因为它们没有添加 key 属性。

### v-show
带有v-show的元素始终会被渲染并保留在DOM中。v-show只是简单地切换元素的CSS属性display
>v-show不支持\<template>元素，也不支持v-else

### v-if vs v-show
v-if 是“真正”的条件渲染，因为它会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建。

v-if 也是惰性的：如果在初始渲染时条件为假，则什么也不做——直到条件第一次变为真时，才会开始渲染条件块。

相比之下，v-show 就简单得多——不管初始条件是什么，元素总是会被渲染，并且只是简单地基于 CSS 进行切换。

一般来说，v-if 有更高的切换开销，而 v-show 有更高的初始渲染开销。因此，如果需要非常频繁地切换，则使用 v-show 较好；如果在运行时条件很少改变，则使用 v-if 较好。


当 v-if 与 v-for 一起使用时，v-for 具有比 v-if 更高的优先级。

- - -

## 列表渲染
### 用 v-for 把一个数组对应为一组元素
我们用 v-for 指令根据一组数组的选项列表进行渲染。v-for 指令需要使用 item in items 形式的特殊语法，items 是源数据数组并且 item 是数组元素迭代的别名。

在 v-for 块中，我们拥有对父作用域属性的完全访问权限。v-for 还支持一个可选的第二个参数为当前项的索引。
~~~
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
~~~

~~~
var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
~~~

你也可以用 of 替代 in 作为分隔符，因为它是最接近 JavaScript 迭代器的语法：
    <div v-for="item of items"></div>

### 一个对象的 v-for
你也可以用 v-for 通过一个对象的属性来迭代。
~~~
<ul id="v-for-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
~~~

~~~
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      firstName: 'John',
      lastName: 'Doe',
      age: 30
    }
  }
})
~~~

+ John
+ Doe
+ 30

~~~
<div v-for="(value, key) in object">
  {{ key }}: {{ value }}
</div>
~~~
+ firstName: 'John'
+ lastName: 'Doe'
+ age: 30

#### key
当Vue.js用v-for正在更新已渲染过的元素列表时，它默认用“就地复用”的策略。

如果不想这样，你需要为每项提供一个唯一的key属性。理想的key值是每项都有且唯一的id。
~~~
<div v-for="item in items" :key="item.id">
  <!-- 内容 -->
</div>
~~~

建议尽可能在使用 v-for 时提供 key，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为以获取性能上的提升。

#### 注意事项
1. 当你利用索引值直接设置一个项时，例如:`vm.items[indexOfItem] = newValue`
2. 当你修改数组的长度时，例如：vm.items.length=newLength

为了解决第一类问题，以下两种方式都可以实现和 vm.items[indexOfItem] = newValue 相同的效果，同时也将触发状态更新：
~~~
Vue.set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
~~~

为了解决第二类问题，你可以使用 splice：
  vm.items.splice(newLength)

#### splice()方法
从数组中添加/删除项目，然后返回被删除的项目。该方法会改变原始数组
  arrayObject.splice(index,howmany,item1,.....,itemX)

参数 | 描述
---|---
index | 必需。整数，规定添加/删除项目的位置，使用负数可从数组结尾处规定位置。
howmany | 必需。要删除的项目数量。如果设置为 0，则不会删除项目。
item1, ..., itemX | 可选。向数组添加的新项目。

~~~
var arr = new Array(6)
arr[0] = "George"
arr[1] = "John"
arr[2] = "Thomas"
arr[3] = "James"
arr[4] = "Adrew"
arr[5] = "Martin"
arr.splice(2,3,"William")

George,John,Thomas,James,Adrew,Martin
George,John,William,Martin
~~~

### 对象更改检测注意事项
对于已经创建的实例，Vue 不能动态添加根级别的响应式属性。但是，可以使用 Vue.set(object, key, value) 方法向嵌套对象添加响应式属性。例如，对于：
~~~
var vm = new Vue({
  data: {
    userProfile: {
      name: 'Anika'
    }
  }
})
~~~
你可以添加一个新的 age 属性到嵌套的 userProfile 对象：
  Vue.set(vm.userProfile, 'age', 27)

你还可以使用 vm.$set 实例方法，它只是全局 Vue.set 的别名：
  vm.$set(vm.userProfile, 'age', 27)

有时你可能需要为已有对象赋予多个新属性
~~~
vm.userProfile = Object.assign({}, vm.userProfile, {
  age: 27,
  favoriteColor: 'Vue Green'
})
~~~

#### ==和===区别
==代表相同（值是否相等），===代表严格相同(数据类型和值都相等)

==比较时，如果类型不同，会转换成相同类型后再进行比较

### 显示过滤/排序结果
显示一个数组的过滤或排序副本，而不实际改变或重置原始数据
~~~
<li v-for="n in evenNumbers">{{ n }}</li>
~~~
~~~
data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
computed: {
  evenNumbers: function () {
    return this.numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
~~~

~~~
<li v-for="n in even(numbers)">{{ n }}</li>

data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
methods: {
  even: function (numbers) {
    return numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
~~~

#### v-for with v-if
当处于同一节点，`v-for`的优先级比`v-if`更高，`v-if`将分别重复运行于每个`v-for`循环中

#### style的scoped属性
在Vue组件中，在style标签上添加scoped属性，以表示它的样式作用于当下的模块


## 组件
~~~
// 定义一个名为 button-counter 的新组件
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
~~~

### data必须是一个函数
一个组件的 data 选项必须是一个函数，因此每个实例可以维护一份被返回对象的独立的拷贝
~~~
data: function () {
  return {
    count: 0
  }
}
~~~

unshift()方法可向数组的开头添加一个或更多元素，并返回新的长度

- - -
## Prop
#### Prop类型
以字符串数组形式列出的 prop
  props: ['title', 'likes', 'isPublished', 'commentIds', 'author']

但是，通常你希望每个 prop 都有指定的值类型。这时，你可以以对象形式列出 prop，这些属性的名称和值分别是 prop 各自的名称和类型：
~~~
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object
}
~~~


#### 传入一个对象的所有属性
如果你想要将一个对象的所有属性都作为prop传入，你可以使用不带参数的`v-bind`（取代 `v-bind:prop-name`）。
~~~
post: {
  id: 1,
  title: 'My Journey with Vue'
}
~~~
下面的模板
~~~
<blog-post v-bind="post"></blog-post>
~~~
等价于：
~~~
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
~~~

### 单向数据流
所有的prop都使得其父子prop之间形成了一个**单向下行绑定**：父级prop的更新会向下流动到子组件中，但是反过来则不行。

额外的，每次父级组件发生更新时，子组件中所有的prop都将会刷新为最新的值。这意味着你不应该在一个子组件内部改变 prop。如果你这样做了，Vue 会在浏览器的控制台中发出警告。

> 注意在 JavaScript 中对象和数组是通过引用传入的，所以对于一个数组或对象类型的 prop 来说，在子组件中改变这个对象或数组本身将会影响到父组件的状态。


实例传数据给子组件      通过在子组件定义props属性，实例就可以直接通过属性的方式传递数据给组件
子组件传数据给实例      通过在子组件使用`$emit`来触发事件，在实例中使用`v-on`来监听事件

父组件可使用`$children`来访问子组件
子组件可使用`$parent`来访问父组件


例如
~~~
//子组件
<button @click="$emit('close')"></button>

//父组件
@close="xxx"
~~~

当xxx复杂时，直接把JS代码写在`v-on`指令中是不可行的。因此`v-on`还可以接收一个需要调用的方法名称
~~~
<div id="example-2">
  <!-- `greet` 是在下面定义的方法名 -->
  <button v-on:click="greet">Greet</button>
</div>

var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // 在 `methods` 对象中定义方法
  methods: {
    greet: function (event) {
      // `this` 在方法里指向当前 Vue 实例
      alert('Hello ' + this.name + '!')
      // `event` 是原生 DOM 事件
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})

// 也可以用 JavaScript 直接调用方法
example2.greet() // => 'Hello Vue.js!'
~~~

>[Vue warn]: Avoid mutating a prop directly since the value will be overwritten whenever the parent component re-renders. Instead, use a data or computed property based on the prop's value. Prop being mutated: "selected"

vue不提倡在组件里面修改`props`的值，`props`仅用来数据传递（而非修改数据）。我们可以自己在内部定义变量`data`来起到一样的效果

- - -

前端可以根据带锚点的方式实现简单路由（不需要刷新页面）  

  http://localhost:8080/#/about/

路由让我们可以访问诸如`http://localhost:8080/#/about/`这些页面的时候不带刷新，直接展示

  \<router-view>\</router-view>

这句代码在页面中放入一个路由试图容器，当我们访问`http://localhost:8080/#/about/`的时候会将about的内容放进去

如果不是组件的话，正常data的写法可以直接写一个对象，比如`data:{msg:'下载'}`，但由于组件是会在多个地方引用的，JS中直接共享对象会造成引用传递，也就是说修改了msg后所有按钮的msg都会跟着修改，所以这里用function来每次返回一个对象实例

在组件中，props是专门用来暴露组件的属性接口


`$emit`，触发机制，父组件监听，子组件触发。由引用方（暂且叫做父组件）监听子组件的内置方法；同时在子组件中，需要触发这个事件
~~~
//quiButton.vue
//子组件中的代码
<script>
  export default {
    props: {
      msg: {
        default: '下载'
      }
    },
    methods: {
      btnClickEvent: function(){
        alert("先弹出默认的文案");
        this.$emit('btnClickEvent');//关键代码父组件触发自定义事件
      }
    }
  }
</script>
~~~

~~~
//监听子组件的事件 
<qui-btn v-on:btnClickEvent="doSth" msg="我可以点击" ></qui-btn>
~~~
上面的代码在引用组件的时候，注册了一个事件，这个btnClickEvent事件是之前在按钮组件中绑定到按钮的click事件中的，然后我们给这个事件一个自定义的方法doSth



slot标签，插槽
~~~
<template>
  <button class="qui-btn" v-on:click="btnClickEvent">
    <slot name="icon"></slot><!--重点在这里-->
    <span>{{msg}}</span>
  </button>
</template>
~~~

~~~
//pageQuiButton.vue
<qui-btn msg="下载" class="with-icon">
  <img slot="icon" class="ico" src="xxx.png" />
</qui-btn>
~~~
img上有个关键字`slot="icon"`，对应组件中的`name="icon"`，渲染的时候，会将img整个替换掉组件中的对应name的`<slot>`标签。slot的翻译是插槽的意思，相当于把img这块内容查到一个名为icon的插槽里面去。


~~~
beforeCreate:function(){},//组件实例化之前
created:function(){},//组件实例化了
beforeMount:function(){},//组件写入dom结构之前
mounted:function(){//组件写入dom结构了
  console.log(this.$el);
  console.log(this.$children);  //通过数组获取  [0].msg
  console.log(this.$refs);      //获取组件对象信息
  console.log(this.$refs.child1.msg); //通过对象集合获取
},
beforeUpdate:function(){},//组件更新前
updated:function(){},//组件更新比如修改了文案
beforeDestroy:function(){},//组件销毁之前
destroyed:function(){}//组件已经销毁
~~~