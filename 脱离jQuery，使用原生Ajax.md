# 脱离jQuery，使用原生Ajax 

标签： Ajax translate

---

英文出处：[《A Guide to Vanilla Ajax Without jQuery》](http://www.sitepoint.com/guide-vanilla-ajax-without-jquery/)

翻译： [刘健超 J.c](https://github.com/JChehe)

注意：未经许可，禁止转载！


----------


[Ajax](https://en.wikipedia.org/wiki/Ajax_%28programming%29) 是异步的JavaScript和XML的简称，是一种更新页面某部分的机制。它赋予了你从服务器获取数据后，更新页面某部分的权力，从而避免了刷新整个页面。另外，以此方式实现页面局部更新，不仅能有效地打造流畅的用户体验，而且减轻了服务器的负载。

下面是对一个基本的 Ajax 请求进行剖析：

    var xhr = new XMLHttpRequest();
    xhr.open('GET', 'send-ajax-data.php');
    xhr.send(null);

在这里， 我们创建了一个能向服务器发出 HTTP 请求的类的实例。然后调用其 `open` 方法，其中第一个参数是 HTTP 请求方法，第二个参数是请求页面的 URL。最后，我们调用参数为 null 的 `send` 方法。假如使用 POST 请求方法（这里我们使用了 GET），那么 send 方法 的参数应该包含任何你想发送的数据。

下面是我们如何处理服务器的响应：

    xhr.onreadystatechange = function(){
        var DONE = 4; // readyState 4 代表已向服务器发送请求
        var OK = 200; // status 200 代表服务器返回成功
        if(xhr.readyState === DONE){
            if(xhr.status === OK){
                console.log(xhr.responseText); // 这是返回的文本
            } else{
                console.log("Error: "+ xhr.status); // 在此次请求中发生的错误
            }
        }
    }

`onreadystatechange` 是异步的，那么这就意味着它将可在任何时候被调用。这种类型的函数被称为回调函数——一旦某些处理完成后，它就会被调用。在此案例中，这个处理发生在服务器。

对于想学习更多关于 Ajax 基础知识的同学，可关注 MDN 的这篇[教程](https://developer.mozilla.org/en-US/docs/AJAX/Getting_Started)。

###到底选择 jQuery 还是选择原生 JavaScript 呢？
嗯，好消息是上述代码兼容所有最新的主流浏览器。而坏消息是使用起来十分复杂。的确令人恶心！我已经苦思出一个优雅的解决方案了。

如果使用 jQuery，则能把上述代码压缩成这样：

    $.ajax({
        url: "send-ajax-data.php"
    }).done(function(res){
        console.log(res);
    }).fail(function(){
        console.log("Error: " + err.status);
    })

非常简洁易懂。对于大多数人（我想也包括你）来说，jQuery 已经成为了解决 Ajax 的默认标准。但你知道吗？情况不一定是这样的。jQuery 的存在是为了解决丑陋的 DOM API。但 Ajax 真的是丑陋或复杂的吗？

在文章的剩余部分，我打算用原生 JavaScript 使 Ajax API 有所改善。关于 Ajax 的完整文档可以在 [W3C](http://www.w3.org/TR/XMLHttpRequest/) 找到。然而这份说明的标题使我非常受打击。竟然不是“XMLHttpRequest Level 2”，而是“XMLHttpRequest Level 1”——因为在2011年已将两个文档合并。展望未来，它们将被视为单一实体，而现存标准将其称为 [XMLHttpRequest](https://xhr.spec.whatwg.org/)。这表明社区坚持承诺只有一个标准，这对于想脱离 jQuery 的开发人员来说，是个好消息。

所以，让我们一起开始吧！

在这篇文章，我使用 [Node.js](https://nodejs.org/)作为后端。没错，这就可以全栈（浏览器和服务器）JS了。Node.js 是很简洁的，我鼓励你能在 [Github](https://github.com/sitepoint-editors/VanillaAjaxNojQuery)下载demo，并关注该项目。下面是服务器端的代码：
    
    // app.js
    var app = http.createServer(function(req, res){
        if(req.url.indexOf("/scripts/") >= 0){
            render(req.url.slice(1), "application/javascript", httpHandler);
        } else if(req.headers['x-requested-with'] === 'XMLHttpRequest'){
            // Send Ajax response
        } else{
            render('views/index.html', 'text/html', httpHandler);
        }
    }
该代码段通过检测请求 URL，确定该app返回的相应内容。如果该请求来自 `scripts` 目录，那么服务器将返回内容类型（content type）为 `application/javascript` 的相应文件。如果请求头部的 `x-requested-with` 被设为 `XMLHttpRequest`，那么该请求是 Ajax 请求，然后返回相应数据。除了以上两种情况，服务器将会返回 `views/index.html`。

下面我将会展开上一代码段处理 Ajax 请求的注释部分进行深入讲解。在 Node.js端，我已处理了 `render` 和 `httpHandler` 的体力活：
    
    // app.js
    function render(path, contentType, fn) {
        fs.readFile(__dirname + '/' + path, 'utf-8', function (err, str) {
            fn(err, str, contentType);
        });
    }
    var httpHandler = function (err, str, contentType) {
        if (err) {
            res.writeHead(500, {'Content-Type': 'text/plain'});
            res.end('An error has occured: ' + err.message);
        } else {
            res.writeHead(200, {'Content-Type': contentType});
            res.end(str);
        }
    };
`render` 函数异步读取被请求文件的内容。该函数向被作为回调函数的 `httpHandler` 传递一个引用。
`httpHandler` 函数检测 error 对象是否存在（如：被请求文件不能被打开，该对象就会存在）。另外，指定类型是好的做法，那么服务器返回的文件内容就会拥有适当的 HTTP 状态码（status code）和内容类型（content type）。

###测试 API
让我们为后端API编写一些单元测试，从而确保它们能正确运行。对于这类测试，我会请求 [supertest](https://www.npmjs.com/package/supertest) 和 [mocha](https://www.npmjs.com/package/mocha)帮助。

    // test/app.request.js
    it("responds with html", function(done){
        request(app)
            .get("/")
            .expect("Content-Type", /html/)
            .expect(200, done);
    });
    it('responds with javascript', function (done) {
    request(app)
        .get('/scripts/index.js')
        .expect('Content-Type', /javascript/)
        .expect(200, done);
    });
    it('responds with json', function (done) {
    request(app)
        .get('/')
        .set('X-Requested-With', 'XMLHttpRequest')
        .expect('Content-Type', /json/)
        .expect(200, done);
    });
这些测试确保了我们的 app 对于不同请求能返回正确的内容类型(content type)和HTTP 状态码（status code）。一旦你安装了这些依赖，那么你就能使用命令 `npm test` 运行这些测试。

###界面
现在，让我们看看用户界面的 HTML 代码：

    // views/index.html
    <h1>Vanilla Ajax without jQuery</h1>
    <button id="retrieve" data-url="/">Retrieve</button>
    <p id="results"></p>
上述的 HTML 代码看起来很简洁。没错，正如你所看到的，所有令人兴奋的事情都发生在 JavaScript。

###onreadystate vs onload
如果你看过任何一本权威的、关于 Ajax 的书，你可能会发现 `onreadystate` 在书上随处可见。该回调函数需要通过嵌套的 ifs 或多个 case 语句完成，这使得难以记忆。让我们再次回顾 `onreadystate` 和  `onload` 事件。

    (function() {
        var retrieve = document.getElementById('retrieve'),
            results = document.getElementById('results'),
            toReadyStateDescription = function(state) {
                switch (state) {
                    case 0:
                        return 'UNSENT';
                    case 1:
                        return 'OPENED';
                    case 2:
                        return 'HEADERS_RECEIVED';
                    case 3:
                        return 'LOADING';
                    case 4:
                        return 'DONE';
                    default:
                        return '';
                }
            };
        retrieve.addEventListener('click', function(e) {
            var oReq = new XMLHttpRequest();
            oReq.onload = function() {
                console.log('Inside the onload event');
            };
            oReq.onreadystatechange = function() {
                console.log('Inside the onreadystatechange ev![此处输入图片的描述][1]ent with readyState: ' +
                    toReadyStateDescription(oReq.readyState));
            };
            oReq.open('GET', e.target.dataset.url, true);
            oReq.send();
        });
    }());

上述代码会在 控制台（console） 输出以下语句：

![此处输入图片的描述][2]

 `onreadystatechange` 事件能在请求的任何过程中被触发。如能在每个请求前、请求末。但根据文档，`onload` 事件只会在请求成功后触发。又因为 `onload` 事件是一个常见的 API，所以你能在很短时间内掌握它。`onreadystatechange` 事件可作为后备（原文是backwards compatible 向后兼容？）方案。而 `onload` 事件应该是你的首选。而且 `onload` 事件与 jQuery 的 `success` 回调函数类似，难道不是吗？
 
 ###设置请求头部
 jQuery 私下帮你设置请求头部了，所以后端能检测这是一个 Ajax 请求。一般来说，后端并不关心 GET 请求是从哪而来，只要能返回正确的响应即可。当你相用同样的 web API 返回 Ajax 或 HTML 时，这就派上用场了。让我们看看如何通过原生 JavaScript 设置请求头部：
 
    var oReq = new XMLHttpRequest();
    oReq.open('GET', e.target.dataset.url, true);
    oReq.setRequestHeader('X-Requested-With', 'XMLHttpRequest');
    oReq.send();
    
与此同时，我们在 Node.js 做一个检测：
    
    if (req.headers['x-requested-with'] === 'XMLHttpRequest') {
        res.writeHead(200, {'Content-Type': 'application/json'});
        res.end(JSON.stringify({message: 'Hello World!'}));
    }

正如你所看到的，原生 Ajax 是一个灵活且现代化的前端 API。你可以利用请求头部做很多事情，其中一种是版本控制。例如，我想让某个 web API 支持多个版本。但我又不想利用 URL，取而代之的是：通过设置请求头部，使客户端能选择它们想要的版本。所以，我们能这样设置请求头部：

    oReq.setRequestHeader('x-vanillaAjaxWithoutjQuery-version', '1.0');
    
然后，在后端写上相应代码：
    
    if (req.headers['x-requested-with'] === 'XMLHttpRequest' &&
        req.headers['x-vanillaajaxwithoutjquery-version'] === '1.0') {
        // Send Ajax response
    }

我们能利用 Node.js 为我们提供的 headers 对象进行相应检测。而唯一需要注意的地方是：以小写字母读取它们。

###响应类型
你可能想知道为什么 `responseText` 返回的是字符串，而不是能被我们操作的普通 JSON（Plain Old JSON）。原来是因为我没有设置合适的 `responseType` 属性。该 Ajax 属性会很好地告诉前端 API 所期望服务器返回的数据类型。所以，我们要好好利用它：

    var oReq = new XMLHttpRequest();
    oReq.onload = function (e) {
        results.innerHTML = e.target.response.message;
    };
    oReq.open('GET', e.target.dataset.url, true);
    oReq.responseType = 'json';
    oReq.send();
    
哇，这样我们就不必再对返回的纯文本解析为 JSON 了，我们能告诉 API 我们期待接收的数据类型。该特性几乎得到了所有最新主流浏览器的支持。当然，jQuery 会自动帮我们转为适当的类型。但现在的原生 JavaScript 也具有方便的、完成同样事件的方法。 原生 Ajax 已经支持很多其它响应类型，如 XML。

但遗憾的是，到 IE11 为止，开发团队仍未对 [xhr.responseType='json'](https://connect.microsoft.com/IE/feedback/details/794808) 进行支持。虽然该特性目前在 [Microsoft Edge](http://caniuse.com/#feat=xhr2) 得到支持。但这个 Bug 提出几乎两年了。我坚信 Microsoft 团队一直在努力地改进浏览器。让我们期待 Microsoft Edge、aka Project Spartan 当初提出的承诺。
当然，你可以这个解决这个 IE 问题：

    oReq.onload = function (e) {
        var xhr = e.target;
        if (xhr.responseType === 'json') {
            results.innerHTML = xhr.response.message;
        } else {
            results.innerHTML = JSON.parse(xhr.responseText).message;
        }
    };
    
###避免缓存
对 Ajax 请求进行缓存的浏览器特性都快被我们忘记了。例如，IE 就默认是这样。我还曾因此导致我的 Ajax 不执行而苦恼了几个小时。幸运的是，jQuery 默认清除浏览器缓存。当然，你能在纯 Ajax 达到该目的，而且相当简单：

    var bustCache = '?' + new Date().getTime();
    oReq.open('GET', e.target.dataset.url + bustCache, true);
    
查看 [jQuery 文档](http://api.jquery.com/jQuery.ajax/#jQuery-ajax-settings)，可知道 jQuery 在每个请求（GET）后面追加一个时间戳作为查询字符串。这在某个程度上让请求变得独一无二，并避免浏览器缓存。每当你触发 HTTP Ajax 请求，你能看到类似如下请求：
![此处输入图片的描述][3]

OK！这就没有戏剧性的事情发生了。

###总结
我希望你能喜欢这篇关于原生 Ajax 的文章。Ajax 在过去某段时间里，被人们看作是一种可怕的怪兽。而事实上，我们已经覆盖了原生 Ajax 所有基础知识。


最后，我会留给你一个简洁的方式进行Ajax调用：
    
    var oReq = new XMLHttpRequest();
    oReq.onload = function (e) {
        results.innerHTML = e.target.response.message;
    };
    oReq.open('GET', e.target.dataset.url + '?' + new Date().getTime(), true);
    oReq.responseType = 'json';
    oReq.send();

不要忘记，你能在 [Github](https://github.com/sitepoint-editors/VanillaAjaxNojQuery) 找到整个案例。我希望在评论里看到你对原生 Ajax 的想法。


----------


感谢您的阅读。

  [2]: http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/08/1439883041onreadystate_vs_onload.jpg
  [3]: http://dab1nmslvvntp.cloudfront.net/wp-content/uploads/2015/08/1439883644cache_busting_vanilla_ajax.jpg
