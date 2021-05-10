##  原型到原型链

- 在javascript中，每个函数都有一个prototype属性，每个对象都有 ____proto____ 属性。

  函数的prototype属性指向一个对象，这个对象就是调用该构造函数而创建的实例的原型(说起来有点拗口 - -)。

  那什么是原型呢？ 可以这样理解，每个javascript对象在创建的时候都会关联一个对象，这个对象

  就是所谓的原型，每个对象都会从原型 “继承” 属性

  eg：

  ```javascript
  function Person(){}
  Person.prototype.name = 'kangkang'
  let person1 = new Person()
  person1.name // kangkang
  /**
  	从上面代码中分析：
  	Person 是一个构造函数， 有 prototype 属性
  	person1 是Person 的一个实例对象， 有 __proto__属性
  	
  	Person 和 实例person1 都有共同的原型对象, 即：
  	person1.__proto__ === Person.prototype
  	
  	原型对象上有一个 constructor属性，指向关联的构造函数，即：
  	Person.prototype.constructor === Person
  	person1.__proto__.constructor === Person	
  **/

### 实例与原型

- 当读取实例的属性时，如果找不到，就会查找原型上的对应属性，一直找到最顶层(null)

  eg：

  ```javascript
  function Person() {
    this.name = 'kangkang'
  }
  Person.prototype.name = 'maria'
  let person1 = new Person()
  person1.name // kangkang
  delete person1.name
  person1.name // maria
  delete Person.prototype.name
  person1.name // undefined
  ```



## javascript 继承

- 原型链继承

  ```javascript
  function Parent() {
  	this.name = 'kevin'
  }
  Parent.prototype.getName = function() {
    return this.name
  }
  funtion Child() {}
  Child.prototype = new Parent()
  var child1 = new Child()
  child1.getName // kevin
  
  /**
  	问题：
  	1.引用类型的属性被所有实例共享
  	2.不能向Parent传参		
  **/
  ```

- 借用构造函数(经典继承)

  ```javascript
  function Parent(name){
    this.name = name
  }
  function Child(name) {
    Parent.call(this,name)
  }
  var child1 = new Child(['kevin', 'kangkang'])
  child1.name.push('maria')
  var child2 = new Child(['kevin', 'kangkang'])
  child1.name // ['kevin', 'kangkang', 'maria']
  child2.name // ['kevin', 'kangkang']
  
  /**
  	优点：
  	解决了上面原型链继承导致的引用共享问题
  	可以在Child中向Parent传参
  	
  	缺点：
  	方法都在构造函数中定义，每次创建实例都会创建一遍方法(因为Child只有call方法)
  **/
  ```

- 组合继承

  原型链继承和经典继承双剑合璧

  ```javascript
  function Parent(name) {
    this.name = name
    this.habits = ['swimming', 'running']
  }
  Parent.Prototype.getName = function() {
    return this.name
  }
  function Child(name, age) {
    Parent.call(this, name)
    this.age = age
  }
  Child.prototype = new Parent() // 继承Parent原型上的属性， 此时Child的构造函数指向 Parent
  Child.prototype.constructor = Child // 重新让Child的构造函数指向Child
  
  var child1 = new Child('kangkang', '36')
  child1.habits.push('sleep')
  child1.name // kangkang
  child1.getName() // kangkang
  child1.age // 36
  child1.habits // ['swimming', 'running', 'sleep']
  
  var child2 = new Child('daisy', '20')
  child1.habits // ['swimming', 'running']
  
  /**
  	优点：融合原型链继承和构造函数的优点，是 JavaScript 中最常用的继承模式
  **/
  ```

- 原型式继承

  ```javascript
  function createObj(o) {
    function F(){}
    F.prototype = o
    return new F()
  }
  
  /**
  	就是 ES5 Object.create 的模拟实现，将传入的对象作为创建的对象的原型
  	缺点：与最上面的原型链继承一样，有引用共享的问题
  **/
  ```

- 寄生式继承

  ```javascript
  function creatObj(o) {
    var clone = Object.create(o)
    clone.testFunc = function() {}
    return clone
  }
  
  /**
  创建一个仅用于封装继承过程的函数，该函数在内部以某种形式来做增强对象，最后返回对象。
  缺点：跟借用构造函数模式一样，每次创建对象都会创建一遍方法
  **/
  ```

- 寄生组合式继承

  ```javascript
  function Parent (name) {
      this.name = name;
      this.colors = ['red', 'blue', 'green'];
  }
  
  Parent.prototype.getName = function () {
      console.log(this.name)
  }
  
  function Child (name, age) {
      Parent.call(this, name);
      this.age = age;
  }
  
  // 关键的三步
  var F = function () {};
  
  F.prototype = Parent.prototype;
  
  Child.prototype = new F();
  
  
  var child1 = new Child('kevin', '18');
  
  console.log(child1);
  
  // 封装一下
  function object(o) {
      function F() {}
      F.prototype = o;
      return new F();
  }
  
  function prototype(child, parent) {
      var prototype = object(parent.prototype);
      prototype.constructor = child;
      child.prototype = prototype;
  }
  
  // 当我们使用的时候：
  prototype(Child, Parent);
  
  /**
  这种方式的高效率体现它只调用了一次 Parent 构造函数，并且因此避免了在 Parent.prototype 上面创建不必要的、多余的属性。与此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf。开发人员普遍认为寄生组合式继承是引用类型最理想的继承范式。
  **/
  ```




## 执行上下文栈

- 执行上下文

  ​	js的es5之前的可执行上下文有 全局代码，函数，eval代码，多个执行上下文组成执行上下文栈

  * 变量对象(vo)

    vars,functions,arguments

    变量对象是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明，

    变量对象里面主要存储了 变量，函数声明，函数形参(argument)

  * this

    context object

  * ScopeChain(作用域链)

    variable object + all parents scopes

    当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)**执行上下文**的**变量对象**中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做作用域链

  [诠释了作用域链，变量对象，执行上下文的关系](https://github.com/kuitos/kuitos.github.io/issues/18)



## 闭包

- 定义

  闭包是指那些能够访问**自由变量**的函数

  自由变量：自由变量是指在函数中使用的，但既不是函数参数也不是函数的局部变量的变量

  Eg:

  ```javascript
  var a = 1;
  
  function foo() {
      console.log(a);
  }
  
  foo();
  
  /**
  foo 函数可以访问变量 a，但是 a 既不是 foo 函数的局部变量，也不是 foo 函数的参数，所以 a 就是自由变量
  **/
  ```

  

- ECMAScript中，闭包指的是:

  [文章链接](https://github.com/mqyqingfeng/Blog/issues/9)

  1. 从理论角度：所有的函数。因为它们都在创建的时候就将上层上下文的数据保存起来了。哪怕是简单的全局变量也是如此，因为函数中访问全局变量就相当于是在访问自由变量，这个时候使用最外层的作用域
  2. **从实践角度：以下函数才算是闭包：**
     		1. 即使创建它的上下文已经销毁，它仍然存在（比如，内部函数从父函数中返回）
       		2. 在代码中引用了自由变量

  Eg:

  ```js
  var scope = "global scope";
  function checkscope(){
      var scope = "local scope";
      function f(){
          return scope;
      }
      return f;
  }
  
  var foo = checkscope();
  foo();
  
  
  /**************************************/
  // 必刷题
  
  var data = [];
  
  for (var i = 0; i < 3; i++) {
    data[i] = function () {
      console.log(i);
    };
  }
  
  data[0]();
  data[1]();
  data[2]()
  
  ```

  

## 变量提升

- 主要就是词法分析，其实就是变量对象(vo)保存的值， 执行代码的时候再去更新这些值，

  在词法分析时，其实可以看成是变量对象的创建过程；变量对象会包括：

  1. 函数的所有形参 (如果是函数上下文)
     - 由名称和对应值组成的一个变量对象的属性被创建
     - 没有实参，属性值设为 undefined
  2. 函数声明
     - 由名称和对应值（函数对象(function-object)）组成一个变量对象的属性被创建
     - 如果变量对象已经存在相同名称的属性，则完全替换这个属性
  3. 变量声明
     - 由名称和对应值（undefined）组成一个变量对象的属性被创建；
     - 如果变量名称跟已经声明的形式参数或函数相同，则变量声明不会干扰已经存在的这类属

  

  - 变量对象的创建过程
    1. 全局上下文的变量对象初始化是全局对象
    2. 函数上下文的变量对象初始化只包括 Arguments 对象
    3. 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值
    4. 在代码执行阶段，会再次修改变量对象的属性值

  

  Eg:

  ```js
  function foo(a) {
    var b = 2;
    function c() {}
    var d = function() {};
  
    b = 3;
  
  }
  
  foo(1);
  
  
  /**
  在进入执行上下文后，这时候的 AO 是
  AO = {
      arguments: {
          0: 1,
          length: 1
      },
      a: 1,
      b: undefined,
      c: reference to function c(){},
      d: undefined
  }
  
  
  代码执行
  在代码执行阶段，会顺序执行代码，根据代码，修改变量对象的值
  
  还是上面的例子，当代码执行完后，这时候的 AO 是
  
  AO = {
      arguments: {
          0: 1,
          length: 1
      },
      a: 1,
      b: 3,
      c: reference to function c(){},
      d: reference to FunctionExpression "d"
  }
  **/
  ```

  

  

## [this的指向](https://github.com/mqyqingfeng/Blog/issues/7)

- 如何判断this的指向？

  1. 计算 MemberExpression 的结果赋值给 ref

     什么是MemberExpression

     - PrimaryExpression // 原始表达式 可以参见《JavaScript权威指南第四章》
     - FunctionExpression // 函数定义表达式
     - MemberExpression [ Expression ] // 属性访问表达式
     - MemberExpression . IdentifierName // 属性访问表达式
     - new MemberExpression Arguments // 对象创建表达式

     Eg:

     ```js
     function foo() {
         console.log(this)
     }
     
     foo(); // MemberExpression 是 foo
     
     function foo() {
         return function() {
             console.log(this)
         }
     }
     
     foo()(); // MemberExpression 是 foo()
     
     var foo = {
         bar: function () {
             return this;
         }
     }
     
     foo.bar(); // MemberExpression 是 foo.bar
     ```

     **所以简单理解 MemberExpression 其实就是()左边的部分。**

     

  2. 判断 ref 是不是一个 Reference 类型

     1. 如果 ref 是 Reference，并且 IsPropertyReference(ref) 是 true, 那么 this 的值为 GetBase(ref)

        Reference构成：

        - base value

          它的值只可能是 undefined, an Object, a Boolean, a String, 

          a Number, or an environment record 其中的一种

        - referenced name

        - strict reference

        Eg:

        ```js
        var foo = 1;
        
        // 对应的Reference是：
        var fooReference = {
            base: EnvironmentRecord,
            name: 'foo',
            strict: false
        };
        ```

        

        IsPropertyReference(ref):如果 base value 是一个对象，就返回true,否则返回false

     2. 如果 ref 是 Reference，并且 base value 值是 Environment Record(可以看成是不是在执行上下文是不是全局上下文), 那么this的值为 ImplicitThisValue(ref)

        ImplicitThisValue(ref)返回的值永远是undefined， 非严格模式下 undefined下的tihs永远指向window

        GetBase(): 返回 reference 的 base value

     3. 如果 ref 不是 Reference，那么 this 的值为 undefined

  

- 判断 ref 是不是一个 Reference 类型

  关键就在于看规范是如何处理各种 MemberExpression，返回的结果是不是一个Reference类型

  Eg:

  ```js
  var value = 1;
  
  var foo = {
    value: 2,
    bar: function () {
      return this.value;
    }
  }
  
  //示例1
  console.log(foo.bar());
  //示例2
  console.log((foo.bar)());
  //示例3
  console.log((foo.bar = foo.bar)());
  //示例4
  console.log((false || foo.bar)());
  //示例5
  console.log((foo.bar, foo.bar)());
  ```



## [JavaScript：立即执行函数表达式(IIFE)](https://segmentfault.com/a/1190000003985390)

- 定义：**IIFE**（ 立即调用函数表达式）是一个在定义时就会立即执行的javascript函数

  ```js
  (function () {
      statements
  })();
  ```


- 目的：
  1. 不必为函数命名，避免了污染全局变量
  2. IIFE内部形成了单独的作用域，可以封住一些外部无法读取的私有变量



## [instanceof和typeof实现原理](https://juejin.cn/post/6844903613584654344)

- typeof只能准确判断基础数据类型，所有引用数据类型结构都为 "object",

  要想判断引用数据类型，可以用instanceof这个操作符来判断

  

- typeof

  - 为什么typeof null === "object"

    js在底层存储变量的时候，会在机器码的低位1-3位存储其类型信息

    - 000：对象
    - 010：浮点数
    - 100：字符串
    - 110：布尔
    - 1：整数

    需要注意的是：

    - null：所有机器码都为0

      这就是为什么 typeof null === "object" 结果为 true

    - `undefined`：用 −2^30 整数来表示

- instanceof

  之前我们提到了 `instanceof` 来判断对象的具体类型，其实 `instanceof` 主要的作用就是判断一个实例是否属于某种类型

  Eg:

  ```js
  let person = function () {
  }
  let nicole = new person()
  nicole instanceof person // true
  ```

  当然，`instanceof` 也可以判断一个实例是否是其父类型或者祖先类型的实例

  ```js
  let person = function () {
  }
  let programmer = function () {
  }
  programmer.prototype = new person()
  let nicole = new programmer()
  nicole instanceof person // true
  nicole instanceof programmer // true
  ```

  instanceof原理

  ```js
  // 示例代码
  function new_instance_of(leftVaule, rightVaule) { 
      let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
      leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
      while (true) {
      	if (leftVaule === null) {
              return false;	
          }
          if (leftVaule === rightProto) {
              return true;	
          } 
          leftVaule = leftVaule.__proto__ 
      }
  }
  
  /**
  其实 instanceof 主要的实现原理就是只要右边变量的 prototype 在左边变量的原型链上即可。因此，instanceof 在查找的过程中会遍历左边变量的原型链，直到找到右边变量的 prototype，如果查找失败，则会返回 false，告诉我们左边变量并非是右边变量的实例。
  /**
  ```

- 图解

  ​	![原型链图解](https://user-gold-cdn.xitu.io/2018/5/28/163a55d5d35b866d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- Object.prototype.toString可以准确判断具体类型




## call & apply

### call

- call函数特点
  1. 立即执行，例如 `fn.call(null)`  这里的fn会立即执行
  2. 第一个参数是需要绑定this指向的对象， 如果不传或者没有指定绑定对象 this 将指向window
  3. 第二个到最后的参数是传入的值
  4. call函数有返回值

- 模拟实现思路：

  1. 将函数设为对象的属性

  2. 执行该函数

  3. 删除该函数

​	

最简单的实现:

```js
// 第一版
Function.prototype.call2 = function(context) {
    // 首先要获取调用call的函数，用this可以获取
  	// 注意， 这里的this指向bar函数
    context.fn = this;
    context.fn();
    delete context.fn;
}

// 测试一下
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

bar.call2(foo); // 1
```



考虑携带参数和返回值之后的模拟实现：

```js
Function.prototype.call2 = function(context) {
  	context = context || window
  	context.fn = this
  	var args = []
  	for(var i=1, i<arguments.length; i++) {
      args.push('arguments['+i+']')
    }
  	/**
  		下面的args会自动调用Array.prototype.toString()转换成字符串
  		因为eval接收的是一个字符串， js接收到数组时会将数组转换成字符串，参照js类型转换规则
  	**/
  	var result = eval('context.fn('+args+')') 
    delete context.fn
  	return result
}



/***************************/
// 测试一下
var value = 2;

var obj = {
    value: 1
}

function bar(name, age) {
    console.log(this.value);
    return {
        value: this.value,
        name: name,
        age: age
    }
}

bar.call2(null); // 2

console.log(bar.call2(obj, 'kevin', 18));
// 1
// Object {
//    value: 1,
//    name: 'kevin',
//    age: 18
// }
```



	### apply

- apply与call类似，

  模拟实现代码：

  ````js
  Function.prototype.apply = function (context, arr) {
      var context = Object(context) || window;
      context.fn = this;
  
      var result;
      if (!arr) {
          result = context.fn();
      }
      else {
          var args = [];
          for (var i = 0, len = arr.length; i < len; i++) {
              args.push('arr[' + i + ']');
          }
          result = eval('context.fn(' + args + ')')
      }
  
      delete context.fn
      return result;
  }
  ````

  

## bind的实现

- [规范链接](https://262.ecma-international.org/5.1/#sec-15.3.4.5)

- bind() 方法会创建一个新函数。当这个新函数被调用时，bind() 的第一个参数将作为它运行时的 this，之后的一序列参数将会在传递的实参前传入作为它的参数。(来自于 MDN )
- bind函数的两个特点
  1. 返回一个函数
  
  2. 传参与call类似，是若干个参数
  
  3. 与call和apply不同的是， bind函数不会立即执行(返回一个函数，主动调用执行), 因为bind函数不会立即执行， 所以可以多次传参数
  
     Eg:
  
     ```js
     var foo = {
         value: 1
     };
     
     function bar(name, age) {
         console.log(this.value);
         console.log(name);
         console.log(age);
     
     }
     
     var bindFoo = bar.bind(foo, 'daisy');
     bindFoo('18');
     // 1
     // daisy
     // 18
     ```
  
     

Eg:

```js
var foo = {
    value: 1
};

function bar() {
    console.log(this.value);
}

// 返回了一个函数
var bindFoo = bar.bind(foo); 

bindFoo(); // 1
```



模拟实现代码：

```js
Function.prototype.bind2 = function (context) {

    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function () {};

    var fBound = function () {
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
    }

    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```

 



## [javascript之函数柯里化](https://github.com/mqyqingfeng/Blog/issues/42)

- 定义

  在数学和计算机科学中，柯里化是一种将使用多个参数的一个函数转换成一系列使用一个参数的函数的技术

- 这里理解感觉好一点

  ​	用闭包把参数保存起来，当参数的数量足够执行函数了，就开始执行函数

- 用途

  - 参数复用（降低通用型，提高适用性？？啥意思）

    ​	降低通用性，提高适用性：

    ​	假如一个函数是 fn()()(), 到第三个()才会去执行， 那我去定义两个函数: fn1 = fn(), 和fn2 = fn()()

    ​	这样fn1就保持了传了一个参数的状态， fn2保持了传了两个参数的状态，这样就达到了降低通用型，同时提高适用性的目的

- curry函数简易版实现

  ```js
  const curry = (fn, ...curryArgs) => (...args) =>
    args.length >= fn.length
      ? fn(...curryArgs, ...args)
      : curry(fn, ...curryArgs, ...args);
  ```




## [v8垃圾回收机制](https://juejin.cn/post/6844904016325902344)

### v8引擎内存限制

64位操作系统：约1.4G

32位操作系统：约0.7G

为什么会有这样的限制？

​	起初v8只是作为浏览器端javascript的执行环境，很少使用到大量内存的场景，

​	此外，如果内存使用过高，那么必然会导致垃圾回收的过程缓慢，也就会导致主线程的等待时间越长，

​	浏览器也就越长时间得不到响应。

在node中可以设置 --max-old-space-size=xxxx  --max-semi-space-size=xxxx  来改变内存限制

同时也可以使用 `process.memoryUsage()` 方法来查看当前node进程所占用的实际内存大小

### v8垃圾回收策略

​	[分代式垃圾回收机制](https://blog.csdn.net/qq_42535651/article/details/109603874)：

​	根据**对象的存活时间**将内存的垃圾回收进行不同的分代，然后对不同的分代采用不同的垃圾回收算法



### v8的内存结构

#### 内存结构

- 新区(new_space)：

  大多数的对象开始都会被分配在这里，**这个区域相对较小**但是垃圾回收特别频繁，该区域被分为两半，一半用来分配内存，另一半用于在垃圾回收时将需要保留的对象复制过来。

  新生代主要用于存放存活时间较短的对象。新生代内存是由两个`semispace(半空间)`构成的，内存最大值在`64`位系统和`32`位系统上分别为`32MB`和`16MB`，在新生代的垃圾回收过程中主要采用了`Scavenge`算法

  **`Scavenge`算法是一种典型的牺牲空间换取时间的算法**，对于老生代内存来说，可能会存储大量对象，如果在老生代中使用这种算法，势必会造成内存资源的浪费，但是在新生代内存中，大部分对象的生命周期较短，在时间效率上表现可观，所以还是比较适合这种算法。

  > 在`Scavenge`算法的具体实现中，主要采用了`Cheney`算法，它将新生代内存一分为二，每一个部分的空间称为`semispace`，也就是我们在上图中看见的new_space中划分的两个区域，其中处于激活状态的区域我们称为`From`空间，未激活(inactive new space)的区域我们称为`To`空间。这两个空间中，始终只有一个处于使用状态，另一个处于闲置状态。我们的程序中声明的对象首先会被分配到`From`空间，当进行垃圾回收时，如果`From`空间中尚有存活对象，则会被复制到`To`空间进行保存，非存活的对象会被自动回收。当复制完成后，`From`空间和`To`空间完成一次角色互换，`To`空间会变为新的`From`空间，原来的`From`空间则变为`To`空间

  

- 旧区(old_space)：

  新生代中的对象在存活一段时间后就会被转移到老生代内存区，相对于新生代该内存区域的垃圾回收频率较低。老生代又分为`老生代指针区`和`老生代数据区`，前者包含大多数可能存在指向其他对象的指针的对象，后者只保存原始数据对象，这些对象没有指向其他对象的指针。

- 大对象区(large_object_space)

- 代码区(code_space)

- map区(map_space)

**垃圾回收过程主要集中在新区和旧区，其他的就不展开了**

####  对象晋升

- 当一个对象在经过多次复制之后依旧存活，那么它会被认为是一个生命周期较长的对象，在下一次进行垃圾回收时，该对象会被直接转移到老生代中，这种对象从新生代转移到老生代的过程我们称之为`晋升`

  对象晋升条件

  1. 对象是否经历过一次`Scavenge`算法
  2. `To`空间的内存占比是否已经超过`25%`

#### 旧区

在老生代中，因为管理着大量的存活对象，如果依旧使用`Scavenge`算法的话，很明显会浪费一半的内存，因此已经不再使用`Scavenge`算法，而是采用新的算法`Mark-Sweep(标记清除)`和`Mark-Compact(标记整理)`来进行管理。

> `Mark-Sweep(标记清除)`分为`标记`和`清除`两个阶段，在标记阶段会遍历堆中的所有对象，然后标记活着的对象，在清除阶段中，会将死亡的对象进行清除。`Mark-Sweep`算法主要是通过判断某个对象是否可以被访问到，从而知道该对象是否应该被回收，具体步骤如下：
>
> - 垃圾回收器会在内部构建一个`根列表`，用于从根节点出发去寻找那些可以被访问到的变量。比如在JavaScript中，`window`全局对象可以看成一个根节点。
> - 然后，垃圾回收器从所有根节点出发，遍历其可以访问到的子节点，并将其标记为活动的，根节点不能到达的地方即为非活动的，将会被视为垃圾。
> - 最后，垃圾回收器将会释放所有非活动的内存块，并将其归还给操作系统。
>
> 以下几种情况都可以作为根节点：
>
> 1. 全局对象
> 2. 本地函数的局部变量和参数
> 3. 当前嵌套调用链上的其他函数的变量和参数

但是`Mark-Sweep`算法存在一个问题，就是在经历过一次标记清除后，内存空间可能会出现不连续的状态，因为我们所清理的对象的内存地址可能不是连续的，所以就会出现内存碎片的问题，导致后面如果需要分配一个大对象而空闲内存不足以分配，就会提前触发垃圾回收，而这次垃圾回收其实是没必要的，因为我们确实有很多空闲内存，只不过是不连续的。



为了解决这种内存碎片的问题，`Mark-Compact(标记整理)`算法被提了出来，该算法主要就是用来解决内存的碎片化问题的，回收过程中将死亡对象清除后，在整理的过程中，会将活动的对象往堆内存的一端进行移动，移动完成后再清理掉边界外的全部内存



#### 如何避免内存泄漏

1. 尽可能少地创建全局变量

2. 手动清除定时器

3. 谨慎使用闭包

4. 清除dom引用

5. 弱引用

   WeakMap

   WeakSet

   WeakMap和WeakSet就是为了解决内存泄漏的问题而诞生的。其表示`弱引用`，它的键名所引用的对象均是弱引用，弱引用是指垃圾回收的过程中不会将键名对该对象的引用考虑进去，只要所引用的对象没有其他的引用了，垃圾回收机制就会释放该对象所占用的内存



## 运算符优先级





## javascript 错误类型

