# 关于JavaScript单线程的一些事

标签（空格分隔）： JavaScript 单线程

---

最近被同学问道 JavaScript 单线程的一些事，我竟回答不上。好吧，感觉自己的 JavaScript 白学了。下面是我这几天整理的一些关于 JavaScript 单线程的一些事。

##首先，说下为什么 JavaScript 是单线程？
总所周知，JavaScript是以单线程的方式运行的。说到线程就自然联想到进程。那它们有什么联系呢？
    
> 进程和线程都是操作系统的概念。进程是应用程序的执行实例，每一个进程都是由私有的虚拟地址空间、代码、数据和其它系统资源所组成；进程在运行过程中能够申请创建和使用系统资源（如独立的内存区域等），这些资源也会随着进程的终止而被销毁。而线程则是进程内的一个独立执行单元，在不同的线程之间是可以共享进程资源的，所以在多线程的情况下，需要特别注意对临界资源的访问控制。在系统创建进程之后就开始启动执行进程的主线程，而进程的生命周期和这个主线程的生命周期一致，主线程的退出也就意味着进程的终止和销毁。主线程是由系统进程所创建的，同时用户也可以自主创建其它线程，这一系列的线程都会并发地运行于同一个进程中。

显然，在多线程操作下可以实现应用的**并行处理**，从而以更高的CPU利用率提高整个应用程序的性能和吞吐量。特别是现在很多语言都支持多核并行处理技术，然而JavaScript却以单线程执行，为什么呢？

其实这与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。若以多线程的方式操作这些DOM，则可能出现操作的冲突。假设有两个线程同时操作一个DOM元素，线程1要求浏览器删除DOM，而线程2却要求修改DOM样式，这时浏览器就无法决定采用哪个线程的操作。当然，我们可以为浏览器引入“锁”的机制来解决这些冲突，但这会大大提高复杂性，所以 JavaScript 从诞生开始就选择了单线程执行。

另外，因为 JavaScript 是单线程的，在某一时刻内只能执行特定的一个任务，并且会阻塞其它任务执行。那么对于类似I/O等耗时的任务，就没必要等待他们执行完后才继续后面的操作。在这些任务完成前，JavaScript完全可以往下执行其他操作，当这些耗时的任务完成后则以回调的方式执行相应处理。这些就是JavaScript与生俱来的特性：异步与回调。

当然对于不可避免的耗时操作（如：繁重的运算，多重循环），HTML5提出了**Web Worker**，它会在当前JavaScript的执行主线程中利用Worker类新开辟一个额外的线程来加载和运行特定的JavaScript文件，这个新的线程和JavaScript的主线程之间并不会互相影响和阻塞执行，而且在Web Worker中提供了这个新线程和JavaScript主线程之间数据交换的接口：postMessage和onMessage事件。但在HTML5 Web Worker中是不能操作DOM的，任何需要操作DOM的任务都需要委托给JavaScript主线程来执行，所以虽然引入HTML5 Web Worker，但仍然没有改线JavaScript单线程的本质。

##并发模式与Event Loop

**JavaScript 有个基于“Event Loop”并发的模型。**
啊，并发？不是说 JavaScript是单线程吗？ 没错，的确是单线程，但是并发与并行是有区别的。
前者是逻辑上的同时发生，而后者是物理上的同时发生。所以，单核处理器也能实现并发。

![并发与并行][1]
并发与并行

并行大家都好理解，而**所谓“并发”是指两个或两个以上的事件在同一时间间隔中发生。**如上图的第一个表，由于计算机系统只有一个CPU，故ABC三个程序从“微观”上是交替使用CPU，但交替时间很短，用户察觉不到，形成了“宏观”意义上的并发操作。


###Runtime 概念
下面的内容解释一个理论上的模型。现代 JavaScript 引擎已着重实现和优化了以下所描述的几个概念。

![Stack、Heap、Queue][2]

####Stack（栈）
这里放着JavaScript正在执行的任务。每个任务被称为帧（stack of frames）。

    function f(b){
      var a = 12;
      return a+b+35;
    }
    
    function g(x){
      var m = 4;
      return f(m*x);
    }
    
    g(21);

上述代码调用 `g` 时，创建栈的第一帧，该帧包含了 `g` 的参数和局部变量。当 `g` 调用 `f` 时，第二帧就会被创建，并且置于第一帧之上，当然，该帧也包含了 `f` 的参数和局部变量。当 `f` 返回时，其对应的帧就会出栈。同理，当 `g` 返回时，栈就为空了（**栈的特定就是后进先出** Last-in first-out (LIFO)）。

####Heap（堆）
一个用来表示内存中一大片非结构化区域的名字，对象都被分配在这。

####Queue（队列）
一个 JavaScript runtime 包含了一个任务队列，该队列是由一系列待处理的任务组成。而每个任务都有相对应的函数。当栈为空时，就会从任务队列中取出一个任务，并处理之。该处理会调用与该任务相关联的一系列函数（因此会创建一个初始栈帧）。当该任务处理完毕后，栈就会再次为空。**（Queue的特点是先进先出 First-in First-out (FIFO)）。**

为了方便描述与理解，作出以下约定：

 - Stack栈为**主线程**
 - Queue队列为**任务队列（等待调度到主线程执行）**

OK，上述知识点帮助我们理清了一个 JavaScript runtime 的相关概念，这有助于接下来的分析。

###Event Loop
之所以被称为Event loop，是因为它以以下类似方式实现：

    while(queue.waitForMessage()){
      queue.processNextMessage();
    }
    
正如上述所说，“任务队列”是一个事件的队列，如果I/O设备完成任务或用户触发事件（该事件指定了回调函数），那么相关事件处理函数就会进入“任务队列”，当主线程空闲时，就会调度“任务队列”里第一个待处理任务，（FIFO）。当然，对于定时器，当到达其指定时间时，才会把相应任务插到“任务队列”尾部。

####“执行至完成”
每当某个任务执行完后，其它任务才会被执行。也就是说，当一个函数运行时，它不能被取代且会在其它代码运行前先完成。
当然，这也是Event Loop的一个**缺点**：当一个任务完成时间过长，那么应用就不能及时处理用户的交互（如点击事件），甚至导致该应用奔溃。一个比较好解决方案是：将任务完成时间缩短，或者尽可能将一个任务分成多个任务执行。

####绝不阻塞
JavaScript与其它语言不同，其Event Loop的一个特性是永不阻塞。I/O操作通常是通过事件和回调函数处理。所以，当应用等待 indexedDB 或 XHR 异步请求返回时，其仍能处理其它操作（如用户输入）。

例外是存在的，如alert或者同步XHR，但避免它们被认为是最佳实践。注意的是，例外的例外也是存在的（但通常是实现错误而非其它原因)。

###定时器
####定时器的一些概念
上面也提到，在到达指定时间时，定时器就会将相应回调函数插入“任务队列”尾部。这就是“定时器(timer)”功能。

[定时器](http://dev.w3.org/html5/spec-preview/timers.html) 包括setTimeout与setInterval两个方法。它们的第二个参数是指定其回调函数推迟\每隔多少毫秒数后执行。
对于第二个参数有以下需要注意的地方：

 - 当第二个参数缺省时，默认为0；
 - 当指定的值小于4毫秒，则增加到4ms（4ms是HTML5标准指定的，对于2010年及之前的浏览器则是10ms）；

如果你理解上述知识，那么以下代码就应该对你没什么问题了：

    console.log(1);
    setTimeout(function(){
        console.log(2);
    },10);
    console.log(3);
    // 输出：1 3 2

####深入了解定时器
#####零延迟 setTimeout(func, 0)
零延迟并不是意味着回调函数立刻执行。它取决于主线程当前是否空闲与“任务队列”里其前面正在等待的任务。

看看以下代码：
    
    (function () {

      console.log('this is the start');
    
      setTimeout(function cb() {
        console.log('this is a msg from call back');
      });
    
      console.log('this is just a message');
    
      setTimeout(function cb1() {
        console.log('this is a msg from call back1');
      }, 0);
    
      console.log('this is the  end');
    
    })();
    
    // 输出如下：
    this is the start
    this is just a message
    this is the end
    undefined // 立即调用函数的返回值
    this is a msg from callback
    this is a msg from a callback1

#####**setTimeout(func, 0)的作用**

 - 让浏览器渲染当前的变化（很多浏览器UI render和js执行是放在一个线程中，线程阻塞会导致界面无法更新渲染） 
 - 重新评估”scriptis running too long”警告
 - 改变执行顺序

再看看以下代码：

    <button id='do'> Do long calc!</button>
    <div id='status'></div>
    <div id='result'></div>
     
     
    $('#do').on('click', function(){
      
      $('#status').text('calculating....');// 此处会触发redraw事件，但会放到队列里执行，直到long()执行完。
      
      // 没设定定时器，用户将无法看到“calculating...”
      long();// 执行长时间任务，造成阻塞
       
      // 设定了定时器，用户就如期看到“calculating...”
      //setTimeout(long,50);// 大约50ms后，将耗时长的long回调函数插入“任务队列”末尾，根据先进先出原则，其将在redraw之后被调度到主线程执行
      
     });
      
    function long(){
      var result = 0
      for (var i = 0; i<1000; i++){
        for (var j = 0; j<1000; j++){
          for (var k = 0; k<1000; k++){
            result = result + i+j+k
          }
        } 
      }
      $('#status').text('calclation done'); // 在本案例中，该语句必须放到这里，这将使它与回调函数的行为类似
    }

######**正版与翻版setInterval的区别**
大家都可能知道通过setTimeout可以模仿setInterval的效果，下面我们看看以下代码的区别：

    // 利用setTimeout模仿setInterval
    setTimeout(function(){
        /* 执行一些操作. */
        setTimeout(arguments.callee, 10);
    }, 1000);
    
    setInterval(function(){
        /* 执行一些操作 */
    }, 1000);

可能你认为这没什么区别。的确，当回调函数里的操作耗时很短时，并不能看出它们有什么区别。
其实：上面案例中的 setTimeout 总是会在其回调函数执行后延迟 10ms（或者更多，但不可能少）再次执行回调函数，从而实现setInterval的效果，而 setInterval 总是 10ms 执行一次，而不管它的回调函数执行多久。

所以，如果 setInterval 的回调函数执行时间比你指定的间隔时间相等或者更长，那么其回调函数会连在一起执行。

你可以试试运行以下代码：

    var counter = 0;
	var initTime = new Date().getTime();
	var timer = setInterval(function(){
		if(counter===2){
			clearInterval(timer);
		}
		if(counter === 0){
			for(var i = 0; i < 1990000000; i++){
				;
			}
		}

		console.log("第"+counter+"次：" + (new Date().getTime() - initTime) + " ms");

		counter++;
	},1000);
	
我电脑Chrome浏览器的输入如下：

    第0次：2007 ms
    第1次：2013 ms
    第2次：3008 ms

###浏览器
####浏览器不是单线程的
上面说了这么多关于JavaScript是单线程的，下面说说其宿主环境——浏览器。
**浏览器的内核是多线程**的，它们在内核制控下相互配合以保持同步，一个浏览器至少实现三个常驻线程：

 1. javascript引擎线程 javascript引擎是基于事件驱动单线程执行的，JS引擎一直等待着任务队列中任务的到来，然后加以处理，浏览器无论什么时候都只有一个JS线程在运行JS程序。
 2. GUI渲染线程 GUI渲染线程负责渲染浏览器界面，当界面需要重绘（Repaint）或由于某种操作引发回流(reflow)时,该线程就会执行。但需要注意GUI渲染线程与JS引擎是互斥的，当JS引擎执行时GUI线程会被挂起，GUI更新会被保存在一个队列中等到JS引擎空闲时立即被执行。
 3. 浏览器事件触发线程 事件触发线程，当一个事件被触发时该线程会把事件添加到“任务队列”的队尾，等待JS引擎的处理。这些事件可来自JavaScript引擎当前执行的代码块如setTimeOut、也可来自浏览器内核的其他线程如鼠标点击、AJAX异步请求等，但由于JS是单线程执行的，所有这些事件都得排队等待JS引擎处理。

在Chrome浏览器中，为了防止因一个标签页奔溃而影响整个浏览器，其每个标签页都是一个**进程**。当然，对于同一域名下的标签页是能够相互通讯的，具体可看 [浏览器跨标签通讯](http://web.jobbole.com/82225/)。在Chrome设计中存在很多的进程，并利用进程间通讯来完成它们之间的同步，因此这也是Chrome快速的法宝之一。对于Ajax的请求也需要特殊线程来执行，当需要发送一个Ajax请求时，浏览器会开辟一个新的线程来执行HTTP的请求，它并不会阻塞JavaScript线程的执行，当HTTP请求状态变更时，相应事件会被作为回调放入到“任务队列”中等待被执行。

看看以下代码：

    document.onclick = function(){
		console.log("click")
	}

	for(var i = 0; i< 100000000; i++);

解释一下代码：首先向document注册了一个click事件，然后就执行了一段耗时的for循环，在这段for循环结束前，你可以尝试点击页面。当耗时操作结束后，console控制台就会输出之前点击事件的"click"语句。这视乎证明了点击事件（也包括其它各种事件）是由额外单独的线程触发的，事件触发后就会将回调函数放进了“任务队列”的末尾，等待着JavaScript主线程的执行。

##总结

 - JavaScript是单线程的，同一时刻只能执行特定的任务。而浏览器是多线程的。
 - 异步任务（各种浏览器事件、定时器等）都是先添加到“任务队列”（定时器则到达其指定参数时）。当Stack栈（JS主线程）为空时，就会读取Queue队列（任务队列）的第一个任务（队首），然后执行。


JavaScript为了避免复杂性，而实现单线程执行。而今JavaScript却变得越来越不简单了，当然这也是JavaScript迷人的地方。

##参考资料：

 1. [JavaScript 运行机制详解：再谈Event Loop](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)
 2. [JavaScript单线程和浏览器事件循环简述](http://www.cnblogs.com/whitewolf/p/javascript-single-thread-and-browser-event-loop.html)
 3. [Javascript是单线程的深入分析](http://www.cnblogs.com/Mainz/p/3552717.html)
 4. [Concurrency model and Event Loop](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop)
 5. [也谈setTimeout](http://imweb.io/topic/56642e21d91952db73b41f52)
 6. [单线程的Javascript](https://leohxj.gitbooks.io/front-end-database/content/theory/single-thread.html)


  [1]: http://images.51cto.com/files/uploadimg/20090804/1503300.jpg
  [2]: https://developer.mozilla.org/files/4617/default.svg