# JavaScript（ES5）的面向对象精要

标签： JavaScript 面向对象 读书笔记

---

2016年1月16日-17日两天看完了《JavaScript 面向对象精要》（参加异步社区的活动送的），这本书虽然不够100页，但都是精华，不愧是《JavaScript 高级程序设计》作者 Nicholas C.Zakas 的最新力作。

下面是我的读书笔记（ES5）：

## 1.原始类型和引用类型

### 1.1 什么是类型

**原始类型** 保存为简单数据值。
**引用类型** 保存为对象，其本质是指向内存位置的引用。

为了让开发者能够把原始类型和引用类型按相同的方式处理，JavaScript 花费了很大的努力来保证语言的一致性。

其他编程语言用栈存原始类型，用对存储引用类型。而 JavaScript 则完全不同：它使用一个变量对象追踪变量的生存期。原始值被直接保存在变量对象内，而引用值则作为一个指针保存在变量对象内，该指针指向实际对象在内存中的存储位置。

### 1.2 原始类型
原始类型代表照原样保存的一些简单数据。
JavaScript 共有 **5** 种原始类型：

 - boolean    布尔，值为 `true` or `false`
 - number     数字，值为任何整型或浮点数值
 - string     字符串，值为由单引号或双引号括住的单个字符或连续字符
 - null       空类型，仅有一个值：null
 - undefined  未定义，只有一个值：undefined（undefined 会被赋给一个还没有初始化的变量）

JavaScript 和许多其他语言一样，原始类型的变量直接保存原始值（而不是一个指向对象的指针）。

```js    
var color1 = "red";
var color2 = color1;

console.log(color1); // "red"
console.log(color2); // "red"

color1 = "blue";

console.log(color1); // "blue"
console.log(color2); // "red"
```

#### 鉴别原始类型

鉴别原始类型的最佳方式是使用 `typeof` 操作符。

```js    
console.log(typeof "Nicholas"); // "string"
console.log(typeof 10);         // "number"
console.log(typeof true);       // "boolean"
console.log(typeof undefined);  // "undefined"
```

至于空类型（null）则有些棘手。

```js    
console.log(typeof null); // "object"
```
    
对于 typeof null，结果是"object"。（其实这已被设计和维护 JavaScript 的委员会 TC39 认定是一个错误。在逻辑上，你可以认为 `null` 是一个空的对象指针，所以结果为"object"，但这还是很令人困惑。）

判断一个值是否为空类型（null）的最佳方式是直接和 `null` 比较：

```js
console.log(value === null); // true or false
```

> **注意：以上这段代码使用了三等号（全等 ===）**，因为三等号（全等）不会将变量强制转换为另一种类型。

```js
console.log("5" == 5); // true
console.log("5" === 5); // false

console.log(undefined == null); // true
console.log(undefined === null); // false
```
    
#### 原始方法

虽然字符串、数字和布尔值是原始类型，但是它们也拥有方法（null 和 undefined 没有方法）。

```js 
var name = "Nicholas";
var lowercaseName = name.toLowerCase(); // 转为小写

var count = 10;
var fixedCount = count.toFixed(2); // 转为10.00

var flag = true;
var stringFlag = flag.toString(); // 转为"true"

console.log("YIBU".charAt(0)); // 输出"Y"
```    

> 尽管原始类型拥有方法，但它们不是对象。JavaScript 使它们看上去像对象一样，以此来提高语言上的一致性体验。

### 1.3 引用类型

引用类型是指 JavaScript 中的对象，同时也是你在该语言中能找到最接近类的东西。
引用值是引用类型的实例，也是对象的同义词（后面将用对象指代引用值）。对象是属性的无序列表。属性包含键（始终是字符串）和值。如果一个属性的值是函数，它就被称为方法。除了函数可以运行以外，一个包含数组的属性和一个包含函数的属性没有什么区别。

#### 创建对象

有时候，把 JavaScript 对象想象成哈希表可以帮助你更好地理解对象结构。

![Object][1]  

JavaScript 有好几种方法可以创建对象，或者说实例化对象。第一种是使用 `new` 操作符和构造函数。
构造函数就是通过 `new` 操作符来创建对象的函数——任何函数都可以是构造函数。根据命名规范，JavaScript  中的构造函数用**首字母大写**来跟非构造函数进行区分。

```js    
var object = new Object();
```

因为引用类型不再变量中直接保存对象，所以本例中的 `object` 变量实际上并**不包含对象的实例，而是一个指向内存中实际对象所在位置的指针（或者说引用）**。这是对象和原始值之间的一个基本差别，原始值是直接保存在变量中。

当你将一个对象赋值给变量时，实际是赋值给这个变量一个指针。这意味着，将一个变量赋值给另外一个变量时，两个变量各获得了一份指针的拷贝，指向内存中的同一个对象。

```js
var obj1 = new Object();
var obj2 = obj1;
```

![copy_obj][2]  

#### 对象引用解除

JavaScript 语言有垃圾收集的功能，因此当你使用引用类型时无需担心内存分配。**但最好在不使用对象时将其引用解除，让垃圾收集器对那块内存进行释放。解除引用的最佳手段是将对象变量设置为 `null`。**

```js
var obj1 = new Object();
// dosomething
obj1 = null; // dereference
```

#### 添加删除属性

在 JavaScript 中，你可以随时添加和删除其属性。

```js
var obj1 = new Object();
var obj2 = obj1;

obj1.myCustomProperty = "Awsome!";
console.log(obj2.myCustomProperty); // "Awsome!" 因为 obj1 和 obj2 指向同一个对象。
```

### 1.4 内建类型实例化

内建类型如下：

 - Array       数组类型，以数字为索引的一组值的有序列表
 - Date        日期和时间类型
 - Error       运行期错误类型
 - Function    函数类型
 - Object      通用对象类型
 - RegExp      正则表达式类型    
 
可使用 `new` 来实例化每一个内建引用类型：

```js    
var items = new Array();
var now = new Date();
var error = new Error("Something bad happened.");
var func = new Function("console.log('HI');");
var object = new Object();
var re = new RegExp();
```

#### 字面形式

内建引用类型有字面形式。字面形式允许你在不需要使用 `new` 操作符和构造函数显示创建对象的情况下生成引用值。属性的**键**可以是标识符或字符串（若含有空格或其他特殊字符）

```js    
var book = {
  name: "Book_name",
  year: 2016
}
```

上面代码与下面这段代码等价：

```js
var book = new Object();
book.name = "Book_name";
book.year = 2016;
```

> 虽然使用字面形式并没有调用 new Object()，但是 JavaScript 引擎背后做的工作和 new Object() 一样，除了没有调用构造函数。其他引用类型的字面形式也是如此。


### 1.5 访问属性

可通过 `.` 和 `中括号` 访问对象的属性。
中括号 `[]` 在需要动态决定访问哪个属性时，特别有用。因为你可以用**变量**而不是字符串字面形式来指定访问的属性。

### 1.6 鉴别引用类型

函数是最容易鉴别的引用类型，因为对函数使用 `typeof` 操作符时，返回"function"。

```js    
function reflect(value){
  return value;
}
console.log(typeof reflect); // "function"
```

对其他引用类型的鉴别则较为棘手，因为对于所有非函数的引用类型，`typeof` 返回 `object`。为了更方便地鉴别引用类型，可以使用 JavaScript 的 `instanceof` 操作符。

```js    
var items = [];
var obj = {};
function reflect(value){
  return value;
}

console.log(items instanceof Array); // true;
console.log(obj instanceof Object); // true;
console.log(reflect instanceof Function); // true;
```
   
`instanceof` 操作符可鉴别继承类型。这意味着所有对象都是 `Oject` 的实例，因为所有引用类型都继承自 `Object`。

> 虽然 instanceof 可以鉴别对象类型（如数组），但是有一个列外。JavaScript 的值可以在同一个网页的不用框架之间传来传去。由于每个网页拥有它自己的全局上下文—— Object、Array 以及其他内建类型的版本。所以当你把一个对象（如数组）从一个框架传到另外一个框架时，instanceof 就无法识别它。

### 1.8 原始封装类型

原始封装类型有 `3` 种：String、Number 和 Boolean。
当读取字符串、数字或布尔值时，原始封装类型将被自动创建。

```js    
var name = "Nicholas";
var firstChar = name.charAt(0); // "N"
```
这在背后发生的事情如下：
    
```js
var name = "Nichola";
var temp = new String(name);
var firstChar = temp.charAt(0);
temp = null;
```

由于第二行把字符串当成对象使用，JavaScript 引擎创建了一个字符串的实体让 `charAt(0)` 可以工作。字符串对象的存在仅用于该语句并在随后销毁（一种被称为自动打包的过程）。为了测试这一点，试着给字符串添加一个属性看看它是不是对象。

```js
var name = "Nicholas";
name.last = "Zakas";

console.log(name.last); // undefined;
```

下面是在 JavaScript 引擎中实际发生的事情：

```js
var name = "Nicholas";
var temp = new String(name);
temp.last = "Zakas";
temp = null; // temporary object destroyed

var temp = new String(name);
console.log(temp.last);
temp = null;
```

新属性 `last` 实际上是在一个立刻就被销毁的临时对象上而不是字符串上添加。之后当你试图访问该属性时，另一个不同的临时对象被创建，而新属性并不存在。

虽然原始封装类型会被自动创建，在这些值上进行 `instanceof` 检查对应类型的返回值却是 `false`。
这是因为**临时对象仅在值被读取时创建**。`instanceof` 操作符并没有真的读取任何东西，也就没有临时对象的创建。

当然你也可以手动创建原始封装类型。

```js
var str = new String("me");
str.age = 18;

console.log(typeof str); // object
console.log(str.age); // 18
```
    
如你所见，手动创建原始封装类型实际会创建出一个 `object`。这意味着 `typeof` 无法鉴别出你实际保存的数据的类型。
    
另外，手动创建原始封装类型和使用原始值是有一定区别的。所以尽量避免使用。

```js
var found = new Boolean(false);
if(found){
  console.log("Found"); // 执行到了，尽管对象的值为 false
}
```

这是因为一个对象(如 `{}` )在条件判断语句中总被认为是 `true`;

> MDN: Any object whose value is not undefined or null, including a Boolean oject whose value is false, evaluates to true when passed to a conditional statement.

### 1.9 总结

第一章的东西都是我们一些比较熟悉的知识。但是也有一些需要注意的地方：

 - 正确区分原始类型和引用类型
 - 对于 `5` 种原始类型都可以用 typeof 来鉴别，而空类型必须直接跟 `null` 进行全等比较。
 - 函数也是对象，可用 `typeof` 鉴别。其它引用类型，可用 `instanceof` 和一个构造函数来鉴别。（当然可以用  `Object.prototype.toString.call()` 鉴别，它会返回 [object Array]之类的）。
 - 为了让原始类型看上去更像引用类型，JavaScript提供了 `3` 种封装类型。JavaScript会在背后创建这些对象使得你能够像使用普通对象那样使用原始值。但这些临时对象在使用它们的语句结束时就立刻被销毁。虽然可手动创建，但不建议。

## 2. 函数

函数也是对象，使对象不同于其它对象的决定性特点是函数存在一个被称为 `[[Call]]` 的内部属性。  
**内部属性无法通过代码访问而是定义了代码执行时的行为**。ECMAScript为JavaScript的对象定义了多种内部属性，这些**内部属性都用双重中括号来标注**。

**[[Call]] 属性是函数独有的，表明该对象可以被执行。由于仅函数拥有该属性，ECMAScript 定义 typeof 操作符对任何具有 [[Call]] 属性的对象返回 "function"**。过去因某些浏览器曾在正则表达式中包含 `[[Call]]` 属性，导致正则表达式被错误鉴别为函数。
    
    
### 2.1 声明还是表达式

两者的一个重要区别是：函数声明会被提升至上下文（要么是该函数被声明时所在的函数范围，要么是全局范围）的顶部。
    
### 2.2 函数就是值

可以像使用对象一样使用函数（因为函数本来就是对象，Function 构造函数更加容易说明）。

### 2.3 参数

函数参数保存在类数组对象 `arguments` （`Array.isArray(arguments)` 返回 `false`）中。可以接收任意数量的参数。
函数的 `length` 属性表明其期望的参数个数。

### 2.4 重载

大多数面向对象语言支持函数重载，它能让一个函数具有多个签名。函数签名由函数的名字、参数的个数及其类型组成。
而 JavaScript 可以接收任意数量的参数且参数类型完全没有限制。这说明 JavaScript 函数根本就没有签名，因此也不存在重载。

```js    
function sayMessage(message){
  console.log(message);
}
function sayMessage(){
  console.log("Default Message");
}

sayMessage("Hello!"); // 输出"Default Message";
```

在 JavaScript 里，当你试图定义多个同名的函数时，只有最后的定义有效，之前的函数声明被完全删除（函数也是对象，变量只是存指针)。

```js
var sayMessage = new Function("message", "console.log(message)");
var sayMessage = new Function("console.log(\"Default Message\");");

sayMessage("Hello!"); 
```

当然，你可以根据传入参数的数量来模仿重载。

### 2.5 对象方法

对象的值是函数，则该属性被称为方法。

#### 2.5.1 this 对象

JavaScript 所有的函数作用域内都有一个 `this` 对象代表调用该函数的对象。在全局作用域中，`this` 代表全局对象（浏览器里的 window）。当一个函数作为对象的方法调用时，默认 `this` 的值等于该对象。
**this在函数调用时才被设置。**

```js
function sayNameForAll(){
  console.log(this.name);
}

var person1 = {
  name: "Nicholas",
  sayName: sayNameForAll
}

var name = "Jack";
    
person1.sayName(); // 输出 "Nicholas"
sayNameforAll(); // 输出 "Jack"
```

#### 2.5.2 改变 this

有 `3` 种函数方法运行你改变 `this` 值。

 1. fun.call(thisArg[, arg1[, arg2[, ...]]]);
 2. fun.apply(thisArg, [argsArray]);
 3. fun.bind(thisArg[, arg1[, arg2[, ...]]])
 
使用 `call` 或 `apply` 方法，就不需要将函数加入每个对象——你显示地指定了 `this` 的值而不是让 JavaScript 引擎自动指定。

`call` 与 `apply` 的不同地方是，`call` 需要把所有参数一个个列出来，而 `apply` 的参数需要一个数组或者类似数组的对象（如 `arguments` 对象）。

`bind` 是 ECMAScript 5 新增的，它会创建一个新函数返回。其参数与 `call` 类似，而且其所有参数代表需要被**永久**设置在新函数中的命名参数（绑定了的参数（没绑定的参数依然可以传入），就算调用时再传入其它参数，也不会影响这些绑定的参数）。

```js
function sayNameForAll(label){
  console.log(label + ":" + this.name);
}
var person = {
  name: "Nicholas"
}

var sayNameForPerson = sayNameForAll.bind(person);
sayNameForPerson("Person"); // 输出"Person:Nicholas"

var sayName = sayNameForAll.bind(person, "Jc");

sayName("change"); // 输出"Jc:Nicholas" 因为绑定的形参，会忽略调用时再传入参数
```

### 2.6 总结

 - 函数也是对象，所以它可以被访问、复制和覆盖。
 - 函数与其他对象最大的区别在于它们有一个特殊的内部属性 `[[Call]]`，包含了该函数的执行指令。
 - 函数声明会被提升至上下文的顶部。
 - 函数是对象，所以存在一个 `Function` 构造函数。但这会使你的代码难以理解和调试，除非函数的真实形式要直到运行时才能确定的时候才会利用它。

## 理解对象

JavaScript 中的对象是动态的，可在代码执行的任意时刻发生改变。基于类的语言会根据类的定义锁定对象。

### 3.1 定义属性

当一个属性第一次被添加到对象时，JavaScript 会在对象上调用一个名为 `[[Put]]` 的内部方法。`[[Put]]` 方法会在对象上创建一个新节点来保存属性。
当一个已有的属性被赋予一个新值时，调用的是一个名为 `[[Set]]` 的方法。

### 3.2 属性探测

检查对象是否已有一个属性。JavaScript 开发新手错误地使用以下模式检测属性是否存在。

```js
if(person.age){
  // do something with ag
}
```

上面的问题在于 JavaScript 的类型强制会影响该模式的输出结果。  
当 if 判断中的值如下时，会判断为**真**：

 - 对象
 - 非空字符串
 - 非零
 - true

当 if 判断中的值如下时，会判断为**假**：

 - null
 - undefined
 - 0
 - false
 - NaN
 - 空字符串

因此判断属性是否存在的方法是使用 `in` 操作符。
`in` 操作符会检查**自有属性和原型属性**。
所有的对象都拥有的 `hasOwnProperty()` 方法（其实是 `Object.prototype` 原型对象的），该方法在给定的属性存在且为**自有属性**时返回 `true`。

```js
var person = {
  name: "Nicholas"
}

console.log("name" in person); // true
console.log(person.hasOwnpropert("name")); // true

console.log("toString" in person); // true
console.log(person.hasOwnproperty("toString")); // false
```
    
### 3.3 删除属性

设置一个属性的值为 `null` 并不能从对象中彻底移除那个属性，这只是调用 `[[Set]]` 将 `null` 值替换了该属性原来的值而已。
`delete` 操作符针对单个对象属性调用名为 `[[Delete]]` 的内部方法。删除成功时，返回 `true`。

```js
var person = {
  name: "Nicholas"
}

person.name = null;
console.log("name" in person); // true
delete person.name;
console.log(person.name); // undefined 访问一个不存在的属性将返回 undefined
console.log("name" in person); // false
```

### 3.4 属性枚举

所有人为添加的属性默认都是可枚举的。可枚举的内部特征 `[[Enumerable]]` 都被设置为 `true`。
`for-in` 循环会枚举一个对象所有的可枚举属性。

> 我的备注：在 Chrome 中，对象属性会按 ASCII 表排序，而不是定义时的顺序。

ECMAScript 5 的 Object() 方法可以获取可枚举属性的名字的数组。

```js
var person = {
  name: "Ljc",
  age: 18
}

Object.keys(person); // ["name", "age"];
```

`for-in` 与 `Object.keys()` 的一个区别是：前者也会遍历原型属性，而后者返回自有(实例)属性。

实际上，对象的大部分原生方法的 `[[Enumerable]]` 特征都被设置为 `false`。可用 `propertyIsEnumerable()` 方法检查一个属性是否为可枚举的。

```js
var arr = ["abc", 2];
console.log(arr.propertyIsEnumerable("length")); // false
```

### 3.5 属性类型

属性有两种类型：**数据属性**和**访问器属性**。
数据属性包含一个值。`[[Put]]` 方法的默认行为是创建**数据属性**。
访问器属性不包含值而是定义了一个当属性被读取时调用的函数（称为 `getter`）和一个当属性被写入时调用的函数（称为 `setter`）。访问器属性仅需要 `getter` 或 `setter` 两者中的任意一个，当然也可以两者。

```js    
// 对象字面形式中定义访问器属性有特殊的语法：
var person = {
  _name: "Nicholas",

  get name(){
    console.log("Reading name");
    return this._name;
  },
  set name(value){
    console.log("Setting name to %s", value);
    this._name = value;
  }
};
    
console.log(person.name); // "Reading name" 然后输出 "Nicholas"

person.name = "Greg";
console.log(person.name); // "Setting name to Greg" 然后输出 "Greg"
```    

> 前置下划线 _ 是一个约定俗成的命名规范，表示该属性是私有的，实际上它还是公开的。

访问器就是定义了我们在对象读取或设置属性时，触发的动作（函数），`_name` 相当于一个内部变量。
当你希望赋值（读取）操作会触发一些行为，访问器就会非常有用。

> 当只定义 getter 或 setter 其一时，该属性就会变成只读或只写。

### 3.6 属性特征

在 ECMAScript 5 之前没有办法指定一个属性是否可枚举。实际上根本没有方法访问属性的任何内部特征。为了改变这点，ECMAScript 5 引入了多种方法来和属性特征值直接互动。

#### 3.6.1 通用特征

数据属性和访问器属性均由以下两个属性特制：
`[[Enumerable]]` 决定了是否可以遍历该属性；
`[[Configurable]]` 决定了该属性是否可配置。

所有人为定义的属性默认都是可枚举、可配置的。

可以用 `Object.defineProperty()` 方法改变属性特征。
其参数有三：拥有该属性的对象、属性名和包含需要设置的特性的属性描述对象。

```js    
var person = {
  name: "Nicholas"
}
Object.defineProperty(person, "name", {
  enumerable: false
})

console.log("name" in person); // true
console.log(person.propertyIsEnumerable("name")); // false

var properties = Object.keys(person);
console.log(properties.length); // 0

Object.defineProperty(person, "name",{
  configurable: false
})

delete person.name; // false
console.log("name" in person); // true

Object.defineProperty(person, "name",{ // error! 
  // 在 chrome：Uncaught TypeError: Cannot redefine property: name
  configurable: true
})
```    

> 无法将一个不可配置的属性变为可配置，相反则可以。

#### 3.6.2 数据属性特征

数据属性额外拥有两个访问器属性不具备的特征。
`[[Value]]` 包含属性的值(哪怕是函数)。
`[[Writable]]` 布尔值，指示该属性是否可写入。所有属性默认都是可写的。

```js
var person = {};

Object.defineProperty(person, "name", {
  value: "Nicholas",
  enumerable: true,
  configurable: true,
  writable: true
})
```

在 `Object.defineProperty()` 被调用时，如果属性本来就有，则会按照新定义属性特征值去覆盖默认属性特征（`enumberable`、`configurable` 和 `writable` 均为 `true`）。但如果用该方法定义新的属性时，没有为所有的特征值指定一个值，则所有布尔值的特征值会被默认设置为 `false`。即不可枚举、不可配置、不可写的。
当你用 `Object.defineProperty()` 改变一个已有的属性时，只有你指定的特征会被改变。

#### 3.6.3 访问器属性特征

访问器属性额外拥有两个特征。`[[Get]]` 和 `[[Set]]`，内含 `getter` 和 `setter` 函数。
使用访问其属性特征比使用对象字面形式定义访问器属性的优势在于：可以为已有的对象定义这些属性。而后者只能在创建时定义访问器属性。

```js
var person = {
  _name: "Nicholas"
};

Object.defineProperty(person, "name", {
  get: function(){
    return this._name;
  },
  set: function(value){
    this._name = value;
  },
  enumerable: true,
  configurable: true
})

for(var x in person){
  console.log(x); // _name \n(换行) name（访问器属性）
}
```    
    
设置一个不可配置、不可枚举、不可以写的属性：

```js
Object.defineProperty(person, "name",{
  get: function(){
    return this._name;
  }
})
```

对于一个新的访问器属性，没有显示设置值为布尔值的属性，默认为 `false`。

#### 3.6.4 定义多重属性

`Object.defineProperties()` 方法可以定义任意数量的属性，甚至可以同时改变已有的属性并创建新属性。

```js    
var person = {};

Object.defineProperties(person, {

  // data property to store data
  _name: {
    value: "Nicholas",
    enumerable: true,
    configurable: true,
    writable: true
  },

  // accessor property
  name: {
    get: function(){
      return this._name;
    },
    set: function(value){
      this._name = value;
    }
  }
})
```

#### 3.6.5 获取属性特征

`Object.getOwnPropertyDescriptor()` 方法。该方法接受两个参数：对象和属性名。如果属性存在，它会返回一个属性描述对象，内涵 `4` 个属性：`configurable` 和 `enumerable`，另外两个属性则根据属性类型决定。

```js
var person = {
  name: "Nicholas"
}

var descriptor = Object.getOwnPropertyDescriptor(person, "name");

console.log(descriptor.enumerable); // true
console.log(descriptor.configuable); // true
console.log(descriptor.value); // "Nicholas"
console.log(descriptor.wirtable); // true
```

### 3.7 禁止修改对象

对象和属性一样具有指导其行为的内部特性。其中， `[[Extensible]]` 是布尔值，指明该对象本身是否可以被修改。默认是 `true`。当值为 `false` 时，就能禁止新属性的添加。

> 建议在 "use strict"; 严格模式下进行。

#### 3.7.1 禁止扩展

`Object.preventExtensions()` 创建一个不可扩展的对象（即**不能添加新属性**）。
`Object.isExtensible()` 检查 `[[Extensible]]` 的值。

```js
var person = {
  name: "Nocholas"
}

Object.preventExtensions(person);

person.sayName = function(){
  console.log(this.name)
}

console.log("sayName" in person); // false
```

#### 3.7.2 对象封印

一个被封印的对象是不可扩展的且其所有属性都是不可配置的（即不能添加、删除属性或修改其属性类型（从数据属性变成访问器属性或相反））。**只能读写它的属性**。
Object.seal()。调用此方法后，该对象的 `[[Extensible]]` 特征被设置为 `false`，其所有属性的 `[[configurable]]` 特征被设置为 `false`。
`Object.isSealed()` 判断一个对象是否被封印。

#### 3.7.3 对象冻结
被冻结的对象不能添加或删除属性，不能修改属性类型，也不能写入任何数据属性。简言而之，被冻结对象是一个**数据属性都为只读**的被封印对象。

 - `Object.freeze()` 冻结对象。
 - `Object.isFrozen()` 判断对象是否被冻结。

### 3.8 总结

 - `in` 操作符检测自有属性和原型属性，而 `hasOwnProperty()` 只检查自有属性。
 - 用 `delete` 操作符删除对象属性。
 - 属性有两种类型：数据属性和访问器属性。
 - 所有属性都有一些相关特征。`[[Enumerable]]` 和 `[[Configurable]]` 的两种属性都有的，而数据属性还有 `[[Value]]` 和 `[[Writable]]`，访问器属性还有 `[[Get]]` 和 `[[Set]]`。可通过 `Object.defineProperty()` 和 `Object.defineProperties()` 改变这些特征。用 `Object.getOwnPropertyDescriptor()` 获取它们。
 - 有 `3` 种可以锁定对象属性的方式。

    
## 4. 构造函数和原型对象

由于 JavaScript(ES5) 缺乏类，但可用构造函数和原型对象给对象带来与类相似的功能。

### 4.1 构造函数

构造函数的函数名首字母应大写，以此区分其他函数。
当没有需要给构造函数传递参数，可忽略小括号：

```js    
var Person = {
  // 故意留空
}
var person = new Person;
```

尽管 Person 构造函数没有显式返回任何东西，但 new 操作符会自动创建给定类型的对象并返回它们。

每个对象在创建时都自动拥有一个构造函数属性（constructor，其实是它们的原型对象上的属性），其中包含了一个指向其构造函数的引用。
通过对象字面量形式（{}）或 Object 构造函数创建出来的泛用对象，其构造函数属性（constructor）指向  Object；而那些通过自定义构造函数创建出来的对象，其构造函数属性指向创建它的构造函数。
 
```js  
console.log(person.constructor === Person); // true
console.log(({}).constructor === Object); // true
console.log(([1,2,3]).constructor === Object); // true

// 证明 constructor 是在原型对象上
console.log(person.hasOwnPrototype("constructor")); // false
console.log(person.constructor.prototype.hasOwnPrototype("constructor")); // true
```

尽管对象实例及其构造函数之间存在这样的关系，但还是建议使用 `instanceof` 来检查对象类型。这是因为构造函数属性可以被覆盖。（person.constructor = ""）。

当你调用构造函数时，new 会自动自动创建 this 对象，且其类型就是构造函数的类型（构造函数就好像类，相当于一种数据类型）。

> 你也可以在构造函数中显式调用 return。如果返回值是一个对象，它会代替新创建的对象实例而返回，如果返回值是一个原始类型，它会被忽略，新创建的对象实例会被返回。

始终确保要用 new 调用构造函数；否则，你就是在冒着改变全局对象的风险，而不是创建一个新的对象。

```js
var person = Person("Nicholas"); // 缺少 new

console.log(person instanceof Person); // false
console.log(person); // undefined，因为没用 new，就相当于一个普通函数，默认返回 undefined
console.log(name); // "Nicholas"
```

当 Person 不是被 new 调用时，构造函数中的 this 对象等于全局 this 对象。

> 在严格模式下，会报错。因为严格模式下，并没有为全局对象设置 this，this 保持为 undefined。

以下代码，通过 new 实例化 100 个对象，则会有 100 个函数做相同的事。因此可用 `prototype` 共享同一个方法会更高效。

```js
var person = {
  name: "Nicholas",
  sayName: function(){
    console.log(this.name);
  }
}
```

### 4.2 原型对象

可以把原型对象看作是对象的基类。几乎所有的函数（除了一些内建函数）都有一个名为 prototype 的属性，该属性是一个原型对象用来创建新的对象实例。所有创建的对象实例（同一构造函数，当然，可能访问上层的原型对象）**共享**该原型对象，且这些对象实例可以访问原型对象的属性。例如，hasOwnProperty() 定义在 Object 的原型对象中，但却可被任何对象当作自己的属性访问。

```js
var book = {
  title : "book_name"
}

"hasOwnProperty" in book; // true
book.hasOwnProperty("hasOwnProperty"); // false
Object.property.hasOwnProperty("hasOwnProperty"); // true
```    

**鉴别一个原型属性**
   
```js
function hasPrototypeProperty(object, name){
  return name in object && !object.hasOwnProperty(name);
}
```

#### 4.2.1 [[Prototype]] 属性

一个对象实例通过内部属性 [[Prototype]] 跟踪其原型对象。该属性是一个指向该实例使用的原型对象的指针。当你用 new 创建一个新的对象时，构造函数的原型对象就会被赋给该对象的 [[Prototype]] 属性。

![prototype][3]  

由上图可以看出，[[Prototype]] 属性是如何让多个对象实例引用同一个原型对象来减少重复代码。

Object.getPrototypeOf() 方法可读取 [[Prototype]] 属性的值。

```js
var obj = {};
var prototype = Object.getPrototypeOf(Object);

console.log(prototype === Object.prototype); // true
```

> 大部分 JavaScript 引擎在所有对象上都支持一个名为 `__proto__` 的属性。该属性使你可以直接读写 [[Prototype]] 属性。

isPrototypeOf() 方法会检查某个对象是否是另一个对象的原型对象，该方法包含在所有对象中。

```js
var obj = {}
console.log(Object.prototype.isPrototypeOf(obj)); // true
```

当读取一个对象的属性时，JavaScript 引擎首先在该对象的自有属性查找属性名。如果找到则返回。否则会搜索 [[Prototype]] 中的对象，找到则返回，找不到则返回 undefined。

```js
var obj = new Object();
console.log(obj.toString()); // "[object Object]"

obj.toString = function(){
  return "[object Custom]";
}
console.log(obj.toString()); // "[object Custom]"

delete obj.toString; // true
console.log(obj.toString()); // "[object Object]"

delete obj.toString; // 无效，delete不能删除一个对象从原型继承而来的属性
cconsole.log(obj.toString()); // // "[object Object]"
```

> MDN：delete 操作符不能删除的属性有：①显式声明的全局变量不能被删除,该属性不可配置（not configurable）； ②内置对象的内置属性不能被删除； ③不能删除一个对象从原型继承而来的属性(不过你可以从原型上直接删掉它)。

一个重要概念：无法给一个对象的原型属性赋值。但我们可以通过 `obj.constructor.prototype.sayHi = function(){console.log("Hi!")}` 向原型对象添加属性。

![此处输入图片的描述][4]   
（图片中间可以看出，为对象 obj 添加的 toString 属性代替了原型属性）

#### 4.2.2 在构造函数中使用原型对象

##### 在原型对象上定义公用方法

##### 在原型对象上定义数据类型

开发中需要注意原型对象的数据是否共享。

```js
function Person(name){
  this.name = name
}

Person.prototype.sayName = function(){
  console.log(this.name);
}

Person.prototype.position = "school";
Person.prototype.arr = [];

var person1 = new Person("xiaoming");
var person2 = new Person("Jc");

console.log("原始类型")
console.log(person1.position); // "school"
console.log(person2.position); // "school"

person1.position = 2; // 这是在当前属性设置position，引用类型同理
console.log(person1.hasOwnProperty("position")); // true
console.log(person2.hasOwnProperty("position")); // false

console.log("引用类型");
person1.arr.push("pizza"); // 这是在原型对象上设置，而不是直接在对象上
person2.arr.push("quinoa"); // 这是在原型对象上设置
console.log(person1.hasOwnProperty("arr")); // false
console.log(person2.hasOwnProperty("arr")); // false
console.log(person1.arr); // ["pizza", "quinoa"]
console.log(person2.arr); // ["pizza", "quinoa"]
```

上面是在原型对象上一一添加属性，下面一种更简洁的方式：以一个对象字面形式替换原型对象

```js
function Person(name){
  this.name = name
}

Person.prototype = {
  sayName: function(){
    console.log(this.name);
  },
  toString: function(){
    return "[Person ]" + this.name + "]";
  }
}
```

这种方式有一种副作用：因为原型对象上具有一个 `constructor` 属性，这是其他对象实例所没有的。当一个函数被创建时，它的  `prototype` 属性也会被创建，且该原型对象的 `constructor` 属性指向该函数。当使用字面量时，因没显式设置原型对象的 `constructor` 属性，因此其 `constructor` 属性是指向 `Object` 的。
因此，当通过此方式设置原型对象时，可手动设置 `constructor` 属性。

```js    
function Person(name){
  this.name = name
}

// 建议第一个属性就是设置其 constructor 属性。
Person.prototype = {
  constructor: Person,

  sayName: function(){
    console.log(this.name);
  },
  toString: function(){
    return "[Person ]" + this.name + "]";
  }
}
```

构造函数、原型对象和对象实例之间的关系最有趣的一方面也许是：
对象实例和构造函数直接没有直接联系。（对象实例只有 `[[Prototype]]` 属性指向其相应的原型对象，而原型对象的 `constructor` 属性指向构造函数，而构造函数的 `prototype` 指向原型对象）
![obj_constructor_prototype][5]

#### 4.2.3 改变原型对象

因为每个对象的 `[[Prototype]]` 只是一个指向原型对象的指针，所以原型对象的改动会立刻反映到所有引用它的对象。
当对一个对象使用封印 `Object.seal()` 或冻结 `Object.freeze()` 时，完全是在操作对象的自有属性，但任然可以通过在原型对象上添加属性来扩展这些对象实例。

#### 4.2.4 内建对象（如Array、String）的原型对象

```js
String.prototype.capitalize = function(){
  return this.charAt(0).toUpperCase() + this.substring(1);
}
```

### 总结

 - 构造函数就是用 `new` 操作符调用的普通函数。可用过 `instanceof` 操作符或直接访问 `constructor`(实际上是原型对象的属性) 来鉴别对象是被哪个构造函数所创建的。
 - 每个函数都有一个 `prototype` 对象，它定义了该构造函数创建的所有对象共享的属性。而 `constructor` 属性实际上是定义在原型对象里，供所有对象实例共享。
 - 每个对象实例都有 `[[Prototype]]` 属性，它是指向原型对象的指针。当访问对象的某个属性时，先从对象自身查找，找不到的话就到原型对象上找。
 - 内建对象的原型对象也可被修改


## 5. 继承

### 5.1 原型对象链和 Object.prototype

JavaScript 内建的继承方法被称为 原型对象链（又叫原型对象继承）。
原型对象的属性可经由对象实例访问，这就是继承的一种形式。对象实例继承了原型对象的属性，而原型对象也是一个对象，它也有自己的原型对象并继承其属性，以此类推。这就是原型对象链。

所有对象（包括自义定的）都自动继承自 `Object`，除非你另有指定。更确切地说，所有对象都继承自 `Object.prototype`。任何以对象字面量形式定义的对象，其 `[[Prototype]]` 的值都被设为 `Object.prototype`，这意味着它继承 `Object.prototype` 的属性。

#### 5.1.1 继承自 Object.prototype 的方法

Object.prototype 一般有以下几个方法

 - hasOwnProperty()             检测是否存在一个给定名字的自有属性
 - propertyIsemumerable()       检查一个自有属性是否可枚举
 - isPrototypeOf                检查一个对象是否是另一个对象的原型对象
 - valueOf()                    返回一个对象的值表达
 - toString()                   返回一个对象的字符串表达

这 5 种方法经由继承出现在所有对象中。
因为所有对象都默认继承自 `Object.prototype`，所以改变它就会影响所有的对象。所以不建议。

### 5.2 继承

对象继承是最简单的继承类型。你唯需要做的是指定哪个对象是新对象的 `[[Prototype]]`。对象字面量形式会隐式指定 `Object.prototype` 为其 `[[Protoype]]`。当然我们可以用 ES5 的 `Object.create()` 方法显式指定。该方法接受两个参数，第一个是新对象的的 `[[Prototype]]` 所指向的对象。第二个参数是可选的一个属性描述对象，其格式与 `Object.definePrototies()`一样。

```js
var obj = {
  name: "Ljc"
};

// 等同于
var obj = Object.create(Object.prototype, {
  name: {
    value: "Ljc",
    configurable: true,
    enumberable: true,
    writable: true
  }
});
```

下面是继承其它对象：

```js    
var person = {
  name: "Jack",
  sayName: function(){
    console.log(this.name);
  }
}

var student = Object.create(person, {
  name:{
    value: "Ljc"
  },
  grade: {
    value: "fourth year of university",
    enumerable: true,
    configurable: true,
    writable: true
  }
});

person.sayName(); // "Jack"
student.sayName(); // "Ljc"

console.log(person.hasOwnProperty("sayName")); // true
console.log(person.isPrototypeOf(student)); // true
console.log(student.hasOwnProperty("sayName")); // false
console.log("sayName" in student); // true
```

![对象继承][6]

当访问一个对象属性时，JavaScript 引擎会执行一个搜索过程。如果在对象实例存在该自有属性，则返回，否则，根据其私有属性 `[[Protoype]]` 所指向的原型对象进行搜索，找到返回，否则继承上述操作，知道继承链末端。末端通常是 `Object.prototype`，其 `[[Prototype]]` 是 `null`。

当然，也可以用 `Object.create()` 常见一个 `[[Prototype]]` 为 `null` 的对象。

```js    
var obj = Object.create(null);

console.log("toString" in obj); // false
```

该对象是一个没有原型对象链的对象，即是一个没有预定义属性的白板。

### 5.3 构造函数继承

JavaScript 中的对象继承也是构造函数继承的基础。
第四章提到，几乎所有函数都有 `prototype` 属性，它可被修改或替换。该 `prototype` 属性被自动设置为一个新的继承自 `Object.prototype` 的泛用对象，该对象(原型对象)有一个自有属性 `constructor`。实际上，JavaScript 引擎为你做了下面的事情。

```js
// 你写成这样
function YourConstructor(){
  // initialization
}

// JavaScript引擎在背后为你做了这些处理
YourConstructor.prototype = Object.create(Object.prototype, {
  constructor: {
    configurable: true,
    enumerable: true,
    value: YourConstructor,
    writable: true
  }
})
```

你不需要做额外的工作，这段代码帮你把构造函数的 `prototype` 属性设置为一个继承自 `Object.prototype` 的对象。这意味着 `YourConstructor` 创建出来的任何对象都继承自 `Object.prototype`。

由于 prototype 可写，你可以通过改变它来改变原型对象链。

> MDN：instanceof 运算符可以用来判断某个构造函数的 prototype 属性是否存在另外一个要检测对象的原型链上。

```js
function Rectangle(length, width){
  this.length = length;
  this.width = width
}

Rectangle.prototype.getArea = function(){
  return this.length * this.width
}

Rectangle.prototype.toString = function(){
  return "[Rectangle " + this.length + "x" + this.width + "]";
}

// inherits from Rectangle
function Square(size){
  this.length = size;
  this.width = size;
}

Square.prototype = new Rectangle(); // 尽管是 Square.prototype 是指向了 Rectangle 的对象实例，即Square的实例对象也能访问该实例的属性（如果你提前声明了该对象，且给该对象新增属性）。
// Square.prototype = Rectangle.prototype; // 这种实现没有上面这种好，因为Square.prototype 指向了 Rectangle.prototype，导致修改Square.prototype时，实际就是修改Rectangle.prototype。
console.log(Square.prototype.constructor); // 输出 Rectangle 构造函数

Square.prototype.constructor = Square; // 重置回 Square 构造函数
console.log(Square.prototype.constructor); // 输出 Square 构造函数

Square.prototype.toString = function(){
  return "[Square " + this.length + "x" + this.width + "]";
}

var rect = new Rectangle(5, 10);
var square = new Square(6);

console.log(rect.getArea()); // 50
console.log(square.getArea()); // 36

console.log(rect.toString()); // "[Rectangle 5 * 10]", 但如果是Square.prototype = Rectangle.prototype，则这里会"[Square 5 * 10]"
console.log(square.toString()); // "[Square 6 * 6]"

console.log(square instanceof Square); // true
console.log(square instanceof Rectangle); // true
console.log(square instanceof Object); // true
```

![构造函数继承][7]  
    
`Square.prototype` 并不真的需要被改成为一个 `Rectangle` 对象。事实上，是 `Square.prototype` 需要指向 `Rectangle.prototype` 使得继承得以实现。这意味着可以用 `Object.create()` 简化例子。

```js
// inherits from Rectangle
function Square(size){
  this.length = size;
  this.width = size;
}

Square.prototype= Object.create(Rectangle.prototype, {
  constructor: {
    configurable: true,
    enumerable: true,
    value: Square,
    writable: true
  }
})
```

> 在对原型对象添加属性前要确保你已经改成了原型对象，否则在改写时会丢失之前添加的方法（因为继承是将被继承对象赋值给需要继承的原型对象，相当于重写了需要继承的原型对象）。

### 5.4 构造函数窃取

由于 JavaScript 中的继承是通过原型对象链来实现的，因此不需要调用对象的父类的构造函数。如果确实需要在子类构造函数中调用父类构造函数，那就可以在子类的构造函数中利用 `call`、`apply` 方法调用父类的构造函数。

```js
// 在上面的代码基础上作出修改
// inherits from Rectangle
function Square(size){
  Rectangle.call(this, size, size);

  // optional: add new properties or override existing ones here
}
```

一般来说，需要修改 `prototype` 来继承方法并用构造函数窃取来设置属性，由于这种做法模仿了那些基于类的语言的类继承，所以这通常被称为伪类继承。

### 5.5 访问父类方法

其实也是通过指定 `call` 或 `apply` 的子对象调用父类方法。

## 6 对象模式

### 6.1 私有成员和特权成员

JavaScript 对象的所有属性都是公有的，没有显式的方法指定某个属性不能被外界访问。

#### 6.1.1 模块模式

模块模式是一种用于创建**拥有私有数据的单件对象**的模式。
基本做法是使用立即调用函数表达式（IIFE）来返回一个对象。原理是利用闭包。

```js    
var yourObj = (function(){
  // private data variables

  return {
    // public methods and properties
  }
}());
```

模块模式还有一个变种叫暴露模块模式，它将所有的变量和方法都放在 `IIFE` 的头部，然后将它们设置到需要被返回的对象上。

```js
//  一般写法
var yourObj = (function(){
  var age = 25;

  return {
    name: "Ljc",

    getAge: function(){
      return age;
    }
  }
}());


// 暴露模块模式
var yourObj = (function(){
  var age = 25;
  function getAge(){
    return age;
  };
  return {
    name: "Ljc",
    getAge: getAge
  }
}());
```

#### 6.1.2 构造函数的私有成员（不能通过对象直接访问）

模块模式在定义单个对象的私有属性十分有效，但对于那些同样需要私有属性的自定义类型呢？你可以在构造函数中使用类似的模式来创建每个实例的私有数据。

```js
function Person(name){
  // define a variable only accessible inside of the Person constructor
  var age = 22;

  this.name = name;
  this.getAge = function(){
    return age;
  };
  this.growOlder = function(){
    age++;
  }
}

var person = new Person("Ljc");

console.log(person.age); // undefined
person.age = 100;
console.log(person.getAge()); // 22

person.growOlder();
console.log(person.getAge()); // 23
```

这里有个问题：如果你需要**对象实例**拥有私有数据，就不能将相应方法放在 `prototype` 上。

如果你需要所有实例共享私有数据。则可结合模块模式和构造函数，如下：

```js    
var Person = (function(){
  var age = 22;

  function InnerPerson(name){
    this.name = name;
  }

  InnerPerson.prototype.getAge = function(){
    return age;
  }
  InnerPerson.prototype.growOlder = function(){
    age++;
  };

  return InnerPerson;
}());

var person1 = new Person("Nicholash");
var person2 = new Person("Greg");

console.log(person1.name); // "Nicholash"
console.log(person1.getAge()); // 22

console.log(person2.name); // "Greg"
console.log(person2.getAge()); // 22

person1.growOlder();
console.log(person1.getAge()); // 23
console.log(person2.getAge()); // 23
```

### 6.2 混入

这是一种伪继承。一个对象在不改变原型对象链的情况下得到了另外一个对象的属性被称为“混入”。因此，和继承不同，混入让你在创建对象后无法检查属性来源。
纯函数实现：

```js
function mixin(receiver, supplier){
  for(var property in supplier){
    if(supplier.hasOwnProperty(property)){
      receiver[property] = supplier[property];
    }
  }
}
```

这是浅拷贝，如果属性的值是一个引用，那么两者将指向同一个对象。

### 6.3 作用域安全的构造函数

构造函数也是函数，所以不用 new 也能调用它们来改变 `this` 的值。在非严格模式下， `this` 被强制指向全局对象。而在严格模式下，构造函数会抛出一个错误（因为严格模式下没有为全局对象设置 `this`，`this` 保持为 `undefined`）。
而很多内建构造函数，例如 `Array`、`RegExp` 不需要 `new` 也能正常工作，这是因为它们被设计为作用域安全的构造函数。
当用 `new` 调用一个函数时，`this` 指向的新创建的对象是属于该构造函数所代表的自定义类型。因此，可在函数内用 `instanceof` 检查自己是否被 `new` 调用。

```js
function Person(name){
  if(this instanceof Person){
    // called with "new"
  }else{
    // called without "new"
  }
}
```

具体案例：

```js    
function Person(name){
  if(this instanceof Person){
    this.name = name;
  }else{
    return new Person(name);
  }
}
```

## 总结

看了两天的书，做了两天的笔记。当然这只是 ES5 的。过几天 ES6 新书又来了。最后感谢 [异步社区](http://www.epubit.com.cn/) 送我这本好书 [《JavaScript面向对象精要》](http://www.epubit.com.cn/book/details/1798)，让我的前端根基更加稳固，希望自己的前端之路越走越顺。

  [1]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/Object_hash.jpg
  [2]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/copy_obj.jpg
  [3]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/prototype.jpg
  [4]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/%E6%97%A0%E6%A0%87%E9%A2%98.jpg
  [5]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/obj_constructor_prototype.jpg
  [6]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/%E5%AF%B9%E8%B1%A1%E7%BB%A7%E6%89%BF.jpg
  [7]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0%E7%BB%A7%E6%89%BF.jpg
