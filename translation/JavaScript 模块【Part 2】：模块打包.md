# JavaScript 模块【Part 2】：模块打包

标签： JavaScript module ES6 translate

---

本文由 [伯乐在线][1] - [刘健超-J.c][2] 翻译，等待校稿。未经许可，禁止转载！

英文出处：[JavaScript Modules Part 2: Module Bundling][3]。欢迎加入翻译组。

---

本文共有两章节：

 1. [JavaScript 模块：初学者指南][4]
 2. [JavaScript 模块：模块打包][5]

![此处输入图片的描述][6]

在文章的 [Part 1][8]，我讲解了模块是什么、为何要使用模块和为程序整合为模块的各种方式。而在 Part 2，我将会详细讲解模块“打包”：为什么要打包模块，以不同的方式进行打包和模块在 web 开发上的未来。


## 什么是模块打包？
总体上看，模块打包只是简单地将一组模块（和它们所依赖的模块）以正确的顺序整合为单一文件（或文件组）。我们也知道：对于 web 开发，细节才是可怕的地方。 :）。

## 究竟为什么需要打包模块？
当你将程序分为各个模块时，通常会将这些模块放到不同文件或文件夹下。当然，你所使用的库（如 Underscore 或 React）也是模块。

因此，每个文件都必须以一个 `<script>` 标签引入到主 HTML 文件中。然后当用户访问你的主页时，浏览器就会加载这些文件。分离的 `<script>` 标签就意味着浏览器必须单独地加载每个文件（一个接一个）。

...这无疑是页面加载时间的噩耗。

为了解决该问题，我们需要打包或“拼接”所有文件，从而生成一个大文件（或几个文件，视情况而定）以减少请求数量。当你听到开发者讨论“构建步骤”或“构建处理”时，这大概就是他们所讨论的内容了。

另一个加快打包操作的普遍做法是：“压缩”打包的代码。压缩就是从源代码中移除不必要的字符（如空格、注释和换行符等），这样能减少内容的整体大小且不会改变代码的功能。

更少的数据就意味着浏览器处理的时间更短，而且反过来也减少了下载文件的时间。如果你曾看到文件拥有扩展名“min”（如 [underscore-min.js][11]），你应该会注意到压缩版本会比 [完整版][12] 小很多（当然，无可读性可言）。

构建工具（如 Gulp 和 Grunt）能为开发者直接执行拼接（concatenation）和压缩（minification）操作，并确保在打包生成利于浏览器执行的代码的同时，也会导出一份开发者可读的代码。

## 打包模块的不同方式是什么？
当使用标准的模块模式（module pattern，在文章的前一节中所讨论的）定义模块时，拼接和压缩文件都能很好运行。你实际所做的是将各个原生 JavaScript 代码混合在一起。

然而，如果你使用的是非原生的模块系统，如 CommonJS 或 AMD（甚至是原生的 ES6 模块格式，因为浏览器仍不支持该语法），浏览器就不能解析识别了。此时你需要使用特定工具将模块转为顺序正确且对浏览器友好的代码。这些工具可以是 Browserify、RequireJS、Webpack 或其它“模块打包工具”或“模块加载器”。

除了打包和（或）加载模块，模块打包工具也提供了很多额外功能，如自动重编译（当你对代码作出修改或为了调试而生成 source maps 时）。

下面是一些常见的模块打包方法：

## 打包 CommonJS
正如你从 [Part 1][15] 可知，CommonJS 是同步加载模块的，但这对于浏览器来说并不切合实际。我在 Part 1 提到了一种解决方案 —— 其中一种是模块打包工具 Browserify。Browserify 是一种将 CommonJS 模块编译成浏览器能执行的代码的工具。

举个例子，main.js 文件导入一个用于计算 `number数组` 平均数的模块：

    var myDependency = require(‘myDependency’);
    
    var myGrades = [93, 95, 88, 0, 91];
    
    var myAverageGrade = myDependency.average(myGrades);

因此，main.js 文件有一个依赖项（myDependency）。当使用以下命令时，Browserify 会递归打包所有由 main.js 文件开始引入的模块，到一个名为 bundle.js 的文件：

    browserify main.js -o bundle.js

Browserify 要实现以上功能，它要解析 [抽象语法树（AST）][17] 的每个 `require` 调用，以遍历项目的整个依赖图。一旦它解决了依赖的构造关系，就能将模块以正确的顺序打包进一个单独文件内。然后，在 html 里插入一个用于引入 `“bundle.js”` 的 `<script>` 标签，从而确保你的源代码在一个 HTTP 请求中完成下载。

同样地，如果多个文件拥有多个依赖，你只需简单地告诉 Browserify 你的入口文件（entry file），然后休息一会等待它完成魔法即可。

最终产品：打包文件需要通过 Minify-JS 之类的工具压缩打包后的代码。

## 打包 AMD
如果你使用的是 AMD，你需要使用 AMD 加载器，如 RequireJS 或 Curl。一个模块加载器（与打包工具不同）会动态加载程序需要运行的模块。

再次提醒，AMD 与 CommonJS 的主要区别是：AMD 以异步的方式加载模块。也就是说， 对于 AMD，你实际上不需要将模块打包到一个文件的这个构建步骤，因为它是以异步方式加载模块——也就意味着当用户第一次访问网页时，浏览器会循序渐进地下载程序实际需要执行的文件，而不是一次性下载所有文件。

然而，在实际生产环境中，随着用户操作，大容量的请求开销并不会产生多大意义。但大多数开发者为了优化性能，仍然使用构建工具（如 RequireJS 优化工具和 [r.js][19]）打包和压缩它们的 AMD 模块。

总的来说，AMD 与 CommonJS 之间的打包差异是：在开发期间，AMD 应用无须任何构建步骤即可运行。当然，在代码上线前，要使用优化工具（如 r.js）进行优化。

想了解更多关于 CommonJS vs. AMD 的有趣讨论，可看看 [Tom Dale’s blog][21] 的这篇文章 : )。

## Webpack
就打包工具而言，Webpack 是这方面的新生儿。它与你所使用的具体模块系统无关，也就是说它允许开发者使用 CommonJS、AMD 或 ES6。

你可能会疑惑：我们已经有其它打包工具（如 Browserify 和 RequireJS）完成相应工作并做得相当好了，为什么还需要 Webpack。没错，Webpack 提供了一些有用的功能，如“代码分割（code splitting）”——一种将代码库分割为“块（chunks）”的方式，从而能实现按需加载。 

例如，如果 web 应用的某段代码块在某种环境下才被用到时，却直接将整个代码库放进一个庞大的打包文件，显然不那么高效。因此，你可使用“代码分割”，将其提取出来成为“打包块（bundled chunks）”，然后按需加载。对于大多数用户只需应用程序的核心部分这种情况，就避免了前期负荷过重的问题。

代码分割只是 Webpack 提供的众多引人注目的功能之一，网上有很多关于 “Webpack 与 Browserify 谁更好”的激烈讨论。下面列出了一些围绕该问题的、能理清思路的讨论：

 - https://gist.github.com/substack/68f8d502be42d5cd4942
 - http://mattdesl.svbtle.com/browserify-vs-webpack
 - http://blog.namangoel.com/browserify-vs-webpack-js-drama

## ES6 模块
跟得上吧？很好！因为接下来要讲 ES6 模块，某种意义上它在未来能削弱对打包工具的需求。（你马上会明白我的意思。）首先，让我们知道 ES6 模块如何被加载。

当前的 JS 模块规范（CommonJS、AMD）与 ES6 模块之间最重要的区别是：设计 ES6 模块时考虑到了静态分析。其意思是：当你导入模块时，该导入在编译时（换言之，在脚本开始执行前。）已执行。这允许我们在运行程序前移除那些不被其它模块使用的导出模块（exports）。移除不被使用的模块能节省空间，且有效地减少浏览器的压力。

一个常被提起的问题是：使用 UglifyJS 之类的工具压缩代码后（即消除冗余代码 dead code elimination）会有何不同？答案是：“视情况而定”。

> （注意：消除冗余代码是一个优化步骤，它能移除无用的代码和变量——即移除打包程序不需要执行的冗余代码）。

有时 UglifyJS 与 ES6 模块的消除冗余代码的工作完全相同，有时则不是。如果你想了解相关知识，可看看 [Rollup’s wiki][23] 的案例。

导致 ES6 模块不同的原因是它以不同方式去完成消除冗余代码的效果，我们称该方式为“tree shaking”。Tree shaking 本质与消除冗余代码相反。它仅包含打包文件需要运行的代码，而不是排除打包文件不需要的代码。让我们看看 tree shaking 的一个案例：

假设有一个带有多个函数的 utils.js 文件，每个函数都用 ES6 的语法导出：

    export function each(collection, iterator) {
      if (Array.isArray(collection)) {
        for (var i = 0; i < collection.length; i++) {
          iterator(collection[i], i, collection);
        }
      } else {
        for (var key in collection) {
          iterator(collection[key], key, collection);
        }
      }
     }
    
    export function filter(collection, test) {
      var filtered = [];
      each(collection, function(item) {
        if (test(item)) {
          filtered.push(item);
        }
      });
      return filtered;
    }
    
    export function map(collection, iterator) {
      var mapped = [];
      each(collection, function(value, key, collection) {
        mapped.push(iterator(value));
      });
      return mapped;
    }
    
    export function reduce(collection, iterator, accumulator) {
        var startingValueMissing = accumulator === undefined;
    
        each(collection, function(item) {
          if(startingValueMissing) {
            accumulator = item;
            startingValueMissing = false;
          } else {
            accumulator = iterator(accumulator, item);
          }
        });
    
    	return accumulator;
    }

接着，假设我们不知道程序需要 utils.js 里的哪些函数，所以直接将上述模块内的所有函数导入到 main.js，如下：
 
    import * as Utils from ‘./utils.js’;

最终我们只用到了 each 函数：
    
    import * as Utils from ‘./utils.js’;

    Utils.each([1, 2, 3], function(x) { console.log(x) });

“tree shaken” 版本的 main.js 看起来如下（一旦模块被加载后）：

    function each(collection, iterator) {
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
    
    each([1, 2, 3], function(x) { console.log(x) });

注意：只导出我们使用的 `each` 函数。

或者我们决定使用 filter 函数，而不是 each 函数，则最终看到的代码如下：

    import * as Utils from ‘./utils.js’;
    
    Utils.filter([1, 2, 3], function(x) { return x === 2 });

tree shaken 版本如下：
        
    function each(collection, iterator) {
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
    
    function filter(collection, test) {
      var filtered = [];
      each(collection, function(item) {
        if (test(item)) {
          filtered.push(item);
        }
      });
      return filtered;
    };
    
    filter([1, 2, 3], function(x) { return x === 2 });

此刻，`each` 和 `filter` 函数都被包含进来。这是因为 `filter` 在定义时使用了 `each`。因此也需要导出该函数模块以保证程序正常运行。

很聪明，对吧？

我要向你发起挑战，在 Rollup.js 的 [线上案例与编辑器][25] 中探索 tree shaking 吧。

## 构建 ES6 模块
现在我们知道加载 ES6 模块与其它模块规范是不同的，但我们还没讲使用 ES6 模块时的构建步骤。

不幸的是，由于浏览器到现在仍不支持加载原生 ES6 模块，如果现在要使用 ES6 模块则需要其它额外的工作。

![此处输入图片的描述][26]

下面有两个实现构建/转化 ES6 模块（以至浏览器能执行）的方法，第一个是现在最常用的方式：

 1. 使用转译器（如 Babel 或 Traceur）以 CommonJS、AMD 或 UMD 其中一种规范将 ES6 代码转译为 ES5 代码。然后通过模块打包工具（如 Browserify 或 Webpack）将转译后的代码打包成一个或多个文件。
 2. 使用 [Rollup.js][28]，这与前一个方式很相似，不同的是 Rollup 拥有 ES6 模块的静态分析代码（ES6 代码）与依赖的能力。它利用 “tree shaking” 让打包文件拥有最精简的代码。总言之，对于 ES6 模块，使用 Rollup.js （相较于 Browserify 或 Webpack）的最大好处是 tree shaking 能让打包文件更小。需要提醒你的是：Rollup 提供了几种打包代码的规范，包括 ES6、CommonJS、AMD、UMD 和 IIFE（立即调用函数表达式）。IIFE 和 UMD 的打包能直接在浏览器运行，但如果你选择打包 AMD、CommonJS 或 ES6 模块时，需要寻找能将代码转成浏览器能理解运行的代码的方法（例如，使用 Broserify、Webpack、RequireJS 等）。

## 跨越障碍
作为 Web 开发者，我们不得不跨越很多障碍。例如，将优美的 ES6 模块转为浏览器能识别的代码并不总是一帆风顺。

问题是，ES6 模块什么时候才能脱离上述的代码构建开销呢？

答案是：“尽快”。

ECMAScript 目前有一个解决方案叫 [ECMAScript 6 module loader API][30]。简言之，这是一个纲领性的、基于 Promise 的 API，它支持动态加载模块并缓存模块，以便后续的导入不需要重新加载模块。

它看起来如下：

**myModule.js**

    export class myModule {
      constructor() {
        console.log('Hello, I am a module');
      }
    
      hello() {
        console.log('hello!');
      }
    
      goodbye() {
        console.log('goodbye!');
      }
    }

**main.js**
    
    System.import(‘myModule’).then(function(myModule) {
      new myModule.hello();
    });
    
    // ‘hello!’

你亦可直接对 script 标签指定 “type=module” 来定义模块，如：

    <script type="module">
      // loads the 'myModule' export from 'mymodule.js'
      import { hello } from 'mymodule';
    
      new Hello(); // 'Hello, I am a module!'
    </script>

如果你还没看过 the module API polyfill 的 repo，我强烈建议你 [看看][32]。 

此外，如果你想试试该方法，那就看看 [SystemJS][35]，它构建于 [ES6 Module Loader polyfill][36] 之上。SystemJS 能在浏览器和 Node 上动态加载任何模块规范（ES6 模块、AMD、CommonJS、全局脚本）。它在一个 “模块注册器（module registry）”上保存了所有已加载模块的路径，从而避免重新加载先前已加载的模块。更不用说它能自动转译 ES6 模块（只需简单配置）和拥有从任何类型模块中加载任何类型模块的能力了。

## 有了原生的 ES6 模块后，还需要模块打包吗？
对于日益普及的 ES6 模块，下面有一些有趣的观点：

### HTTP/2 会淘汰模块打包吗？
HTTP/1 只允许每个 TCP 连接带一个请求。这就是加载多个资源时需要多个请求的原因。而 HTTP/2 是完全多路复用的，这意味着多个请求和响应可并行执行。因此，我们可用单独一个链接同时处理多个请求。

由于每个 HTTP 请求（HTTP/2）的成本远低于 HTTP/1，从长远来说，加载多个模块不再是一个严重的性能问题。一些人认为模块打包不再需要了。这当然是有可能的，但这要具体情况具体分析了。

举个例说，HTTP/2 不享有模块打包提供的优势，例如移除未被使用的导出模块以节省空间。如果一个网站的每一丁点性能都至关重要，那么长远来看，打包能带来增量效益。当然，如果你对性能需求不那么极端，你可能会通过跳过该构建步骤（打包文件），以最小的成本节省时间。

总的来说，要让大多数网站使用 HTTP/2 协议仍有很长的路要走。我预测构建处理至少在短期内仍会保留。

PS：如果你对 HTTP/2 与 HTTP/1.x 的差异感兴趣，可看看这份 [优秀的资源][38]。

### CommonJS、AMD 与 UMD 会被淘汰吗？
一旦 ES6 成为模块标准，我们还需要其它非原生的模块规范吗？

我持怀疑态度。

若 Web 开发遵守一个标准方法进行导入和导出模块，将获益匪浅，而且省去了中间步骤（译者注：一些构建处理）。但 ES6 成为模块规范需要多长时间呢？

机会是有，但得等一段时间 ;)

再者，众口难调，所以“一个标准的方法”可能永远不会成为现实。

## 总结
我希望文章的两章节能让你理清一些开发者口中的模块和模块打包的相关概念。如果发现上文有令你困惑的地方，可看看 [part I][40]。

一如既往，可以在评论区和我尽情交流或回答问题！


  [1]: http://web.jobbole.com/
  [2]: http://www.jobbole.com/members/q574805242
  [3]: https://medium.com/@preethikasireddy/javascript-modules-part-2-module-bundling-5020383cf306
  [4]: https://github.com/JChehe/blog/blob/master/translation/JavaScript%20%E6%A8%A1%E5%9D%97%E3%80%90Part%201%E3%80%91%EF%BC%9A%E5%88%9D%E5%AD%A6%E8%80%85%E6%8C%87%E5%8D%97.md
  [5]: https://github.com/JChehe/blog/blob/master/translation/JavaScript%20%E6%A8%A1%E5%9D%97%E3%80%90Part%202%E3%80%91%EF%BC%9A%E6%A8%A1%E5%9D%97%E6%89%93%E5%8C%85.md
  [6]: https://cdn-images-1.medium.com/max/2000/1*e0eQH_9X8jN7yC6AEqlvdQ.jpeg
  [7]: https://medium.freecodecamp.com/javascript-modules-a-beginner-s-guide-783f7d7a5fcc?source=latest---------1
  [8]: https://github.com/JChehe/blog/blob/master/translation/JavaScript%20%E6%A8%A1%E5%9D%97%E3%80%90Part%201%E3%80%91%EF%BC%9A%E5%88%9D%E5%AD%A6%E8%80%85%E6%8C%87%E5%8D%97.md
  [9]: https://github.com/jashkenas/underscore/blob/master/underscore-min.js
  [10]: https://github.com/jashkenas/underscore/blob/master/underscore.js
  [11]: https://github.com/jashkenas/underscore/blob/master/underscore-min.js
  [12]: https://github.com/jashkenas/underscore/blob/master/underscore.js
  [13]: https://medium.freecodecamp.com/javascript-modules-a-beginner-s-guide-783f7d7a5fcc#.y8hs0nsne
  [14]: https://medium.freecodecamp.com/javascript-modules-a-beginner-s-guide-783f7d7a5fcc#.y8hs0nsne
  [15]: https://github.com/JChehe/blog/blob/master/translation/JavaScript%20%E6%A8%A1%E5%9D%97%E3%80%90Part%201%E3%80%91%EF%BC%9A%E5%88%9D%E5%AD%A6%E8%80%85%E6%8C%87%E5%8D%97.md
  [16]: https://en.wikipedia.org/wiki/Abstract_syntax_tree
  [17]: https://en.wikipedia.org/wiki/Abstract_syntax_tree
  [18]: http://requirejs.org/docs/optimization.html
  [19]: http://requirejs.org/docs/optimization.html
  [20]: http://tomdale.net/2012/01/amd-is-not-the-answer/
  [21]: http://tomdale.net/2012/01/amd-is-not-the-answer/
  [22]: https://github.com/rollup/rollup
  [23]: https://github.com/rollup/rollup
  [24]: http://rollupjs.org/
  [25]: http://rollupjs.org/
  [26]: https://cdn-images-1.medium.com/max/800/1*lpAgpggDLcK1a3MBEbmODg.png
  [27]: http://rollupjs.org/
  [28]: http://rollupjs.org/
  [29]: https://github.com/ModuleLoader/es6-module-loader
  [30]: https://github.com/ModuleLoader/es6-module-loader
  [31]: https://github.com/ModuleLoader/es6-module-loader
  [32]: https://github.com/ModuleLoader/es6-module-loader
  [33]: https://github.com/systemjs/systemjs
  [34]: https://github.com/ModuleLoader/es6-module-loader
  [35]: https://github.com/systemjs/systemjs
  [36]: https://github.com/ModuleLoader/es6-module-loader
  [37]: https://http2.github.io/faq/#what-are-the-key-differences-to-http1x
  [38]: https://http2.github.io/faq/#what-are-the-key-differences-to-http1x
  [39]: https://medium.freecodecamp.com/javascript-modules-a-beginner-s-guide-783f7d7a5fcc#.y8hs0nsne
  [40]: https://github.com/JChehe/blog/blob/master/translation/JavaScript%20%E6%A8%A1%E5%9D%97%E3%80%90Part%201%E3%80%91%EF%BC%9A%E5%88%9D%E5%AD%A6%E8%80%85%E6%8C%87%E5%8D%97.md
