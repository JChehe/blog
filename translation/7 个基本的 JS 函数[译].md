# 7 个基本的 JS 函数

标签： JavaScript translate

---

我记得早期的 JavaScript ，要完成任何事情几乎都绕不开一些简单的函数，因为浏览器提供商实现功能有所差异，而且不只是边缘功能，基础功能也一样，如 `addEventListener` 和 `attachEvent`。虽然时代变了，但仍有一些函数是每个开发者都应该掌握的，以便于完成某些功能和提高性能。

### [debounce](http://davidwalsh.name/javascript-debounce-function)

对于高耗能事件，debounce 函数是一种不错解决方案。如果你不对 `scroll`、`resize`、和 `key*` 事件使用 debounce  函数，那么你几乎等同于犯了错误。下面的 `debounce` 函数能让你的代码保持高效：

    // 返回一个函数，如果它被不间断地调用，它将不会得到执行。该函数在停止调用 N 毫秒后，再次调用它才会得到执行。如果有传递 ‘immediate’ 参数，会马上将函数安排到执行队列中，而不会延迟。
    function debounce(func, wait, immediate) {
	    var timeout;
	    return function() {
		    var context = this, args = arguments;
		    var later = function() {
			    timeout = null;
			    if (!immediate) func.apply(context, args);
		    };
		    var callNow = immediate && !timeout;
		    clearTimeout(timeout);
		    timeout = setTimeout(later, wait);
		    if (callNow) func.apply(context, args);
	    };
    };

    // 用法
    var myEfficientFn = debounce(function() {
        // 所有繁重的操作
    }, 250);
    window.addEventListener('resize', myEfficientFn);
`debounce` 函数不允许回调函数在指定时间内执行多于一次。当为一个会频繁触发的事件分配一个回调函数时，该函数显得尤为重要。

### [poll](http://davidwalsh.name/javascript-polling)
尽管上面我提及了 `debounce` 函数，但如果事件不存在时，你就不能插入一个事件以判断所需的状态，那么就需要每隔一段时间去检查状态是否达到你的要求。

    function poll(fn, callback, errback, timeout, interval) {
        var endTime = Number(new Date()) + (timeout || 2000);
        interval = interval || 100;

        (function p() {
            // 如果条件满足，则执行！
            if (fn()) {
                callback();
            }
            // 如果条件不满足，但并未超时，再来一次
            else if (Number(new Date()) < endTime) {
                setTimeout(p, interval);
            }
            // 不匹配且时间消耗过长，则拒绝！
            else {
                errback(new Error('timed out for ' + fn + ': ' + arguments));
            }
        })();
    }

    // 用法：确保元素可见
    poll(
        function() {
            return document.getElementById('lightbox').offsetWidth > 0;
        },
        function() {
            // 执行，成功的回调函数
        },
        function() {
            // 错误，失败的回调函数
        }
    );
Polling 在 web 中已被应用很长时间了，并在将来仍会被使用。

### [once](http://davidwalsh.name/javascript-once)
有时候，你想让一个给定的功能只发生一次，类似于 `onload` 事件。下面的代码提供了你所说的功能：

    function once(fn, context) {
        var result;

        return function() {
            if (fn) {
                result = fn.apply(context || this, arguments);
                fn = null;
            }

            return result;
        };
    }

    // 用法
    var canOnlyFireOnce = once(function() {
        console.log('Fired!');
    });

    canOnlyFireOnce(); // "Fired!"
    canOnlyFireOnce(); // // 没有执行指定函数
`once` 函数确保给定函数只能被调用一次，从而防止重复初始化！

### [getAbsoluteUrl](http://davidwalsh.name/get-absolute-url)
从一个字符串变量得到一个绝对 URL，并不是你想象中这么简单。对于某些 `URL` 构造器，如果你不提供必要的参数就会出问题（而有时候你真的不知道提供什么参数）。下面有一个优雅的技巧，只需要你传递一个字符串就能得到相应的绝对 URL。
    
    var getAbsoluteUrl = (function() {
        var a;
        return function(url) {
            if (!a) a = document.createElement('a');
            a.href = url;

            return a.href;
        };
    })();

    // 用法
    getAbsoluteUrl('/something'); // http://davidwalsh.name/something

a 元素的 `href` 处理和 url 处理看似无意义，而 return 语句返回了一个可靠的绝对 URL。

### [isNative](http://davidwalsh.name/detect-native-function)
如果你想知道一个指定函数是否是原生的，或者能不能通过声明来覆盖它。下面这段便于使用的代码能给你答案：

    ;(function() {

        // Used to resolve the internal `[[Class]]` of values
        var toString = Object.prototype.toString;

        // Used to resolve the decompiled source of functions
        var fnToString = Function.prototype.toString;

        // Used to detect host constructors (Safari > 4; really typed array specific)
        var reHostCtor = /^\[object .+?Constructor\]$/;

        // Compile a regexp using a common native method as a template.
        // We chose `Object#toString` because there's a good chance it is not being mucked with.
        var reNative = RegExp('^' +
            // Coerce `Object#toString` to a string
            String(toString)
            // Escape any special regexp characters
            .replace(/[.*+?^${}()|[\]\/\\]/g, '\\$&')
            // Replace mentions of `toString` with `.*?` to keep the template generic.
            // Replace thing like `for ...` to support environments like Rhino which add extra info
            // such as method arity.
            .replace(/toString|(function).*?(?=\\\()| for .+?(?=\\\])/g, '$1.*?') + '$'
        );

        function isNative(value) {
            var type = typeof value;
            return type == 'function'
                // Use `Function#toString` to bypass the value's own `toString` method
                // and avoid being faked out.
                ? reNative.test(fnToString.call(value))
                // Fallback to a host object check because some environments will represent
                // things like typed arrays as DOM methods which may not conform to the
                // normal native pattern.
                : (value && type == 'object' && reHostCtor.test(toString.call(value))) || false;
        }

        // export however you want
        module.exports = isNative;
    }());
这个函数虽不完美，但它能完成任务！

### [insertRule](http://davidwalsh.name/add-rules-stylesheets)
我们都知道能通过选择器（通过 `document.querySelectorAll` ）获取一个 NodeList ，并可为每个元素设置样式，但有什么更高效的方法为选择器设置样式呢（例如你可以在样式表里完成）：

    var sheet = (function() {
        // 创建 <style> 标签
        var style = document.createElement('style');

        // 如果你需要指定媒介类型，则可以在此添加一个 media (和/或 media query) 
        // style.setAttribute('media', 'screen')
        // style.setAttribute('media', 'only screen and (max-width : 1024px)')

        // WebKit hack :(
        style.appendChild(document.createTextNode(''));

        // 将 <style> 元素添加到页面
        document.head.appendChild(style);

        return style.sheet;
    })();

    // 用法
    sheet.insertRule("header { float: left; opacity: 0.8; }", 1);
这对于一个动态且重度依赖 AJAX 的网站来说是特别有用的。如果你为一个选择器设置样式，那么你就不需要为每个匹配到的元素都设置样式（现在或将来）。

### [matchesSelector](http://davidwalsh.name/element-matches-selector)
我们经常会在进行下一步操作前进行输入校验，以确保是一个可靠值，或确保表单数据是有效的，等等。但我们平时是怎么确保一个元素是否有资格进行进一步操作呢？如果一个元素有给定匹配的选择器，那么你可以使用 `matchesSelector` 函数来校验：

    function matchesSelector(el, selector) {
        var p = Element.prototype;
        var f = p.matches || p.webkitMatchesSelector || p.mozMatchesSelector || p.msMatchesSelector || function(s) {
            return [].indexOf.call(document.querySelectorAll(s), this) !== -1;
        };
        return f.call(el, selector);
    }

    // 用法
    matchesSelector(document.getElementById('myDiv'), 'div.someSelector[some-attribute=true]')

就这样啦，上述 7 个 JavaScript 函数是每个开发者都应该时刻记着的。有哪个函数我错过了呢？请把它分享出来！

本文由 [伯乐在线](http://web.jobbole.com/) - [刘健超-J.c](http://www.jobbole.com/members/q574805242) 翻译，[周进林](http://www.jobbole.com/members/8zjl8) 校稿。未经许可，禁止转载！
英文出处：[davidwalsh.name](http://davidwalsh.name/essential-javascript-functions)。

感谢您的阅读。
