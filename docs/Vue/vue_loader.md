### Vue-Loader 源码解读

#### 介绍

用过 Vue 的小伙伴都知道，Vue 的一个很大的特色就是 SFC 组件(Single File Component),也就是单文件组件。它可以将一个组件的三要素(HTML、CSS、Javascript)组合在一个文件中,在 Vue 底层实现中，凡是出现 template 字符串模板或者是<template>标签的，最后都转换成 render 函数。而转换的方式有多种：

- 1、结合构建工具比如 webpack，通过配置 Vue-Loader 和 VueLoaderPlugin 在编译时对.vue 文件进行解析

Vue-Service 就是基于这种方式来解析的

#### 流程概述
