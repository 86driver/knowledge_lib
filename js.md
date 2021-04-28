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

  



## 运算符优先级



