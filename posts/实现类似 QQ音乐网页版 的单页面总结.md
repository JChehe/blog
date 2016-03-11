# 实现类似 QQ音乐网页版 的单页面总结

标签： iframe javascript hash

---

最近需要对创业团队的网站进行改版，而我负责前端设计和实现。  
下面是一些总结与体会：    
当设计完成之前，我就跟和我配合的Java 后台说用iframe实现，结果说麻烦不肯，到最后突然对我说还是用iframe吧，说他以前也用过，很简单--！...其实我之间也基本没用iframe，对它比较陌生，但是 QQmusic 网页版就是用iframe 做的，印象比较深刻！  

我设计的页面总体结构是与QQmusic网页版类似，网页头部和脚部都是固定，中间内容是通过iframe来展示。

用iframe之前，我想到要解决的问题有：

 1. iframe虽然有height这个属性，但是每次加载到里面的内容都是不同的，而且我实现的部分页面是需要动态加载能容的。
    我的解决办法是：
    iframe 首次加载成功后和 `iframe` 页面内有动态增加内容时（导致整体高度有变化）都调用下面这个函数，来设置 `iframe` 高度

        function iframeLoaded() {
            var contentFrame = document.getElementById('contentFrame');
            if(contentFrame) {
                contentFrame.height = contentFrame.contentWindow.document.documentElement.scrollHeight + "px";
            }
        }
    

 2. 因为是做成像QQmusic网页版的单页面，所以我也利用hash变化来欺骗浏览器，让浏览器为我的页面跳转生成历史记录，但是后来发现，每次点击链接加载不用iframe时，都会生成两条相同的历史记录。这是为什么呢？后来发现，hash值改变会导致一次，iframe的src改变也会生成一次历史记录(包括hash的改变)。然后我又去看看QQmusic 是怎么实现只产生一次历史记录的，惊奇的发现，它的iframe的src一直没变（一直是 about:blank）。

        <iframe name="contentFrame" id="contentFrame" width="100%" height="2207px" allowtransparency="true" scrolling="no" border="0" frameborder="0" src="about:blank"></iframe>
        
    因此发现我之前对iframe有些误解，之前认为iframe只能通过src来获取内容，其实个人认为正确的是：iframe中的内容不一定是通过设置src后获取的，也可以是通过其它方式（例如通过ajax获取html后替换原视图而成）；所以对于QQmusic来说，iframe只是一块渲染视图的容器，它里面显示的内容是由另外的逻辑来控制的。这样无须改变iframe的src（也就不会产生历史记录），从而可以动态修改iframe的内容了。如：
    
        window.onload = function(){
            var iframe = document.getElementById("iframe")
            var newDiv = document.createElement("div")
            newDiv.innerHTML = "我要跑到iframe里"
            iframe.contentWindow.document.body.appendChild(newDiv)
        }
    
    对于整体页面（用jQuery发出异步请求）
    
        $.ajax({
            url: "原本iframe的src值",    // 改为异步请求加载
            type: "GET"
        })
        .done(function(data){
             var iframe = document.getElementById("iframe")
             iframe.contentWindow.document.documentElement.innerHTML = data;  // 获取后直接插到iframe
        })

    所以，iframe的内容一直在改变，而它的src却一直是about:blank没变。因此也不会产生新的历史记录。
此时此刻，我心情是很开心的，然而，当我与iframe内的元素交互时，发现除了 CSS 的交互效果，<script src=""></script>和写在<script></script>内的js代码都是不执行的（测试后发现直接添加到元素的onclick事件可以执行），而且在测试的过程中发现有时候CSS加载比较慢，导致HTML裸露出来，不知道为什么。这个要继续查查。

    所以，最后我不采用通过ajax获取内容，再添加到iframe的做法。  
而是采用移除原来iframe，再新建一个iframe插入的方式，这种方式也是不会产生新的历史记录。

    解决了产生两条历史记录的问题后，剩下的问题就是iframe内的链接跳转，因为整体是通过主页面的hash值来路由的，所以iframe内的链接跳转是不能直接跳转的，他的跳转需要一定处理的流程：
    
    处理由iframe内发送的请求：
      ①首先接收iframe的参数
      ②根据参数生成对应的hash，并修改main_frame 的hash值
      ③根据hash值，修改iframe的src（我是通过删除原来的iframe，再添加新iframe的做法）
    
    如：主页面的全局有一个接收iframe内a标签的data-hash的函数:
    
        function handleIframeRequst( iframeHash){
            window.location.hash = iframeHash;
        }

    假设iframe内的一个a标签 ： <a href="javascript:;" data-hash="main&sub"></a>
点击这个标签时，就会调用主页面的一个处理该data-hash的函数
parent.handleIframeRequst($(this).data("hash")); // 通过parent调用主页面的函数
 
    主页面接收参数后，修改自身的hash值，然后解析hash值，生成相应的 iframe src值。
 
    这样基本就可以将网站做成 类似 QQmusic 那样的单页面网站啦。 满足感 Up！Up！Up！
