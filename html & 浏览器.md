## 行内元素与块级元素

- 块级元素

  1. 总是在新行上开始，占据一整行；

  2. 高度，行高以及外边距和内边距都可控制；

  3. 宽带始终是与浏览器宽度一样，与内容无关；

  4. 它可以容纳内联元素和其他块元素。

- 行内元素
  1. 和其他元素都在一行上；
  2. 高，行高及外边距和内边距部分可改变；
  3. 宽度只与内容有关；
  4. 行内元素只能容纳文本或者其他行内元素。
     不可以设置宽高，其宽度随着内容增加，高度随字体大小而改变，内联元素可以设置外边界，但是外边界不对上下起作用，只能对左右起作用，也可以设置内边界，但是内边界在ie6中不对上下起作用，只能对左右起作用

## [跨标签页通信](https://juejin.cn/post/6844903811232825357)

### **同源页面**间的跨页面通信

#### [BroadCast Channel](https://developer.mozilla.org/en-US/docs/Web/API/BroadcastChannel)

**Broadcast Channel**可以帮我们创建一个用于广播的通信频道。当所有页面都监听同一频道的消息时，其中某一个页面通过它发送的消息就会被其他所有页面收到。它的API和用法都非常简单

⚠️：**safari和ie不兼容**

示例：

```html
<!-- post页面 -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <button id="button">点击</button>
    <script>
        let button=document.getElementById('button')
        button.onclick=function(){
           	//创建一个名字是mychannel的对象。记住这个名字，下面会用到
            let cast = new BroadcastChannel('mychannel');
            myObj = { from: "children1", content: "add" };
            cast.postMessage(myObj);
        }     
    </script>
</body>
</html>


<!----------------------------------->

<!-- receive页面 -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <script>
        let cast1 = new BroadcastChannel('mychannel');//创建一个和刚才的名字一样的对象
        cast1.onmessage = function (e) {
            //e.data就是刚才的信息; 
            console.log(e.data);
        };
    </script>
</body>
</html>

```



####  [Service Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API)

- 可以用于PWA项目



#### LocalStorage

当 LocalStorage 变化时，会触发`storage`事件。利用这个特性，我们可以在发送消息时，把消息写入到某个 LocalStorage 中；然后在各个页面内，通过监听`storage`事件即可收到通知。

示例：

```js
window.addEventListener('storage', function (e) {
    if (e.key === 'ctc-msg') {
        const data = JSON.parse(e.newValue);
        const text = '[receive] ' + data.msg + ' —— tab ' + data.from;
        console.log('[Storage I] receive message:', text);
    }
});

```

⚠️：`storage`事件只有在值真正改变时才会触发

示例：

```js
window.localStorage.setItem('test', '123');
window.localStorage.setItem('test', '123');
```

由于第二次的值`'123'`与第一次的值相同，所以以上的代码只会在第一次`setItem`时触发`storage`事件



#### [Shared Worker](https://developer.mozilla.org/zh-CN/docs/Web/API/SharedWorker)

⚠️： 兼容性不怎么好



#### [indexedDB](https://developer.mozilla.org/zh-CN/docs/Web/API/IndexedDB_API)



#### window.open + window.opener



### 非同源页面之间的通信

#### iframe

​	iframe 与父页面间可以通过指定`origin`来忽略同源限制，因此可以在每个页面中嵌入一个 iframe （例如：`http://sample.com/bridge.html`），而这些 iframe 由于使用的是一个 url，因此属于同源页面，其通信方式可以复用上面第一部分提到的各种方式。

​	实现方式

​		1.首先需要在页面中监听 iframe 发来的消息，做相应的业务处理

​		2.然后，当页面要与其他的同源或非同源页面通信时，会先给 iframe 发送消息

​		3.其他 iframe 收到通知后，则会将该消息同步给所属的页面





## [history路由模式和hash路由模式](https://blog.csdn.net/Charissa2017/article/details/104779412)

### hash模式

​	**概述**

​	使用`window.location.hash`属性及窗口的`onhashchange`事件，可以实现监听浏览器地址hash值变化，执行相应的js切换网页。下面	具体介绍几个使用过程中必须理解的要点：

1. hash指的是地址中#号以及后面的字符，也称为散列值。hash也称作锚点，本身是用来做页面跳转定位的。如http://localhost/index.html#abc，这里的#abc就是hash；
2. 散列值是不会随请求发送到服务器端的，所以改变hash，不会重新加载页面；
3. 监听 window 的 hashchange 事件，当散列值改变时，可以通过 location.hash 来获取和设置hash值；
4. location.hash值的变化会直接反应到浏览器地址栏；



触发**hashchange**事件的几种情况

- 浏览器地址栏散列值的变化（包括浏览器的前进、后退）会触发window.location.hash值的变化，从而触发onhashchange事件；
- 当浏览器地址栏中URL包含哈希如 `http://www.baidu.com/#home`，这时按下输入，浏览器发送`http://www.baidu.com/`请求至服务器，请求完毕之后设置散列值为#home，进而触发onhashchange事件；
- 当只改变浏览器地址栏URL的哈希部分，这时按下回车，浏览器不会发送任何请求至服务器，这时发生的只是设置散列值新修改的哈希值，并触发onhashchange事件；
- html中<a>标签的属性 href 可以设置为页面的元素ID如 #top，当点击该链接时页面跳转至该id元素所在区域，同时浏览器自动设置 window.location.hash 属性，地址栏中的哈希值也会发生改变，并触发onhashchange事件；



### history模式

**概述**：

- window.history 属性指向 History 对象，它表示当前窗口的浏览历史。当发生改变时，只会改变页面的路径，不会刷新页面；
- History 对象保存了当前窗口访问过的所有页面网址。通过 history.length 可以得出当前窗口一共访问过几个网址；
- 由于安全原因，浏览器不允许脚本读取这些地址，但是允许在地址之间导航；
- 浏览器工具栏的“前进”和“后退”按钮，其实就是对 History 对象进行操作;

**History**对象主要有两个属性

- `History.length`：当前窗口访问过的网址数量（包括当前网页）
- `History.state`：History 堆栈最上层的状态值（详见下文）

**方法**

- `History.back()`：移动到上一个网址，等同于点击浏览器的后退键。对于第一个访问的网址，该方法无效果。

- `History.forward()`：移动到下一个网址，等同于点击浏览器的前进键。对于最后一个访问的网址，该方法无效果。

- `History.go()`：接受一个整数作为参数，以当前网址为基准，移动到参数指定的网址。如果参数超过实际存在的网址范围，该方法无效果；如果不指定参数，默认参数为0，相当于刷新当前页面。

- `History.pushState()`:

  ​	语法：`history.pushState(object, title, url) `

  ​	该方法接受三个参数，依次为：
  
  ​		`object`：是一个对象，通过 pushState 方法可以将该对象内容传递到新页面中。如果不需要这个对象，此处可以填 null。
  ​		`title`：指标题，几乎没有浏览器支持该参数，传一个空字符串比较安全。
  ​		`url`：新的网址，必须与当前页面处在同一个域。不指定的话则为当前的路径，如果设置了一个跨域网址，则会报错。
  
  ​	⚠️：	如果 pushState 的 URL 参数设置了一个新的锚点值（即 hash），并不会触发 hashchange 事件。反过来，如果 URL 的				锚点值变了，则会在 History 对象创建一条浏览记录
  
- `History.replaceState()`: 该方法用来修改 History 对象的当前记录，用法与 pushState() 方法一样

- `History.popstate()`: 每当 history 对象出现变化时，就会触发 popstate 事件

  ⚠️：

  - 仅仅调用`pushState()`方法或`replaceState()`方法 ，并不会触发该事件；
  - 只有用户点击浏览器倒退按钮和前进按钮，或者使用 JavaScript 调用`History.back()`、`History.forward()`、`History.go()`方法时才会触发。
  - 另外，该事件只针对同一个文档，如果浏览历史的切换，导致加载不同的文档，该事件也不会触发。
  - 页面第一次加载的时候，浏览器不会触发`popstate`事件

  ​				

⚠️： 移动到以前访问过的页面时，页面通常是从浏览器缓存之中加载，而不是重新要求服务器发送新的网页

⚠️： history 致命的缺点就是当改变页面地址后，强制刷新浏览器时，（如果后端没有做准备的话）会报错，因为刷新是拿当前地址去请			求服务器的，如果服务器中没有相应的响应，会出现 404 页面。 所以vue在启用history模式需要在nginx层配置 try file



## [DOM树](https://blog.poetries.top/browser-working-principle/guide/part5/lesson22.html#javascript-%E6%98%AF%E5%A6%82%E4%BD%95%E5%BD%B1%E5%93%8D-dom-%E7%94%9F%E6%88%90%E7%9A%84)

#### DOM树如何生成

- 通过**HTML解析器**将HTML字节转换成DOM树

  

  ![转换图解](http://blog.poetries.top/img-repo/2019/11/57.png)

  

  转换的三个阶段：

  	1. **通过分词器将字节流转换为 Token**
   	2. **至于后续的第二个和第三个阶段是同步进行的，需要将 Token 解析为 DOM 节点，并将 DOM 节点添加到 DOM 树中**

  

#### JavaScript 是如何影响 DOM 生成的

当HTML解析器遇到script标签时(如果该script是同步执行的)，渲染引擎判断这是一段脚本， 此时HTML解析器会暂停DOM解析，因为接下来 Javascript脚本可能会修改当前DOM结构。 这就是为什么通常要延迟执行 script脚本； 一方面会引起重绘， 一方面是为了防止DOM还未构建从而导致访问错误。



## [事件模型](https://javascript.ruanyifeng.com/dom/event.html)



## [缓存策略](https://juejin.cn/post/6844903593275817998)

**概述**：浏览器的缓存机制也就是我们说的HTTP缓存机制，其机制是根据HTTP报文的缓存标识进行的

请求(响应)行、HTTP头(HTTP头(通用信息头，请求头，实体头) )、请求(响应)主体

#### 强制缓存

- 强制缓存规则：

  当浏览器向服务器发起请求时，服务器会将缓存规则放入HTTP响应报文的HTTP头中和请求结果一起返回给浏览器，控制强制缓存的字段分别是Expires和Cache-Control，其中Cache-Control优先级比Expires高。

  - Expires
  - Cache-Control

- 不存在该缓存结果和缓存标识，强制缓存失效，则直接向服务器发起请求

![](https://user-gold-cdn.xitu.io/2018/4/19/162db63596c9de23?imageView2/0/w/1280/h/960/webp/ignore-error/1)\



- 存在该缓存结果和缓存标识，但该结果已失效，强制缓存失效，则使用协商缓存

  ​	![](https://user-gold-cdn.xitu.io/2018/4/19/162db63597182316?imageView2/0/w/1280/h/960/webp/ignore-error/1)

- 存在该缓存结果和缓存标识，且该结果尚未失效，强制缓存生效，直接返回该结果

  ![](https://user-gold-cdn.xitu.io/2018/4/19/162db6359acd19d3?imageView2/0/w/1280/h/960/webp/ignore-error/1)



#### 协商缓存

同样，协商缓存的标识也是在响应报文的HTTP头中和请求结果一起返回给浏览器的，控制协商缓存的字段分别有：Last-Modified / If-Modified-Since和Etag / If-None-Match，其中Etag / If-None-Match的优先级比Last-Modified / If-Modified-Since高。

- Last-Modified / If-Modified-Since
  - Last-Modified
  - If-Modified-Since
- Etag / If-None-Match
  - Etag
  - If-None-Match

协商缓存就是强制缓存失效后，浏览器携带缓存标识向服务器发起请求，由服务器根据缓存标识决定是否使用缓存的过程，主要有以下两种情况:

- 协商缓存生效，返回304，如下

![](https://user-gold-cdn.xitu.io/2018/4/19/162db635cbfff69d?imageView2/0/w/1280/h/960/webp/ignore-error/1)

- 协商缓存失效，返回200和请求结果结果

  ![](https://user-gold-cdn.xitu.io/2018/4/19/162db635cf070ff5?imageView2/0/w/1280/h/960/webp/ignore-error/1)







#### 缓存过程分析

![分析图解](https://user-gold-cdn.xitu.io/2018/4/19/162db6359673e7d0?imageView2/0/w/1280/h/960/webp/ignore-error/1)



#### 总结

强制缓存优先于协商缓存进行，若强制缓存(Expires和Cache-Control)生效则直接使用缓存，若不生效则进行协商缓存(Last-Modified / If-Modified-Since和Etag / If-None-Match)，协商缓存由服务器决定是否使用缓存，若协商缓存失效，那么代表该请求的缓存失效，重新获取请求结果，再存入浏览器缓存中；生效则返回304，继续使用缓存，如下图

![](https://user-gold-cdn.xitu.io/2018/4/19/162db635ed5f6d26?imageView2/0/w/1280/h/960/webp/ignore-error/1)







## [浏览器渲染](https://xie.infoq.cn/article/5d36d123bfd1c56688e125ad3)



## [浏览器工作原理](https://www.html5rocks.com/zh/tutorials/internals/howbrowserswork/)



## [内存泄漏](https://segmentfault.com/a/1190000020231307)



- **ES6 Set Map WeakSet** 这个是重点，需要着重看一下

- 引用计数垃圾收集

  这是最初级的垃圾收集算法。此算法把“对象是否不再需要”简化定义为“对象有没有其他对象引用到它”。如果没有引用指向该对象（零引用），对象将被垃圾回收机制回收。

  ES6 把**引用**有区分为**强引用**(Set, Map)和**弱引用**(WeakSet, WeakMap)，这个目前只有再 Set 和 Map 中才有， **强引用**才会有**引用计数**叠加，只有引用计数为 0 的对象的内存才会被回收，所以一般需要手动回收内存（手动回收的前提在于**标记清除法**还没执行，还处于当前执行环境）。

  而**弱引用**没有触发**引用计数**叠加，只要引用计数为 0，弱引用就会自动消失，无需手动回收内存。

  

- 标记清除法