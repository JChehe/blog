# Web应用上线之前，每个程序员应该了解的技术细节

标签： Web translation

---

【伯乐在线注】：《Web 应用上线前，程序员应考虑哪些技术细节呢？》这是 StackExchange 上面的一个经典问题贴。
最赞回复有 2200+ 顶，虽然大多数人可能都听过其中大部分内容，但应该会有你没有深入了解的内容。一起来看看。


问题
===
Web 应用上线前，程序员应考虑哪些技术细节呢？ 如果 Jeff Atwood 忘记把 [HttpOnly cookies、sitemaps](http://programmers.stackexchange.com/users/37/jeff-atwood) 和 [cross-site request forgeries](http://www.codinghorror.com/blog/archives/001171.html) 放在同一个网站，那我会把什么重要的东西也会忘掉呢？

我以一个 Web 开发人员的角度思考这个问题，别人为网站进行美化设计并填充内容。因此，他们可能认为可用性和内容比平台更重要，程序员在这方面没多少发言权了。而你需要考虑到的是：你实现平台的稳定性、安全性和满足其它商业目的（如成本不要太高、耗时不要太长和网站排名）。

而以一位已经在相当可信的环境下，完成了几个企业内网应用程序项目的开发者角度思考，并在一个流行且权威网站上为整个糟糕的万维网打响第一枪。

另外，我希望能回答得更加具体一点，而不仅仅是一个“Web标准”这样模糊的答案。通过 HTTP 传输的 HTML、JavaScript、CSS 是必须要掌握的，特别是针对那些资深 Web 开发者。所以，超出上述范围，哪一个标准？在什么环境下，并且为什么这样？**麻烦您提供一个跳转到该标准说明的链接。**

最佳回复
===
下面列表里的大部分内容，我们大多数人都应该已经听过了。所以在这之前，你可能只有一到两个项目没有深入查看和理解透彻，或甚至没听过。

界面和用户体验
---
- 应意识到浏览器实现标准不一致，并确保你的网站能在所有主流浏览器上合理运行。至少起码在最近的 [Gecko](https://en.wikipedia.org/wiki/Gecko_%28layout_engine%29) 引擎（[Firefox](http://firefox.com/)）、Webkit 引擎（[Safari](http://www.apple.com/safari/) 和一些移动端浏览器）、[Chrome](http://www.google.com/chrome)、支持 [IE 浏览器](https://en.wikipedia.org/wiki/Internet_Explorer)（利用 [Application Compatibility VPC Images](http://www.microsoft.com/Downloads/details.aspx?FamilyID=21eabb90-958f-4b64-b5f1-73d0a413c8ef&displaylang=en) 进行测试）和 [Opera](http://www.opera.com/)。另外，也要考虑浏览器在不同操作系统下是如何渲染网站的。
- 要考虑到用户除了通过主流浏览器来浏览网站外，还有其它方式：手机、屏幕阅读器和搜索引擎等。 这有一些相关信息：[WAI](http://www.w3.org/WAI/) 和 [Section508](http://www.section508.gov/)，移动开发：[MobiForge](http://mobiforge.com/)。
- Staging：如何部署更新而不影响用户。进行一次或多次测试或 staging 环境可用来实现架构的更改，确保代码或全部内容能部署在一个可控的方式而不会破坏任何东西。有一个自动化的方式部署批准改变网站。最有效地实现方法是使用版本控制系统(Git、CVS、Subversion 等)和一个自动构建机制( Ant、 NAnt 等)。
- 不要向用户直接显示不友好的错误提示。
- 不要以纯文本的方式显示用户的 Email 地址，否则他们将会收到该死的垃圾邮件。
- 为用户链接添加属性 rel = “nofollow” 来 [避免垃圾邮件](https://en.wikipedia.org/wiki/Nofollow)。
- [为你的网站建立深思熟虑的限制](http://www.codinghorror.com/blog/archives/001228.html) – 这也属于下面将要讲到的安全性。
- 学会如何实现网页的 [渐进增强](https://en.wikipedia.org/wiki/Progressive_enhancement)。
- [POST 提交成功后，要重定向](https://en.wikipedia.org/wiki/Post/Redirect/Get)，以防止再次提交引起刷新。
- 别忘了考虑到访问性（accessibility，即残障人士如何使用网站）。这一直是好想法并且有时这是法定要求。[WAI-ARIA](http://www.w3.org/WAI/intro/aria) 和 [WCAG 2](http://www.w3.org/TR/WCAG20/) 都是这方面很好的资源。
- 别让用户思考如何操作。

安全性
---
- 阅读 [《OWASP开发指南》](http://www.owasp.org/index.php/Category%3aOWASP_Guide_Project)，它提供了全面的网站安全指导。
- 知道注入相关的知识，尤其是 [SQL 注入](https://en.wikipedia.org/wiki/SQL_injection)，并知道如何防止它。
- 千万别相信用户的输入，也不要相信任何请求（其中包括 cookies 和 表单域的隐藏字段值！）。
- 使用 salt（密码散列技术）散列密码并为你的彩虹表行使用不同的 salts 来防止 rainbow 攻击。 使用一个效率较低的散列算法，如 bcrypt （ 久经试验的）或 scrypt （更新，甚至更强）（[1](http://www.tarsnap.com/scrypt.html)，[2](http://it.slashdot.org/comments.pl?sid=1987632&cid=35149842)），来存储密码。（[如何安全地存储一个密码](http://codahale.com/how-to-safely-store-a-password/)）。NIST 也批准用 [PBKDF2](https://security.stackexchange.com/q/7689/396) 散列密码，[FIPS](https://security.stackexchange.com/a/2136/396) 认可 [.NET](https://security.stackexchange.com/a/2136/396) （想了解更多信息，请 [点击](https://security.stackexchange.com/questions/211/how-to-securely-hash-passwords)）。应避免直接使用 MD5 或 SHA 家族。
- [别尝试提出你自己喜欢的认证系统](http://stackoverflow.com/questions/1581610/how-can-i-store-my-users-passwords-safely/1581919#1581919)。这很容易在微妙且不可测的方式下出现错误，而且你可能直到被入侵才知道发生什么事
- 了解 [处理信用卡的规则](https://www.pcisecuritystandards.org/)。（[也可以看看这里这个问题](https://stackoverflow.com/questions/51094/payment-processors-what-do-i-need-to-know-if-i-want-to-accept-credit-cards-on-m)）
- 在登录页和任何涉及敏感数据的网页（如信用卡信息），使用 [SSL](http://www.mozilla.org/projects/security/pki/nss/ssl/draft302.txt) / [HTTPS](https://en.wikipedia.org/wiki/Https)。
- 防止 [会话（session）劫持](https://en.wikipedia.org/wiki/Session_hijacking#Prevention)。
- 避免 [跨站脚本攻击](https://en.wikipedia.org/wiki/Cross-site_scripting)（XSS）。
- 避免 [跨站请求伪造攻击](https://en.wikipedia.org/wiki/Cross-site_request_forgery)（CSRF）。
- 避免 [点击劫持](https://en.wikipedia.org/wiki/Clickjacking)。
- 系统补丁要保持更新。
- 保证数据库连接信息安全。
- 你自身要保持关注最新的攻击技术和影响你平台的漏洞。
- 阅读 [Google 的《浏览器安全手册》](http://code.google.com/p/browsersec/wiki/Main)。
- 阅读 [《Web应用黑客手册》](http://amzn.com/0470170778)。
- 考虑 [最小特权原则](https://en.wikipedia.org/wiki/Principle_of_least_privilege)。尝试将你的应用程序在 [非根模式（non-root）](https://security.stackexchange.com/questions/47576/do-simple-linux-servers-really-need-a-non-root-user-for-security-reasons)的服务器下运行。（[tomcat 案例](http://tomcat.apache.org/tomcat-8.0-doc/security-howto.html#Non-Tomcat_settings)）

性能
---
- 如有必要，就实现缓存。了解和正确地使用 HTTP 缓存（caching）和 HTML 5 离线缓存。
- 优化图片 – 别使用一个 20 KB 大小的图片做为一个重复背景。
- 学习如何用 [gzip / deflate 压缩内容](http://developer.yahoo.com/performance/rules.html#gzip)（[deflate更好](https://stackoverflow.com/questions/1574168/gzip-vs-deflate-zlib-revisited)）。
- 合并多个样式表单或脚本文件，以减少浏览器发送请求次数，而且要利用 gzip 压缩文件之间重复的部分。
- 浏览 [Yahoo Exceptional Performance](http://developer.yahoo.com/performance/)（雅虎优越性能）网站，里面有很多优秀的指引，其中就包括提高前端性能和它们的 [YSlow](http://developer.yahoo.com/yslow/) 工具（需要 Fixfox、Safari、Chrome 或 Opera）。另外，[Google PageSpeed](https://developers.google.com/speed/docs/best-practices/rules_intro) （以 浏览器扩展 的方式](https://developers.google.com/speed/pagespeed/insights_extensions)）是另一个测试性能的工具，而且它也会优化你的图片。
- 为较小且有关联的图片使用 [CSS 图片精灵](http://alistapart.com/articles/sprites) 技术，如工具栏（看“把 HTTP 请求减到最低”那点建议）
- 繁忙 Web 站点应考虑将 [网页的内容分开存放](http://developer.yahoo.com/performance/rules.html#split) 在不同的域名下。特别是…
- 静态内容（也就是图片、CSS、JavaScript 和无需通过 cookies 获取的一般内容）应放进独立且 [不使用 cookies](http://blog.stackoverflow.com/2009/08/a-few-speed-improvements/) 的域名上，因为所有域名和其子域名为客户端生成的 cookies 都会伴随请求发送回给自己。 一个很好的选择是使用内容分发网络（CDN），但要考虑到这种情况：CDN（包括可替代的 CDN）可能会失效，这时本地副本能代替它来进行传输。
- 将浏览器渲染页面所需 HTTP 请求数量最少化。
- 用Google的 [Closure Compiler](http://developers.google.com/closure/compiler/) 压缩 JavaScript，当然也可以使用 [其他压缩工具](http://developer.yahoo.com/yui/compressor/)。
- 确保有一个 favicon.ico 文件在网站的根目录，也就是说 /favicon.ico。[浏览器会自动请求它](http://mathiasbynens.be/notes/rel-shortcut-icon)，即使在 HTML 中并未提及到它。如果没有 /favicon.ico，那么请求返回的结果是 大量的 404 错误，这将会耗尽服务器的带宽。

SEO（搜索引擎优化）
---
- 使用“对搜索引擎友好”的链接，比如说 example.com/pages/45-article-title 优于 example.com/index.php?page=45。是 googlebot（Google 的 web 爬虫）用来替换 #! 的。换句话说，./#!page=1 会被Google搜索引擎转成 ./?_escaped_fragments_=page=1。 （通常来说 URL 中的 # 后的东西都不会被传到服务器上，所以，为了要让 Google 可以抓取 AJAX 的东西，你需要使用 #!，而 Google 会把“#!”转成“_escaped_fragment_”来向服务器发请求，Twitter 的大量链接者是#!的，比如：https://twitter.com/#!/your_activity —— 陈皓注）。另外，用户也许会使用 Firefox 或 Chromium，那么 history.pushState({"foo":"bar"}, "About", "./?page=1"); 是一个很不错的命令。因为即使地址栏上的地址改变了，页面也不会重新加载。这可让你使用 ? 而不是 #!来动态加载内容了，也告诉服务器，当下次访问该页面时给该链接发邮件，AJAX 无须再发送一个额外的请求了。
- 别使用 [“点击这里”](https://ux.stackexchange.com/questions/12100/why-shouldnt-we-use-the-word-here-in-a-textlink) 这类的链接。这是浪费一个 SEO 的机会，并且会对使用屏幕朗读器用户造成困惑。
- 拥有一个 [XML 网站地图](http://www.sitemaps.org/)，它的默认路径最好是 /sitemap.xml。
- 当你有多个 URL 指向同一个内容时，请使用 [<link rel="canonical" ... />](http://googlewebmastercentral.blogspot.com/2009/02/specify-your-canonical.html)。这个问题可利用 [Google Webmaster Tools](http://www.google.com/webmasters/) 解决。
- 使用 [Google Webmaster Tools](http://www.google.com/webmasters/) 和 [Bing Webmaster Tools](http://www.bing.com/toolbox/webmaster)。
- 在一开始就正确安装 [Google Analytics](http://www.google.com/analytics/) （或一个开源的分析工具，如 [Piwik](http://piwik.org/)）。
- 要知道 [robots.txt](https://en.wikipedia.org/wiki/Robots_exclusion_standard) 和搜索引擎爬虫是如何工作的。
- 重定向请求（使用 301 永久性移走），要求 www.example.com 重定向到 example.com （或反过来），从而防止分裂两个站点之间的谷歌排名。
- 知道并不是所有的爬虫都是好的，有些爬虫的行为并不好。
- 如果有非文本内容（如视频等）需要添加到 Google 网站地图的话，你可以到 [Tim Farley’s answer](http://stackoverflow.com/questions/72394/what-should-a-developer-know-before-building-a-public-web-site#167608) 看看，里面有一些关于这方面的，而且不错的信息。

技术
---
- 搞懂 HTTP 协议，以及诸如 GET 、POST、sessions 和 cookies 这些概念，而且要知道“无状态”是什么意思。
- 根据 [W3C 文档](http://www.w3.org/TR/) 编写你的 [XHTML](http://www.w3.org/TR/xhtml1/) / [HTML](http://www.w3.org/TR/REC-html40/) 和 [CSS](http://www.w3.org/TR/CSS2/) 代码，并确保它们 [有效](http://validator.w3.org/)。这里的目的是避免浏览器的怪异模式，并让它们更容易在非传统浏览器（如屏幕阅读器和移动设备）上运行。
- 搞懂浏览器是如何处理 JavaScript。
- 搞懂页面上的 JavaScript、样式表单和其他资源是如何加载和运行的，并考虑它们对性能的影响。现在广泛认同的做法是：除了通用脚本，如 analytics apps 或 HTML5 shims，[将其它脚本放到页面底部](http://developer.yahoo.com/blogs/ydn/posts/2007/07/high_performanc_5/)。
- 搞懂 JavaScript 沙箱如何工作，特别是你打算用 iframes。
- 要意识到 JavaScript 可能会被禁用，因此 AJAX 也只是一个扩展，不一定会被运行。即使大多数普通的用户并不会理会 JavaScript 被禁用，但要记住 [NoScript](http://noscript.net/) 正变得更流行，移动设备可能默认禁止 JavaScript，而且 Google 在索引你的网站时，并不会执行大多数 JavaScript。
- 学会区分 [301 和 302 重定向](http://www.bigoakinc.com/blog/when-to-use-a-301-vs-302-redirect/) 的不同之处（这也是一个 SEO 问题）。
- 尽可能多地学习你部署平台的相关知识。
- 考虑使用 [Reset Style Sheet](http://stackoverflow.com/questions/167531/is-it-ok-to-use-a-css-reset-stylesheet)（重置样式表单） 或 [normalize.css](http://necolas.github.com/normalize.css/)。
- 考虑使用 JavaScript 框架（如 [jQuery](http://jquery.com/)、[MooTools](http://mootools.net/)、[Prototype](http://www.prototypejs.org/)、[Dojo](http://dojotoolkit.org/) 或 [YUI 3](http://developer.yahoo.com/yui/3/)），它们会解决很多在使用 JavaScript 操作 DOM 时的浏览器差异问题。
- 把性能和 JS 框架合在一起讨论，考虑使用诸如 [Google Libraries API](http://developers.google.com/speed/libraries/devguide) 服务来加载框架， 以至于浏览器能使用已缓存框架的副本，而不是从你的网站下载同样的副本。
- 不要重复造轮子。在做任何事之前，可搜索一个组件或案例是如何实现的。但有 99% 机会是其它人已经做过了，并发布了 OSS 版本的代码。
- 另外，即时确定你需要的是什么，但也别使用太多库。特别是在 Web 客户端，保持轻量、快速和灵活非常重要。

BUG 修复
---
- 要明白你将花费 20% 时间敲代码，而剩下 80% 的时间是在维护你的代码，所以代码质量很重要。
- 建立一个良好的错误报告解决方案。
- 为用户提供一个能向你提交建议与批评的系统。
- 为将来的维护和技术支持人员撰写文档，解释清楚系统是怎么运行的。
- 经常备份！（并确保那些备份是可用的）除了备份机制，你还必须有一个恢复机制。
- 使用版本控制系统来存储你的文件，如 [Subversion](http://subversion.apache.org/)、[Mercurial](http://mercurial.selenic.com/) 或 [Git](http://git-scm.org/)。
- 别忘记进行验收测试。框架（如 [Selenium](http://seleniumhq.org/)）能为你提供相应帮助。特别是如果你想完全自动化测试，也可通过使用持续集成工具，比如 [Jenkins](http://jenkins-ci.org/)。
- 在网站运行时，要确保你有足够的日志，当然你可以使用框架，如 [log4j](http://logging.apache.org/log4j/)、[log4net](http://logging.apache.org/log4net/) 或 [log4r](http://log4r.rubyforge.org/)。因为当你的网站某部分发生错误，你将需要一种方式找出是哪里发生的。
- 当日志能确保你能同时捕捉到处理异常和未处理异常。那么可通过记录/分析输出的日志，可显示网站的关键问题出现在哪里。

其他
---
- 服务器端和客户端都要监控和分析（应主动而不是被动）。
- 使用能与用户保持联系的服务（如 UserVoice 和 Intercom，或其它类似的工具）。
- 采用 [Vincent Driessen](http://nvie.com/about/) 的 Git 分支模型（[Git branching model](http://nvie.com/posts/a-successful-git-branching-model/)）。

文中有很多省略掉的东西，并不是因为它们不是有用的答案，而是它们过于详细，且超出本问题的范围。而对于想懂得更多的人来说，他们希望学到更多的东西，因此他们应该知道这些概述。另外，我也欢迎大家编辑补充这个答案，因为我可能忽略了一些东西或犯了一些错误。



本文由 [伯乐在线](http://blog.jobbole.com/) - [刘健超-J.c](http://www.jobbole.com/members/q574805242) 翻译，黄利民 校稿。未经许可，禁止转载！
英文出处：[stackexchange](http://programmers.stackexchange.com/questions/46716/what-technical-details-should-a-programmer-of-a-web-application-consider-before)。

谢谢您的阅读。
