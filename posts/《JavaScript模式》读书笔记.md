# 《JavaScript 模式》读书笔记

标签： 读书笔记

---

## 简介
在软件开发过程中，模式是指一个通用问题的解决方案。一个模式不仅仅是一个可以用来复制粘贴的代码解决方案，更多地是提供了一个更好的实践经验、有用的抽象化表示和解决一类问题的模板。

对象有两大类：

 - 本地对象（Native）：由ECMAScript标准定义的对象
 - 宿主对象（Host）：由宿主环境创建的对象（比如浏览器环境）

本地对象也可以被归类为内置对象（比如Array、Date）或自定义对象（var o = {}）。  
宿主对象包含window和所有DOM对象。如果你想知道你是否在使用宿主对象，将你的代码迁移到一个非浏览器环境中运行一下，如果正常工作，那么你的代码就只用到了本地对象。  

“GoF”的书中提到一条通用规则，“组合优于继承”。

[console 对象](https://developer.mozilla.org/en-US/docs/Web/API/Console)

## 基本技巧

易维护的代码具有如下特性：

 - 可读的
 - 风格一致
 - 可预测的
 - 看起来像一个人写的
 - 有文档

### 尽量少用全局变量
#### 全局变量的问题
隐式创建全部变量有两种情况：

 - 未经声明的变量就为全局对象所有。

        function sum(x, y){
            result = x + y; // result 是全局变量;
        }

 - 带有 `var` 声明的链式赋值
    
        function foo(){
            var a = b = 0; // b 是全局变量
        }
    由于 `=` 的运算顺序是从右到左。即 `var a = b = 0;` 等价于 `var a = (b = 0)`。  
    因此，对于链式赋值建议做法如下：
        
        function foo(){
            var a, b;
            a = b = 0;
        }
 
隐式创建的全局变量与明确定义的全局变量的不同之处在于：是否能被 `delete` 操作符删除。

 - 使用 var 创建的全局变量（在函数外创建）不能删除
 - 不使用 var 创建的隐式全局变量（就算在函数内创建）可以删除
```
var a = 1;
b = 2;
(function(){
    c = 3;
})();

delete a; // false;
delete b; // true;
delete c; // true

typeof a; // number
typeof b; // undefined
typeof c; // undefined
```
在ES5 strict模式下，为未声明的变量赋值会抛出错误。

### 单一var模式
在函数顶部对所有变量通过一个 var 进行声明。好处如下：

 - 可以在同一个位置找到函数所需的所有变量
 - 避免在变量声明之前使用这个变量时产生的逻辑错误
 - 提醒你不要忘记声明变量，顺便减少潜在的全局变量
 - 代码量更少
 
例子：

    function func(){
        var a = 1,
            b = 2,
            sum = a + b,
            myobject = {},
            i,
            j;
        console.log(sum)
        // 函数体
    }
    func()
    

> 使用逗号操作符可以在一条语句中执行多个操作。多用于声明多个变量，但还可以用于赋值，总会返回表达式的最后一项。
(1) 逗号表达式的运算过程为：从左往右逐个计算表达式。  
(2) 逗号表达式作为一个整体，它的值为最后一个表达式的值。`var num = (5,4,1,0); // num 为 0`。 
(3) 逗号运算符的优先级别在所有运算符中最低。  

### 避免使用隐式类型转换
使用 `===` 或 `!==` 进行比较。增强代码可阅读性，避免猜测。
另外，`switch` 语句的 `case` 进行比较时，使用的是 `===`。

### 避免使用 eval()
new Functin() 与 eval()的不同：

第一点：
new Function()中的代码将在局部函数空间运行，因此代码中任何采用var定义的变量不会自动成为全局变量（即在函数内）。  
eval()则会自动成为全局变量，但可通过立即调用函数对其进行封装。
    
    console.log(typeof un);// "undefined"
    console.log(typeof deux); // "undefined"
    console.log(typeof trois); // "undefined"
    
    var jsstring = "var un = 1; console.log(un);";
    eval(jsstring); // 打印出 "1"
    
    jsstring = "var deux = 2; console.log(deux);";
    new Function(jsstring)(); // 打印出 "2"
    
    jsstring = "var trois = 3; console.log(trois);";
    (function () {
        eval(jsstring);
    }()); // 打印出 "3"
    
    console.log(typeof un); // "number"
    console.log(typeof deux); // "undefined"
    console.log(typeof trois); // "undefined"
第二点：
eval()会影响到作用域链，而Function则像一个沙盒，无论在哪里执行Function，它都仅能看到全局作用域链。因此对局部变量的影响比较小。

    (function () {
        var local = 1;
        eval("local = 3; console.log(local)"); // 打印出 3
        console.log(local); // 打印出 3
    }());
    
    (function () {
        var local = 1;
        Function("console.log(typeof local);")(); // 打印出 undefined
    }());

### 使用parseInt()进行数字转换
ECMAScript3中以0为前缀的字符串会被当作八进制数处理，这一点在ES5中已经有了改变。为了避免转换类型不一致而导致的意外结果，应当总是指定第二个参数：
    
    var month = "06",
    year = "09";
    month = parseInt(month, 10);
    year = parseInt(year, 10);
    
字符串转换为数字还有两种方法：
    
    +"08" // 结果为8，隐式调用Number()
    Number("08") // 结果为8
    
这两种方法要比parseInt()更快一些，因为顾名思义parseInt()是一种“解析”而不是简单的“转换”。但当你期望将“08 hello”这类字符串转换为数字，则必须使用parseInt()，其他方法都会返回NaN。

### 命名约定

 - 构造函数首字母大写
 - 函数用小驼峰式（getFirstName），变量用“所有单词小写，并用下划线分隔各个单词”（first_name）。这样就能区分函数和变量了。
 - 常量和全局变量的所有字符大写
 - 私有成员函数用下划线（_）前缀命名
 
当然，还要正确编写注释和更新注释。最好能编写 API 文档。

## 字面量与构造函数
    
    // 一种方法，使用字面量
    var car = {goes: "far"};
    
    // 另一种方法，使用内置构造函数
    // 注意：这是一种反模式
    var car = new Object();
    car.goes = "far";
字面量写法的一个明显优势是，它的代码更少。“创建对象的最佳模式是使用字面量”还有一个原因，它可以强调对象就是一个简单的可变的散列表，而不必一定派生自某个类。  
另外一个使用字面量而不是Object()构造函数创建实例对象的原因是，对象字面量不需要“作用域解析”（scope resolution）。因为可能存在一个同名的构造函数Object()，当你调用Object()的时候，解析器需要顺着作用域链从当前作用域开始查找，直到找到全局Object()构造函数为止。

Object()构造函数仅接受一个参数，且依赖于传递的值，该Object()根据值委派另一个内置构造函数来创建对象，并返回另外一个对象实例，而这往往不是你想要的。

    // 空对象
    var o = new Object();
    console.log(o.constructor === Object); // true
    
    // 数值对象
    var o = new Object(1);
    console.log(o.constructor === Number); // true
    console.log(o.toFixed(2)); // "1.00"
    
    // 字符串对象
    var o = new Object("I am a string");
    console.log(o.constructor === String); // true
    // 普通对象没有substring()方法，但字符串对象有
    console.log(typeof o.substring); // "function"
    
    // 布尔值对象
    var o = new Object(true);
    console.log(o.constructor === Boolean); // true
    
### 强制使用new的模式
对于构造函数，若忘记使用 `new` 操作符，会导致构造函数中的this指向全局对象（严格模式下，指向undeinfed）。

为了防止忘记 `new`，我们使用下面的方法：在构造函数中首先检查this是否是构造函数的实例，如果不是，则**通过new再次调用自己**：

    function Waffle() {
    
    // Waffle 可换成 arguments.callee（指向当前执行的函数）
    if (!(this instanceof Waffle)) { 
        return new Waffle();
    }
    this.tastes = "yummy";
    
    }
    Waffle.prototype.wantAnother = true;
    
    // 测试
    var first = new Waffle(),
        second = Waffle();
    
    console.log(first.tastes); // "yummy"
    console.log(second.tastes); // "yummy"
    
    console.log(first.wantAnother); // true
    console.log(second.wantAnother); // true
    
    
## 函数
### 重定义函数
函数可以被动态定义，也可以被赋值给变量。如果将你定义的函数赋值给已经存在的函数变量的话，则新函数会覆盖旧函数。这样做的结果是，旧函数的引用被丢弃掉，变量中所存储的引用值替换成了新的函数。这样看起来这个变量指代的函数逻辑就发生了变化，或者说函数进行了“重新定义”或“重写”。

    var scareMe = function () {
        alert("Boo!");
        scareMe = function () {
            alert("Double boo!");
        };
    };
    // 使用重定义函数
    scareMe(); // Boo!
    scareMe(); // Double boo!

当函数中包含一些初始化操作，并希望这些初始化操作只执行一次，那么这种模式是非常合适的，因为我们要避免重复执行不需要的代码。在这个场景中，函数执行一次后就被重写为另外一个函数了。

使用这种模式可以帮助提高应用的执行效率，因为重新定义的函数执行的代码量更少。

> 这种模式的另外一个名字是“函数的懒惰定义”，因为直到函数执行一次后才重新定义，可以说它是“某个时间点之后才存在”，简称“懒惰定义”。

这种模式有一个明显的缺陷，就是之前给原函数添加的功能在重定义之后都丢失了。同时，如果这个函数被重定义为不同的名字，被赋值给不同的变量，或者是作为对象的方法使用，那么重定义的部分并不会生效，原来的函数依然会被执行。

### 条件初始化
条件初始化（也叫条件加载）是一种优化模式。当你知道某种条件在整个程序生命周期中都不会变化的时候，那么对这个条件的探测只做一次就很有意义。浏览器探测（或者特征检测）是一个典型的例子。
    
    // 接口
    var utils = {
        addListener: null,
        removeListener: null
    };
    
    // 实现
    if (typeof window.addEventListener === 'function') {
        utils.addListener = function (el, type, fn) {
            el.addEventListener(type, fn, false);
        };
        utils.removeListener = function (el, type, fn) {
            el.removeEventListener(type, fn, false);
        };
    } else if (typeof document.attachEvent === 'function') { // IE
        utils.addListener = function (el, type, fn) {
            el.attachEvent('on' + type, fn);
        };
        utils.removeListener = function (el, type, fn) {
            el.detachEvent('on' + type, fn);
        };
    } else { // older browsers
        utils.addListener = function (el, type, fn) {
            el['on' + type] = fn;
        };
        utils.removeListener = function (el, type, fn) {
            el['on' + type] = null;
        };
    }

当然，重定义函数也能实现这种效果。

### 函数属性——记忆模式（Memoization）
任何时候都可以给函数添加自定义属性。添加自定义属性的一个有用场景是缓存函数的执行结果（返回值），这样下次同样的函数被调用的时候就不需要再做一次那些可能很复杂的计算。缓存一个函数的运行结果也就是为大家所熟知的记忆模式。

    var myFunc = function (param) {
        if (!myFunc.cache[param]) {
            var result = {};
            // ……复杂的计算……
            myFunc.cache[param] = result;
        }
        return myFunc.cache[param];
    };
    
    // 缓存
    myFunc.cache = {};
    
上面的代码假设函数只接受一个参数param，并且这个参数是原始类型（比如字符串）。如果你有更多更复杂的参数，则通常需要对它们进行序列化。比如，你需要将arguments对象序列化为JSON字符串，然后使用JSON字符串作为cache对象的key：

    var myFunc = function () {
    
        var cachekey = JSON.stringify(Array.prototype.slice.call(arguments)),
            result;
    
        if (!myFunc.cache[cachekey]) {
            result = {};
            // ……复杂的计算……
            myFunc.cache[cachekey] = result;
        }
        return myFunc.cache[cachekey];
    };
    
    // 缓存
    myFunc.cache = {};
    
前面代码中的函数名还可以使用arguments.callee来替代，这样就不用将函数名硬编码。不过尽管现阶段这个办法可行，但是仍然需要注意，arguments.callee在ECMAScript5的严格模式中是不被允许的。

### 柯里化
是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术。  
下面是通用的柯里化函数：

    function cury(fn){
        var slice = Array.prototype.slice;
        var stored_args = slice.call(arguments, 1);

        return function(){
            var new_args = slice.call(arguments),
                args = stored_args.concat(new_args);
            return fn.apply(null, args);
        }
    }
    
    // 测试
    function sum(){
        var result = 0;
        for(var i = 0, len = arguments.length; i < len; i++){
            result += arguments[i];
        }
        return result;
    }

    var newSum = cury(sum, 1,2,3,4,5,6);
    console.log(newSum(2,3,5,4)); // 35
    
#### 什么时候使用柯里化
当你发现自己在调用同样的函数并且传入的参数大部分都相同的时候，就是考虑柯里化的理想场景了。你可以通过传入一部分的参数动态地创建一个新的函数。这个新函数会存储那些重复的参数（所以你不需要再每次都传入），然后再在调用原始函数的时候将整个参数列表补全。

> apply()接受两个参数：第一个是在函数内部绑定到this上的对象，第二个是一个参数数组，参数数组会在函数内部变成一个类似数组的arguments对象。如果第一个参数为 `null`，那么this将指向全局对象，这正是当你调用一个函数（且这个函数不是某个对象的方法）时发生的事情。

### 小结
在介绍完背景和函数的语法后，介绍了一些有用的模式，按分类列出：

1. API模式，它们帮助我们为函数给出更干净的接口，包括：
	- 回调模式

		  传入一个函数作为参数
	- 配置对象

		   帮助保持函数的参数数量可控
	- 返回函数

		   函数的返回值是另一个函数
	- 柯里化

		   新函数在已有函数的基础上再加上一部分参数构成
2. 初始化模式，这些模式帮助我们用一种干净的、结构化的方法来做一些初始化工作（在web页面和应用中非常常见），通过一些临时变量来保证不污染全局命名空间。这些模式包括：
	- 即时函数

		   当它们被定义后立即执行
	- 对象即时初始化

		   初始化工作被放入一个匿名对象，这个对象提供一个可以立即被执行的方法
	- 条件初始化

		   使分支代码只在初始化的时候执行一次，而不是在整个程序生命周期中反复执行
3. 性能模式，这些模式帮助提高代码的执行速度，包括：
	- 记忆模式

		   利用函数的属性，使已经计算过的值不用再次计算
	- 自定义函数

		   重写自身的函数体，使第二次及后续的调用做更少的工作
		   


## 对象创建模式
JavaScript语言本身很简单、直观，也没有其他语言的一些语言特性：命名空间、模块、包、私有属性以及静态成员。本章将介绍一些常用的模式，以此实现这些语言特性。

我们将对命名空间、依赖声明、模块模式以及沙箱模式进行初探——它们可以帮助我们更好地组织应用程序的代码，有效地减少全局污染的问题。除此之外，还会讨论私有和特权成员、静态和私有静态成员、对象常量、链式调用以及一种像类式语言一样定义构造函数的方法等话题。

### 命名空间模式
使用命名空间可以减少全局变量的数量，与此同时，还能有效地避免命名冲突和前缀的滥用。

本章后续要介绍的沙箱模式则可以避免这些缺点。
```
// 重构前：5个全局变量
// 注意：反模式
// 构造函数
function Parent() {} 
function Child() {}
// 一个变量
var some_var = 1;

// 一些对象
var module1 = {}; 
module1.data = {a: 1, b: 2}; 
var module2 = {};
```
可以通过创建一个全局对象（通常代表应用名）比如MYAPP来重构上述这类代码，然后将上述例子中的函数和变量都变为该全局对象的属性：
```
// 重构后：一个全局变量
// 全局对象
var MYAPP = {};

// 构造函数
MYAPP.Parent = function () {}; 
MYAPP.Child = function () {};

// 一个变量
MYAPP.some_var = 1;

// 一个对象容器
MYAPP.modules = {};

// 嵌套的对象
MYAPP.modules.module1 = {}; 
MYAPP.modules.module1.data = {a: 1, b: 2}; 
MYAPP.modules.module2 = {};
```
这种模式在大多数情况下非常适用，但也有它的缺点：

 - 代码量稍有增加；在每个函数和变量前加上这个命名空间对象的前缀，会增加代码量，增大文件大小
 - 该全局实例可以被随时修改
 - 命名的深度嵌套会减慢属性值的查询
 
本章后续要介绍的沙箱模式则可以避免这些缺点。

#### 通用命名空间函数
随着程序复杂度的提高，代码会被分拆在不同的文件中以按照页面需要来加载，这样一来，就不能保证你的代码一定是第一个定义命名空间或者某个属性的，甚至会发生属性覆盖的问题。所以，在创建命名空间或者添加属性的时候，最好先检查下是否存在，如下所示：
```
// 不安全的做法
var MYAPP = {};
// 更好的做法
if (typeof MYAPP === "undefined") {
    var MYAPP = {}; 
}
// 简写
var MYAPP = MYAPP || {};
```
如上所示，如果每次做类似操作都要这样检查一下就会有很多重复的代码。例如，要声明MYAPP.modules.module2，就要重复三次这样的检查。所以，我们需要一个可复用的namespace()函数来专门处理这些检查工作，然后用它来创建命名空间，如下所示：
```
// 使用命名空间函数
MYAPP.namespace('MYAPP.modules.module2');

// 等价于：
// var MYAPP = {
//  modules: {
//      module2: {}
//  }
// };
```
下面是上述namespace函数的实现示例。这种实现是非破坏性的，意味着如果要创建的命名空间已经存在，则不会再重复创建：
```
var MYAPP = MYAPP || {};

MYAPP.namespace = function(ns_string){
    var parts = ns_string.split("."),
        parent = MYAPP;

    if(parts[0] === "MYAPP"){
        parts.shift();
    }

    for(var i = 0, len = parts.length; i < len; i++){
        if(parent[parts[i]] === undefined){
            parent[parts[i]] = {}
        }
        parent = parent[parts[i]]
    }
    return parent;
}
var module2 = MYAPP.namespace("MYAPP.modules.module2");
console.log(module2 === MYAPP.modules.module2); // true

var modules = MYAPP.namespace("modules");
console.log(modules === MYAPP.modules); // true
```

### 依赖声明
JavaScript库往往是模块化而且有用到命名空间的，这使得你可以只使用你需要的模块。比如在YUI2中，全局变量YAHOO就是一个命名空间，各个模块都是全局变量的属性，比如YAHOO.util.Dom（DOM模块）、YAHOO.util.Event（事件模块）。

将你的代码依赖在函数或者模块的顶部进行声明是一个好主意。声明就是创建一个本地变量，指向你需要用到的模块：
```
var myFunction = function () {
    // 依赖
    var event = YAHOO.util.Event,
        dom = YAHOO.util.Dom;

    // 在函数后面的代码中使用event和dom……
};
```
**这是一个相当简单的模式，但是有很多的好处：**

 - 明确的依赖声明是告知使用你代码的开发者，需要保证指定的脚本文件被包含在页面中。
 - 将声明放在函数顶部使得依赖很容易被查找和解析。
 - 本地变量（如dom）永远会比全局变量（如YAHOO）要快，甚至比全局变量的属性（如YAHOO.util.Dom）还要快，这样会有更好的性能。使用了依赖声明模式之后，全局变量的解析在函数中只会进行一次，在此之后将会使用更快的本地变量（备注：本地变量直接指向最后一级对象，event）。
 - 一些高级的代码压缩工具比如YUI Compressor和Google Closure compiler会重命名本地变量（比如event可能会被压缩成一个字母，如A），这会使代码更精简，但这个操作不会对全局变量进行，因为这样做不安全。
 
### 私有属性和方法
#### 私有成员
通过闭包实现：
```
function Gadget() {
    // 私有成员
    var name = 'iPod';
    // 公有函数
    this.getName = function () {
        return name;
    };
}
var toy = new Gadget();

// name是是私有的
console.log(toy.name); // undefined
// 公有方法可以访问到name
console.log(toy.getName()); // "iPod"
```

#### 特权方法
特权方法的概念不涉及到任何语法，它只是一个给可以访问到私有成员的公有方法的名字（就好像它们有更多权限一样）。   
在前面的例子中，getName()就是一个特权方法，因为它有访问name属性的特殊权限。

#### 私有成员失效
当你直接通过特权方法返回一个私有变量，而这个私有变量恰好是一个对象或者数组时，外部的代码可以修改这个私有变量，因为它是按引用传递的。
```
function Gadget() {
    // 私有成员
    var specs = {
        screen_width: 320,
        screen_height: 480,
        color: "white"
    };

    // 公有函数
    this.getSpecs = function () {
        return specs; // 直接返回对象（数组也是对象），会导致私有对象能在外面被修改
    };
}
```
**解决方法：**返回精简后新对象（返回需要用到的部分属性），或对私有对象进行复制（返回副本）。

> 众所周知的“最低授权原则”（Principle of Least Authority，简称POLA），指永远不要给出比真实需要更多的东西。

#### 原型和私有成员
使用构造函数创建私有成员的一个弊端是，每一次调用构造函数创建对象时这些私有成员都会被创建一次。
```
function Gadget() {
    // 私有成员
    var name = 'iPod';
    // 公有函数
    this.getName = function () {
        return name;
    };
}

Gadget.prototype = (function () {
    // 私有成员
    var browser = "Mobile Webkit";
    // 公有函数
    return {
        getBrowser: function () {
            return browser;
        }
    };
}());

var toy = new Gadget();
console.log(toy.getName()); // 自有的特权方法 
console.log(toy.getBrowser()); // 来自原型的特权方法
```

#### 将私有函数暴露为公有方法
“暴露模式”是指将已经有的私有函数暴露为公有方法。

我们来看一个例子，它建立在对象字面量的私有成员模式之上：
```
var myarray;

(function () {

    var astr = "[object Array]",
        toString = Object.prototype.toString;

    function isArray(a) {
        return toString.call(a) === astr;
    }

    function indexOf(haystack, needle) {
        var i = 0,
            max = haystack.length;
        for (; i < max; i += 1) {
            if (haystack[i] === needle) {
                return i;
            }
        }
        return −1;
    }

    myarray = {
        isArray: isArray,
        indexOf: indexOf,
        inArray: indexOf
    };

}());
```

### 模块模式
模块模式使用得很广泛，因为它可以为代码提供特定的结构，帮助组织日益增长的代码。不像其它语言，JavaScript没有专门的“包”（package）的语法，但模块模式提供了用于创建独立解耦的代码片段的工具，这些代码可以被当成黑盒，当你正在写的软件需求发生变化时，这些代码可以被添加、替换、移除。

模块模式是我们目前讨论过的好几种模式的组合，即：

 - 命名空间模式
 - 即时函数模式
 - 私有和特权成员模式
 - 依赖声明模式

第一步是初始化一个命名空间。我们使用本章前面部分的namespace()函数，创建一个提供数组相关方法的套件模块：
```
MYAPP.namespace('MYAPP.utilities.array');
```

下一步是定义模块。使用一个即时函数来提供私有作用域供私有成员使用。即时函数返回一个对象，也就是带有公有接口的真正的模块，可以供其它代码使用：
```
MYAPP.utilities.array = (function () {
    return {
        // todo...
    };
}());
```

下一步，给公有接口添加一些方法：
```
MYAPP.utilities.array = (function () {
    return {
        inArray: function (needle, haystack) {
            // ...
        },
        isArray: function (a) {
            // ...
        }
    };
}());
```

如果需要的话，你可以在即时函数提供的闭包中声明私有属性和私有方法。同样，依赖声明放置在函数顶部，在变量声明的下方可以选择性地放置辅助初始化模块的一次性代码。函数最终返回的是一个包含模块公共API的对象：
```
MYAPP.namespace('MYAPP.utilities.array');
MYAPP.utilities.array = (function () {

        // 依赖声明
    var uobj = MYAPP.utilities.object,
        ulang = MYAPP.utilities.lang,

        // 私有属性
        array_string = "[object Array]",
        ops = Object.prototype.toString;

        // 私有方法
        // ……

        // 结束变量声明

    // 选择性放置一次性初始化的代码
    // ……

    // 公有API
    return {

        inArray: function (needle, haystack) {
            for (var i = 0, max = haystack.length; i < max; i += 1) {
                if (haystack[i] === needle) {
                    return true;
                }
            }
        },

        isArray: function (a) {
            return ops.call(a) === array_string;
        }
        // ……更多的方法和属性
    };
}());
```
模块模式被广泛使用，是一种值得强烈推荐的模式，它可以帮助我们组织代码，尤其是代码量在不断增长的时候。
#### 暴露模块模式
我们在本章中讨论私有成员模式时已经讨论过暴露模式。模块模式也可以用类似的方法来组织，将所有的方法保持私有，只在最后暴露需要使用的方法来初始化API。

上面的例子可以变成这样：
```
MYAPP.utilities.array = (function () {

        // 私有属性
    var array_string = "[object Array]",
        ops = Object.prototype.toString,

        // 私有方法
        inArray = function (haystack, needle) {
            for (var i = 0, max = haystack.length; i < max; i += 1) {
                if (haystack[i] === needle) {
                    return i;
                }
            }
            return −1;
        },
        isArray = function (a) {
            return ops.call(a) === array_string;
        };
        // 结束变量定义

    // 暴露公有API
    return {
        isArray: isArray,
        indexOf: inArray
    };
}());

```

#### 在模块中引入全局上下文
作为这种模式的一个常见的变种，你可以给包裹模块的即时函数传递参数。你可以传递任何值，但通常情况下会传递全局变量甚至是全局对象本身。引入全局上下文可以加快函数内部的全局变量的解析，因为引入之后会作为函数的本地变量：
```
MYAPP.utilities.module = (function (app, global) {

    // 全局对象和全局命名空间都作为本地变量存在

}(MYAPP, this));
```

## 代码复用模式
在做代码复用的工作的时候，谨记Gang of Four在书中给出的关于对象创建的建议：“优先使用对象创建而不是类继承”。

类式（传统）继承（classical inheritance） vs 现代继承模式
类式继承：按照类的方式考虑JavaScript，并产生了一些假定在类的基础上的开发思路和继承模式。
现代继承模式：其他任何不需要以类的方式考虑的模式。

当需要给项目选择一个继承模式时，有不少的备选方案。你应该尽量选择那些现代继承模式，除非团队已经觉得“无类不欢”。

跳过类继承..

### 通过复制属性实现继承
在这种模式中，一个对象通过简单地复制另一个对象来获得功能。下面是一个简单的实现这种功能的extend()函数：
```
function extend(parent, child) {
    var i;
    child = child || {};
    for (i in parent) {
        if (parent.hasOwnProperty(i)) {
            child[i] = parent[i];
        }
    }
    return child;
}
```

上面给出的实现叫作对象的“浅拷贝”（shallow copy），与之相对，“深拷贝”是指检查准备复制的属性本身是否是对象或者数组，如果是，也遍历它们的属性并复制。如果使用浅拷贝的话（因为在JavaScript中对象是按引用传递），如果你改变子对象的一个属性，而这个属性恰好是一个对象，那么你也会改变父对象。实际上这对方法来说可能很好（因为函数也是对象，也是按引用传递），但是当遇到其它的对象和数组的时候可能会有些意外情况。

现在让我们来修改一下extend()函数以便实现深拷贝。你需要做的事情只是检查一个属性的类型是否是对象，如果是，则递归遍历它的属性。另外一个需要做的检查是这个对象是真的对象还是数组，可以使用第三章讨论过的数组检查方式。最终深拷贝版的extend()是这样的：
```
function extendDeep(parent, child) {
    var i,
        toStr = Object.prototype.toString,
        astr = "[object Array]";

    child = child || {};

    for (i in parent) {
        if (parent.hasOwnProperty(i)) {
            if (typeof parent[i] === "object") {
                child[i] = (toStr.call(parent[i]) === astr) ? [] : {};
                extendDeep(parent[i], child[i]);
            } else {
                child[i] = parent[i];
            }
        }
    }
    return child;
}
```

这种模式并不高深，因为根本没有原型牵涉进来，而只跟对象和它们的属性有关。

### 混元（Mix-ins）
“混元”模式，从任意多数量的对象中复制属性，然后将它们混在一起组成一个新对象。
```
function mix() {
    var arg, prop, child = {};
    for (arg = 0; arg < arguments.length; arg += 1) {
        for (prop in arguments[arg]) {
            if (arguments[arg].hasOwnProperty(prop)) {
                child[prop] = arguments[arg][prop];
            }
        }
    }
    return child;
}
```
这里我们只是简单地遍历、复制自有属性，并没有与父对象有任何链接。

### 借用方法
apply、call、bind（ES5）。  
apply是接受数组，而call是接受一个一个的参数。

在低于ES5的环境中时如何实现Function.prototype.bind()：
```
if (typeof Function.prototype.bind === "undefined") {
    Function.prototype.bind = function (thisArg) {
        var fn = this,
        slice = Array.prototype.slice,
        args = slice.call(arguments, 1);

        return function () {
            return fn.apply(thisArg, args.concat(slice.call(arguments)));
        };
    };
}
```

### 小结
在JavaScript中，继承有很多种方案可以选择，在本章中你看到了很多类式继承和现代继承的方案。学习和理解不同的模式是有好处的，因为这可以增强你对这门语言的掌握能力。

但是，也许在开发过程中继承并不是你经常面对的一个问题。一部分是因为这个问题已经被使用某种方式或者某个你使用的类库解决了，另一部分是因为你不需要在JavaScript中建立很长很复杂的继承链。在静态强类型语言中，继承可能是唯一可以复用代码的方法，但在JavaScript中有更多更简单更优化的方法，包括借用方法、绑定、复制属性、混元等。

记住，代码复用才是目标，继承只是达成这个目标的一种手段。

## DOM与浏览器模式
### 延迟加载
所谓的延迟加载是指在页面的load事件之后再加载外部文件。通常，将一个大的合并后的文件分成两部分是有好处的：

 - 一部分是页面初始化和绑定UI元素的事件处理函数必须的
 - 第二部分是只在用户交互或者其它条件下才会用到的

分成两部分的目标就是逐步加载页面，让用户尽快可以进行一些操作。剩余的部分在用户可以看到页面的时候再在后台加载。

加载第二部分JavaScript的方法也是使用动态script元素，将它加在head或者body中：
```
    ……页面主体部分……

    <!-- 第二块结束 -->
    <script src="all_20100426.js"></script>
    <script>
    window.onload = function () {
        var script = document.createElement("script");
        script.src = "all_lazy_20100426.js";
        document.documentElement.firstChild.appendChild(script);
    };
    </script>
</body>
</html>
```
 ### 按需加载
创建一个require()函数或者方法，它接受一个需要被加载的脚本文件的文件名，还有一个在脚本被加载完毕后执行的回调函数。
```
require("extra.js", function () {
    functionDefinedInExtraJS();
});
```
```
function require(file, callback) {

    var script = document.getElementsByTagName('script')[0], newjs = document.createElement('script');

    // IE
    newjs.onreadystatechange = function () {
        if (newjs.readyState === 'loaded' || newjs.readyState === 'complete') {
            newjs.onreadystatechange = null;
            callback();
        }
    };

    // 其它浏览器
    newjs.onload = function () {
        callback();
    };

    newjs.src = file;
    script.parentNode.insertBefore(newjs, script);
}

```
这个实现的几点说明：

 - 在IE中需要监听readystatechange事件，然后判断状态是否为"loaded"或者"complete"。其它的浏览器会忽略这里。
 - 在Firefox，Safari和Opera中，通过onload属性监听load事件。
 - 这个方法在Safari 2中无效。如果必须要处理这个浏览器，需要设一个定时器，周期性地去检查某个指定的变量（在脚本中定义的）是否有定义。当它变成已定义时，就意味着新的脚本已经被加载并执行。
 
### 预加载JavaScript
在延迟加载模式和按需加载模式中，我们加载了当前页面需要用到的脚本，除此之外，我们也可以加载当前页面不需要但可能在接下来的页面中需要的脚本。这样的话，当用户进入第二个页面时，脚本已经被预加载过，整体体验会变得更快。

预加载可以简单地通过动态脚本模式实现，但这也意味着脚本会被解析和执行。解析仅仅会在页面加载时间中增加预加载消耗的时间，但执行却可能导致JavaScript错误，因为预加载的脚本会假设自己运行在第二个页面上，比如找一个特定的DOM节点就可能出错。

仅加载脚本而不解析和执行是可能的，这也同样适用于CSS和图像。

在IE中，你可以使用熟悉的图片信标模式来发起请求：
```
new Image().src = "preloadme.js";
```

在其它的浏览器中，你可以使用<object>替代script元素，然后将它的data属性指向脚本的URL：
```
var obj = document.createElement('object');
obj.data = "preloadme.js";
document.body.appendChild(obj);
```
为了阻止object可见，你应该设置它的width和height属性为0。

你可以创建一个通用的preload()函数或者方法，使用条件初始化模式（第四章）来处理浏览器差异：
```
var preload;
if (/*@cc_on!@*/false) { // IE支持条件注释
    preload = function (file) {
        new Image().src = file;
    };
} else {
    preload = function (file) {
        var obj = document.createElement('object'),
            body = document.body;

        obj.width = 0;
        obj.height = 0;
        obj.data = file;
        body.appendChild(obj);
    };
}
```
使用这个新函数：
```
preload('my_web_worker.js');
```

这种模式的坏处在于存在用户代理（浏览器）嗅探，但这里无法避免，因为特性检测没有办法告知足够的浏览器行为信息。比如在这个模式中，理论上你可以测试typeof Image是否是"function"来代替嗅探，但这种方法其实没有作用，因为所有的浏览器都支持new Image()；只是有一些浏览器会为图片单独做缓存，意味着作为图片缓存下来的组件（文件）在第二个页面中不会被作为脚本取出来，而是会重新下载。

> 浏览器嗅探中使用条件注释很有意思，这明显比在`navigator.userAgent`中找字符串要安全得多，因为用户可以很容易地修改这些字符串。 比如： `var isIE = /*@cc_on!@*/false`; 会在其它的浏览器中将`isIE`设为`false`（因为忽略了注释），但在IE中会是`true`，因为在条件注释中有取反运算符!。在IE中就像是这样： var isIE = !false; // true

预加载模式可以被用于各种组件（文件），而不仅仅是脚本。比如在登录页就很有用，当用户开始输入用户名时，你可以使用打字的时间开始预加载（非敏感的东西），因为用户很可能会到第二个也就是登录后的页面。
