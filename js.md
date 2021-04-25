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

  