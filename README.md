# 学习 `Vuex 4` 源码整体架构，深入理解provide、inject原理

>你好，我是[若川](https://lxchuan12.gitee.io)，微信搜索[「若川视野」](https://mp.weixin.qq.com/s/c3hFML3XN9KCUetDOZd-DQ)关注我，专注前端技术分享。欢迎加我微信`ruochuan12`，加群交流学习。

>这是`学习源码整体架构系列`第九篇。
学习源码整体架构系列文章([有哪些必看的JS库](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650746362&idx=1&sn=afe3a26cdbde1d423aae4fa99355f369&chksm=88662e76bf11a760a7f0a8565b9e8d52f5e4f056dc2682f213eec6475127d71f6f1d203d6c3a&scene=21#wechat_redirect))：[jQuery](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744496&idx=1&sn=0f149e9436cb77bf9fc1bfb47aedd334&chksm=8866253cbf11ac2a53b385153cd8e9a0c4018b6b566750cf0b5d61d17afa2e90b52d36db8054&scene=21#wechat_redirect)、[underscore](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744505&idx=1&sn=26801ad6c2a5eb9cf64e7556b6478d39&chksm=88662535bf11ac23eea3f76335f6777e2acbf4ee660b5616148e14ffbefc0e8520806db21056&scene=21#wechat_redirect)、[lodash](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744514&idx=1&sn=776336d888d06bfe72cb4d5b07a4b90c&chksm=8866254ebf11ac5822fc078082603f77a4b4d9b487c9f4d7069acb12c727c46c75946fa9b0cd&scene=21#wechat_redirect)、[sentry](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744551&idx=1&sn=4d79c2fa97d7c737aab70055c7ec7fa3&chksm=8866256bbf11ac7d9e2269f3638a705d5e5f45056d53ad2faf17b814e4c46ec6b0ba52571bde&scene=21#wechat_redirect)、[vuex](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744584&idx=1&sn=b14f8a762f132adcf0f7e3e075ee2ded&chksm=88662484bf11ad922ed27d45873af838298949eea381545e82a511cabf0c6fc6876a8370c6fb&scene=21#wechat_redirect)、[axios](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744604&idx=1&sn=51d8d865c9848fd59f7763f5fb9ce789&chksm=88662490bf11ad86061ae76ff71a1177eeddab02c38d046eecd0e1ad25dc16f7591f91e9e3b2&scene=21#wechat_redirect)、[koa](https://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744703&idx=1&sn=cfb9580241228993e4d376017234ff79&chksm=886624f3bf11ade5f5e37520f6b1291417bcea95f222906548b863f4b61d20e7508eb419eb85&token=192125900&lang=zh_CN&scene=21#wechat_redirect)、[redux](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650745007&idx=1&sn=1fd6f3caeff6ab61b8d5f644a1dbb7df&chksm=88662b23bf11a23573509a01f941d463b0c61e890b2069427c78c26296197077da359c522fe8&scene=21#wechat_redirect)。整体架构这词语好像有点大，姑且就算是源码整体结构吧，主要就是学习是代码整体结构，不深究其他不是主线的具体函数的实现。本篇文章学习的是实际仓库的代码。

>[本文仓库地址](https://github.com/lxchuan12/vuex4-analysis.git)：`git clone https://github.com/lxchuan12/vuex4-analysis.git`，本文最佳阅读方式，克隆仓库自己动手调试，容易吸收消化。

>**要是有人说到怎么读源码，正在读文章的你能推荐我的源码系列文章，那真是无以为报**。

源码类文章，一般阅读量不高。已经有能力看懂的，自己就看了。不想看，不敢看的就不会去看源码。
所以我的文章，尽量写得让想看源码又不知道怎么看的读者能看懂。我都是推荐使用**搭建环境断点调试源码学习**，**哪里不会点哪里**，边调试边看，而不是硬看。正所谓：**授人与鱼不如授人予渔**。

阅读本文后你将学到：TODO:

- 1. git subtree 管理子仓库
- 2. 如何学习`Vuex 4`源码
- 3. Vuex 4 和 Vuex 3 的异同
- 4. Vuex 4 composition API如何使用
- 5. Vue.provide / inject API使用
- 等等

如果对于谷歌浏览器调试还不是很熟悉的读者，可以看这篇文章[chrome devtools source面板](https://mp.weixin.qq.com/s/lMlq4IKtHj2V3Hv2iB723w)，写的很详细。谷歌浏览器是我们前端常用的工具，所以建议大家深入学习，毕竟**工欲善其事，必先利其器**。

之前写过`Vuex 3`的源码文章[学习 vuex 源码整体架构，打造属于自己的状态管理库](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744584&idx=1&sn=b14f8a762f132adcf0f7e3e075ee2ded&chksm=88662484bf11ad922ed27d45873af838298949eea381545e82a511cabf0c6fc6876a8370c6fb&scene=21#wechat_redirect)`http://lxchuan12.gitee.io/vuex`，仓库有很详细的注释和看源码方法，所以本文不会过多赘述与`Vuex 3`源码相同的地方。

最近抽空看了下`Vuex 4`的源码，顺便学习了下`composition API`，写下这篇文章。

## 1. `Vuex` 原理简述

结论先行：Vuex原理可以拆解为三个关键点。
第一点、其实就是每个组件实例里都混入了Store实例，挂载在provides中。
那么每个组件如何获取组件实例中的Store实例，composition API中本质上则是使用inject函数。
第二点、Store实例中的各种方法都是为Store中的属性服务的。
第三点、Store中的属性变更触发视图更新。

本文主要讲解第一点。第二点在我的上一篇文章[学习 vuex 源码整体架构，打造属于自己的状态管理库](http://mp.weixin.qq.com/s?__biz=MzA5MjQwMzQyNw==&mid=2650744584&idx=1&sn=b14f8a762f132adcf0f7e3e075ee2ded&chksm=88662484bf11ad922ed27d45873af838298949eea381545e82a511cabf0c6fc6876a8370c6fb&scene=21#wechat_redirect)详细讲了，本文就不赘述了。第三点两篇文章都没有讲述。

```js
// 简版
class Store{
  constructor(){
    this.name = 'Store 实例';
  }
  dispatch(){

  }
  commit(){}
  // 省略
}


const store = new Store();
var rootInstance = {
  Store: store,
};
var parentInstance = {
  Store: store,
};
var childInstance1 = {
  Store: store,
};
var childInstance2 = {
  Store: store
};

store.name = '我被修改了';
```

![provide,inject](./images/components_provide.png)

所以要学习的问题是：怎么混入的。

1、

全局的`Store` 实例对象。通过`Vue.reactive()`监测数据。

## 2. `Vuex 4` 重大改变

`Vuex 4`发布了[Vuex 4 release](https://github.com/vuejs/vuex/releases/tag/v4.0.0)`https://github.com/vuejs/vuex/releases/tag/v4.0.0`。

[Vuex 4 官方文档](https://next.vuex.vuejs.org/) `https://next.vuex.vuejs.org/`

`Vuex 4`的重点是兼容性。`Vuex 4`支持使用`Vue 3`开发，并且直接提供了和`Vuex 3`完全相同的`API`，因此用户可以在`Vue 3`项目中复用现有的`Vuex`代码。

相比Vuex3.x版本。主要有如下重大改变（其他的在上方链接中）：

### 2.1 安装过程

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

### 2.2 核心模块导出了 `createLogger` 函数

```js
import { createLogger } from 'vuex'
```

**接下来我们从源码的角度来看这些重大改变**。

## 3. 从源码角度看 `Vuex 4` 重大变化

### 3.1 `chrome` 调试 `Vuex 4` 源码准备工作

```sh
git subtree add --prefix=vuex https://github.com/vuejs/vuex.git 4.0
```

这种方式保留了`vuex4`仓库的`git`记录信息。更多`git subtree`使用方式可以查看这篇文章[用 Git Subtree 在多个 Git 项目间双向同步子项目，附简明使用手册](https://segmentfault.com/a/1190000003969060)。

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

### 3.2 Vuex.createStore 函数

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

**监测数据**

和`Vuex 3`不同的是，监听数据不再是用`new Vue()`，而是`Vue 3`提供的`reactive`方法。

`Vue.reactive` 函数方法，本文就不展开讲解了。因为展开来讲，又可以写篇新的文章了。只需要知道主要功能是监测数据改变，变更视图即可。

### 3.3 app.use() 方法

```js
// runtime-core.esm-bundler.js
function createAppAPI(render, hydrate) {
    return function createApp(rootComponent, rootProps = null) {
      // 代码有删减
      const installedPlugins = new Set();
      const app = (context.app = {
        use(plugin, ...options) {
          // 已经有插件，并且 不是生产环境，报警告。
            if (installedPlugins.has(plugin)) {
                (process.env.NODE_ENV !== 'production') && warn(`Plugin has already been applied to target app.`);
            }
            // 插件的install 是函数，则添加插件，并执行 install 函数
            else if (plugin && isFunction(plugin.install)) {
                installedPlugins.add(plugin);
                // 断点
                plugin.install(app, ...options);
            }
            // 插件本身 是函数，则添加插件，并执行 插件本身函数
            else if (isFunction(plugin)) {
                installedPlugins.add(plugin);
                plugin(app, ...options);
            }
            // 如果都不是报警告
            else if ((process.env.NODE_ENV !== 'production')) {
                warn(`A plugin must either be a function or an object with an "install" ` +
                    `function.`);
            }
            // 支持链式调用
            return app;
        },
        provide(){ 
          // 省略... 后文再讲
        }
      });
    }
}

```

跟着断点走到下一步，`install`函数。

### 3.4 install 函数

```js
export class Store{
    // 省略若干代码...
    install (app, injectKey) {
       // 为 composition API 中使用
      //  可以传入 injectKey  如果没传取默认的 storeKey 也就是 store
        app.provide(injectKey || storeKey, this)
        // 为 option API 中使用
        app.config.globalProperties.$store = this
    }
    // 省略若干代码...
}
```

#### 3.4.1 app.provide

```js
provide(key, value) {
    // 如果已经有值了警告
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

从上方代码中搜索`context`，可以发现这一句代码

```js
const context = createAppContext();
```

接着我们来看函数 `createAppContext`。
context 为上下文

```js
// runtime-core.esm-bundler.js
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

[Vue3 文档应用配置(app.config)](https://v3.cn.vuejs.org/api/application-config.html) `https://v3.cn.vuejs.org/api/application-config.html`

也就是说在`AppContext.provides`中注入了一个Store实例对象。

#### 3.4.2 app.config.globalProperties

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

也就能解释为什么每个组件都可以使用 `this.$store.xxx` 访问 `vuex`中的方法和属性了。

至此我们就看完，`createStore(store)`，`app.use(store)`两个API

`app.provide` 其实是用于`composition API`使用的。

接下来，我们看下源码具体实现，为什么每个组件实例中都能获取到的。

### 3.5 `composition API` 中如何使用`Vuex 4`

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

#### 3.5.1 Vuex.useStore 源码实现

```js
// vuex/src/injectKey.js
import { inject } from 'vue'

export const storeKey = 'store'

export function useStore (key = null) {
  return inject(key !== null ? key : storeKey)
}
```

#### 3.5.2 Vue.inject 源码实现

```js
// runtime-core.esm-bundler.js
function inject(key, defaultValue, treatDefaultAsFactory = false) {
    // fallback to `currentRenderingInstance` so that this can be called in
    // a functional component
    // 如果是被一个函数式组件调用则取 currentRenderingInstance
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
        // 如果参数大于1个 第二个则是默认值 ，第三个参数是 true，并且第二个值是函数则执行函数。
        else if (arguments.length > 1) {
            return treatDefaultAsFactory && isFunction(defaultValue)
                ? defaultValue()
                : defaultValue;
        }
        // 警告没找到
        else if ((process.env.NODE_ENV !== 'production')) {
            warn(`injection "${String(key)}" not found.`);
        }
    }
    // 如果没有当前实例则说明则报警告。
    // 也就是是说inject必须在setup中调用或者在函数式组件中使用
    else if ((process.env.NODE_ENV !== 'production')) {
        warn(`inject() can only be used inside setup() or functional components.`);
    }
}
```

接着我们继续来看`inject`的，`provide`。

#### 3.5.3  Vue.provide 源码实现

```js
// runtime-core.esm-bundler.js
function provide(key, value) {
    if (!currentInstance) {
        if ((process.env.NODE_ENV !== 'production')) {
            warn(`provide() can only be used inside setup().`);
        }
    }
    else {
        let provides = currentInstance.provides;
        // by default an instance inherits its parent's provides object
        // but when it needs to provide values of its own, it creates its
        // own provides object using parent provides object as prototype.
        // this way in `inject` we can simply look up injections from direct
        // parent and let the prototype chain do the work.
        const parentProvides = currentInstance.parent && currentInstance.parent.provides;
        if (parentProvides === provides) {
            provides = currentInstance.provides = Object.create(parentProvides);
        }
        // TS doesn't allow symbol as index type
        provides[key] = value;
    }
}
```

上文`inject`、`provide`函数中都有个变量`currentInstance`当前实例，那么当前实例对象是怎么来的呢。

为什么每个组件就能访问到，依赖注入的思想。
有一个讨巧的方法，就是在文件`runtime-core.esm-bundler.js`中搜索`provides`，则能搜索到`createComponentInstance`函数

接下来我们`createComponentInstance`函数如何创建组件实例。

### 3.6 createComponentInstance 创建组件实例

```js
// runtime-core.esm-bundler.js
const emptyAppContext = createAppContext();
let uid$1 = 0;
function createComponentInstance(vnode, parent, suspense) {
    const type = vnode.type;
    const appContext = (parent ? parent.appContext : vnode.appContext) || emptyAppContext;
    const instance = {
        uid: uid$1++,
        vnode,
        type,
        parent,
        appContext,
        root: null,
        next: null,
        subTree: null,
        // ...
        provides: parent ? parent.provides : Object.create(appContext.provides),
        // ...
    }
    instance.root = parent ? parent.root : instance;
    // ...
    return instance;
}
```

// APP
emptyAppContext.provides = Object.create(null)

// 
instance.provides = Object.create(Object.create(null))

`getCurrentInstance`

`getCurrentInstance` 支持访问内部组件实例，用于高阶用法或库的开发。

```js
import { getCurrentInstance } from 'vue'

const MyComponent = {
  setup() {
    const internalInstance = getCurrentInstance()

    internalInstance.appContext.config.globalProperties // 访问 globalProperties
  }
}
```

## 其他基本和Vuex3.x版本没什么区别

## `Vuex 4`其他能力

核心模块导出了 `createLogger` 函数

```js
import { createLogger } from 'vuex'
```

## 总结

本文总有讲述了`Vuex4`把`Store`实例注入到各个组件中的原理，展开讲述了`Vuex4`相对与`Vuex3`安装方式的改变`Vuex.createStore`、`app.use(store)` ，深入源码分析`Vue.inject`、`Vue.provide`实现原理。

TODO:
- [ ] 原型图
- [ ] 补devtools图
- [ ] 补组件图