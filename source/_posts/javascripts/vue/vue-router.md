---
title: Vue Router 食用指南
date: '2021-12-29 14:16'
categories:
  - - JavaScript
    - Vue
tags:
  - Vue
  - 原理
abbrlink: 217af96f
---

### 1. 起步
1. 使用模块化，要调用 Vue.use(VueRouter)
2. 注入路由器，任何组件都可通过 this.$router 访问路由器，this.$route 访问当前路由

![1586929561190.png](https://cdn.nlark.com/yuque/0/2020/png/448234/1586942111407-979d831d-2d0a-4e1c-9b20-1688f38fd04f.png#align=left&display=inline&height=408&name=1586929561190.png&originHeight=408&originWidth=696&size=44854&status=done&style=none&width=696)
### 2. 动态路由匹配

1. 路径参数使用 ：冒号标记，当匹配到一个路由，参数会被设置到 this.$route.params

![1586930691665.png](https://cdn.nlark.com/yuque/0/2020/png/448234/1586942187698-19783596-ccd7-42a3-b582-ad71add0b83b.png#align=left&display=inline&height=169&name=1586930691665.png&originHeight=169&originWidth=297&size=10752&status=done&style=none&width=297)

2. 动态路由的参数改变（id 改变）,例如从 /user/111 到 /user/222 ，原来的组件会被复用，意味着组件的生命周期钩子函数不会重复调用，如想复用，可用以下方法
- 用 watch 监测变化


```javascript
watch:{
        '$route'(to,from){}
    }
```

- 使用2.2的 beforeRouteUpdate 导航守卫
```javascript
beforeRouteUpdate(to, from, next) {
    next()
  }
```

3. 捕获所有路由或 404 Not found 路由
```javascript
{path:'*'} // 匹配所有路径，放在最后
{path:'/user-*'} // 匹配'/user-'开头的任意路径，放在后面
```
### 3. 嵌套路由

1. 父组件包含自己的嵌套 <router-view />, VueRouter 的参数使用 children 配置
### 4. 编程式的导航
![1586935811809.png](https://cdn.nlark.com/yuque/0/2020/png/448234/1586942936614-865b790d-d9cb-4acf-b511-0e08b03f3e5a.png#align=left&display=inline&height=97&name=1586935811809.png&originHeight=97&originWidth=407&size=4006&status=done&style=none&width=407)

1. 使用 router.push() ，这个方法会向 history 栈添加一个新的记录，所以当用户点击回退按钮，会回到之前的 URL。
2. router.go(n) n为整数，意思是在 history 记录中前进或后退几步，类似 window.history.go(n)
> `router.push`、 `router.replace` 和 `router.go` 跟 [`window.history.pushState`、 `window.history.replaceState` 和 `window.history.go`](https://developer.mozilla.org/en-US/docs/Web/API/History)好像， 实际上它们确实是效仿 `window.history` API 的。

### 5. 命名路由
在 routes 配置中给路由设置名称
```javascript
const router = new VueRouter({
  routes: [
    {
      path: '/user/:userId',
      name: 'user',
      component: User
    }
  ]
})
```
### 6. 命名视图
用于同级展示多个视图，并非嵌套展示，如侧导航+主内容的布局
### 7. 重定向和别名
```javascript
// 重定向
{ path: '/a', redirect: '/b' }
// 别名
{ path: '/a', component: A, alias: '/b' }
```
### 8. 导航守卫
用来通过跳转或取消的方式守卫导航，有多种机会植入路由的导航过程中：全局、单个路由独享、组件级。
注意：参数或查询的改变不会触发进入/离开的导航守卫

- router.beforeEach(to,from,next) 全局前置守卫：在导航被确认之前会进行全局拦截，常用于判断用户是否登录
- router.beforeResolve(to,from,next) 全局解析守卫：在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后
- afterEach(to,from) 全局后置钩子：这些钩子不会接受 next 函数也不会改变导航本身
- beforeEnter 路由独享守卫：在路由配置上定义
```javascript
{
  path: '/foo',
  component: Foo,
  beforeEnter: (to, from, next) => {
        // ...
    next()
  }
}
```

- 组件内守卫 beforeRouteEnter、beforeRouteUpdate、beforeRouteLeave

參考：[https://www.jianshu.com/p/59c005511d73](https://www.jianshu.com/p/59c005511d73)
### 9. 路由元信息（meta）
可通过 this.$route 拿到
### 10. 过渡动效
```javascript
// 给所有路由设置
<transition>
  <router-view></router-view>
</transition>
```
如果需要每个路由组件有各自的过渡效果，可在各组件内使用 <transition>
### 11. 数据获取

1. **导航完成之后获取**：先完成导航，后在接下来的组件生命周期钩子中获取数据。在数据获取期间显示“加载中”之类的指示。
2. **导航完成之前获取**：导航完成前，在路由进入的守卫中获取数据，在数据获取成功后执行导航。
### 12. 滚动行为
创建路由实例时，可提供一个 scrollBehavior(to, from, savedPosition) 方法, savedPosition 仅当 popstate 导     航（浏览器前进/后退）时触发。
注意：滚动行为只在支持 history.pushState 的浏览器中可用
### 13. 路由懒加载
Webpack 2 ，可以使用动态 import 语法来定义代码分块点，如果使用 Babel，需要添加 [plugin-syntax-dynamic-import](https://babeljs.io/docs/en/babel-plugin-syntax-dynamic-import/)  插件
```javascript
{ path: '/foo', component: () => import('./Foo.vue') }
```

