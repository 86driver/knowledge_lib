## vuex和单纯的全局对象的区别

- Vuex 的状态存储是响应式的。当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变
    化，那么相应的组件也会相应地得到高效更新。
- 你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我 们更好地了解我们的应用。



## vuexInit

vuex store 在组件的注入其实就是通过全局混入的方式， 在 `beforecreated`生命钩子中注入的，注入函数为 `vuexInit`

```js
function vuexInit () {
	const options = this.$options // store injection
  // 下面的代码保证 this.$store 访问的是同一个对象
	if (options.store) {
		this.$store = typeof options.store === 'function' ? options.store() : options.store
	} else if (options.parent && options.parent.$store) { 
    this.$store = options.parent.$store
	} 
}
```



## vuex 插件

Vuex的插件：是通过一个数组维护的， 数组里面定义的是一个一个的函数， 函数的参数是 store实例，然后通过在该函数里面调用 store.subscribeAction函数向 `store._subscribers`数组中插入回调函数， 然后 vuex 会在 commit或者 dispatch的时候会去循环执行 `_subscribers`中存在的钩子函数。（这就是vuex的插件模式 - -）

## xxx