## Vue的选项配置合并

### el的合并

`el`提供一个在页面上已存在的 `DOM` 元素作为 `Vue` 实例的挂载目标,因此它只在创建`Vue`实例才存在，在子类或者子组件中无法定义`el`选项，因此`el`的合并策略是在保证选项只存在于根的`Vue`实例的情形下使用默认策略进行合并。

### data的合并

合并的原则是，将父类的数据整合到子类的数据选项中， 如若父类数据和子类数据冲突时，保留子类数据。如果对象有深层嵌套，则需要递归调用`mergeData`进行数据合并

FAQ:

Q：为什么`Vue`组件的`data`是一个函数，而不是一个对象呢？

A：**避免造成数据污染**， 因为如果data是一个对象的时候， 子组件在合并data的时候合并的是一个引用， 改变引用会造成数据的污染，为了复用原则， 应该返回一个函数创建的副本，这样才不会造成组件之前的数据污染。

### 自带资源选项合并

对于 `directives、filters` 以及 `components` 等资源选项，父类选项将以原型链的形式被处理。子类必须通过原型链才能查找并使用内置组件和内置指令。

### 生命周期钩子函数的合并

1. 如果子类和父类都拥有相同钩子选项，则将子类选项和父类选项合并。
2. 如果父类不存在钩子选项，子类存在时，则以数组形式返回子类钩子选项。
3. 当子类不存在钩子选项时，则以父类选项返回。
4. 子父合并时，是将子类选项放在数组的末尾，这样在执行钩子时，永远是父类选项优先于子类选项执行。



> 如果同时存在 extend和mixin：mixin 的生命周期比extend的生命周期先执行,  全局的先执行, 实例本身的生命周期最后执行



### watch选项合并

对于watch选项的合并，最终和父类选项合并成数组，并且数组的选项成员，可以是回调函数，选项对象，或者函数名。

⚠️： 父类watch先执行

示例：

```js
var Parent = Vue.extend({
  watch: {
    'test': function() {
      console.log('parent change')
    }
  }
})
var Child = Parent.extend({
  watch: {
    'test': {
      handler: function() {
        console.log('child change')
      }
    }
  },
  data() {
    return {
      test: 1
    }
  }
})
var vm = new Child().$mount('#app');
vm.test = 2;
// 输出结果
parent change
child change

```



### props methods inject computed合并

源码的设计将`props.methods,inject,computed`归结为一类，他们的配置策略一致，简单概括就是，如果父类不存在选项，则返回子类选项，子类父类都存在时，用子类选项去覆盖父类选项。











## [Vue 组件的注册](https://ustbhuangyi.github.io/vue-analysis/v2/components/component-register.html#全局注册)

- 全局注册
- 局部注册
- 异步组件的注册







##  深入响应式原理

相关文章：

https://blog.csdn.net/hf872914334/article/details/89098295

https://blog.csdn.net/hf872914334/article/details/105620653

https://blog.csdn.net/swallowblank/article/details/107542882

### initState

通过这个方法对props, data, computed, methods, watch进行初始化

1. initProps

   `props` 的初始化主要过程，就是遍历定义的 `props` 配置。遍历的过程主要做两件事情：一个是调用 `defineReactive` 方法把每个 `prop` 对应的值变成响应式，可以通过 `vm._props.xxx` 访问到定义 `props` 中对应的属性。另一个是通过 `proxy` 把 `vm._props.xxx` 的访问代理到 `vm.xxx` 上

2. initData

   `data` 的初始化主要过程也是做两件事，一个是对定义 `data` 函数返回对象的遍历，通过 `proxy` 把每一个值 `vm._data.xxx` 都代理到 `vm.xxx` 上；另一个是调用 `observe` 方法观测整个 `data` 的变化，把 `data` 也变成响应式，可以通过 `vm._data.xxx` 访问到定义 `data` 返回函数中对应的属性。

3. initComputed

4. initMethods

5. initWatch

### [三种watcher](https://blog.csdn.net/hf872914334/article/details/105620653)

1. normal-watcher(watch中的数据，通过initWatch生成) 记录在_watchers中(所有的watch都会存放在这里
2. computed-watcher(computed) 记录在_computedWatchers中 
3. render-watcher 就是vm._watcher

### 依赖收集

在initState的时候对props, data, computed, methods, watch进行响应式初始化。

在mount的阶段实例化一个渲染watcher, 把 `vm._update(vm._render(), hydrating)` 作为 `expOrFn` 传给渲染watcher, 渲染watcher在构造函数中会执行get方法，接着会执行`value = this.getter.call(vm, vm)`, 这里的getter就是vm._update(vm._render(), hydrating)， 所以会去执行 _render(), 方法，在render()方法中会去访问对应的属性，因为之前进行过initState， 对应属性都被响应式初始化， 即有getter， 在每个属性的getter中会触发 `dep.depend()`, 这个	`dep.depend()` 就是进行依赖收集的。

### 派发更新

当我们在组件中对响应的数据做了修改，就会触发 setter 的逻辑，最后调用 `dep.notify()` 方法， 在 `dep.notify()`方法中遍历所有的 `subs`，也就是 `Watcher` 的实例数组，然后调用每一个 `watcher` 的 `update` 方法，在`update` 方法中，一般组件数据更新的场景，会走到最后一个 `queueWatcher(this)` 的逻辑，它并不会每次数据改变都触发 `watcher` 的回调，而是把这些 `watcher` 先添加到一个队列里，然后在 `nextTick` 后执行 `flushSchedulerQueue`

flushSchedulerQueue方法会执行 `watcher.run()`,

对于渲染 `watcher` 而言，它在执行 `this.get()` 方法求值的时候，会执行 `getter` 方法：

```js
updateComponent = () => {
  vm._update(vm._render(), hydrating)
}
```

所以这就是当我们去修改组件相关的响应式数据的时候，会触发组件重新渲染的原因。

### Dep

```js
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

```

Dep 扮演的角色是调度中心/订阅器，主要的作用就是收集观察者Watcher和通知观察者目标更新

有个<watcher>sub数组，用来存放依赖对应的订阅者，

target: 观察者，这是全局唯一的，因为在任何时候只有一个观察者被处理; (可以看成任意一个响应式属性被访问和修改时都会激活一个watcher, 这时Dep.target =watcher， 此时的sub都是对应这个属性的)

depend: 依赖收集，如果当前有观察者，将该dep放进当前观察者的deps中, 同时，将当前观察者放入观察者列表subs中;

(在getter中中触发,通知当前激活的watcher添加对应依赖的订阅)

addSub： 向dep的观察者列表subs添加观察者

removeSub： 移除观察者（v-if.....）

notify： 通知订阅者派发更新.(循环处理，运行每个观察者的update接口)

Watcher， 对应属性的观察者， 一个属性可能对应多个观察者

Watcher扮演的角色是订阅者/观察者，他的主要作用是为观察属性提供回调函数以及收集依赖（如计算属性computed，vue会把该属性所依赖数据的dep添加到自身的deps中），当被观察的值发生变化时，会接收到来自dep的通知，从而触发回调函数

### nextTick

nextTick: 可以看作是本次微任务之后，下一次宏任务之前执行。

源码中可以缩减为：

```js
let useMacroTask = false
// ...省略部分代码

export function nextTick (cb?: Function, ctx?: Object) { 
  // ......省略部分代码
  if (useMacroTask) {
    macroTimerFunc()
  } else {
    microTimerFunc()
  }
}
```



- macroTimerFunc

  宏任务， 

  `macroTimerFunc = setImmediate ？setImmediate : (MessageChannel?MessageChannel:setTimeout)` 

- microTimerFunc 

  微任务，

  `macroTimerFunc = Promise ? Promise : macroTimerFunc`





## 计算属性 VS 侦听属性

### 计算属性computed （本质上是一个computed watcher）

在initComputed(vm, opts.computed) 进行初始化

- 特点

  一旦我们对计算属性依赖的数据做修改，则会触发 setter 过程，通知所有订阅它变化的 `watcher` 更新，执行 `watcher.update()`，

  触发渲染 `watcher` 重新渲染

### 侦听属性 watch

在 initWatch(vm, opts.watch) 方法中进行初始化

- 特点

  实例化了一个 `watcher`，这里需要注意一点这是一个 `user watcher`，因为 `options.user = true`。通过实例化 `watcher` 的方式，一旦我们 `watch` 的数据发送变化，它最终会执行 `watcher` 的 `run` 方法，执行回调函数 `cb`，并且如果我们设置了 `immediate` 为 true，则直接会执行回调函数 `cb`。最后返回了一个 `unwatchFn` 方法，它会调用 `teardown` 方法去移除这个 `watcher`。



## 组件更新

组件重新渲染也是调用vm._update方法

区别是：根据新旧 `vnode` 是否为 `sameVnode`，会走到不同的更新逻辑

- 新旧节点不同

  以当前旧节点为参考节点创建新节点 -> 更新父的占位节点 -> 删除旧节点

- [新旧节点相同](https://ustbhuangyi.github.io/vue-analysis/v2/reactive/component-update.html#%E6%96%B0%E6%97%A7%E8%8A%82%E7%82%B9%E7%9B%B8%E5%90%8C)

  调用`patchVnode` 把新的 `vnode` `patch` 到旧的 `vnode` 上

总结：

组件更新的过程核心就是新旧 vnode diff，对新旧节点相同以及不同的情况分别做不同的处理。新旧节点不同的更新流程是创建新节点->更新父占位符节点->删除旧节点；而新旧节点相同的更新流程是去获取它们的 children，根据不同情况做不同的更新逻辑。最复杂的情况是新旧节点相同且它们都存在子节点，那么会执行 `updateChildren` 逻辑，这块儿可以借助画图的方式配合理解。



## Props

在 initProps(vm, opts.props) 中初始化，`initProps` 主要做 3 件事情：校验、响应式和代理。

- 子组件props是怎么更新的？

  在父组件重新渲染的最后，会执行 `patch` 过程，进而执行 `patchVnode` 函数，`patchVnode` 通常是一个递归过程，当它遇到组件 `vnode` 的时候，会执行组件更新过程的 `prepatch` 钩子函数， `prepatch`钩子函数中会执行`updateChildComponent`函数去更新子组件

- 子组件重新渲染

  子组件的重新渲染有 2 种情况，一个是 `prop` 值被修改，另一个是对象类型的 `prop` 内部属性的变化。

  先来看一下 `prop` 值被修改的情况，当执行 `props[key] = validateProp(key, propOptions, propsData, vm)` 更新子组件 `prop` 的时候，会触发 `prop` 的 `setter` 过程，只要在渲染子组件的时候访问过这个 `prop` 值，那么根据响应式原理，就会触发子组件的重新渲染。

  再来看一下当对象类型的 `prop` 的内部属性发生变化的时候，这个时候其实并没有触发子组件 `prop` 的更新。但是在子组件的渲染过程中，访问过这个对象 `prop`，所以这个对象 `prop` 在触发 `getter` 的时候会把子组件的 `render watcher` 收集到依赖中，然后当我们在父组件更新这个对象 `prop` 的某个属性的时候，会触发 `setter` 过程，也就会通知子组件 `render watcher` 的 `update`，进而触发子组件的重新渲染。



## 扩展

###  [event](https://ustbhuangyi.github.io/vue-analysis/v2/extend/event.html#%E8%87%AA%E5%AE%9A%E4%B9%89%E4%BA%8B%E4%BB%B6)

- 原生DOM事件：

  调用原生`addEventListener` 进行监听

- 组件自定义事件

  维护了一个 vm._events数组，通过 `vm.$on` 向`vm._events`数组里面添加回调函数， 通过 `vm.$emit` 从 `vm._events`取出回调函数依次执行

  ⚠️： vm.$emit 是给当前的 vm 上派发的实例，之所以我们常用它做父子组件通讯，是因为它的回调函数的定义是在父组件中，对于我们这个例子而言，当子组件的 被点击了，它通过
  this.$emit('select') 派发事件，那么子组件的实例就监听到了这个 事件，并执行它的回调函数——定义在父组件中的 selectHandler 方法，这样就相当于完成了一次父子组件的通讯。

  **（个人理解：也就是说 $on, $emit操作的_event都是当前子组件VNode,  然后执行回调的时候是执行父组件上定义的方法）**

  证明：

  ```vue
  <!-- 父组件 -->
  <template>
    <div id="app">
      <HelloWorld @select="select"/>
    </div>
  </template>
  
  <script>
  import HelloWorld from "./components/HelloWorld";
  
  export default {
    name: "App",
    components: {
      HelloWorld
    },
    mounted() {
      console.log(this._events) // {}
    },
    methods: {
      select() {}
    }
  };
  </script>
  
  <!-- 子组件 -->
  <template>
    <div class="hello"></div>
  </template>
  
  <script>
  export default {
    name: "HelloWorld",
  
    mounted() {
      console.log(this._events) // {select: Array(1)}
    },
  };
  </script>
  
  
  ```

  

- 总结：

  Vue 支持 2 种事件类型，原生 DOM 事件和自定义事件，它们主要的区别在于添加和删除事件的方式不一样，并且自定义事件的派发是往当前实例上派发，但是可以利用在父组件环境定义回调函数来实现父子组件的通讯。另外要注意一点，只有组件节点才可以添加自定义事件，并且添加原生 DOM 事件需要使用 `native` 修饰符；而普通元素使用 `.native` 修饰符是没有作用的，也只能添加原生 DOM 事件。



### v-model

v-mode：双向绑定，双向绑定除了数据驱动 DOM 外， DOM 的变化反过来影响数据，是一个双向关系

`v-model` 即可以作用在普通表单元素上，又可以作用在组件上，它其实是一个语法糖

表单示例：

```vue
<input v-model="message" />
<!-- 
	v-model实际上是：
		addProp(el, 'value', `(${value})`) 
		addHandler(el, event, code, null, true)
	的语法糖。
	上面的代码实际上会转换成下面的代码,
-->

<input v-bind:value="message" v-on:input="message=$event.target.value">



<!-- 
	其实就是动态绑定了 input 的 value 指向了 messgae 变量，并且在触发 input 事件的时候去动态把 message 设置为目标值，这样实际	上就完成了数据双向绑定了，所以说 v-model 实际上就是语法糖。
-->
```



作用于组件上的示例：

```js
let Child = {
  template: '<div>'
  + '<input :value="value" @input="updateValue" placeholder="edit me">' +
  '</div>',
  props: ['value'],
  methods: {
    updateValue(e) {
      this.$emit('input', e.target.value)
    }
  }
}

let vm = new Vue({
  el: '#app',
  template: '<div>' +
  '<child v-model="message"></child>' +
  '<p>Message is: {{ message }}</p>' +
  '</div>',
  data() {
    return {
      message: ''
    }
  },
  components: {
    Child
  }
})

/** ************************************** **/

/**
 当v-model作用在组件上时，实际上就是在组件上绑定了 props:[value]和 `@input`组件自定义事件。
 在组件上使用 v-model中经过了  `transformModel` 函数， 通过这个函数可以看到组件上的v-model的 value和input是可配置的，
 通过 options.model配置
**/

// transformModel源码方法：
function transformModel (options, data: any) {
  const prop = (options.model && options.model.prop) || 'value';
  const event = (options.model && options.model.event) || 'input';
}
```



### slot（插槽）

插槽分为普通插槽和作用域插槽：它们有一个很大的差别是数据作用域，普通插槽是在父组件编译和渲染阶段生成 vnodes ，所以数据的作用域是父组件实例，子组件渲染的时候直接拿到这些渲染好的。而对于作用域插槽，父组件在编译和渲染阶段并不会直接 生成 vnodes ，而是在父节点   的   中保留一个 scopedSlots 对象，存储着不同名称 的插槽以及它们对应的渲染函数，只有在编译和渲染子组件阶段才会执行这个渲染函数生成
vnodes ，由于是在子组件环境执行的，所以对应的数据作用域是子组件实例。
简单地说，两种插槽的目的都是让子组件 slot 占位符生成的内容由父组件来决定，但数据的作用域 会根据它们 vnodes 渲染时机不同而不同。

### keep-alive

#### props:

- include:

  需要缓存的组件

- exlude:

  不需要缓存的组件

- max:

  最大缓存数量

#### keep-alive生命周期

第一次执行的时候跟普通的组件没什么区别， 当被缓存之后再次执行的时候 `created, mounted`等钩子函数不会触发，但是会触发 `keep-alive`独有的生命周期钩子函数**`activated`**,触发时机是组件重新被渲染的时候

#### 缓存规则

如果命中缓存，则直接从缓存中拿 vnode 的组件实例，并且重新调整了 key 的 顺序放在了最后一个;否则把 vnode 设置进缓存，最后还有一个逻辑，如果配置了 max 并且缓存 的⻓度超过了 this.max ，还要从缓存中删除第一个。除了从缓存中删除外，还要判断如果要删除的缓存并的组件 tag 不是当前渲染组件 tag ，也执行 删除缓存的组件实例的 $destroy 方法

缓存代码：

```js
  render () {
    const slot = this.$slots.default
    const vnode: VNode = getFirstComponentChild(slot)
    const componentOptions: ?VNodeComponentOptions = vnode && vnode.componentOptions
    if (componentOptions) {
      // check pattern
      const name: ?string = getComponentName(componentOptions)
      const { include, exclude } = this
      if (
        // not included
        (include && (!name || !matches(include, name))) ||
        // excluded
        (exclude && name && matches(exclude, name))
      ) {
        return vnode
      }
      
      
      // 以下部分是缓存代码

      const { cache, keys } = this
      const key: ?string = vnode.key == null
        // same constructor may get registered as different local components
        // so cid alone is not enough (#3269)
        ? componentOptions.Ctor.cid + (componentOptions.tag ? `::${componentOptions.tag}` : '')
        : vnode.key
      if (cache[key]) {
        vnode.componentInstance = cache[key].componentInstance
        // make current key freshest
        remove(keys, key)
        keys.push(key)
      } else {
        // delay setting the cache until update
        this.vnodeToCache = vnode
        this.keyToCache = key
      }

      vnode.data.keepAlive = true
    }
    return vnode || (slot && slot[0])
  }
```



#### include和exclude优先级判断：

```js
if (
  // not included
  (include && (!name || !matches(include, name))) ||
  // excluded
  (exclude && name && matches(exclude, name))
) {
  return vnode
}

// 符合以上条件的都不缓存
// 规则：满足了配置include且不匹配name或者是配置了 exclude 且匹配name，那么就直接返回这个组件的 vnode
// include在前说明 include优先级高一些
```



#### keep-alive watch

在keep-alive方法中使用vm.$watch手动侦听了 `include` 和 `exclude`

```typescript
watch: {
  include (val: string | RegExp | Array<string>) {
    pruneCache(this, name => matches(val, name))
  },
  exclude (val: string | RegExp | Array<string>) {
    pruneCache(this, name => !matches(val, name))
	} 
}

function pruneCache (keepAliveInstance: any, filter: Function) { 
  const { cache, keys, _vnode } = keepAliveInstance
	for (const key in cache) {
    const cachedNode: ?VNode = cache[key]
    if (cachedNode) {
      // getComponentName, 获取组件 options.name
			const name: ?string = getComponentName(cachedNode.componentOptions) 
      if (name && !filter(name)) {
        // pruneCacheEntry： 移除缓存
				pruneCacheEntry(cache, key, keys, _vnode) 
      }
		} 
  }
}

// 总结：
/**
	逻辑很简单，观测他们的变化执行 pruneCache 函数，其实就是对 cache 做遍历，发现缓存的节点名称和新的规则没有匹配上的时候，就		把这个缓存节点从缓存中摘除
**/
```



#### 渲染

- 首次渲染

  建立缓存， 其他跟普通组件一样渲染

- 缓存渲染

  触发 `$forceUpdate`，

  执行 keep-alive自己定义的render函数， render函数执行完之后，执行 **`activated`**生命钩子函数

  

### [transition](https://juejin.cn/post/6844903858611683336)

内置render函数， 只允许有一个子节点(没有同级子节点)， 这就是为什么`<transition><p>xx</p><p>xxx</p></transition>`会警告的原因，然后递归子节点， 找到第一个非抽象组件的子节点作为vnode返回

`created`钩子函数只有 当节点的创建过程才会执行，而`remove`会在节点销毁的时候执行，这也就印证了
必须要满足 v-if 、动态组件、组件根节点条件之一了。

过渡动画提供了 2 个时机，一个是 create 和 activate 的时候提供了 entering 进入动画，一个是 remove 的时候提供了 leaving 离开动画，那么接下来我们就来分别去分析这两个过程。



总结

- Vue 的过渡实现分为以下几 个步骤:

  1. 自动嗅探目标元素是否应用了 CSS 过渡或动画，如果是，在恰当的时机添加/删除 CSS 类名。
  2. 如果过渡组件提供了 JavaScript 钩子函数，这些钩子函数将在恰当的时机被调用。
  3. 如果没有找到 JavaScript 钩子并且也没有检测到 CSS 过渡/动画，DOM 操作 (插入/删除) 在下一帧 中立即执行。   

  所以真正执行动画的是我们写的 CSS 或者是 JavaScript 钩子函数，而 Vue 的 <transition> 只是帮我 们很好地管理了这些 CSS 的添加/删除，以及钩子函数的执行时机。

  

  个人总结：

  `	<transition></transition>` 内置组件自己实现render函数， 用途是来找到第一个非抽象组件的子节点作为vnode返回，

  然后通过在节点创建(created)、节点移除阶段(remove, transition有这个钩子 - -)插入自定义钩子函数，然后通过执行对应的钩子函数和动态改变节点`class`属性来实现动画的过程

  

### transition-group

参考<transition></transition>,

不同点：

transition-group组件不是抽象组件， 即意味着该组件会渲染成一个真实元素， 通过	`move` 实现过渡动画

## vue的抽象组件

用法： 设置`options.abstract = true`

概述：

- abstract:true 表明该组件是抽象组件
- 抽象组件没有真实的节点，在组件渲染的时候不会解析渲染成真实的dom节点，而只是作为中间的数据过度层处理
- 组件初始化的时候父子组件会显示的建立一层关键，这层关系点顶了父子组件的通信基础

功能：

- 在抽象组件的生命周期中，我们可以拦截包的子组件监听的事件，或者对子组件做Dom操作，这样我们就可以封装我们需要的功能，而不用关心子组件的具体实现



## xxxx

















