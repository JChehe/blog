# JavaScript 模块【Part 1】：初学者指南

标签： JavaScript translate module

---

本文由 [伯乐在线][1] - [刘健超-J.c][2] 翻译，等待校稿。未经许可，禁止转载！

英文出处：[JavaScript Modules: A Beginner’s Guide][3]。欢迎加入翻译组。

---

本文共有两章节：

 1. [JavaScript 模块：初学者指南][4]
 2. [JavaScript 模块：模块打包][5]

![此处输入图片的描述][6]

如果你刚接触 JavaScript，想必已经被“module bundlers vs. module loaders”、“Webpack vs. Browserify”和“AMD vs. CommonJS” 等诸如此类的行业术语所吓到。

JavaScript 模块系统听起来挺吓人的，但明白它是每个 Web 开发者所必备的要求。

在这篇文章中，我将抛开这些行业术语，用通俗易懂的语言（和一些代码案例）向你解释清楚。希望你能从中收益！

> 注意：为了让文章更易理解，我分为两部分进行讲述：第一部分会深入解释「模块是什么」和「为什么要使用它们」。第二部分（下周发布）讲述「模块打包意味着什么」和「用不同方式实现模块打包」。


## Part 1: 你能再次解释模块是什么吗？
优秀的作者会将他的书分为章和节。同理，优秀的程序员能将他的程序划分为各个模块。

就像书的章节，模块就是词（或代码，视情况而定）的集群。

好的模块拥有以下特点：不同功能是高度独立的，并且它们允许被打乱、移除或在必要时进行补充，而不会扰乱系统作为一个整体。

## 为什么使用模块？
使用模块有诸多好处，如利于建立一个扩展性强的、互相依赖的代码库。而在我看来，其最重要是：

**1）可维护性：** 根据定义，模块是独立的。一个设计良好的模块意在尽可能减少依赖代码库的某部分，因此它才能单独地扩展与完善。更新一个从其它代码段解耦出来的独立模块显然来得更简单。

回到书的案例，如果书的某个章节需要进行小改动，而该改动会牵涉到其它所有章节，这无疑是个梦魇。相反，如果每章节都以某种良好方式进行编写，那么改动某章节，则不会影响其它章节。

**2）命名空间：** 在 JavaScript 中，如果变量声明在顶级函数的作用域外，那么这些变量都是全局的（意味着，任何地方都能读写它）。因此，造成了常见的“命名空间污染”，从而导致完全无关的代码却共享着全局变量。

无关代码间共享着全局变量是一个严重的 [编程禁忌][7]。

我们将在本文后面看到，模块通过为变量创建一个私有空间，从而避免了命名空间的污染。

**3）可重用性：** 坦诚地讲：我们都试过复制旧项目的代码到新项目上。例如，我们复制以前项目的某些功能方法到当前项目中。

该做法看似可行，但如果发现那段代码有更好的实现方式（即需要改动），那么你就不得不去追溯并更新任何你所粘贴到的任何地方。

这无疑会浪费大量的时间。因此可重用的模块显然让你编码轻松。


## 如何整合为模块？
整合为模块的方式有很多。下面就看看其中一些方法：

### 模块模式（Module pattern）
模块模式用于模仿类（由于 JavaScript 并不支持原生的类），以致我们能在单个对象中存储公有和私有变量与方法——类似于其它编程语言（如 Java 或 Python ）中的类的用法。模块模式不仅允许我们创建公用接口 API（如果我们需要暴露方法时），而且也能在闭包作用域中封装私有变量和方法。

下面有几种方式能实现模块模式（module pattern）。第一个案例中，我将会使用匿名闭包。只需将所有代码放进匿名函数中，就能帮助我们实现目标（记住：在 JavaScript 中，函数是唯一创建新作用域的方式）。

#### Example 1：匿名闭包（Anonymous closure）

    (function () {
      // We keep these variables private inside this closure scope
      // 让这些变量在闭包作用域内变为私有（外界访问不到这些变量）。
      var myGrades = [93, 95, 88, 0, 55, 91];
    
      var average = function() {
        var total = myGrades.reduce(function(accumulator, item) {
          return accumulator + item}, 0);
    
          return 'Your average grade is ' + total / myGrades.length + '.';
      }
    
      var failing = function(){
        var failingGrades = myGrades.filter(function(item) {
          return item < 70;});
    
        return 'You failed ' + failingGrades.length + ' times.';
      }
    
      console.log(failing());
    
    }());
    
    // ‘You failed 2 times.’

通过这种结构，匿名函数拥有自身的求值环境或”闭包“，并立即执行它。这就实现了对上级（全局）命名空间的隐藏。

这种方法的好处是：能在函数内使用本地变量，而不会意外地重写已存在的全局变量。当然，你也能获取全局变量，如：

    var global = 'Hello, I am a global variable :)';
    
    (function () {
      // We keep these variables private inside this closure scope
    
      var myGrades = [93, 95, 88, 0, 55, 91];
    
      var average = function() {
        var total = myGrades.reduce(function(accumulator, item) {
          return accumulator + item}, 0);
    
        return 'Your average grade is ' + total / myGrades.length + '.';
      }
    
      var failing = function(){
        var failingGrades = myGrades.filter(function(item) {
          return item < 70;});
    
        return 'You failed ' + failingGrades.length + ' times.';
      }
    
      console.log(failing());
      console.log(global);
    }());
    
    // 'You failed 2 times.'
    // 'Hello, I am a global variable :)'

这里需要注意的是，包围着匿名函数的小括号是必须的，这是因为当语句以关键字 function 开头时，它会被认为是一个函数声明语句（记住，JavaScript 中不能拥有未命名的函数声明语句）。因此，该括号会创建一个函数表达式代替它。欲知详情，可点击 [这里][8]。

#### Example 2：全局导入（Global import ）
另一个常见的方式是类似于 [jQuery][9] 的全局导入（global import）。该方式与上述的匿名闭包相似，特别之处是传入了一个全局变量作为参数：

    (function (globalVariable) {
    
      // Keep this variables private inside this closure scope
      var privateFunction = function() {
        console.log('Shhhh, this is private!');
      }
    
      // Expose the below methods via the globalVariable interface while
      // hiding the implementation of the method within the 
      // function() block
      // 通过 globalVariable 接口暴露下面的方法。当然，这些方法的实现则隐藏在 function() 块内
    
      globalVariable.each = function(collection, iterator) {
        if (Array.isArray(collection)) {
          for (var i = 0; i < collection.length; i++) {
            iterator(collection[i], i, collection);
          }
        } else {
          for (var key in collection) {
            iterator(collection[key], key, collection);
          }
        }
      };
    
      globalVariable.filter = function(collection, test) {
        var filtered = [];
        globalVariable.each(collection, function(item) {
          if (test(item)) {
            filtered.push(item);
          }
        });
        return filtered;
      };
    
      globalVariable.map = function(collection, iterator) {
        var mapped = [];
        globalUtils.each(collection, function(value, key, collection) {
          mapped.push(iterator(value));
        });
        return mapped;
      };
    
      globalVariable.reduce = function(collection, iterator, accumulator) {
        var startingValueMissing = accumulator === undefined;
    
        globalVariable.each(collection, function(item) {
          if(startingValueMissing) {
            accumulator = item;
            startingValueMissing = false;
          } else {
            accumulator = iterator(accumulator, item);
          }
        });
    
        return accumulator;
    
      };
    
     }(globalVariable));

在该案例中，`globalVariable` 是唯一的全局变量。这个相对于匿名闭包的优势是：提前声明了全局变量，能让别人更清晰地阅读你的代码。

#### Example 3：对象接口（Object interface）
使用一个独立的对象接口创建模块，如：

    var myGradesCalculate = (function () {
    
      // Keep this variable private inside this closure scope
      var myGrades = [93, 95, 88, 0, 55, 91];
    
      // Expose these functions via an interface while hiding
      // the implementation of the module within the function() block
    
      return {
        average: function() {
          var total = myGrades.reduce(function(accumulator, item) {
            return accumulator + item;
            }, 0);
    
          return'Your average grade is ' + total / myGrades.length + '.';
        },
    
        failing: function() {
          var failingGrades = myGrades.filter(function(item) {
              return item < 70;
            });
    
          return 'You failed ' + failingGrades.length + ' times.';
        }
      }
    })();
    
    myGradesCalculate.failing(); // 'You failed 2 times.' 
    myGradesCalculate.average(); // 'Your average grade is 70.33333333333333.'

正如你所看到的，该方式让你决定哪个变量/方法是私有的（如 `myGrades`），哪个变量/方法是需要暴露出来的（通过将需要暴露出来的变量/方法放在 return 语句中，如 `average` & `failing`）。

#### Example 4: 暴露模块模式（Revealing module pattern）
这与上一个方法非常类似，只不过该方法确保所有变量和方法都是私有的，除非显式暴露它们：

    var myGradesCalculate = (function () {
    
      // Keep this variable private inside this closure scope
      var myGrades = [93, 95, 88, 0, 55, 91];
    
      var average = function() {
        var total = myGrades.reduce(function(accumulator, item) {
          return accumulator + item;
          }, 0);
    
        return'Your average grade is ' + total / myGrades.length + '.';
      };
    
      var failing = function() {
        var failingGrades = myGrades.filter(function(item) {
            return item < 70;
          });
    
        return 'You failed ' + failingGrades.length + ' times.';
      };
    
      // Explicitly reveal public pointers to the private functions 
      // that we want to reveal publicly
    
      return {
        average: average,
        failing: failing
      }
    })();
    
    myGradesCalculate.failing(); // 'You failed 2 times.' 
    myGradesCalculate.average(); // 'Your average grade is 70.33333333333333.'

看似有许多知识需要我们吸收，但这只是模块模式（module patterns）的冰山一角。在我学习这方面知识时，发现了下面这些有用的资源：

 - [Learning JavaScript Design Patterns][10]： 出自 Addy Osmani，他以极其简洁的方式对模块模式进行详细分析。
 - [Adequately Good by Ben Cherry][11]：一篇通过案例对模块模式的高级用法进行概述的文章。
 - [Blog of Carl Danley][12]：一篇对模块模式进行概述并拥有其它 JavaScript 模式资源的文章。

### CommonJS and AMD
上述所有方法都有一个共同点：使用一个全局变量将其代码封装在一个函数中，从而利用闭包作用域为自身创建一个私有的命名空间。

虽每种方式都有效，但他们也有消极的一面。

举个例子说，作为一名开发者，需要以正确的依赖顺序去加载你的文件。更直接地说，假如你在项目中使用 Backbone，那么你需要在文件中用 script 标签引入 Backbone 的源代码。

然而，由于 Backbone 重度依赖于 Underscore.js，因此 Backbone 的 script 标签不能放在 Underscore 的 script 标签前。

作为一名开发者，有时会为了正确处理并管理好依赖而感到头痛。

另一个消极面是：他们仍会导致命名空间污染。例如，两个模块拥有同样的名字，或者一个模块拥有两个版本，而且你同时需要他们俩。

所以，你可能会想到：我们能不能设计一种方法，无须通过全局作用域去请求一个模块接口呢？

答案是能！

有两种流行且实现良好的方法：CommonJS 和 AMD。

#### CommonJS
CommonJS 是一个志愿工作组设计并实现的 JavaScript 声明模块 APIs

CommonJS 模块本质上是一片可重用的 JavaScript 代码段，将其以特定对象导出后，其它模块即可引用它。如果你接触过 Node.js，那么你应该非常熟悉这种格式。

通过 CommonJS，每个 JavaScript 文件保存的模块都拥有其独一无二的模块上下文（就像封装在闭包内）。在此作用域中，我们使用 `module.exports` 对象导出模块，然后通过 require 导入它们。

当你定义一个 CommonJS 模块时，代码类似：

    function myModule() {
      this.hello = function() {
        return 'hello!';
      }
    
      this.goodbye = function() {
        return 'goodbye!';
      }
    }
    
    module.exports = myModule;

我们使用特定对象模块，并将 module.exports 指向我们的函数。这让 CommonJS 模块系统知道我们想导出什么，并让其它文件能访问到它。

然后，当有人想使用 `myModule` 时，他们可在文件内将其 `require` 进来，如：

    var myModule = require('myModule');
    
    var myModuleInstance = new myModule();
    myModuleInstance.hello(); // 'hello!'
    myModuleInstance.goodbye(); // 'goodbye!'
    
该方法相对于我们先前讨论的模块模式有两个显而易见的好处：

 1. 避免了全局命名空间的污染
 2. 让依赖关系更明确

此外，该语法非常紧凑简单，我个人非常喜欢。

另外需要注意的一点是：CommonJS 采用服务器优先的方式，并采用同步的方式加载模块。这点很重要，因为如果我们有其它三个模块需要 `require` 进来的话，这些模块会被一个接一个地加载。

这种工作方式很适合应用在服务器上。但不幸的是，当你将这种方式应用在浏览器端时，就会出现问题。因为相对于硬盘，从 web 上读取模块**更耗时**（网络传输等因素）。而且，只要模块正在加载，就会阻塞浏览器运行其它任务。这是由于 JavaScript 线程会在代码加载完成前被停止。（在 Part 2 的模块打包部分，我会告诉你如何解决此问题。而现在，只需了解到这）。

#### AMD
CommonJS 是不错，但如果我们想异步加载模块呢？答案是异步模块定义（Asynchronous Module Definition），或简称 AMD。

使用 AMD 加载模块的代码类似：

    define(['myModule', 'myOtherModule'], function(myModule, myOtherModule) {
      console.log(myModule.hello());
    });

`define` 函数的第一个参数是一个包含本模块所依赖的模块数组。这些依赖都在后台加载（以不阻塞的方式）。加载完成后，`define` 会调用其指定的回调函数。

接着，回调函数会将加载完成后的依赖作为其参数（一一对应）——在该案例中，是 `myModule` 和 `myOtherModule`。因此，回调函数就能使用这些依赖。当然，这些依赖本身也需要通过 `define` 关键字定义。 
例如，`myModule` 类似：

    define([], function() {
    
      return {
        hello: function() {
          console.log('hello');
        },
        goodbye: function() {
          console.log('goodbye');
        }
      };
    });

不像 CommonJS，AMD 采取浏览器优先的方式，通过异步加载的方式完成任务。（注意，有很多人并不赞成此方式，因为他们坚信在代码开始运行时动态且逐个地加载文件是不好的。我将会在下一节的模块构建（module-building）中探讨更多相关信息）。

除了异步外，AMD 的另一个好处是：模块可以是一个对象、函数、构造函数、字符串、JSON 或其它各种类型，而 CommonJS 仅支持对象作为模块。

话虽如此，AMD 不兼容 io、文件系统（filesystem）和其它通过 CommonJS 实现的面向服务器的功能，而且其通过函数封装的语法与简单的 require 语句相比显得有点啰嗦。

#### UMD
对于需要同时支持 AMD 和 CommonJS 特性的项目，你可选择另一种规范：通用的模块定义（Universal Module Defintion，简称 UMD）。

UMD 在本质上创建了一种使用二者其一的方式，同时也支持定义全局变量。因此，UMD 模块适用于客户端和服务器端。

下面快速浏览 UMD 是如何处理其业务的：

    (function (root, factory) {
      if (typeof define === 'function' && define.amd) {
          // AMD
        define(['myModule', 'myOtherModule'], factory);
      } else if (typeof exports === 'object') {
          // CommonJS
        module.exports = factory(require('myModule'), require('myOtherModule'));
      } else {
        // Browser globals (Note: root is window)
        root.returnExports = factory(root.myModule, root.myOtherModule);
      }
    }(this, function (myModule, myOtherModule) {
      // Methods
      function notHelloOrGoodbye(){}; // A private method
      function hello(){}; // A public method because it's returned (see below)
      function goodbye(){}; // A public method because it's returned (see below)
    
      // Exposed public methods
      return {
          hello: hello,
          goodbye: goodbye
      }
    }));

想获取更多关于 UMD 的案例，可看看 Github 上的 [enlightening repo][13]。
 
### 原生 JS（Native JS）
哊！我没把你绕晕了吧？好吧，下面还有**另一种**定义模块的方式。

可能你已注意到：上述的模块都不是原生 JavaScript 模块。它们只不过是我们用模块模式（module pattern）、CommonJS 或 AMD **模仿**的模块系统。

幸运的是，机智的标准制定者在 TC39（该标准定义了 ECMAScript 的语法与语义）已经为 ECMAScript 6（ES6）引入内置的模块系统了。

ES6 为导入（importing）导出（exporting）模块带来了很多可能性。下面是很好的资源：

 - [jsmodules.io][14]
 - [exploringjs.com][15]

相对于 CommonJS 或 AMD，ES6 模块如何设法提供两全其美的实现方案：简洁紧凑的声明式语法和异步加载，另外能更好地支持循环依赖。

我最喜欢 ES6 模块的特性应该是导入的都是**动态**且只读的导出视图（CommonJS 导入的都是导出的副本，因此不是动态的）。

> 上一句的原文是：Probably my favorite feature of ES6 modules is that imports
> are live read-only views of the exports. (Compare this to CommonJS,
> where imports are copies of exports and consequently not alive).

下面这个例子展示了它（CommonJS）如何运行：

    // lib/counter.js
    
    var counter = 1;
    
    function increment() {
      counter++;
    }
    
    function decrement() {
      counter--;
    }
    
    module.exports = {
      counter: counter,
      increment: increment,
      decrement: decrement
    };
    
    // src/main.js
    
    var counter = require('../../lib/counter');
    
    counter.increment();
    console.log(counter.counter); // 1

在此案例中，我们主要构造了该模块的两个副本：一个是在我们导出它时，另一个是在我们引入它时。

此外，在 main.js 的副本与原来的模块是分离的。这就是为什么当我们的计数器自增时，仍返回 1 —— 因为我们导入的计数器变量（counter）与来自原本模块的计数器副本是分离的。

所以，计算器的自增只会在模块内自增，并不会在复制的版本自增。要修改复制版本的计数器的唯一方式是手动自增。

    counter.counter++;
    console.log(counter.counter); // 2

对于ES6，它会在导入时创建一个动态的、只读的模块视图。

    // lib/counter.js
    export let counter = 1;
    
    export function increment() {
      counter++;
    }
    
    export function decrement() {
      counter--;
    }
    
    // src/main.js
    import * as counter from '../../counter';
    
    console.log(counter.counter); // 1
    counter.increment();
    console.log(counter.counter); // 2

很酷对吧？但我认为动态且只读的视图的真正引人注目的是，它允许你将模块分成更小的片段，而又不导致功能的缺失。

你可以反过来再次合并他们，且不会导致任何问题。

### 期待：模块打包（bundling modules）
哇！时间过得真快。这是个疯狂之旅，但我真心希望本文能让你更好地了解 JavaScript 模块。

在下一节，我将会讲述模块打包（module bundling）和覆盖以下核心主题：

 - 为什么需要模块打包
 - 以不同方式进行打包
 - ECMAScript 的模块加载 API
 - 等等 :）

注意：为了尽可能通俗易懂，我跳过了一些细节（如：循环依赖）。如果我漏了任何重要或有趣的知识，请在评论里告诉我！


  [1]: http://web.jobbole.com/
  [2]: http://www.jobbole.com/members/q574805242
  [3]: https://medium.freecodecamp.com/javascript-modules-a-beginner-s-guide-783f7d7a5fcc
  [4]: https://github.com/JChehe/blog/blob/master/translation/JavaScript%20%E6%A8%A1%E5%9D%97%E3%80%90Part%201%E3%80%91%EF%BC%9A%E5%88%9D%E5%AD%A6%E8%80%85%E6%8C%87%E5%8D%97.md
  [5]: https://github.com/JChehe/blog/blob/master/translation/JavaScript%20%E6%A8%A1%E5%9D%97%E3%80%90Part%202%E3%80%91%EF%BC%9A%E6%A8%A1%E5%9D%97%E6%89%93%E5%8C%85.md
  [6]: http://jbcdn2.b0.upaiyun.com/2016/02/e36f4ad51fef734fae6cdf155b053239.jpeg
  [7]: http://c2.com/cgi/wiki?GlobalVariablesAreBad
  [8]: http://stackoverflow.com/questions/1634268/explain-javascripts-encapsulated-anonymous-function-syntax
  [9]: https://github.com/jquery/jquery/tree/master/src
  [10]: https://addyosmani.com/resources/essentialjsdesignpatterns/book/#modulepatternjavascript
  [11]: http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html
  [12]: https://carldanley.com/js-module-pattern/
  [13]: https://github.com/umdjs/umd
  [14]: http://jsmodules.io/cjs.html
  [15]: http://exploringjs.com/es6/ch_modules.html
