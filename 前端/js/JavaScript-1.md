### JavaScript的组成

- 核心（ECMAScript）
- BOM（文件对象模型）
- DOM（浏览器对象模型）

而ECMAScript的组成部分：

- 语法
- 类型
- 语句
- 关键字
- 保留字
- 操作符
- 对象

### 文档模式

​		IE5.5引入文档模式的概念，这个概念是通过文档类型`doctype`来实现的。最初的两个文档类型是：

- 混杂模式

- 标准模式

  这两种模式主要影响css内容的呈现，某些情况下也会影响到Javascript的解释执行。如果文档没有声明文档类型的话，浏览器默认开启混乱模式，这不是神秘好事，我们应该声明文档类型。

```html
<!-- HTML 5 -->
<!DOCTYPE html>
```

### 变量

​		用`var`声明一个变量，即只存在于它所在的作用域，一出它的作用域，变量即被销毁。

```javascript
function test(){
	var food = "apple";	// 局部变量
	console.log(food);
}

test();
alert(food); // 错误！
```

​		可以省略`var`操作符从而创建一个全局变量，不过不推荐这样做，可能会造成代码的混乱以及不便于代码的维护。

```javascript
function test(){
	food = "apple";	// 全局变量
	console.log(food);
}

test();
alert(food); // apple
```

### 数据类型

​		ECMAScript中有5种简单数据类型（基本数据类型）：

- Undefined

- Null

- Boolean

- Number

- String

  1种复杂数据类型`Object`，`Object`本质是由一组无序名值对组成的。

### typeof操纵符

​		typeof用于检测数据类型，它会返回一个表示数据类型的`字符串`：

- "undefined" - 这个值未定义（未声明）或未初始化。
- "boolean" - 这个值是布尔值。
- "string"  - 这个值是字符串。
- "number" - 这个值是数值。
- "object" - 这个值是对象或者null。
- "function" - 这个值是函数。

```javascript
var message;	// 声明message并取得undefined值

// 未定义的age变量
// var age

alert(message);	// 输出undefined
alert(age);	// 错误

alert(typeof message);	// "undefined"
alert(typeof age);	// "undefined"
```

​		`null`是一个空对象指针，因此`typeof`会返回"object"。

### Number

#### NaN

​		NaN，即非数值（Not a number）是一个特殊的数值，这个数值用来表示一个数值本要返回数值的操作数未返回数值的情况（这样就不会抛出异常了），在ECMAScript中，任何数值除于0会得到NAN，不会导致错误从而影响代码的运行。

​		ECMAScript定义了一个函数`isNAN()`，`isNAN()`在接受一个任意类型的参数时，会将它转为数值类型，如果不能被转换成数值的值都会导致这个函数返回true。

```javascript
alert(isNaN(NaN));		// true
alert(isNaN(10));		// false  10是一个数值
alert(isNaN("10"));		// false  可以转换为数值10
alert(isNaN("blue"));	// true   不能转换为数值
alert(isNaN(true));		// false  能转换为1
```

#### 数值转换

​		有三个函数可以把非数值转换为数值：Number()、parseInt()、parseFloat()。

```javascript
var num1 = Number("hello");		// NaN
var num2 = Number("");			// 0
var num3 = Number("000011");	// 11
var num4 = Number(true);		// 1

var num5 = parseInt("1234blue");	// 1234
var num6 = parseInt("0xA");			// 10(十六进制)
var num7 = parseInt("070");			// 56（八进制）
var num8 = parseInt("AF", 16);		// 175(16为第二参数，表示通过十六进制解析，其它同理)
var num9 = parseInt("");			// NAN
```

### object类型

​		ECMAScript中的对象其实就是一组数据和功能的集合。对象可以通过执行`new`操作符来实例化一个对象，并可以为其添加属性和方法。

​		Object类型所具有的所有属性和方法也存在于更具体的对象中。

- Constructor: 构造函数，保存着用于创建当前对象的函数。
- hasOwnProperty(propertyName)：用于判断给定的属性名在当前的对象实例中是否存在。（而不是在实例的原型中）
- isPrototypeOf(object): 用于检查传入的对象是否为另一个对象的原型。
- propertyIsEnumerable(propertyName): 用于检查给定的属性是否能够使用for-in语句来枚举。
- toLocaleString(): 返回对象的字符串表示，该字符串与执行环境的地区对应。
- toString(): 返回对象的字符串表示。
- valueOf(): 返回对象的字符串、数值或布尔值表示。通常与toString()方法的返回值一致。

### 操作符

#### 全等和不全等

```javascript
var result1 = ("55" == 55);	// true, 因为转换后相等
var result2 = ("55" === 55); // false，因为不同的数据类型不相等

var result1 = ("55" != 55);	// false, 因为转换后相等
var result2 = ("55" !== 55); // true，因为不同的数据类型不相等

```

#### for-in语句

​		for-in语句是一种精准的迭代语句，用于枚举对象的全部属性。

```javascript
for(var propName in window){
	document.write(propName);
}
```

​		上面代码会将`window`对象中的所有属性都枚举一遍。对象的属性没有顺序，`for-in`输出的属性的顺序是无法确定的。如果表示要迭代的兑现对象的变量值为`null`或`undefined`时会报错。ECMAScript5更正了这个行为，对这种情况不在报错，而是终止循环。

​		为了最大限度的保障兼容性，最好判断当前属性是否为`null`或`undefined`。

## 变量，作用域和内存问题

### 复制变量值

```javascript
var num1 = 5;
var num2 = num1;
```

​		num2和num1中的5是完全独立的，在任何的操作中都互不影响。

```javascript
var obj1 = new Object();
var obj2 = obj1;
obj1.name = "chao";
alert(obj2.name);	// "chao"
```

​		对象之间进行复制时，也是将储存在变量对象中的值复制一份放到新对象分配的空间中。不同的是，这个值实际上是个指针，而这个指针指向储存在堆中的一个对象。

​		复制结束后，两个变量实际上指向的是同一个对象，任何一个变量对对象进行修改等操作时，会影响到另一个变量。

### 传递参数

​		ECMAScript中的所有函数的参数都是按值传递的。也就是函数外部传入的参数复制给函数内的参数，就和值从一个变量复制到另一个变量一样。

​		向参数传递基本类型的值时，被传递的值复制给局部变量（即函数的命名参数,有是arguments对象中的一个元素）。向参数传递引用类型的值时，会把这个值的内存复制一份给一个局部变量，因此这个局部变量的变化会反映在外部。看下面例子：

```javascript
function addTen(num){
	num += 10;
	return num;
}

var count = 20;
var result = addTen(count);

alert(count);	// 20, 没有变化
alert(result);	// 30
```

​		`addTen()`有一个参数`num`，这个参数实际上是函数的局部变量。传入参数时是将count的值30复制了一份给局部变量num，num和count互不相识，互不影响。如果参数是按引用传递的，那么count的值最后也将变为30。使用数值等基本数据类型比较好理解参数是按值传递的，但对于对象来说，这个问题就不是很好理解了。

```javascript
function setName(obj){
	obj.name = "chao";
}

ver person = new Object();
setName(person);
alert(person.name);	// "chao"

```

​		在setName()中，obj和person引用的是同一个对象，拥有相同的指针，所以当对象obj的属性改变时，也会影响到person的属性。

​		很容易错误的认为：局部作用域中修改的对象会反映到全局变量，这就说明参数是按引用传递的。

​		为证明参数总是按值传递的，请看下面例子。

```javascript
function setName(obj){
	obj.name = "chao";
	obj = new Object();
	obj.name = "Greg";
}

var person = new Object();
setName(person);
alert(person.name);  // "chao"	
```

​		当参数传递到函数的局部变量时，`obj`和`person`引用的时同一个对象，此时设置`obj`的属性`name`会反映到`person`的`name`属性上，但是后面又给`obj`赋了一个新的对象，并且修改`name`属性为`"Greg"`，此时，如果参数是按引用传递，那么就意味着person也被重新赋了一个新对象，并且`name`属性也被修改为`"Greg"`。但是结果表明，`obj`被赋予新对象后，函数引用的就是一个局部对象了，与`person`互不相关。

​		因此，参数是按值传递的。

### 检测类型

​		`typeof`用于检测基本数据类型。但是当我们需要知道被检测变量具体的引用类型时，就要用`instanceof`了。

​		如果变量是给定的引用类型，则返回`true`.

```javascript
alert(person instanceof Object);	// 变量person是Object吗
alert(colors instanceof Array);		// 变量colors是Array吗
alert(pattern instanceof RegExp);	// 变量pattern是RepExp吗
```

### 执行环境

​		执行环境有全局执行环境（全局环境）和函数执行环境。

​		每次进入一个新的执行环境，都会创建一个用于搜索变量和函数的作用域链。

### 垃圾回收

​		Javascript具有自动垃圾收集的机制，开发时不必考虑内存分配和垃圾回收的问题。

- 离开作用域的值将被自动标记为可回收，因此将在垃圾回收期间被删除。
- “标记清除”是目前主流的垃圾收集算法，这个算法的主要思想是给当前不使用的值加上标记，然后再回收其内存。
- 另一种垃圾收集算法是“引用计数”，现在大多数引擎不再用这种算法，因为循环引用的情况可能导致变量永远无法被回收。

## 引用类型

​		**引用类型的值（对象）是引用类型的一个实例**。在ECMAScript中，引用类型是一个数据结构，用于将数据和功能组织在一起。它也经常被称为类。引用类型有时候也被称为**对象定义**，因为它们描述一类对象的属性和方法。

​		引用类型通常包括：Object类型，Array类型，Date类型，RegExp类型，Function类型,Boolean类型，Number类型等。

​		虽然引用类型和类看起来很相似，但是它们并不是相同的概念。

### 数组

#### 栈方法

​		ECMAScript为数组提供了让数组的行为类似于数据结构的方法。这些方法的表示像**栈**的表示一样（。栈是一种`LFIO`（后进先出，last-in-first-out）的数据结构。在数组最后增加项，也在最后减少项。

​		对应数组的方法为`push()`和`pop()`。

#### 队列方法

​		队列的数据结构为`FIFO`（先进先出，first-in-first-out）。在数组最后增加项，在最前减少项。

​		对应数组的方法为`shift()`和`push()`.

#### 迭代方法

​		ES5为数组定义了五个迭代方法。每个方法都接受两个参数：要在每一项上运行的函数和运行该函数的作用域对象（可选） - 影响`this` 的值。

​		传入这些方法的函数接受三个参数，数组当前项的值，数组当前项的索引，数组本身。

- filter(): 对数组中的每一项运行给定函数，返回该函数返回true的项组成的数组。

  ```javascript
  var numbers = [1,2,3,4,5,4,3,2,1];
  
  var filterResult = numbers.filter(function(item, index, array){
  	return (item > 2);
  });
  
  alert(filterResult);	// [3,4,5,4,3]
  ```

- forEach(): 对数组中的每一项运行给定函数。没有返回值。

  ```javascript
  var numbers = [1,2,3,4,5,4,3,2,1];
  
  numbers.foreEach(function(item, index, array){
  	// 执行某些操作
  });
  ```

- map(): 对数组中的每一项运行给定函数，返回每次函数调用的结果组成的数组。

  ```javascript
  var numbers = [1,2,3,4];
  
  var mapResult = numbers.map(function(item, index, array){
  	return (item * 2);
  });
  
  alert(mapResult);	// [2,4,6,8]
  ```

- every(): 对数组中的每一项运行给定函数，如果该函数对**每**一项都返回true，则返回true。

- some(): 对数组中的每一项运行给定函数，如果该函数对**任**一项都返回true，则返回true。

  ```javascript
  var numbers = [1,2,3,4,5,4,3,2,1];
  
  var everyResult = numbers.every(function(item, index, array){
  	return (item > 2);
  });
  
  var someResult = numbers.some(function(item, index, array){
  	return (item > 2);
  });
  
  alert(everyResult);	// false
  alert(someResult);	// true
  ```

以上方法都不修改原数组中的元素的值。

### RegExp类型

​		ES通过RegExp类型来支持正则表达式。下列语法用于创建一个正则表达式。

```javascript
var experssion = / pattern / flags;	
```

​		模式（pattern）部分可以是任何正则表达式，flags代表匹配模式，分别有3个标志。

- g：表示全局模式（global），匹配符合规则的所有字符串。
- i：表示不区分大小写。（case-insensitive）
- m：表示多行模式。（multiline）

### Function类型

​		函数实际上也是一个函数指针，“函数是对象，函数名是指针”。

​		函数没有类似java的重载，后面的同名函数会将前面的函数覆盖。

```javascript
alert(sum(10, 10));

function sum(num1, num2){
	return num1 + num2;
}
```

​		以上代码正常运行，js引擎将函数声明提升到顶部。

#### 函数内部属性

​		在函数内部，有两个特殊的对象：`arguments`和`this`。其中，arguments是一个储存传入参数的类数组对象。此外，arguments还有一个callee属性，该属性是一个指针，指向拥有arguments这个对象的函数。例如下面的经典阶乘函数。

```javascript
function factorial(num){
	if(num <= 1){
		return 1;
	} else {
		return num * factorial(num - 1);
	}
}
```

​		阶乘函数一般要用到递归算法，上述的执行代码与函数名紧密耦合在一起，在函数名不变的情况下没有任何问题，但是当需求要改变该函数名时，问题就来了。我们可以用arguments中的callee属性完美解决问题。

```javascript
function factorial(num){
	if(num <= 1){
		return 1;
	} else {
		return num * arguments.callee(num - 1);
	}
}
```

​		this引用的是函数据以执行的环境对象 -- 或者可以说是this值（当在网页的全局作用域中调用函数时，this对象引用的就是window）。

```javascript
window.color = "red";
var o = {color: "blue"};

function sayColor(){
	alert(this.color);
}

sayColor();			// red

o.sayColor = sayColor;
o.sayColor();		// blue
```

#### bind(), apply(), call()



#### Number类型

toString(), toLocaleString(), valueOf(), toFixed().



#### String类型

toString(), toLocaleString(), valueOf()

1. 字符方法：charAt(), charCodeAt().
2. 字符串操作方法：concat(), slice(), substr(), substring().
3. 字符串位置方法：indexOf(), lastIndexOf().
4. 字符串大小写转换方法：toLowerCase(), toLocaleLowerCase(), toUpperCase()和toLocaleUpperCase().
5. 字符串模式匹配方法：match(), search(), replace(), split().

### 单体内置对象

​		ES实现提供的内置对象，无需再次实例化。比如Global，Math，Object，String等。

1. URI编码方法：

   encodeURI(), encodeURIComponent(), decodeURI(), decodeURIComponent().

2. eval() 方法：

   eval()方法像是一个ES解析器，它接受一个字符串参数，并将这个参数转化话可执行的代码。

3. Global对象的属性

   undefined, NaN等都是Global对象的属性，对应parseInt()等便是它的方法。

4. window对象

   在全局作用域中声明的变量和方法，都会变成window对象的属性和方法。

## 面向对象的程序设计

​		ECMA-262把对象定义为：“无序属性的集合，其属性可以包含基本值、对象或函数。”严格的讲，也就是说对象是一组没有特定顺序的值。可以把对象想象成散列表：就是一组名值对，值可以是数据或者函数。

​		每个对象都是基于一个引用类型创建的，也可以是开发人员定义的类型。

### 属性类型

1. 数据属性

   数据属性有四个描述其行为的特性

   - [[Configurable]]: 表示能否通过delete删除属性，重新定义属性胡总这能否把属性修改为访问器属性。
   - [[Enumerable]]：能否通过for-in返回属性。
   - [[Writable]]: 表示能否修改属性的值。
   - [[Value]]: 包含这个属性的数据值。

   修改属性的默认特性，必须使用Object.defineProperty()方法。接受三个参数：属性所在的对象、属性的名字和一个描述符对象。

   ```javascript
   var person = {};
   Object.defineProperty(person, "name", {
   	writable: false,
   	value: "Nicholas"
   });
   
   alert(person.name);		// "Nicholas"
   person.name = "Grep";
   alert(person.name);		// "Nicholas"
   ```

2. 访问器属性

   访问器属性不包含数据值；它们包含一对getter和setter函数，分别在读取和写入时调用相应访问器。

   - [[Configurable]]: 表示能否通过delete删除属性，重新定义属性胡总这能否把属性修改为访问器属性。
   - [[Enumerable]]：能否通过for-in返回属性。
   - [[Get]]: 在读取属性时调用的函数。默认值为undefined。
   - [[Set]]: 在写入属性时调用的函数。默认值为undefined。

   访问器不能直接定义，必须使用Object.defineProperty()来定义。

   ```javascript
   var book = {
   	_year: 2004,
   	edition: 1
   };
   
   Object.defineProperty(book, "year", {
   	get: function(){
   		return this._year;
   	},
   	
   	set: funciton(newValue){
   		if (newValue > 2004) {
   			this._year = newValue;
   			this.edition += newValue - 2004;
   		}
   	}
   });
   
   book.year = 2005;
   alert(book.edition);	// 2
   ```


#### 定义多个属性

​		ES5提供一个方法Object.defineProperties()。利用此方法可以定义多个属性，接受两个参数：第一个参数是要进行添加和修改其属性的对象，第二个参数是为该对象添加或修改的对象所组成的对象。

```javascript
var book = {};

Object.defineProperties(book, {
	_year: {
		value: 2004
	},
	
	edition: {
		value: 1
	},
	
	year: {
		get: function(){
			return this._year;
		},
		
		set: function(newValue){
			if(newValue > 24){
				this._year = newValue;
				this.edition += newValue - 2004;
			}
		}
	}
});
```

### 创建对象

#### 工厂模式

```javascript
function createPerson(name, age, job){
	var o = new Object();
	o.name = name;
	o.age = age;
	o.job = job;
	o.sayName = function(){
		alert(this.name);
	};
	return o;
}

var person1 = createPerson("Nicholas", 29, "teacher");
var person2 = createPerson("Grep", 19, "student");
```

​		工厂模式解决了创建多个相似对象的问题，但没有解决对象识别的问题（知道这个对象的类型）。于是又有了构造函数模式。

#### 构造函数模式

```javascript
function Person(name, age, job){
	this.name = name;
	this.age = age;
	this.job = job;
	this.sayName = function(){
		alert(this.name);
	};
}

var person1 = new Person("Nicholas", 29, "teacher");
var person2 = new Person("Grep", 19, "student");
```

​		Person()函数不同于createPerson()函数的有：

- 没有显式地创建对象；
- 直接将属性和方法赋给了this对象；
- 没有return语句；

​		此外，函数名Person首字母大写，构造函数应该遵守类的命名的规范，尽管这不是类。主要为了区别于其他函数，构造函数也是函数，只是可以用来创建对象。

​		创建Person的实例，必须使用`new`操作符。这种方式调用构造函数会经历4个步骤。

1. 创建一个新对象；
2. 将构造函数的作用域赋给新对象；（因此this就指向了新对象）
3. 执行构造函数中的代码（为这个对象增加属性和方法）；
4. 返回新对象。

​		这样person1和person2都是Person的实例，这两个对象都有一个`constructor`属性，该属性指向Person。还可以用`instanceof`来判断实例对象的类型。

```javascript
alert(person1.constructor == Person);	// true
alert(person2.constructor == Person);	// true
alert(person1 instanceof Person);	// true
alert(person2 instanceof Object);	// true
alert(person1 instanceof Person);	// true
alert(person1 instanceof Object);	// true
```

​		构造函数的缺点是：内部的函数对象在每次实例化的时候都会重新创建一次，就是说每个实例对象中相同的函数名的函数在内存中并不是同一个函数，它们各自拥有不同的Function实例。上述的构造函数代码中的函数创建和`this.sayName = new Function("alert(this.name)");`在逻辑上是一样的。

​		这样会导致不同的作用域链和标识符解析。创建多个完成同样任务的函数是没有必要的。

​		我们可以通过将函数定义在全局作用域再将函数的指针赋给对象中的函数。不过这样的话要定义多个全局函数，对封装而言又很不友好。我们可以通过原型模式来解决。

#### 原型模式

​		每个函数都有一个原型（prototype）属性，它是一个指针。它指向一个对象，这个对象的用途是包含可以由特定的类型的实例共享的属性和方法。

```javascript
function Person(){
}

Person.prototype.name = "Grep";
Person.prototype.age = 29;
Person.prototype.job = "teacher";
Person.prototype.sayName = function(){
	alert(this.name);
};

var person1 = new Person();
person1.sayName();		// Grep

var person2 = new Person();
person2.sayName();		// Grep

alert(person1.sayName == person2.sayName) 	// true	
```

​		ES5中Object提供一个方法来获取某个实例对象的原型，getPrototypeOf()

```javascript
alert(Object.getPrototypeOf(person1) == Person.prototype)	// true
```

​		通过原型模式创建的实例对象，可以创建自己的属性，若与原型中的属性相同，则调用时会使用实例属性，若找不到实例属性才会去原型中找有没有相同的属性。

```javascript
// 上述代码中添加几行
person1.name = "chao";

alert(person1.name);	// chao -- 来自实例
alert(person2.name);	// Grep -- 来自原型

// 可通过delete删除实例属性并恢复与与原型的连接
delete person1.name;
alert(person1.name);	// Grep	-- 来自原型	
```

​		对实例使用hasOwnProperty()方法可以检测一个实例中是否拥有某个属性。

​		对象的`constructor`和`prototype`属性的[[Enumerable]]被设置为了false，是无法迭代的。我们可以通过`Object.keys()`方法接受一个对象来返回它的可枚举属性的字符串数组

​		或使用`Object.getOwnPropertyName()`方法。

```javascript
var keys1 = Object.keys(Person.prototype);
alert(keys1); 	// "name, age, job, sayName"

var keys2 = Object.getOwnPropertyNmae(Person.prototype);
alert(keys2);	// "constructor name, age, job, sayName"
```

​		更简单的原型语法：（只需写一遍，不过相当于重写了prototype对象，导致constructor属性不指向该构造函数，而是指向Object）

```javascript
Person.prototype = {
	constructor: Person, //	需要重新设置指向
	name: "chao",
	age: 29,
	job: "teacher",
	sayName: function(){
		alert(this.name);
	}
};
```

​		这样又会使constructor属性的[[Enumerable]]特性被设置为了ture，为了修改回来，可使用object.definePrototype()。

```javascript
// 重设构造函数
Object.definePrototype(Person.prototype, "constructor", {
	enumerable: true,
	value: Person
});
```

##### 原型的动态性（P-175）

##### 原型的问题

​		忽略了为构造函数初始化参数的步骤，导致实例默认情况下取得的属性相同，最大的问题是原型的共享属性方法模式，这个在引用的类型最为体现，有时我们并不想每个实例都共享一个属性。

#### 组合使用构造函数模式和原型模式

​		这种混合模式，是目前ES中使用最广泛，认同度最高的一种创建自定义类型的方法。可以说是一种默认模式。

#### 动态原型模式（P-178）

### 继承

​		ES中支持继承，主要通过原型链的方式来实现。

#### 原型链（P-182）

​		原型，构造函数和实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。

​		如果让原型对象等于另一个类型的实例，那么此时原型对象将包含一个指向另一个实例的原型的指针，相应的另一个原型也包含一个指向它自身的构造函数的指针。这样的关系层层递进，就形成了一个原型的链条。

```javascript
function SuperType(){
	this.property = true;
}

SuperType.prototype.getSuperValue = function(){
	return this.property;
};

function SubType(){
	this.subProperty = false;
}

SubType.prototype = new SuperType();

SubType.prototype.getSubValse = function(){
	return this.subProperty;
};

var instance = new SubType();
alert(instance.getSubperValue()		// true	
```

​		上述代码例子中实例以及构造函数和原型之间的关系如下图：

![1564755503526](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564755503526.png)

​		同时，所有的引用类型的原型都指向Object的原型，也是链式继承的关系。

![1564755696585](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564755696585.png)

##### 原型链的问题

​		第一个问题是引用类型的原型属性在实例中共享，一个实例修改了原型属性，也会在其它实例中体现出来。

```javascript
function SuperType(){
	this.colors = ["red", "blue", "green"];
}

function SubType(){
}

SubType.prototype = new SubperType();

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);		// ["red", "blue", "green", "black"]

var instace2 = new Subtype();
alert(instance2.colors);		// ["red", "blue", "green", "black"]
```

​		第二个问题是: 创建子类实例的时候不能向超类的构造函数传递参数。

#### 借用构造函数

​		即在子类内部调用超类的构造方法。通过apply()和call()也可以在（将要）新创建的对象上执行构造函数。

```javascript
function SuperType(){
	this.colors = ["red", "blue", "green"];
}

function SubType(){
	// 继承了SuperType
	SuperType.call(this);
}

var instance1 = new SubType();
instance1.colors.push("black");
alert(instance1.colors);		// ["red", "blue", "green", "black"]

var instace2 = new Subtype();
alert(instance2.colors);		// ["red", "blue", "green"]
```

​		**实际上是在（将要）新创建的`SubType`实例的环境下执行了`SuperType`的构造函数。这样，就会在`SubType`对象上执行`SuperType()`的所有对象的初始化代码**。这样每个SubType实例都具有自己的colors属性的副本了。

​		此外，通过借用构造函数，还可传递参数。

```javascript
function SuperType(name){
	this.name = name;
}

function SubType(){
	// 继承了SuperType, 并传递参数
	SuperType.call(this, "chao");
	
	this.age = 20;
}

var instance = new SubType();
alert(instance.name);		// chao
alert(instance.age);		// 20
```

##### 借用构造函数的问题

​		方法都在函数中定义，那么函数的服用就无从谈起了。而且，在超类中定义的方法，在子类中是不可见的，结果所有类型都只能使用构造函数模式。

#### 组合继承

​		组合原型继承和借用构造函数的长处的继承方式。简单的理解：属性继承用借用构造函数继承，方法继承用原型链。

```javascript
function SuperType(name){
	this.name = name;
	this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
	alert(this.name);
};

function SubType(name, age){
	// 继承属性
	SuperType.call(this, name);
	this.age = age;
}

SubType.prototype = new SuperType();	// 继承方法

SubType.prototype.sayAge = function(){
	alert(this.age);
};

var instance1 = new SubType("chao", 20);
instance1.colors.push("black");
alert(instance1.colors);		// ["red", "blue", "green", "black"]
instance1.sayName();			// "chao"
instance1.sayAge();				// 20

var instace2 = new Subtype("Grep", 29);
alert(instance2.colors);		// ["red", "blue", "green"]
instance2.sayName();			// "Grep"
instance2.sayAge();				// 29
```

​		这种组合继承的缺点在于调用了两次SuperType()，这样会在新对象上两次创建实例属性name和colors，于是，这两个属性屏蔽了原型中的两个同名属性。过程如下图

![1564758911339](C:\Users\叶超\AppData\Roaming\Typora\typora-user-images\1564758911339.png)

#### 寄生组合式继承（最理想的继承范式）

​		寄生式组合继承，即通过借用构造函数来继承属性，通过原型链来继承方法。我们所需要的知识一个超类的副本。本质上，就是使用寄生式继承来继承超类的原型，然后再将结果指定给子类型的原型。

​		寄生组合式继承的基本模式如下：

```javascript
function inheritPrototype(subType, superType){
	var prototype = Object(superType.prototype);	// 创建对象
	prototype.constructor = subType;				// 增强对象
	subType.prototype = prototype;					// 指定对象
}
```

​		这个函数第一步创建了超类的副本，第二步为创建的副本添加constructor属性，为弥补因重写丢失的默认constructor属性。最后一步将新创建的对象（即副本）赋值给子类型的原型。通过调用这个函数，即可替换为子类型原型赋值的语句了。

```javascript
function SuperType(name){
	this.name = name;
	this.colors = ["red", "blue", "green"];
}

SuperType.prototype.sayName = function(){
	alert(this.name);
};

function SubType(name, age){
	// 继承属性
	SuperType.call(this, name);
	this.age = age;
}

inherirPrototype(SubType, SuperType);

SubType.prototype.sayAge = function(){
	alert(this.age);
};
```

​		好处：只调用一次超类构造函数、可传递参数、原型链保持不变、不会创建多余的属性。

