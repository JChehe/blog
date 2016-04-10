# white-space:nowrap 的妙用

标签： CSS white-space

---

对于多个元素同在同一行的布局，如比较常见的是轮播。下面我将探讨这这一布局的做法：  
首先约定`html`结果如下：
	
    div.row
      div.col
      div.col
      div.col
      ...

### 做法一：
设定`div.row`的宽度为`div.col宽度*div.col的个数`，然后设置`div.col`为`float:left`或`display:inline-block`
对于 `float:left`, `div.row` 需要清除浮动。  
对于 `display:inline-block`，需要压缩html或者为`div.row`设置 `font-size:0` 以去除 `div.col` 之间的水平间隙，后者也顺便去除了垂直方向的间隙（line-height为相对单位时，其最终值为line-height值*font-size）。对于前者，还有垂直方面的间隙未去除，我们可以为`div.col`设置 `vertical-align:top` 或为`div.row`设置 `line-height:0`。推荐前者（即vertical-align），因为当 `div.col` 高度不相等时，会按顶部对齐，当然你也可以`bottom`或`middle`。而且，如果`div.col`内含有行内元素或inline-block元素时，`div.col`会按其子元素最后一行`inline/inline-block`元素的基线进行垂直方向上的对齐（vertical-align默认值是baseline）。因此最好显式设置`div.col`的垂直方向上的对齐。
![baseline基线][1]   
baseline基线

![水平与竖直方向上的间隙][2]    
水平与竖直方向上的间隙

![按其子元素最后一行inline/inline-block元素的基线进行垂直方向上的对齐][3]
按其子元素最后一行`inline/inline-block`元素的基线进行垂直方向上的对齐
> 这也符合张鑫旭老师的《[CSS深入理解vertical-align和line-height的基友关系][4]》这篇文章讲到的：一个inline-block元素，如果里面没有inline内联元素，或者overflow不是visible，则该元素的基线就是其margin底边缘，否则，其基线就是元素里面最后一行内联元素的基线。


补充知识：line-height的值为**数字**与**百分比**的区别
> 百分比是当前元素的字体大小`*`百分比，算出的值让后代元素继承（固定值，后代元素均用此值）；而数字是让后代元素的字体大小`*`数值（相对值，后代元素根据自身字体大小算出适合的行高）。具体可以看 《[深入了解css的行高Line Height属性][5]》。

当然，如果`div.row`内有行内元素或inline-block元素，它们会继承父元素的font-size与line-height，因此需要重新设置font-size和line-height，以覆盖`div.row`对应的值。

做法一的案例有：淘宝首页的主轮播（通过子元素浮动，父元素清除浮动）。
这种做法的好处有：①兼容性好，无须清除`div.col`直接的间隙，因为浮动后的元素会一直与当前行框（line box）顶部对齐，vertical-align对齐无效。
不好的地方：要计算`div.row`的宽度。


### 做法二（这也是我想讲的巧妙）
直接上代码：

    *{
		margin: 0;
		padding: 0;
	}
	.row{
		white-space: nowrap; // 让div.col放置在同一行
		background-color: rgb(0,0,0); // 背景色，以方便观察
		font-size: 0; // 去除水平+垂直间隙
	}
	.col{
		display: inline-block;
		*display: inline; // 兼容IE 6/7，模拟inline-block效果
		*zoom: 1; // 兼容IE 6/7，模拟inline-block效果
		width: 20%; 
		margin-right: 30px;
		height: 100px;
		background-color: red;
		font-size: 14px; // 覆盖父元素的font-size
		vertical-align: top; // 向上对齐，同时去除垂直间隙
	}

![此处输入图片的描述][6]  
黑色背景是`div.row`，红色背景是 `div.col`。

可看出这与与应用了`white-space:nowrap`的文本容器效果一样。

####做法二的好处有：
①兼容性好（IE5都正常），无须计算宽度，可任意放多个 `div.col`，而 `div.row` 的宽度等于其父元素的宽度（但IE6则会将div.row撑大，在IE6中，`width`如同`min-width`效果，`height`也是）。
![IE5/6效果][7]  
IE5上的效果，IE6应该也一样。

如果还有其它想法，欢迎大家在评论处告诉我哦。

[github-JChehe][8]。
[静态博客][9] <- 小心这心机婊
    


  [1]: http://7xq7nb.com1.z0.glb.clouddn.com/nowrap-baseline.jpg
  [2]: http://7xq7nb.com1.z0.glb.clouddn.com/nowrap-jianxi.jpg
  [3]: http://7xq7nb.com1.z0.glb.clouddn.com/nowrap-inlineORinline-block.jpg
  [4]: http://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/
  [5]: http://www.cnblogs.com/fengzheng126/archive/2012/05/18/2507632.html
  [6]: http://7xq7nb.com1.z0.glb.clouddn.com/nowrap-GIF.gif
  [7]: http://7xq7nb.com1.z0.glb.clouddn.com/nowrap-GIF-IE56.gif
  [8]: https://github.com/JChehe/blog
  [9]: http://jchehe.github.io/resume/
