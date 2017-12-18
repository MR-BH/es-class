# JavaScript与Typescript中的class  

## ES5中的对象

在js中，除了字符串、数字、布尔值、undefined和null以外，其它的值都是对象。也就是说，数组和函数也是对象。


对象的属性除了名字和值外，还有一些与之相关的值，称为“属性特性”。

* 可写（设置该属性的值）
* 可枚举（for/in）
* 可配置 (delete或修改该属性的属性特性)  


除了包含属性外，每个对象还拥有三个相关的对象特性。

* 对象的原型
* 对象的类
* 对象的扩展标记


对象的分类

* 内置对象
* 宿主对象
* 自定义对象


属性的分类

* 自有属性    
* 继承属性  


### 什么是原型？

每一个JS对象（null除外）都和另一个对象相关联。这里说的“另一个对象”就是这个JS对象的原型，每一个对象都从原型继承属性。  

* 所有通过对象直接量创建的对象都具有同一个原型, Object.prototype
* 通过new创建的对象的原型就是构造函数的prototype
* 通过Object.create创建的对象原型就是create函数的第一个参数
* 特别的，Object.prototype的原型为null

### 什么是原型链？

一个🌰 :

所有的内置构造函数都具有一个继承自Object.prototype的原型，比如Array的原型的原型就是Object.prototype。

Object.getPrototypeOf(Array.prototype) === Object.prototype

这一系列链接的原型对象就是所谓的属性和方法。

### JS中的继承
JS对象具有自有属性，同样也有一些属性是从原型对象继承而来的（继承属性）。那么JS对象是如何访问属性的呢？

一个🌰 :  

在访问obj.a时，如果obj存在a,则返回a；如果obj中不存在a，那么将继续在o的原型对象中查询属性a；如果原型对象中也没有a，但这个原型对象也有原型，那么继续在这个原型对象的原型上查询，直到找到x或者查找到一个原型是null的对象为止。对象的原型构成了一个链，通过这个链可以实现属性的继承。

当然，继承属性会被同名的自有属性覆盖。同样的，对象无法修改继承属性，只能修改自有属性。也就是说，对JS对象进行属性赋值操作时，首先检查原型链，判断是否允许复制操作。如果允许复制操作，它总是在原始对象上创建属性或对已有的属性赋值，而不会去修改原型链。but!有一个例外，如果对象的某个继承属性是一个具有setter方法的accessor属性，此时将调用setter方法而不是给原始对象创建一个同名自有属性。

    obj = {
      set a(arg) { obj._a = arg },
      get a() { return obj._a }
    }

    obj2 = Object.create(obj)
    obj2.a = 111
    obj.a // 111

另一段有意思的代码

    var obj = { a: 1 }
    var obj2 = Object.create(obj)
    Object.defineProperty(obj, 'b', {
      get() { return obj._b },
      set(_b) { obj._b = _b},
      enumerable: true,
      configurable: true,
    })
    obj2.b = 3
    obj.b // undefined
    obj.b = 4

### 对象属性访问为啥要判空？
查询一个不存在的属性并不会报错，但是如果对象不存在。那么试图查询这个不存在的对象的属性就会报错。（undefined和null都没有属性）所以代码中利用&&的短路行为避免获取对象深层属性时的类型错误异常。

### 对象的原型可以修改么
自定义对象是可以修改原型的，但是内置构造函数和ES6的class声明是不可修改原型的。(设置失败但不会报错)

### 什么情况不能给对象设置属性
* 对象o的属性p是只读的。不能给只读属性重新复制（但是可以通过Object.defineProperty对可配置不可读的属性重新赋值）
* 对象o有继承属性p，且p是只读的。不能通过同名自有属性覆盖只读的继承属性
* 对象o中不存在自有属性p。对象o没有setter的继承属性p，并且对象o是不可扩展的。

### 删除属性
* delete只是断开属性和宿主对象的联系，而不会去操作属性中的属性。（原理类似于浅拷贝）
* delete只能删除自有属性，不能删除继承属性

很有意思的一段代码：

    var obj = { say() {} }
    delete obj.say
    obj.say() // TypeError

    class A {
      say() {}
    }
    var a = new A()
    delete a.say
    a.say()

这段代码说明ES6的class只是原型继承的一种语法糖，通过class创建的对象的实例方法其实都是继承方法。当然，这也解释了为什么ES6的class的实例属性必须在constructor中声明而不能像实例方法一样声明（会报错），因为这样形式声明的属性都是继承属性，而继承属性是不能被修改的。

* delete不能删除那些可配置性为false的属性（可以删除不可扩展对象的可配置属性）


### 检测属性
可以通过in、hasOwnProperty、propertyIsEnumerable来判断某个属性是否存在于某个对象中。

* in判断对象中是否包括同名自有属性和继承属性
* hasOwnProperty判断对象中是否有同名自有属性
* propertyIsEnumerable判断是否有同名属性且是否可枚举

### 枚举属性
* in获取所有可枚举的自有属性和继承属性 
* Object.keys获取所有可枚举的自有属性的名称
* Object.getOwnPropertyNames获取所有自有属性的名称

### 属性的特性（属性描述符）
数据属性的四个特性分别是
* 值 value
* 可写性 writable
* 可枚举性 enumerable
* 可配置性 configurable

存取器属性的四个特性是
* 读取 getter
* 写入 setter
* 可枚举性 enumerable
* 可配置性 configurable

查询和设置属性描述符
* Object.getOwnPropertyDescriptor获取对象某个特定自有属性的属性描述符
* Object.getOwnPropertyDescriptors （ES2017添加）获取对象的所有自有属性的属性描述符
* Object.defineProperty设置对象某个特定自有属性的属性描述符
* Object.definePropertys设置对象多个自有属性的属性描述符

注意: 在设置对象属性的属性描述符时不必包含所有四个特性。若是新创建，则默认是false或undefined；若是修改，则保持原值。

### 对象的三个属性
* 原型属性 getPrototypeOf
* 类属性 Object.prototype.toString.call(obj)
* 可扩展性
  * Object.isExtensible判断是否可扩展
  * Object.preventExtensions设置对象不可扩展（不可逆）
  * Object.seal 设置对象不可扩展，对象所有自有属性不可配置
  * Object.isSealed 判断对象是否封闭
  * Object.freeze 冻结对象，不可扩展，所有自有属性不可配置、不可扩展
  * Object.isFrozen 判断对象是否冻结
  * 对象的存取器属性不受影响


## ES5中基于原型链的继承
在JS中，类的实现是基于其原型继承机制的。如果两个实例都从同一个原型对象上继承了属性，我们说它们是同一个类的实例。

原型对象是类的唯一标识，当且仅当两个对象继承自同一个原型对象时，它们才是属于同一个类的实例。而构造函数则不能作为类的标识，两个构造函数的prototype可能指向同一个原型对象。那么这两个构造函数创建的实例是属于同一类的。

    obj instanceof A

检查的是obj是否继承自A.prototype

JS中的类牵扯三种不同的对象。

* 构造函数对象。任何添加到这个构造函数对象中的属性都是类字段和类方法（静态属性和静态方法）
* 原型对象。原型对象的属性被类的所有实例继承，原型对象的方法就是类的实例方法。
* 实例对象。实例对象的非函数属性就是实例的字段。

JS中定义类的步骤

* 定义一个构造函数，设置初始化对象的实例属性。
* 给构造函数的原型对象定义实例方法。
* 给构造函数定义类字段和类属性。

JS中基于原型的继承是动态的：对象从其原型继承属性，如果创建对象之后原型的属性发生改变，也会影响到集成这个原型的所有实例对象。  
下面的代码再一次说明了class只是基于原型的继承的语法糖。

    var obj = { a: 1 }
    var obj2 = Object.create(obj)
    obj.b = 2
    obj2.b = 2

    class A {}
    var a = new A()
    A.prototype.b = 2
    a.b // 2

再来看一段有意思的代码

    function B() {}  
    B.prototype = { b: 2 }  
    var b = new B()  
    b.b // 2  
    B.prototype = { c: 3 }  
    b.c // undefined  
    var b2 = new B()  
    b2.c // 3  
    Object.getPrototypeOf(b2) !== Object.getPrototypeOf(b) // true  
    b instanceof B // false  
    b2 instanceof B // true

    class A {}
    var a = new A()
    A.prototype.a = 1
    a.a // 1
    A.prototype = { d: 4 }
    a.d // undefined
    a2.d // undefined
    a2.a // 1
    Object.getPrototypeOf(a) === Object.getPrototypeOf(a2)

也即是说，ES6中的class保证了class的构造函数的原型对象是不可变的，但是你可以往原型对象上添加属性。

	Object.getOwnPropertyDescriptors(A)
	
	{length: {…}, prototype: {…}, name: {…}}
	length:{value: 0, writable: false, enumerable: false, configurable: true}
	name:{value: "A", writable: false, enumerable: false, configurable: true}
	prototype:{value: {…}, writable: false, enumerable: false, configurable: false}
	__proto__:Object
	

## ES6对象的扩展

### 对象字面量语法扩展
* 对象属性的简写
* 对象方法的简写
	* 使用简写的对象方法会有name属性
* 可计算属性名

### 增加的方法
* Object.is 进行”同值相等“的比较 与===有两点不同
	* +0 不等于 -0
	* NaN 等于自身
* Object.assign ***deprecated*** 建议使用对象扩展符...; 值得一提的是Object.assign和...对于具有getter方法的accessor属性会求值处理为数据属性。

Object.getOwnPropertyDescriptors可以用来

* 配合Object.defineProperties解决Object.assign无法正确拷贝get和set属性的问题
* 配合Object.create方法，将对象属性克隆到一个新对象。这属于浅拷贝。

### 属性的可枚举性与遍历

以下操作只操作对象的可枚举属性；其中for-in操作自有和继承属性，其余操作的都是自有属性

* for-in
* Object.keys
* JSON.stringify
* Object.assign/...  

值得一提的是，所有class的原型方法都是不可枚举的。

	class A {
	 sayName() { return 'myName';}
	}
	Object.getOwnPropertyDescriptors(A.prototype)
	
	{constructor: {…}, sayName: {…}}
	constructor:{writable: true, enumerable: false, configurable: true, value: ƒ}
	sayName:{writable: true, enumerable: false, configurable: true, value: ƒ}
	__proto__:Object


遍历对象属性的5种方法：

* for-in 遍历可枚举的自有和继承属性
* Object.keys 可枚举的自有属性
* Object.getOwnPropertyNames 自有属性
* Object.getOwnPropertySymbols 自有symbol属性
* Reflect.ownKeys 自有属性（包括Symbol属性）


ES6规定了自有属性枚举顺序；自有属性枚举顺序的基本规则是：

* 所有数字键按升序排序
* 所有字符串键按加入对象的顺序排序
* 所有symbol键按加入对象的顺序排序

Object.getOwnPropertyNames、Reflect.ownKeys、Object.assign和对象扩展符...都会受该规则影响；而for-in、Object.keys、JSON.stringify的枚举顺序则并不明确(由浏览器的具体实现决定，v8的顺序与规则一致)

### 增强对象原型

`__proto__` 属性; 用来读取或设置当前兑现的原型对象。只有现代浏览器支持该属性。不推荐使用。deprecated。如果需要使用，可以这样进行定义。

	Object.defineProperty(Object.prototype, '__proto__', {
	  get() {
	    let _thisObj = Object(this);
	    return Object.getPrototypeOf(_thisObj);
	  },
	  set(proto) {
	    if (this === undefined || this === null) {
	      throw new TypeError();
	    }
	    if (!isObject(this)) {
	      return undefined;
	    }
	    if (!isObject(proto)) {
	      return undefined;
	    }
	    let status = Reflect.setPrototypeOf(this, proto);
	    if (!status) {
	      throw new TypeError();
	    }
	  },
	});
	
	function isObject(value) {
	  return Object(value) === value;
	}

推荐使用

* Object.setPrototypeOf
* Object.getPrototypeOf

### 正式的方法定义
对象方法的简写能够让JS引擎确定定义的是对象的方法。这个方法会有一个额内部的[[HomeObject]]属性来容纳这个方法从属的对象。

### 简化原型访问的super引用
super指向当前对象的原型对象，只能用在对象的方法之中。使用super等同于Object. getPrototypeOf(this)

### Obect.values和Object.entries
Object.values 返回对象可遍历的自有属性的值
Object.entries 返回对象可遍历的自有属性的键值对数组

## ES6中的class

如前文所述，ES6中的class只是ES5基于原型的继承的语法糖。但是ES6为我们提供了很多的便利，让我们可以更简单的使用类的特性。

	class A {}
	typeof A // "function"
	A === A.prototype.constructor // true

类的数据类型就是函数，类本身就指向构造函数。
构造函数的prototype属性，在ES的“类”上继续存在。类的所有实例方法都定义在类的prototype属性上。在类的实例上调用方法，其实就是调用原型上的方法。prototype的constructor属性直接指向类本身，这与ES5的行为是一致的。

就像上文所说的，类的内部定义的所有方法都是不可枚举的。实例的属性除非显示定义在其本身，否则都是定义在原型上。类的所有实例共享一个原型对象。

类的内部无法添加实例属性，只能通过constructor添加。但是可以在类的内部添加set/get属性。

### 类的静态方法
类的属性（方法）分为实例属性（方法）和静态属性（方法）

	class A {
	  static say() {}
	}

say就是A的一个静态方法。类的实例属性（方法）定义在类的原型上。类的静态属性（方法）定义在类本身（即类的构造函数）。静态方法中的this指向类本身。

父类的静态方法，可是被子类集成。

	class A {
	  static say() { console.log(this.name) }
	}
	class B extends A {}
	Object.getPrototypeOf(B) === A // true
	
子类的原型是父类（构造函数）; 所以父类的静态方法和静态属性都能被子类继承。
在JS中创建子类的关键之处就在于，采用合适的方法对原型对象进行初始化。如果类B继承与类A,B的prototype必须是A.prototype的后嗣。所以在ES5中，往往有这样的代码来实现继承：

	function A() {}
	A.prototype = {
		do: function () {},
		constructor: A,
	}
	function B() {}
	B.prototype = Object.create(A.prototype);
	B.prototype.constructor = B
	b instanceof B // true
	b instanceof A // true
	
而在ES6中，相当于多加了这么一句

	Object.setPrototypeOf(B, A)
	
结果就是: B的原型是A，B.prototype的原型是A.prototype，完美的实现了类的实例属性和静态属性的继承。
	
### 语法糖

我们来看Babel转译后的class

编译前

	class A {
	  	static getName() {
	    	return A.name
	    }
	  	static name = 'AAA'
		sayHello() {
	     console.log('Hello')
	    }
		canSay = true
	}
	class B extends A {
		sayHello() {
			super.sayHello();
	      	console.log('Again!');
		}
	}


编译后(只保留了关键部分)

	
	'use strict';
	
	var _get = function get(object, property, receiver) { 
		if (object === null) object = Function.prototype; 
		var desc = Object.getOwnPropertyDescriptor(object, property); 
		if (desc === undefined) { 
			// 若过当前对象没有该属性，则查找该对象的原型对象，重复该流程
			var parent = Object.getPrototypeOf(object); 
			if (parent === null) { 
				// 如果Object.prototype上依然没有该属性，则返回undefined
				return undefined; 
			} else { 
				return get(parent, property, receiver); 
			} 
		} else if ("value" in desc) { 
			// 处理数据属性
			return desc.value; 
		} else { 
			// 处理get/set属性
			var getter = desc.get; 
			if (getter === undefined) { 
				return undefined; 
			} 
			return getter.call(receiver); 
		} 
	};
	
	var _createClass = function () { 
		function defineProperties(target, props) { 
			for (var i = 0; i < props.length; i++) { 
				// 设置属性特性
				var descriptor = props[i]; 
				descriptor.enumerable = descriptor.enumerable || false;
				descriptor.configurable = true; 
				if ("value" in descriptor) descriptor.writable = true; 
				Object.defineProperty(target, descriptor.key, descriptor); 
			} 
		} 
		return function (Constructor, protoProps, staticProps) { 
			if (protoProps) defineProperties(Constructor.prototype, protoProps); 
			if (staticProps) defineProperties(Constructor, staticProps); return Constructor; }; 
	}();
	
	
	function _inherits(subClass, superClass) { 
		subClass.prototype = Object.create(superClass.prototype, { 
			constructor: { value: subClass, enumerable: false, writable: true, configurable: true }, 
		}); 
		Object.setPrototypeOf ? 
			Object.setPrototypeOf(subClass, superClass) 
			: 
			subClass.__proto__ = superClass; 
	}

	
	var A = function () {
	  function A() {

	    this.canSay = true;
	  }
	
	  _createClass(A, [{
	    key: 'sayHello',
	    value: function sayHello() {
	      console.log('Hello');
	    }
	  }], [{
	    key: 'getName',
	    value: function getName() {
	      return A.name;
	    }
	  }]);
	
	  return A;
	}();
	
	A.name = 'AAA';
	
	var B = function (_A) {
	  _inherits(B, _A);
	
	  function B() {

	  }
	
	  _createClass(B, [{
	    key: 'sayHello',
	    value: function sayHello() {
	      _get(B.prototype.__proto__ || Object.getPrototypeOf(B.prototype), 'sayHello', this).call(this);
	      console.log('Again!');
	    }
	  }]);
	
	  return B;
	}(A);

## TypeScript中的class

本段内容皆摘抄自TypeScript中文网，待对TS有更深入的使用和了解后再行更新。  

TS是JS的超集，ES6所有关于class的语法TS都支持，而且TS还支持了更多的特性

一个最简单的🌰：

	class Greeter {
	    greeting: string;
	    constructor(message: string) {
	        this.greeting = message;
	    }
	    greet() {
	        return "Hello, " + this.greeting;
	    }
	}
	
	let greeter = new Greeter("world");

复杂一点的🌰：

	class Animal {
	    name: string;
	    constructor(theName: string) { this.name = theName; }
	    move(distanceInMeters: number = 0) {
	        console.log(`${this.name} moved ${distanceInMeters}m.`);
	    }
	}
	
	class Snake extends Animal {
	    constructor(name: string) { super(name); }
	    move(distanceInMeters = 5) {
	        console.log("Slithering...");
	        super.move(distanceInMeters);
	    }
	}
	
	class Horse extends Animal {
	    constructor(name: string) { super(name); }
	    move(distanceInMeters = 45) {
	        console.log("Galloping...");
	        super.move(distanceInMeters);
	    }
	}
	
	let sam = new Snake("Sammy the Python");
	let tom: Animal = new Horse("Tommy the Palomino");
	
	sam.move();
	tom.move(34);


### 熟悉的公共，私有与受保护的修饰符（public，private，protected）

在TypeScript里，成员都默认为 public。

	class Animal {
	    public name: string;
	    public constructor(theName: string) { this.name = theName; }
	    public move(distanceInMeters: number) {
	        console.log(`${this.name} moved ${distanceInMeters}m.`);
	    }
	}
	===
	class Animal {
	    name: string;
	    constructor(theName: string) { this.name = theName; }
	    move(distanceInMeters: number) {
	        console.log(`${this.name} moved ${distanceInMeters}m.`);
	    }
	}
	
当成员被标记成 private时，它就不能在声明它的类的外部访问。

	class Animal {
	    private name: string;
	    constructor(theName: string) { this.name = theName; }
	}
	
	new Animal("Cat").name; // 错误: 'name' 是私有的.
	
TypeScript使用的是结构性类型系统。 当我们比较两种不同的类型时，并不在乎它们从何处而来，如果所有成员的类型都是兼容的，我们就认为它们的类型是兼容的。
然而，当我们比较带有 private或 protected成员的类型的时候，情况就不同了。 如果其中一个类型里包含一个 private成员，那么只有当另外一个类型中也存在这样一个 private成员， 并且它们都是来自同一处声明时，我们才认为这两个类型是兼容的。 对于 protected成员也使用这个规则。

	class Animal {
	    private name: string;
	    constructor(theName: string) { this.name = theName; }
	}
	
	class Rhino extends Animal {
	    constructor() { super("Rhino"); }
	}
	
	class Employee {
	    private name: string;
	    constructor(theName: string) { this.name = theName; }
	}
	
	let animal = new Animal("Goat");
	let rhino = new Rhino();
	let employee = new Employee("Bob");
	
	animal = rhino;
	animal = employee; // 错误: Animal 与 Employee 不兼容.
	

protected修饰符与 private修饰符的行为很相似，但有一点不同， protected成员在派生类中仍然可以访问。例如：

	class Person {
	    protected name: string;
	    constructor(name: string) { this.name = name; }
	}

	class Employee extends Person {
	    private department: string;
	
	    constructor(name: string, department: string) {
	        super(name)
	        this.department = department;
	    }
	
	    public getElevatorPitch() {
	        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
	    }
	}
	
	let howard = new Employee("Howard", "Sales");
	console.log(howard.getElevatorPitch());
	console.log(howard.name); // 错误
	
构造函数也可以被标记成 protected。 这意味着这个类不能在包含它的类外被实例化，但是能被继承。有趣的是，在ES6里我们是通过判断new.target来实现这一特性的。

	class Person {
	    protected name: string;
	    protected constructor(theName: string) { this.name = theName; }
	}
	
	// Employee 能够继承 Person
	class Employee extends Person {
	    private department: string;
	
	    constructor(name: string, department: string) {
	        super(name);
	        this.department = department;
	    }
	
	    public getElevatorPitch() {
	        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
	    }
	}
	
	let howard = new Employee("Howard", "Sales");
	let john = new Person("John"); // 错误: 'Person' 的构造函数是被保护的.
	
### readonly
	
你可以使用 readonly关键字将属性设置为只读的。 只读属性必须在声明时或构造函数里被初始化。

	class Octopus {
	    readonly name: string;
	    readonly numberOfLegs: number = 8;
	    constructor (theName: string) {
	        this.name = theName;
	    }
	}
	let dad = new Octopus("Man with the 8 strong legs");
	dad.name = "Man with the 3-piece suit"; // 错误! name 是只读的.

### 参数属性

参数属性可以方便地让我们在一个地方定义并初始化一个成员。 下面的例子是对之前 Animal类的修改版，使用了参数属性

	class Animal {
	    constructor(private name: string) { }
	    move(distanceInMeters: number) {
	        console.log(`${this.name} moved ${distanceInMeters}m.`);
	    }
	}

注意看我们是如何舍弃了 theName，仅在构造函数里使用 private name: string参数来创建和初始化 name成员。 我们把声明和赋值合并至一处。

参数属性通过给构造函数参数添加一个访问限定符来声明。 使用 private限定一个参数属性会声明并初始化一个私有成员；对于 public和 protected来说也是一样。

### 存取器

语法基本和JS保持一致。只带有get不带有set的存取器自动被推断为 readonly。

	let passcode = "secret passcode";
	
	class Employee {
	    private _fullName: string;
	
	    get fullName(): string {
	        return this._fullName;
	    }
	
	    set fullName(newName: string) {
	        if (passcode && passcode == "secret passcode") {
	            this._fullName = newName;
	        }
	        else {
	            console.log("Error: Unauthorized update of employee!");
	        }
	    }
	}
	
	let employee = new Employee();
	employee.fullName = "Bob Smith";
	if (employee.fullName) {
	    alert(employee.fullName);
	}
	
### 静态属性

类的实例成员，仅当类被实例化的时候才会被初始化的属性。我们也可以创建类的静态成员，这些属性存在于类本身上面而不是类的实例上。

	class Grid {
	    static origin = {x: 0, y: 0};
	    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
	        let xDist = (point.x - Grid.origin.x);
	        let yDist = (point.y - Grid.origin.y);
	        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
	    }
	    constructor (public scale: number) { }
	}
	
	let grid1 = new Grid(1.0);  // 1x scale
	let grid2 = new Grid(5.0);  // 5x scale
	
	console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
	console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
	
### 抽象类
抽象类做为其它派生类的基类使用。它们一般不会直接被实例化。不同于接口，抽象类可以包含成员的实现细节。abstract关键字是用于定义抽象类和在抽象类内部定义抽象方法。

	abstract class Animal {
	    abstract makeSound(): void;
	    move(): void {
	        console.log('roaming the earch...');
	    }
	}
	
抽象类中的抽象方法不包含具体实现并且必须在派生类中实现。抽象方法的语法与接口方法相似。两者都是定义方法签名但不包含方法体。然而，抽象方法必须包含 abstract关键字并且可以包含访问修饰符。

	abstract class Department {
	
	    constructor(public name: string) {
	    }
	
	    printName(): void {
	        console.log('Department name: ' + this.name);
	    }
	
	    abstract printMeeting(): void; // 必须在派生类中实现
	}
	
	class AccountingDepartment extends Department {
	
	    constructor() {
	        super('Accounting and Auditing'); // 在派生类的构造函数中必须调用 super()
	    }
	
	    printMeeting(): void {
	        console.log('The Accounting Department meets each Monday at 10am.');
	    }
	
	    generateReports(): void {
	        console.log('Generating accounting reports...');
	    }
	}
	
	let department: Department; // 允许创建一个对抽象类型的引用
	department = new Department(); // 错误: 不能创建一个抽象类的实例
	department = new AccountingDepartment(); // 允许对一个抽象子类进行实例化和赋值
	department.printName();
	department.printMeeting();
	department.generateReports(); // 错误: 方法在声明的抽象类中不存在
	
	
### 高级技巧

#### 构造函数
当你在TypeScript里声明了一个类的时候，实际上同时声明了很多东西。首先就是类的实例的类型。
	
	class Greeter {
	    greeting: string;
	    constructor(message: string) {
	        this.greeting = message;
	    }
	    greet() {
	        return "Hello, " + this.greeting;
	    }
	}
	
	let greeter: Greeter;
	greeter = new Greeter("world");
	console.log(greeter.greet());
	
	
这里，我们写了`let greeter: Greeter`，意思是Greeter类的实例的类型是Greeter。

我们也创建了一个叫做 构造函数的值。这个函数会在我们使用new创建类实例的时候被调用。上面这段代码被编译后的js代码：

	let Greeter = (function () {
	    function Greeter(message) {
	        this.greeting = message;
	    }
	    Greeter.prototype.greet = function () {
	        return "Hello, " + this.greeting;
	    };
	    return Greeter;
	})();
	
	let greeter;
	greeter = new Greeter("world");
	console.log(greeter.greet());
	
上面的代码里，let Greeter将被赋值为构造函数。当我们调用new并执行了这个函数后，便会得到一个类的实例。这个构造函数也包含了类的所有静态属性。 换个角度说，我们可以认为类具有实例部分与静态部分这两个部分。

	class Greeter {
	    static standardGreeting = "Hello, there";
	    greeting: string;
	    greet() {
	        if (this.greeting) {
	            return "Hello, " + this.greeting;
	        }
	        else {
	            return Greeter.standardGreeting;
	        }
	    }
	}
	
	let greeter1: Greeter;
	greeter1 = new Greeter();
	console.log(greeter1.greet());
	
	let greeterMaker: typeof Greeter = Greeter;
	greeterMaker.standardGreeting = "Hey there!";
	
	let greeter2: Greeter = new greeterMaker();
	console.log(greeter2.greet());

我们创建了一个叫做greeterMaker的变量。这个变量保存了这个类或者说保存了类构造函数。然后我们使用typeof Greeter，意思是取Greeter类的类型，而不是实例的类型。或者更确切的说，"告诉我 Greeter标识符的类型"，也就是构造函数的类型。这个类型包含了类的所有静态成员和构造函数。之后，就和前面一样，我们在 greeterMaker上使用 new，创建Greeter的实例。

#### 把类当做接口使用

	类定义会创建两个东西：类的实例类型和一个构造函数。因为类可以创建出类型，所以你能够在允许使用接口的地方使用类。
	
	class Point {
	    x: number;
	    y: number;
	}
	
	interface Point3d extends Point {
	    z: number;
	}
	
	let point3d: Point3d = {x: 1, y: 2, z: 3};
	
## 参考资料
* JavaScript权威指南
* 深入理解ES6
* ECMAScript 6 入门
* TypeScript中文网
