# BetterScroll

## 一. 起步

### 1. BetterScroll 的页面结构

```html
<div class="wrapper" ref="wrapper">
  <ul class="content">
    <li>...</li>
    <li>...</li>
    ...
  </ul>
  <!-- 这里可以放一些其它的 DOM，但不会影响滚动 -->
</div>
```

1. 获取整个 wrapper 作为可滚动的区域默认区域
2. BetterScroll 只会获取其第一个元素作为滚动的内容，这里便是 content 的内容，其后面的所有 DOM 元素均不会被作为滚动内容。
   * 注意：版本 2.0.4 的 BetterScroll 可以通过 [specifiedIndexAsContent](https://better-scroll.github.io/docs/zh-CN/guide/base-scroll-options.html#specifiedindexascontent-2-0-4) 来指定 wrapper 的哪个子元素作为 content。

### 2. 初始化 BetterScroll

* `{}` 内为初始化 BetterScroll 的参数，如滚动方向、滚动类型、元素等

```js
import BScroll from '@better-scroll/core'
// 原生
let wrapper = document.querySelector('.wrapper')
let scroll = new BScroll(wrapper, {})

// vue2
let scroll = new BScroll(this.$refs.wrapper, {})

// vue3
const wrapper = ref(null) // 注意导入ref
let scroll = new BScroll(wrapper.value, {})
```

* 具体可配置项参照文档 ：[配置项](https://better-scroll.github.io/docs/zh-CN/guide/base-scroll-options.html#startx)



## 二. 使用

### 1. 基础滚动 - 只引用核心功能

* 只有基础的滚动功能

```javascript
import BScroll from '@better-scroll/core'
const wrapper = ref(null)
let bs = new BScroll(wrapper.value, {
	// ...... 具体详见配置项
})
```

### 2. 增强型滚动 - 在基础滚动上增加几个插件功能

* 通过插件给滚动添加几个额外的功能，如上拉加载、下滑刷新等。具体见：[插件](https://better-scroll.github.io/docs/zh-CN/plugins)

```javascript
import BScroll from '@better-scroll/core'
import Pullup from '@better-scroll/pull-up'

// 注册组件
BScroll.use(Pullup)

let bs = new BScroll(wrapper.value, {
    // 启用上拉加载功能
	probeType: 3, // 是否派发scroll
	pullUpLoad: true,
})
```

[probeType](https://better-scroll.github.io/docs/zh-CN/guide/base-scroll-options.html#probetype) ：决定是否派发 scroll 事件，对页面的性能有影响，尤其是在 `useTransition` 为 true 的模式下。

1. probeType 为 0 ，任何时候都不会派发 scroll 事件
2. probeType 为 1 ，仅仅当手指按在滚动区域上，每隔 momentumLimitTime 毫秒派发一次 scroll 事件
3. probeType 为 2 ，仅仅当手指按住滚动区域上，一直派发 scroll 事件，
4. probeType 为 3 ，任何时候都派发 scroll 事件，包括调用 scrollTo 或者 触发 momentum 滚动动画



### 3. 全能力滚动 - 不考虑性能一次性导入所有的功能

* 一次性导入滚动的所有的插件，但会导致体积过大、推荐**按需引入**

```js
import BScroll from '@better-scroll'

const wrapper = ref(null)
let scroll = new BScroll(wrapper.value, {
	// ...
 	pullUpLoad: true,
 	wheel: true,
  	scrollbar: true,
  	// and so on
  	// 需要用到什么，就启用什么插件功能
})
```



## 三. 核心滚动设置

[核心滚动 | BetterScroll 2.0 (better-scroll.github.io)](https://better-scroll.github.io/docs/zh-CN/guide/base-scroll.html#上手)



## 四. 相关设置

* [配置项 | BetterScroll 2.0 (better-scroll.github.io)](https://better-scroll.github.io/docs/zh-CN/guide/base-scroll-options.html)

* [API | BetterScroll 2.0 (better-scroll.github.io)](https://better-scroll.github.io/docs/zh-CN/guide/base-scroll-api.html)

* [插件 | BetterScroll 2.0 (better-scroll.github.io)](https://better-scroll.github.io/docs/zh-CN/plugins/)