## Vue 基本知识点

### 指令

指令与组件 component 的不同在于指令通常与 dom 操作绑定的。

- 1、内置指令

```Javascript
    1、v-text:更新元素的textContent
    <span v-text="msg"></span>
    等同于
    <span>{{msg}}</span>
    2、v-html:更新元素的innerHTML，并且不会作为Vue模板进行编译
    如果要组合模板，考虑使用组件。这个直接插入html有xss攻击的危险只在可信内容上使用，永不直接使用用户的html。另外在单文件组件里，scoped的样式不适用于html，因为该html片段并没有被vue编译。
    3、v-show
    改变元素的display css style
    4、v-if
    有条件渲染元素，当切换时，元素及其数据绑定/组件会被销毁。
    5、v-else
    6、v-else-if
    7、v-for
    会原地修改元素而不是移动它们。要强制重排元素，使用key attribute
    8、v-on 缩写@
    修饰符：
    .stop - 调用event.stopPropagation()
    .prevent - 调用event.preventDefault()
    .capture - 添加事件侦听器使用capture模式
    .self - 只当事件是从侦听器绑定的元素本身触发时才触发回调。
    .native - 监听组件根元素的原生事件
    .once - 只触发一次回调。
    .left - (2.2.0) 只当点击鼠标左键时触发。
    .right - (2.2.0) 只当点击鼠标右键时触发。
    .middle - (2.2.0) 只当点击鼠标中键时触发。
    .passive - (2.3.0) 以 { passive: true } 模式添加侦听器
    9、v-bind 缩写:
    修饰符：
    .prop
    10、v-model
    随着表单控件类型不同而不同
    限于：<input>、<textarea>、<select>、components
    11、v-slot 缩写#
    例子:
    // user-component
    <div>
        <head>
            <slot name="header" :user="user" ></slot>
        </head>
    <div>

    // usage
    <user-component>
        <template v-slot:header="{name,age}">
            <label>name:</label>
            <span>{{name}}</span>
        </template>
    </user-component>
    12、v-pre   不需要表达式
    跳过这个元素和它的子元素的编译过程。可以用来显示原始 Mustache 标签。跳过大量没有指令的节点会加快编译。
    <div>{{this will not be compiled}}</div>

    13、v-cloak  不需要表达式
    这个指令保持在元素上直到关联实例结束编译。和 CSS 规则如 [v-cloak] { display: none } 一起用时，这个指令可以隐藏未编译的 Mustache 标签直到实例准备完毕。
    例子：
    <div v-cloak>{{name}}</div>

    //css
    [v-cloak]{
        display:none;
    }
    14、v-once
    只渲染元素和组件一次。随后的重新渲染，元素/组件及其所有的子节点将被视为静态内容并跳过。这可以用于优化更新性能。

```

### 特殊 attribute

- 1、key
  key 的特殊 attribute 主要用在 Vue 的虚拟 DOM 算法，在新旧 nodes 对比时辨识 VNodes。如果不使用 key，Vue 会使用一种最大限度减少动态元素并且尽可能的尝试就地修改/复用相同类型元素的算法。而使用 key 时，它会基于 key 的变化重新排列元素顺序，并且会移除 key 不存在的元素。

  它也可以用于强制替换元素/组件而不是重复使用它。当你遇到如下场景时它可能会很有用:

```Javascript

    1、完整地触发组件的生命周期钩子
    2、触发过渡

    例子：
    <transition>
        <span :key="text" >{{ text }}</span>
    </transition>

```

- 2、ref
  ref 被用来给元素或子组件注册引用信息。引用信息将会注册在父组件的 $refs 对象上
关于 ref 注册时间的重要说明：因为 ref 本身是作为渲染结果被创建的，在初始渲染的时候你不能访问它们 - 它们还不存在！$refs 也不是响应式的，因此你不应该试图用它在模板中做数据绑定

- 3、is

### 内置组件

- 1、component
  渲染一个“元组件”为动态组件。依 is 的值，来决定哪个组件被渲染。

props:
is:

- 2、transition

- 3、transition-group

- 4、slot

- 5、keep-alive

### 组件选项-数据

- 1、data

- 2、props

- 3、computed
  计算属性将被混入到 Vue 实例中。所有 getter 和 setter 的 this 上下文自动地绑定为 Vue 实例。

注意如果你为一个计算属性使用了箭头函数，则 this 不会指向这个组件的实例，不过你仍然可以将其实例作为函数的第一个参数来访问。

```Javascript
    computed(){
        aDouble: vm => vm.a * 2
    }
```

计算属性的结果会被缓存，除非依赖的响应式 property 变化才会重新计算。注意，如果某个依赖 (比如非响应式 property) 在该实例范畴之外，则计算属性是不会被更新的。

- 4、methods
  methods 将被混入到 Vue 实例中。可以直接通过 VM 实例访问这些方法，或者在指令表达式中使用。方法中的 this 自动绑定为 Vue 实例。
  所以也不要使用箭头函数
- 5、watch
  一个对象，键是需要观察的表达式，值是对应回调函数。值也可以是方法名，或者包含选项的对象。Vue 实例将会在实例化时调用 $watch()，遍历 watch 对象的每一个 property。

```Javascript
var vm = new Vue({
  data: {
    a: 1,
    b: 2,
    c: 3,
    d: 4,
    e: {
      f: {
        g: 5
      }
    }
  },
  watch: {
    a: function (val, oldVal) {
      console.log('new: %s, old: %s', val, oldVal)
    },
    // 方法名
    b: 'someMethod',
    // 该回调会在任何被侦听的对象的 property 改变时被调用，不论其被嵌套多深
    c: {
      handler: function (val, oldVal) { /* ... */ },
      deep: true
    },
    // 该回调将会在侦听开始之后被立即调用
    d: {
      handler: 'someMethod',
      immediate: true
    },
    // 你可以传入回调数组，它们会被逐一调用
    e: [
      'handle1',
      function handle2 (val, oldVal) { /* ... */ },
      {
        handler: function handle3 (val, oldVal) { /* ... */ },
        /* ... */
      }
    ],
    // watch vm.e.f's value: {g: 5}
    'e.f': function (val, oldVal) { /* ... */ }
  }
})
vm.a = 2 // => new: 2, old: 1
```

<strong>注意，不应该使用箭头函数来定义 watcher 函数 </strong>

### 组件选项-dom

- 1、el 只在用 new 创建实例时生效
  提供一个在页面上已存在的 DOM 元素作为 Vue 实例的挂载目标。可以是 CSS 选择器，也可以是一个 HTMLElement 实例。

在实例挂载之后，元素可以用 vm.$el 访问。

如果在实例化时存在这个选项，实例将立即进入编译过程，否则，需要显式调用 vm.$mount() 手动开启编译。

如果 render 函数和 template property 都不存在，挂载 DOM 元素的 HTML 会被提取出来用作模板，此时，必须使用 Runtime + Compiler 构建的 Vue 库。

- 2、template
  一个字符串模板作为 Vue 实例的标识使用。模板将会替换挂载的元素。挂载元素的内容都将被忽略，除非模板的内容有分发插槽。

```html
如果值以 # 开始，则它将被用作选择符，并使用匹配元素的 innerHTML 作为模板。常用的技巧是用 <script type="x-template"> 包含模板
```

- 3、render
  字符串模板的代替方案，允许你发挥 JavaScript 最大的编程能力。该渲染函数接收一个 createElement 方法作为第一个参数用来创建 VNode。

如果组件是一个函数组件，渲染函数还会接收一个额外的 context 参数，为没有实例的函数组件提供上下文信息。
注：如果值以 # 开始，则它将被用作选择符，并使用匹配元素的 innerHTML 作为模板。常用的技巧是用 <script type="x-template"> 包含模板

- 4、renderError
  只在开发者环境下工作
  当 render 函数遭遇错误时，提供另外一种渲染输出。其错误将会作为第二个参数传递到 renderError。这个功能配合 hot-reload 非常实用。
  ```javascript
  new Vue({
    render(h) {
      throw new Error('error!');
    },
    renderError(h, err) {
      return h('span', {}, err.stack);
    },
  }).$mount('#app');
  ```

### 组件选项-生命周期钩子

- 1、beforeCreate
  在实例初始化之后,进行数据侦听和事件/侦听器的配置之前同步调用。
- 2、created

在实例创建完成后被立即同步调用。在这一步中，实例已完成对选项的处理，意味着以下内容已被配置完毕：数据侦听、计算属性、方法、事件/侦听器的回调函数。然而，挂载阶段还没开始，且 $el property 目前尚不可用。

- 3、beforeMount
  在挂载开始之前被调用：相关的 render 函数首次被调用。
  该钩子在服务器端渲染期间不被调用。
- 4、mounted
  实例被挂载后调用，这时 el 被新创建的 vm.$el 替换了。如果根实例挂载到了一个文档内的元素上，当 mounted 被调用时 vm.$el 也在文档内。

注意 mounted 不会保证所有的子组件也都被挂载完成。如果你希望等到整个视图都渲染完毕再执行某些操作，可以在 mounted 内部使用 vm.$nextTick：

```Javascript
mounted: function () {
  this.$nextTick(function () {
        // 仅在整个视图都被渲染之后才会运行的代码
    })
  }
```

该钩子在服务器端渲染期间不被调用。

- 5、beforeUpdate
  在数据发生改变后，DOM 被更新之前被调用。这里适合在现有 DOM 将要被更新之前访问它，比如移除手动添加的事件监听器。

该钩子在服务器端渲染期间不被调用，因为只有初次渲染会在服务器端进行。

- 6、updated
  在数据更改导致的虚拟 DOM 重新渲染和更新完毕之后被调用。

当这个钩子被调用时，组件 DOM 已经更新，所以你现在可以执行依赖于 DOM 的操作。然而在大多数情况下，你应该避免在此期间更改状态。如果要相应状态改变，通常最好使用计算属性或 watcher 取而代之。

注意，updated 不会保证所有的子组件也都被重新渲染完毕。如果你希望等到整个视图都渲染完毕，可以在 updated 里使用 vm.$nextTick：

```Javascript
    updated(){
        this.$nextTicket(function(){
            //整个视图更新后的操作
        })
    }
```

该钩子在服务器端渲染期间不被调用。

- 7、activated
  被 keep-alive 缓存的组件激活时调用。

该钩子在服务器端渲染期间不被调用。

- 8、deactivated
  被 keep-alive 缓存的组件失活时调用。

该钩子在服务器端渲染期间不被调用。

- 9、beforeDestroy
  实例销毁之前调用。在这一步，实例仍然完全可用。

该钩子在服务器端渲染期间不被调用。

- 10、destroyed
  实例销毁后调用。该钩子被调用后，对应 Vue 实例的所有指令都被解绑，所有的事件监听器被移除，所有的子实例也都被销毁。

该钩子在服务器端渲染期间不被调用。

- 11、errorCaptured
  在捕获一个来自后代组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 false 以阻止该错误继续向上传播。

  错误传播的规则：
  1、默认情况下，如果全局的 config.errorHandler 被定义，所有的错误仍会发送它，因此这些错误仍然会向单一的分析服务的地方进行汇报。

2、如果一个组件的 inheritance chain (继承链)或 parent chain (父链)中存在多个 errorCaptured 钩子，则它们将会被相同的错误逐个唤起。

3、如果此 errorCaptured 钩子自身抛出了一个错误，则这个新错误和原本被捕获的错误都会发送给全局的 config.errorHandler。

4、一个 errorCaptured 钩子能够返回 false 以阻止错误继续向上传播。本质上是说“这个错误已经被搞定了且应该被忽略”。它会阻止其它任何会被这个错误唤起的 errorCaptured 钩子和全局的 config.errorHandler。

### 组件选项- 资源

- 1、directives

- 2、filters

- 3、components

### 组件选项- 租户

- 1、parent
  指定已创建的实例之父实例，在两者之间建立父子关系。子实例可以用 this.$parent 访问父实例，子实例被推入父实例的 $children 数组中。

- 2、mixins

mixins 选项接收一个混入对象的数组。这些混入对象可以像正常的实例对象一样包含实例选项，这些选项将会被合并到最终的选项中，使用的是和 Vue.extend() 一样的选项合并逻辑。也就是说，如果你的混入包含一个 created 钩子，而创建组件本身也有一个，那么两个函数都会被调用。

Mixin 钩子按照传入顺序依次调用，并在调用组件自身的钩子之前被调用。

```
    var mixin = {
        created: function () { console.log(1) }
    }
    var vm = new Vue({
        created: function () { console.log(2) },
        mixins: [mixin]
    })
    // => 1
    // => 2
```

- 3、extends
  允许声明扩展另一个组件 (可以是一个简单的选项对象或构造函数)，而无需使用 Vue.extend。这主要是为了便于扩展单文件组件。

这和 mixins 类似。

```Javascript
    var CompA = { ... }

    // 在没有调用 `Vue.extend` 时候继承 CompA
    var CompB = {
    extends: CompA,
    ...
    }
```
