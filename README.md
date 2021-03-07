# 学习 `Vuex 4` 源码整体架构，对比 `Vuex3` 异同

学习源码整体架构系列8篇之vuex4源码，前端面试高频源码，微信搜索「若川视野」关注我，长期交流学习~

>你好，我是[若川](https://mp.weixin.qq.com/s/c3hFML3XN9KCUetDOZd-DQ)，微信搜索「若川视野」关注我，长期交流学习，专注前端技术分享。这是`学习源码整体架构系列`第九篇。
学习源码整体架构系列文章([有哪些必看的JS库](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650746362&idx=1&sn=afe3a26cdbde1d423aae4fa99355f369&chksm=88662e76bf11a760a7f0a8565b9e8d52f5e4f056dc2682f213eec6475127d71f6f1d203d6c3a&scene=21#wechat_redirect))：[jQuery](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744496&idx=1&sn=0f149e9436cb77bf9fc1bfb47aedd334&chksm=8866253cbf11ac2a53b385153cd8e9a0c4018b6b566750cf0b5d61d17afa2e90b52d36db8054&scene=21#wechat_redirect)、[underscore](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744505&idx=1&sn=26801ad6c2a5eb9cf64e7556b6478d39&chksm=88662535bf11ac23eea3f76335f6777e2acbf4ee660b5616148e14ffbefc0e8520806db21056&scene=21#wechat_redirect)、[lodash](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744514&idx=1&sn=776336d888d06bfe72cb4d5b07a4b90c&chksm=8866254ebf11ac5822fc078082603f77a4b4d9b487c9f4d7069acb12c727c46c75946fa9b0cd&scene=21#wechat_redirect)、[sentry](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744551&idx=1&sn=4d79c2fa97d7c737aab70055c7ec7fa3&chksm=8866256bbf11ac7d9e2269f3638a705d5e5f45056d53ad2faf17b814e4c46ec6b0ba52571bde&scene=21#wechat_redirect)、[vuex](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744584&idx=1&sn=b14f8a762f132adcf0f7e3e075ee2ded&chksm=88662484bf11ad922ed27d45873af838298949eea381545e82a511cabf0c6fc6876a8370c6fb&scene=21#wechat_redirect)、[axios](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744604&idx=1&sn=51d8d865c9848fd59f7763f5fb9ce789&chksm=88662490bf11ad86061ae76ff71a1177eeddab02c38d046eecd0e1ad25dc16f7591f91e9e3b2&scene=21#wechat_redirect)、[koa](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744703&idx=1&sn=cfb9580241228993e4d376017234ff79&chksm=886624f3bf11ade5f5e37520f6b1291417bcea95f222906548b863f4b61d20e7508eb419eb85&token=192125900&lang=zh_CN&scene=21#wechat_redirect)、[redux](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650745007&idx=1&sn=1fd6f3caeff6ab61b8d5f644a1dbb7df&chksm=88662b23bf11a23573509a01f941d463b0c61e890b2069427c78c26296197077da359c522fe8&scene=21#wechat_redirect)。整体架构这词语好像有点大，姑且就算是源码整体结构吧，主要就是学习是代码整体结构，不深究其他不是主线的具体函数的实现。本篇文章学习的是实际仓库的代码。


>[本文仓库地址](https://github.com/lxchuan12/vuex4-analysis.git)：`git clone https://github.com/lxchuan12/vuex4-analysis.git`


>`要是有人说到怎么读源码，正在读文章的你能推荐我的源码系列文章，那真是太好了`。

源码类文章，一般阅读量不高。已经有能力看懂的，自己就看了。不想看，不敢看的就不会去看源码。
所以我的文章，尽量写得让想看源码又不知道怎么看的读者能看懂。我都是推荐使用断点调试源码学习，而不是硬看。正所谓：**授人与鱼不如授人予渔**。

阅读本文后你将学到：TODO:

- 1. git subtree 管理子仓库
- 2. 如何学习`Vuex 4`源码
- 3. Vuex 4 和 Vuex 3 的异同
- 4. Vuex 4 composition API如何使用
- 5. Vue.provide / inject API使用
- 等等

之前写过`Vuex 3`的源码文章[微信链接：学习 vuex 源码整体架构，打造属于自己的状态管理库](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744584&idx=1&sn=b14f8a762f132adcf0f7e3e075ee2ded&chksm=88662484bf11ad922ed27d45873af838298949eea381545e82a511cabf0c6fc6876a8370c6fb&scene=21#wechat_redirect)，阅读体验可能不太好，可以访问[博客链接](https://juejin.cn/post/6844904001192853511) `http://lxchuan12.gitee.io/vuex`，仓库很详细的注释和看源码方法，所以本文不会过多赘述与`Vuex 3`源码相同的地方。

最近抽空看了下`Vuex 4`的源码，顺便学习了下`composition API`，写下这篇文章。

## 1. `Vuex` 原理

全局的`Store` 实例对象。通过`Vue.reactive()`监测数据。

## 2. `Vuex 4` 重大改变

`Vuex 4`发布了[Vuex 4 release](https://github.com/vuejs/vuex/releases/tag/v4.0.0)`https://github.com/vuejs/vuex/releases/tag/v4.0.0`。

[Vuex 4 官方文档](https://next.vuex.vuejs.org/) `https://next.vuex.vuejs.org/`

`Vuex 4`的重点是兼容性。`Vuex 4`支持使用`Vue 3`开发，并且直接提供了和`Vuex 3`完全相同的`API`，因此用户可以在`Vue 3`项目中复用现有的`Vuex`代码。

相比Vuex3.x版本。主要有如下重大改变（其他的在上方链接中）：

### 1.1 安装过程

`Vuex 3`是`Vue.use(Vuex)`

`Vuex 4`则是`app.use(store)`

```js
import { createStore } from 'vuex'

export const store = createStore({
  state() {
    return {
      count: 1
    }
  }
})
```

```js
import { createApp } from 'vue'
import { store } from './store'
import App from './App.vue'

const app = createApp(App)

app.use(store)

app.mount('#app')
```

### 1.2 核心模块导出了 `createLogger` 函数

```js
import { createLogger } from 'vuex'
```

**接下来我们从源码的角度来看这些重大改变**。
## 2. 从源码角度看 `Vuex 4` 重大变化

## 2.1 `chrome` 调试 `Vuex 4` 源码准备工作

```sh
git subtree add --prefix=vuex https://github.com/vuejs/vuex.git 4.0
```

这种方式保留了vuex4仓库的git记录信息。

作为读者的你，只需克隆[我的`Vuex 4`源码仓库](https://github.com/lxchuan12/vuex4-analysis.git) `https://github.com/lxchuan12/vuex4-analysis.git` 即可，也欢迎`star`一下。

把`vuex/examples/webpack.config.js`，加个`devtool: 'source-map'`，这样就能开启`sourcemap`调试源码了。

```sh
git clone https://github.com/lxchuan12/vuex4-analysis.git
cd vuex
npm i
npm run dev
# 打开 http://localhost:8080/
# 选择 composition  购物车的例子 shopping-cart
# 打开 http://localhost:8080/composition/shopping-cart/
```

```js
import { createStore } from 'vuex'

export const store = createStore({
  state() {
    return {
      count: 1
    }
  }
})
```

```js
// 
import { createApp } from 'vue'
import App from './components/App.vue'
import store from './store'
import { currency } from './currency'

const app = createApp(App)

app.use(store)

app.mount('#app')
```

接下来，我们从`createApp({})`、`app.use(Store)`两个方面发散开来讲解。

## 2.2 Vuex.createStore 函数

相比 `Vuex 3` 中，`new Vuex.Store`，其实是一样的。只不过为了和`Vue 3` 统一，`Vuex 4` 额外多了一个 `createStore` 函数。

```js
export function createStore (options) {
  return new Store(options)
}
class Store{
  constructor (options = {}){
    // 省略若干代码...
    this._modules = new ModuleCollection(options)
    const state = this._modules.root.state
    resetStoreState(this, state)
    // 省略若干代码...
  }
}
function resetStoreState (store, state, hot) {
  // 省略若干代码...
  store._state = reactive({
    data: state
  })
  // 省略若干代码...
}
```

### 监测数据

和`Vuex 3`不同的是，监听数据不再是用`new Vue()`，而是`Vue 3`提供的`reactive`方法。

```js
Vue.reactive
```

## app.use() 方法

```js
use(plugin, ...options) {
    if (installedPlugins.has(plugin)) {
        (process.env.NODE_ENV !== 'production') && warn(`Plugin has already been applied to target app.`);
    }
    else if (plugin && isFunction(plugin.install)) {
        installedPlugins.add(plugin);
        plugin.install(app, ...options);
    }
    else if (isFunction(plugin)) {
        installedPlugins.add(plugin);
        plugin(app, ...options);
    }
    else if ((process.env.NODE_ENV !== 'production')) {
        warn(`A plugin must either be a function or an object with an "install" ` +
            `function.`);
    }
    return app;
}
```

## 2.3 install 函数

```js
export class Store{
    // ...
    install (app, injectKey) {
        app.provide(injectKey || storeKey, this)
        app.config.globalProperties.$store = this
    }
}
```

### 2.3.1 app.provide

```js
provide(key, value) {
    if ((process.env.NODE_ENV !== 'production') && key in context.provides) {
        warn(`App already provides property with key "${String(key)}". ` +
            `It will be overwritten with the new value.`);
    }
    // TypeScript doesn't allow symbols as index type
    // https://github.com/Microsoft/TypeScript/issues/24587
    context.provides[key] = value;
    return app;
}
```
猜测context

#### context

```js
const context = createAppContext();
```

context 为上下文

```js
function createAppContext() {
    return {
        app: null,
        config: {
            isNativeTag: NO,
            performance: false,
            globalProperties: {},
            optionMergeStrategies: {},
            isCustomElement: NO,
            errorHandler: undefined,
            warnHandler: undefined
        },
        mixins: [],
        components: {},
        directives: {},
        provides: Object.create(null)
    };
}
```

### 2.3.2 app.config.globalProperties

[app.config.globalProperties 官方文档](https://v3.cn.vuejs.org/api/application-config.html#globalproperties)

用法：

```js
app.config.globalProperties.$store = {}

app.component('child-component', {
  mounted() {
    console.log(this.$store) // '{}'
  }
})
```

也就能解释为什么每个组件都可以使用 `$store.state.xxx` 访问 `vuex`中的数据。

至此我们就看完，`createStore(store)`，`app.use(store)`两个API

`app.provide` 其实是用于`composition API`使用的。

接下来，我们看下源码具体实现。
## `composition API` 中如何使用`Vuex 4`

```js
// vuex/examples/classic/counter/Counter.vue
import { computed } from 'vue'
import { useStore } from 'vuex'

export default {
  setup () {
    const store = useStore()

    return {
      count: computed(() => store.state.count),
      evenOrOdd: computed(() => store.getters.evenOrOdd),
      increment: () => store.dispatch('increment'),
      decrement: () => store.dispatch('decrement'),
      incrementIfOdd: () => store.dispatch('incrementIfOdd'),
      incrementAsync: () => store.dispatch('incrementAsync')
    }
  }
}
```

### Vuex.useStore 源码实现

```js
// vuex/src/injectKey.js
import { inject } from 'vue'

export const storeKey = 'store'

export function useStore (key = null) {
  return inject(key !== null ? key : storeKey)
}
```

### Vue.inject 源码实现

```js
function inject(key, defaultValue, treatDefaultAsFactory = false) {
    // fallback to `currentRenderingInstance` so that this can be called in
    // a functional component
    const instance = currentInstance || currentRenderingInstance;
    if (instance) {
        // #2400
        // to support `app.use` plugins,
        // fallback to appContext's `provides` if the intance is at root
        const provides = instance.parent == null
            ? instance.vnode.appContext && instance.vnode.appContext.provides
            : instance.parent.provides;
        if (provides && key in provides) {
            // TS doesn't allow symbol as index type
            return provides[key];
        }
        else if (arguments.length > 1) {
            return treatDefaultAsFactory && isFunction(defaultValue)
                ? defaultValue()
                : defaultValue;
        }
        else if ((process.env.NODE_ENV !== 'production')) {
            warn(`injection "${String(key)}" not found.`);
        }
    }
    else if ((process.env.NODE_ENV !== 'production')) {
        warn(`inject() can only be used inside setup() or functional components.`);
    }
}
```

### Vue.provide 源码实现

## 其他基本和Vuex3.x版本没什么区别

## `Vuex 4`其他能力

核心模块导出了 `createLogger` 函数

```js
import { createLogger } from 'vuex'
```



[chrome source面板](https://mp.weixin.qq.com/s/lMlq4IKtHj2V3Hv2iB723w)