![流程图](https://image-static.segmentfault.com/309/446/3094460228-7e6ede65f143e2c6)



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





## VueRouter对象





## xxxx