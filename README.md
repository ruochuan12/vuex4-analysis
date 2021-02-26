# vuex4-analysis

学习源码整体架构系列8篇之vuex4源码，前端面试高频源码，微信搜索「若川视野」关注我，长期交流学习~

## 视野

## 安装方式改变

### app.provide


### app.provide

```js
import { createApp } from 'vue'
import App from './components/App.vue'
import store from './store'
import { currency } from './currency'

const app = createApp(App)

app.use(store)

app.mount('#app')
```

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
},
```

```js
export class Store{
    // ...
    install (app, injectKey) {
        app.provide(injectKey || storeKey, this)
        app.config.globalProperties.$store = this
    }
}
```

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

```js
```

app.config.globalProperties

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

## 监测数据

```js

```

## Vuex.createStore

## Vuex.useStore

## 其他基本和Vuex3.x版本没什么区别