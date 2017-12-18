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
JS对象具有自有属性，同样也有一些属性是从原型对象继承而来的（继承属性）。那么JS对象是如何访问属性的呢？

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
可以通过in、hasOwnProperty、propertyIsEnumerable来判断某个属性是否存在于某个对象中。
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

_____proto_____ 属性; 用来读取或设置当前兑现的原型对象。只有现代浏览器支持该属性。不推荐使用。deprecated。如果需要使用，可以这样进行定义。

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
构造函数的prototype属性，在ES的“类”上继续存在。类的所有实例方法都定义在类的prototype属性上。

## Typescript中的class

