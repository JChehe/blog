# 如何成为一个JavaScript 大牛？【译】

标签： JavaScript translate

---

在成长的过程中，我的兴趣点不断发散，而且都是看似不相关的领域。我喜欢数学正如我喜欢历史一样。我的目标是成为一个 多才多艺的人 – 博学者-，能在多个领域成为优秀人才。这证实是一项艰巨的任务，我忽然面临着行行皆通，样样稀松的危险。

我开始考虑专注于某些领域，这样即使不能成为像文艺复兴时期的通才，但至少能精通某些方面。那我怎么样才能专注于某一领域的同时，掌握软件开发所需的庞大的知识体系呢？

本帖内容是基于我过去 5 年经验编写的，概述了我成为一个优秀的 JavaScript 开发者所用到的技术和资源。

当今大多数 web 开发者都面临着同样的问题：他们不得不擅长多个不同领域，从数据库到后端架构，再到前端的用户界面，用所精通的 CSS 知识去修改 UI 。

##看书
为了达到精通，专注与努力是首要条件。如果不投入全身心工作，最后你只会一知半解。例如通过阅读一些博客文章，因为初期时间投入较低，所以看起来会比较简单。但从长远来看，这种学习模式将会比专注于学习精髓的过程花费更多的时间。解决这个难题的方法很简单：看书。

书籍让我们站在文明的肩膀上。而精炼的文字让我们的知识代代相传。而对于如何成为 web 技术专家这个问题，你在学习的过程中就要与 web 本身保持一定距离。因为 web 对于学习来说，其本身就是一个混杂且分散的媒介，所以我的第一个建议是阅读相关专业的书籍。

对于 JavaScript，从 [《JavaScript 语言精髓》](http://shop.oreilly.com/product/9780596517748.do) 这本被称为 JavaScript 圣经的书开始。这本书虽然比较旧，但非常适合入门。[《JavaScript权威指南》](http://shop.oreilly.com/product/9780596805531.do) 也是必备的，尽管你可能会将它作为一个参考书籍。另外，jQuery 作者 John Resig 的 [《JavaScript 忍者禁术》](http://blog.ustunozgur.com/javascript/programming/books/videos/2015/06/17/www.manning.com/resig/) 也是不容错过的。如果你在寻找好（在线免费的）书，可以看看 [《JavaScript Allongé》](https://leanpub.com/javascript-allonge/) 、[《You Don’t Know JS》](https://github.com/getify/You-Dont-Know-JS) 和 [《Eloquent JavaScript》](http://eloquentjavascript.net/)（[点击这里](https://watchandcode.com/courses/eloquent-javascript-the-annotated-version) 可以看它的注释版本）。这些都能以电子书或印刷版的形式购买。另外，Mozilla’s Developer Network 也有很好的 “[JavaScript 指南](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)”。

##学习、使用并阅读库
接下来最重要的一步是了解库。如果书籍教会你如何理解语言，则库教你如何表达它。对于库，你有两个重要的事情要做：使用它们并阅读它们的源代码。

通过使用库，开始知道：jQuery、Backbone、underscore 和 React、Angular 、Ember 中的一个。当然，这不是说你必须使用这些库，但称职的 JavaScript 开发者都应该至少有这些库的使用经验（不管好坏）。

对于提高 JavaScript 技能，第二重要的是阅读这些库的源代码。其中，我特别推荐 Backbone 和 underscore 的源代码，因为它们的代码写得特别漂亮。通过阅读和理解 underscore，你的函数编程能力将会得到提高。另一个是其他几个开发者推荐给我的库是 mootools （我个人没有 mootools 的使用和阅读经验，仅仅是传达信息。）

理解上述列表里的其它库，如 React、Ember 等，可能有点难，但值得付出努力。至少略读其它库的源代码，看看它们是如何组织基础代码并尽量发现一些模式。其它一些值得使用和阅读源代码的库还有 d3、 highcharts 和 moment.js。

##练习与问自己问题
成为优秀 JavaScript 开发者的下一步是做大量的实践。理论上，这些实践的重点不在 DOM，而是语言，所以确保有测试工具能在 node.js 上运行。在 node.js 上做大量小练习。通过不同的方式使用 JavaScript 的闭包、原型、array-extras (map, filter) 等。当你经过大量练习后，头脑里就会对 JavaScript 有基本的想法。

我朋友 Armagan 是一名杰出的 JavaScript 程序员兼老师，他在课堂里使用的课本 [《JavaScript 设计模式》](http://www.apress.com/9781590599082) 也是值得一看的。

试着回答诸如：原型继承是如何工作的？闭包的定义是什么？this 关键字是如何改变的？如何使用 apply/bind/map/filter/call？收集一些 JavaScript 开发者常见问题并尝试用自己的语言解释它。用书面或口头的方式向别人解释这些概念，能极大地提高能力。在做实践的同时，尝试做“假设分析”。例如，“如果使用两次 bind，this 将会代表什么？jQuery 是如何确保 this 关键字是引用 jQuery 对象，而不是全局对象？这个库如何完成某个特性？”这些都是值得思考的常见问题。

##学习标准
下一步是学习更多关于 EcmaScript 标准。找到一份最新的 EcmaScript 标准并尝试阅读它。除了这些，也要尝试学习即将推出的 JavaScript 特性，如 ES 6 和 ES 7 新增的。最近有一些新特性如：promises、modules、generators、comprehensions 和 again。可以通过专门的书来学习标准，如 Zakas 的 ( [Understanding EcmaScript 6](https://leanpub.com/understandinges6) ) 或 Dr. Axel Rauschmayer 的 ES6 书 ( [Exploring JS](http://exploringjs.com/) ) 。阅读标准是获取专业知识和发现语言新特性的主要来源。

##使用 web 上的资源
我之前提到使用 web 获取 web 知识的危险性，所以最后的建议是具体如何在 web 中获取最好的资源。Hacker News 是一个很好的资源，然而如果时刻关注它的话，将会花费较多时间，因为信噪比较低（表示 JavaScript 文章比例较低）。取而代之的是，关注 [JavaScript Weekly](http://javascriptweekly.com/) 之类的每周文摘。随着时间的推移，你会看到哪些库或技术是备受关注的。在 Twitter，尝试去关注那些有影响力的 JavaScript 开发者。这里是 Tutsplus 列出的 [33 个值得关注的 JavaScript 开发者](http://code.tutsplus.com/articles/33-developers-you-must-subscribe-to-as-a-javascript-junkie--net-18151)。其它在 web 上的资源还包括一些博客，如 [Toptal Blogs](http://www.toptal.com/section/front-end)、[Rebecca Murphey’s blog](http://rmurphey.com/) （如果你对这个博客的帖子感兴趣，也可以看看 [A Baseline for Front-End [JS] Developers: 2015](http://rmurphey.com/blog/2015/03/23/a-baseline-for-front-end-developers-2015/)）和 [Nicholas Zakas’ blog](http://www.nczonline.net/)。（如果你有其它好博客，请 Email 我，我会将它添加到该列表里。）

另一个重要资源是大会视频和教育视频。对于大会，JSConf 系列都是高质量的。对于教育视频，我强烈建议 Pluralsight，因为他们拥有经验丰富的开发者准备的高质量课程。（我与 Pluralsgiht 没有隶属关系）

##浓缩版
- 从阅读书籍开始，因为书籍能为你提供精华信息。
- 学习主要的库，如 jQuery、underscore、Backbone，并阅读它们的源代码。
- 多实践并尝试用自己的话解释“继承”之类的常见 JavaScript 概念。对上述主题进行演讲和交流。
- 仔细阅读最新标准，并开始使用该语言的最新特性。
- 关注 web 资源，每周关注一次文摘或博客，或观看会议视频和视频教程。

##总结
一直反复这些并完成大量项目，将会极大地提高你的 JavaScript 编程能力。只有努力不懈，才有希望在几年后成为一名专家。我觉得自己是一名优秀的 JavaScript 程序员，离专家仍有一大段路要走，有很多技术需要在我接下来的学习生涯中学到。另外，随时可以通过 [atustun@ustunozgur.com]() 向我提出反馈和修正。

本文由 [伯乐在线](http://web.jobbole.com/) - [刘健超-J.c](http://www.jobbole.com/members/q574805242) 翻译，[shutear](http://www.jobbole.com/members/shutear) 校稿。未经许可，禁止转载！
英文出处：[Ustun Ozgur](http://blog.ustunozgur.com/javascript/programming/books/videos/2015/06/17/how_to_be_a_great_javascript_software_developer.html)。

感谢您的阅读。
