![流程图](https://image-static.segmentfault.com/309/446/3094460228-7e6ede65f143e2c6)



相关文章：

- https://www.cnblogs.com/caizhenbo/p/7297730.html



## 路由注册

### Vue.use

执行`Vue.use(plugin)` 实际上就是在执行 `plugin.install`方法

- install方法

  - installed属性：

    确保install逻辑只执行一次， 即插件只注册一次

  - mixin

    ```js
    Vue.mixin({
        beforeCreate () {
          if (isDef(this.$options.router)) {
            this._routerRoot = this
            this._router = this.$options.router
            this._router.init(this)
            Vue.util.defineReactive(this, '_route', this._router.history.current)
          } else {
            this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
          }
          registerInstance(this, this)
        },
        destroyed () {
          registerInstance(this)
        }
      })
    ```

    `vue-router	` 最重要的就是使用全局混入， 给每一个组件上都注入响应式的 `_router, _route` ， 并初始化路由(init)

    在`beforeCreate`生命钩子初始化

  - registerInstance

    定义在`router-view`组件中， 用来注册组件





## VueRouter对象

- 在install 的 beforeCreate 调用 `init` 方法根据配置信息进行路由初始化和定义方法



## matcher

- 通过 `createRouteMap`创建路由映射表

- match

  返回的是一个路径，它的作用是根据传入的 `raw` 和当前路径`currentRoute`计算出一个新的路径并返回（目标路径）

- addRoutes

  动态添加路由配置，（其实就是调用`createRouteMap`向映射表中添加记录）





## 路径切换

- history.transitionTo

  - 调用 `ensureURL`  更新 url地址栏

    `hash`模式通过`window.location.hash = path 或者 window.location.replace(getUrl(path))`	来个更新url

    `history`通过	`window.history.putState`来更新url, `pushState` 会调用浏览器原生的 history 的 pushState 接口或者 replaceState 接口，更新 浏览器的 url 地址，并把当前 url 压入历史栈中。

    在	`history` 的初始化中会监听window.history的变化， 手动点击浏览器的前进和后退就会调用history.transitionTo方法来触发视图更新

  - 执行钩子函数

    通过构造一个 `queue` 队列，定义一个 `iterator` 迭代器。 通过递归执行`queue` ，在`iterator`的**对应位置**调用导航守卫钩子函数，执行显式的执行 `next` 才会到下一个导航守卫钩子函数中

    **对应位置**：

    1. 在失活的组件里调用离开守卫。
    2. 调用全局的 beforeEach 守卫。
    3. 在重用的组件里调用 beforeRouteUpdate 守卫 。
    4. 在激活的路由配置里调用 beforeEnter 。
    5. 解析异步路由组件。

    拿到激活的异步组件之后：

    1. 在被激活的组件里调用 beforeRouteEnter 。

    2.  调用全局的 beforeResolve 守卫。

    3. 调用全局的 afterEach 钩子。

       

- router-view

  通过 `registerInstance` 和 当前 `route` 函数找到要渲染的组件并进行渲染,

  > 由于我们把根 Vue 实例的 `_route`属性定义成响应式的，我们在每个 <router-view> 执行`render`
  > 函数的时候，都会访问`parent.$route`, 从而会访问到 `this._royterRoot._route`(在install中getter被劫持)，触发它的getter,相当于<router-view> 对它有依赖，然后再执行完  `transitionTo` 后，修改`app._route`的时候，又触发了`setter`,因此会通知`<router-view>`的渲染`watcher`更新， 重新渲染组件





## xxxx