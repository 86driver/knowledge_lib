## [模块机制](https://juejin.cn/post/6844904030905303054)

**Nodejs遵循的是`CommonJS`规范**

### CommJs规范

- 模块定义

  模块上下文提供了`exports`对象用于导入导出当前模块的方法或者变量，并且它是唯一的导出出口。模块中存在一个`module`对象，它代表模块自身，`exports`是module的属性。**一个文件就是一个模块**，将方法作为属性挂载在exports上就可以定义导出的方式：

  ```js
  //math.js
  exports.add = function () {
      let sum = 0, i = 0, args = arguments, l = args.length;
      while(i < l) {
          sum += args[i++];
      }
      return sum;
  }
  // 这样就可像test.js里那样在require()之后调用模块的属性或者方法了。
  ```

- 模块标识

  模块标识就是传递`给require()`方法的参数，它必须是符合小驼峰命名的字符串，或者以`.`、`..`开头的相对路径或者绝对路径，可以没有文件后缀名`.js`.

### Node 的模块实现

在Node中引入模块，需要经历如下四个步骤:

#### 路径分析

Node.js中模块可以通过文件路径或名字获取模块的引用。**模块的引用会映射到一个js文件路径**。 在Node中模块分为两类

- 一是Node提供的模块，称为**核心模块**（内置模块）

  核心模块是Node源码在编译过程中编译进了二进制执行文件。在Node启动时这些模块就被加载进内存中，所以核心模块引入时省去了文件定位和编译执行两个步骤，并且在路径分析中优先判断，因此核心模块的加载速度是最快的。文件模块则是在运行时动态加载，速度比核心模块慢。

- 另一类是用户编写的模块，称为**文件模块**。

**node模块的载入及缓存机制**

1. 载入内置模块（A Core Module）

   Node的内置模块被编译为二进制形式，引用时直接使用名字而非文件路径。当第三方的模块和内置模块同名时，内置模块将覆盖第三方同名模块。因此命名时需要注意不要和内置模块同名。如获取一个http模块:

   ```js
   const http = require('http')
   // 返回的http即是实现了HTTP功能Node的内置模块。
   ```

2. 载入文件模块（A File Module）

   ```js
   // 绝对路径
   const myMod = require('/home/base/my_mod')
   
   // 相对路径
   const myMod = require('./my_mod')
   ```

3. 载入文件目录模块（A Folder Module）

   可以直接require一个目录，假设有一个目录名为folder，如

   ```js
   const myMod = require('./folder')
   ```

   此时，Node将搜索整个folder目录，Node会假设folder为一个包并试图找到包定义文件package.json。如果folder目录里没有包含`package.json`文件，Node会假设默认主文件为`index.js`，即会加载`index.js`。如果`index.js`也不存在， 那么加载将失败。

4. 载入node_modules里的模块

   如果模块名不是路径，也不是内置模块，Node将试图去当前目录的`node_modules`文件夹里搜索。如果当前目录的`node_modules`里没有找到，Node会从父目录的`node_modules`里搜索，这样递归下去**直到根目录**。

5. 自动缓存已载入模块

   对于已加载的模块Node会缓存下来，而不必每次都重新搜索, 示例：

   ```js
   // modA.js
   console.log('模块modA开始加载...')
   exports = function() {
       console.log('Hi')
   }
   console.log('模块modA加载完毕')
   
   //init.js
   var mod1 = require('./modA')
   var mod2 = require('./modA')
   console.log(mod1 === mod2)
   
   /*** 命令行node init.js执行： ***/
   // 模块modA开始加载...
   // 模块modA加载完毕
   // true
   // 可以看到虽然require了两次，但modA.js仍然只执行了一次。mod1和mod2是相同的，即两个引用都指向了同一个模块对象
   ```

   **优先从缓存加载**:

   和浏览器会缓存静态js文件一样，Node也会对引入的模块进行缓存，不同的是，浏览器仅仅缓存文件，而nodejs缓存的是编译和执行后的对象（**缓存内存**） `require()`对相同模块的二次加载一律采用缓存优先的方式，这是第一优先级的，核心模块缓存检查先于文件模块的缓存检查。

   场景：

   ```js
   /** 
   	基于这点：我们可以编写一个模块，用来记录长期存在的变量。例如：我可以编写一个记录接口访问数的模块：
   **/
   let count = {}; // 因模块是封闭的，这里实际上借用了js闭包的概念
   exports.count = function(name){
        if(count[name]){
             count[name]++;
        }else{
             count[name] = 1;
        }
        console.log(name + '被访问了' + count[name] + '次。');
   };
   // 我们在路由的 action 或 controller里这样引用:
   let count = require('count');
   
   export.index = function(req, res){
       count('index');
   };
   // 以上便完成了对接口调用数的统计(不考虑服务器重启)
   ```

   

#### 文件定位

- 文件扩展名分析

  调用`require()`方法时若参数没有文件扩展名，Node会按`.js`、`.json`、`.node`的顺寻补足扩展名，依次尝试。

  在尝试过程中，需要调用**fs模块阻塞式**地判断文件是否存在。因为Node的执行是单线程的，这是一个会引起性能问题的地方。如果是`.node`或者·.json·文件可以加上扩展名加快一点速度。另一个诀窍是：**同步配合缓存**。

- 目录分析和包

  `require()`分析文件扩展名后，可能没有查到对应文件，而是找到了一个目录，此时Node会将目录当作一个包来处理。

  首先， Node在当前目录下查找`package.json`，通过`JSON.parse()`解析出包描述对象，从中取出main属性指定的文件名进行定位。若main（有可能是module，看具体情况）属性指定文件名错误，或者没有`pachage.json`文件，Node会将index当作默认文件名。

  简而言之，如果`require`绝对路径的文件，查找时不会去遍历每一个`node_modules`目录，其速度最快。其余流程如下：

  1. 从`module path`数组中取出第一个目录作为查找基准。

  2. 直接从目录中查找该文件，如果存在，则结束查找。如果不存在，则进行下一条查找。

  3. 尝试添加`.js`、`.json`、`.node`后缀后查找，如果存在文件，则结束查找。如果不存在，则进行下一条。

  4. 尝试将`require`的参数作为一个包来进行查找，读取目录下的`package.json`文件，取得`main`参数指定的文件。

  5. 尝试查找该文件，如果存在，则结束查找。如果不存在，则进行第3条查找。

  6. 如果继续失败，则取出`module path`数组中的下一个目录作为基准查找，循环第1至5个步骤。

  7. 如果继续失败，循环第1至6个步骤，直到module path中的最后一个值。

  8. 如果仍然失败，则抛出异常。

  **一旦加载成功就以模块的路径进行缓存**

  示例图：

  ![](https://user-gold-cdn.xitu.io/2019/12/25/16f38d1d79c552e0?imageView2/0/w/1280/h/960/webp/ignore-error/1)

#### 编译执行

每个模块文件模块都是一个对象，它的定义如下:

```js
function Module(id, parent) {
    this.id = id;
    this.exports = {};
    this.parent = parent;
    if(parent && parent.children) {
        parent.children.push(this);
    }
    this.filename = null;
    this.loaded = false;
    this.children = [];
}

```

对于不同扩展名，其载入方法也有所不同：

- `.js`通过fs模块同步读取文件后编译执行。
- `.node`这是C/C++编写的扩展文件，通过`dlopen()`方法加载最后编译生成的文件
- `.json`同过fs模块同步读取文件后，用`JSON.pares()`解析返回结果

其他当作`.js`

每一个编译成功的模块都会将其文件路径作为索引缓存在`Module._cache`对象上。

**`json` 文件的编译**

```js
.json`文件调用的方法如下:其实就是调用`JSON.parse
```



**`js`模块的编译** 在编译的过程中，Node对获取的javascript文件内容进行了头尾包装，将文件内容包装在一个function中：

```
(function (exports, require, module, __filename, __dirname) {
    var math = require(‘math‘);
    exports.area = function(radius) {
       return Math.PI * radius * radius;
    }
})
复制代码
```

包装之后的代码会通过vm原生模块的`runInThisContext()`方法执行（具有明确上下文，不污染全局），返回一个具体的function对象，最后传参执行，执行后返回`module.exports`.

**runInThisContext** ???

**核心模块编译**

核心模块分为`C/C++`编写和JavaScript编写的两个部分，其中`C/C++`文件放在Node项目的src目录下，JavaScript文件放在lib目录下。



#### 加入内存



### import 和 require

`import`是ES6的模块规范，`require`是commonjs的模块规范

**import是静态加载模块，require是动态加载**



### Node清除`require`缓存

```js
delete require.cache[require.resolve('./server.js')]; // 清除对./server.js的文件引用缓存
app = require('./server.js');
```



## [require原理](http://www.ruanyifeng.com/blog/2015/05/require.html) (直接看链接)

### require的基本用法

### Moudle

### 模块实例的 require 方法

### 模块的绝对路径

### 加载模块



## [Node事件循环](https://learnku.com/articles/38802)

相关文章:

https://www.cnblogs.com/forcheng/p/12723854.html

https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/ (官方文档)

流程图：

> ​     ┌───────────────────────────┐
> ┌>│           								 timers        								  │
> │  └─────────────┬─────────────┘
> │  ┌─────────────┴─────────────┐
> │  │ 							     pending callbacks 							     │
> │  └─────────────┬─────────────┘
> │  ┌─────────────┴─────────────┐
> │  │      								 idle, prepare     							    │
> │  └─────────────┬─────────────┘      ┌───────────────┐
> │  ┌─────────────┴─────────────┐      │  				 incoming:   				│
> │  │       		                           	  poll            					  		│<---┤  				connections, 			   │
> │  └─────────────┬─────────────┘      │   				data, etc. 				   │
> │  ┌─────────────┴─────────────┐      └───────────────┘
> │  │        								   check                                           │
> │  └─────────────┬─────────────┘
> │  ┌─────────────┴─────────────┐
> └─┤     							 close callbacks                                     │
> ​      └───────────────────────────┘

### 阶段概述

- **timers**： 此阶段执行由 setTimeout 和 setInterval 设置的回调
- **pending callbacks**：执行推迟到下一个循环迭代的 `I/O` 回调。 （在poll阶段， 有些I/0不立即执行，而是延时执行，就到了这里）
- **idle, prepare**：只在内部使用（那就忽略吧）
- **poll**： 取出新完成的 `I/O` 事件；执行与 `I/O` 相关的回调（除了关闭回调，计时器调度的回调和 `setImmediate` 之外，几乎所有这些回调） 适当时，`node` 将在此处阻塞。
- **check**：在这里调用 `setImmediate` 回调。
- **close callbacks  **：一些关闭回调，例如 `socket.on('close', ...)`。

在每次事件循环运行之间，Node.js 会检查它是否正在等待任何异步 `I/O` 或 `timers`，如果没有，则将其干净地关闭。

⚠️：轮询阶段具有两个主要功能：

计算应该阻塞并 I/O 轮询的时间
处理轮询队列 (poll queue) 中的事件
当事件循环进入轮询 (poll) 阶段并且没有任何计时器调度 (setTimeout, setInterval) 时，将发生以下两种情况之一：

- 如果轮询队列 (poll queue) 不为空，则事件循环将遍历其回调队列，使其同步执行，直到队列用尽或达到与系统相关的硬限制为止 (到底是哪些硬限制？)。
- 如果轮询队列为空，则会发生以下两种情况之一：
  - 如果已通过 setImmediate 调度了脚本，则事件循环将结束轮询 poll 阶段，并继续执行 check 阶段以执行那些调度的脚本。
  - 如果脚本并没有 setImmediate 设置回调，则事件循环将等待 poll 队列中的回调，然后立即执行它们。

一旦轮询队列 (poll queue) 为空，事件循环将检查哪些计时器 timer 已经到时间。 如果一个或多个计时器 timer 准备就绪，则事件循环将返回到计时器阶段，以执行这些计时器的回调。



### process.nextTick

与事件循环无关， 是Node新增的一个Api, 作用是**在下一次事件循环之前执行回调**,  使用场景可以参考 vue的 $nextTick， 比如在mounted生命钩子 在nextTick中访问dom（dom已经渲染）， 所以场景可以用在 **执行环境准备完毕之后立即调用**



### Node中的 `settimeout` 和 `setImmediate`

`setImmediate` 和 `setTimeout` 相似，但是根据调用时间的不同，它们的行为也不同。

- `setImmediate` 设计为在当前轮询 poll 阶段完成后执行脚本。
- `setTimeout` 计划在以毫秒为单位的最小阈值过去之后运行脚本。

计时器的执行顺序将根据调用它们的上下文而有所不同。 如果两者都是主模块 (main module) 中调用的，则时序将受到进程性能的限制（这可能会受到计算机上运行的其他应用程序的影响）。

例如，如果我们运行以下不在 `I/O` 回调（即主模块）内的脚本，则两个计时器的执行`顺序是不确定`的，因为它受进程性能的约束

```js
// timeout_vs_immediate.js
setTimeout(() => {
  console.log('timeout');
}, 0);

setImmediate(() => {
  console.log('immediate');
});
//////////////////////////////////////
node timeout_vs_immediate.js
timeout
immediate

node timeout_vs_immediate.js
immediate
timeout
```

但是，如果这两个调用在一个 `I/O` 回调中，那么 `immediate` 总是执行第一：

```js
// timeout_vs_immediate.js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => {
    console.log('timeout');
  }, 0);
  setImmediate(() => {
    console.log('immediate');
  });
});

/////////////////////////
$ node timeout_vs_immediate.js
immediate
timeout
```

**总结**

与 `setTimeout` 相比，使用 `setImmediate` 的主要优点是，如果在 `I/O` 周期内 `setImmediate` 总是比任何 `timers` 快。



## [cluster原理](https://www.cnblogs.com/dashnowords/p/10958457.html)

> 在nuxt中， 可以通过pm2.json脚本文件开启集群



##  Node 流机制

### stream模块

四种类型：

- **Readable** - 可读操作。
- **Writable** - 可写操作。
- **Duplex** - 可读可写操作.
- **Transform** - 操作被写入数据，然后读出结果。

## xx

