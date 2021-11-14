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
