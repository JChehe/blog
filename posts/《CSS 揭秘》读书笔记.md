# CSS揭秘

标签： 读书笔记

---

终于看完了《CSS揭秘》了，下面就做一下笔记。

<!-- more -->
对querySelectorAll进行封装：
```javascript
function $$(selector, context){
    context = context || document;
    var elements = context.querySelectorAll(selector);
    return Array.prototype.slice.call(elements);
}
```

检测某个样式属性是否被支持，核心思路是在任意元素的 `element.style` 对象上检查该属性是否存在：
```javascript
funcion testProperty(property){
    var root = document.documentElement; // <html>
    
    if(property in root.style){
        root.classList.add(property.toLowerCase());
        return true;
    }
    
    root.classList.add("no-" + property.toLowerCase());
    return false;
}
```

检测某个具体的**属性值**是否支持，那就需要把它赋给对应的属性，然后再检查浏览器是不是还保存着这个值：
```javascript
function testValue(id, value, property){
    var dummy = document.createElement("p");
    dummy.style[property] = value;
    
    if(dummy.style[property]){
        root.classList.add(id);
        return true;
    }
    
    root.classList.add("no-" + id);
    return false;
}
```

如果要检测选择符和@规则的支持情况：在解析CSS代码时，浏览器总会丢弃它自己无法识别的部分，因此可以动态地应用样式并检查它是否生效，以此来判断浏览器是否可以识别某个特性。当然，**浏览器可以解析某个CSS特性并不代表它已经实现（或正确实现）了这个特性。**

## 第一章 引言
### 浏览器前缀
标准的工作组需要网页开发者这一端的输入，以确保各项规范可以处理真实的开发需求。当实验性的技术被广泛应用到生产时，工作组就被这些技术早期的、实验性的版本捆住手脚了，因此一旦这些技术有变动，那些已经在用这些技术的网站就挂了。显然，这完全否定了让开发者尝试早期标准的好处。

浏览器前缀就是解决方案之一。这个方案是指每个浏览器都可以实现这些实验性的（甚至是私有的、非标准的）特性，但要在名称前加上自己特有的前缀。  
最终标准化的版本会有一个不同的名称（没有前缀），这样就不会与加前缀版本相冲突了。

开发者为了避免浏览器日后更新（从而支持某属性），就先发制人地加上所有可能的浏览器前缀，再把无前缀的版本放在最后，以图一劳永逸。

```css
-moz-border-radius: 10px;
-ms-border-radius: 10px;
-o-border-radius: 10px;
-webkit-border-radius: 10px;
border-radius: 10px;
```
-ms-和-o-是多余的，这两个属性从未在任何浏览器中出现过，因为IE和Opera从开始就直接实现了border-radius这个无前缀版本的。

由于开发者使用无前缀的属性是想确保代码的向前兼容，那么工作组想要修改这些无前缀语法就变得不可能了。我们基本上就跟这些半生不熟的早期规范绑在一起了。**因此，浏览器前缀已是一场史诗般的失败**。

最近，浏览器厂商已经很少以前缀的方式来实验性地实现新特性了。取而代之的是，通过**配置开关**来启用，这样可以有效防止开发者在生产环境中使用它们。

### CSS编码技巧
在软件开发中，保持代码的DRY（Don't Repeat Yourself）和可维护性是最大的挑战之一，而这句话对于CSS也是适用的。

在实践中，代码可维护性的最大要素是尽量减少改动时要编辑的地方。

**当某些值相互依赖时，应该把它们的相互关系用代码表达出来。**如按钮的字体与行高的关系。

**有时，代码易维护和代码量少不可兼得。**如元素的边框为10px宽，但左侧不加边框。
```css
border-width: 10px 10px 10px 0;
```
只需一条声明就可以达到要求了，但如果日后要改动边框的宽度，就需要修改3处。如果把它拆成两条声明，则改起来容易多了，而且可读性或许更好一些：
```css
border-width: 10px;
border-left-width: 0;
```

**currentColor**这个变量的值是当前元素的color值。如果当前元素没有在CSS里显示地指定一个color值，那它的颜色值就遵从CSS规则，从父级元素继承而来。该关键字**兼容IE9及以上**。

**继承**：inherit可以用在任何CSS属性（可继承的），而且它总是绑定到父元素的计算值（对伪元素来说，则会取该伪元素的宿主元素）。

把表单元素的字体设定为与页面的其他部分相同，你无需重复指定字体属性，只需利用inherit即可。
```css
input, select, button{ font: inherit; }
```

把超链接的颜色设定与页面中其它文本相同：
```css
a{ color: inherit; }
```

在创建提示框时，通过:before伪元素生成的小箭头能继承宿主元素的背景和边框样式：
```css
.callout{ position: relative; }

.callout:before{
    content: "";
    background: inherit;
    border: inherit;
    ...
}
```

### 相信你的眼睛，而不是数字
人的眼睛并不是一台完美的输入设备。有时候精准的尺度看起来并不精准，而我们的设计需要顺应这种偏差。

 1. 看一个完美居中的物体时，会感觉它并不居中。实际上，我们应该把这个物体从几何学的中心点再晚上稍微上挪一点。
 2. 宽度相同的圆形和正方形，圆形会显得小一点。
 
上述视觉上的错觉在任何形式的视觉设计中都普遍存在，需要我们有针对性地进行调整。   
常见的例子是：给一个文本容器设置内边距。无论文本多长，四边相同的内边距在实际效果看起来并不相等。原因在于，字母的形状在两端都比较整齐，而顶部和底部则往往参差不齐，从而导致你的眼睛把这些参差不齐的空缺部分感知为多出来的内边距。

### 关于响应式设计
媒体查询不能以一种连续的方式来修复问题。它的工作原理基于某几个特定的断点。  
添加的媒体查询越多，CSS代码就会变得越经不起折腾。应该把它作为最后的手段。

媒体查询的断点不应该由具体的设备来决定，而应该根据设计自身来决定。

避免不必要的媒介查询的建议：

 - 使用百分比长度来取代固定长度。若做不到这点，可尝试使用与视口相关的单位（vw、wh、vmin、vmax）。
 - 需要在较大分辨率下取得固定宽度时，使用max-width而不是width。
 - 不要忘记为替换元素（如img、object、video、iframe等）设置max-width：100%。
 - 假设背景图片需要完整地铺满一个容器，不管容器尺寸如何变化，background-size:cover可以做到这点。需要注意的是，移动端带宽有限，把大图缩小显示往往不太明智。
 - 当图片（或其他元素）以行列式进行布局时，让视口宽度来决定列的数量。
 - 在使用多列文本时，指定column-width而不是column-count。
 
**总的来说，思路是尽最大努力实现弹性可伸缩的布局，并在媒体查询的各个断点区间内指定相应的尺寸。**

### 合理使用简写
合理使用简写是一种良好的防卫性编码方式，可以抵御未来的风险。

如果只为某个属性提供一个值，那它会扩散并应用到列表中的每一项。如：
```css
background: url(tr.png) no-repeat top right / 2em 2em,
            url(br.png) no-repeat bottom right / 2em 2em,
            url(bl.png) no-repeat bottom left / 2em 2em;
```
可将重复的值从简写属性中抽出来写成一个展开式属性：
```css
background: url(tr.png) top right,
            url(br.png) bottom right,
            url(bl.png) bottom left;
background-size: 2em 2em;
background-repeat: no-repeat;
```

对于如background属性的值有歧义的属性，background-position（即使是初始值也要写出来）和background-size中间需要`/`作为分隔。

对于绝大多数的简写属性来说，并没有歧义问题，因而简写属性的多个值往往可以随意排列。

## 第二章 背景与边框
background-clip: border-box（默认值）/padding-box/content-box

### 多重边框
#### box-shadow方案
box-shadow的第四个参数（称作“扩张半径”），通过指定正值或负值，可以让投影面积加大或者减少。  
一个正值的扩张半径加上两个为零的偏移量以及为零的模糊值，得到的“投影”其实就像一道实线边框。  
box-shadow支持逗号分隔符，可创建任意数量的投影。  
box-shadow是层层叠加的，第一层投影位于最顶层，以此类推。

注意点

 - 投影的行为更边框（border）不完全一致，因为它不影响布局，而且也不会受到box-sizing属性的影响。
 - 不会响应鼠标事件
 
#### outline方案
outline: 5px solid #655; 它可通过outline-offset属性来控制它跟元素边缘之间的间距，这个属性甚至可以接受负值。

需要注意的是：它目前并不贴合border-radius，未来可能会改为贴合。

#### 灵活的背景定位
background-origin: padding-box（默认值）/content-box/border-box。

calc()需要注意的是：要在`-`和`+`运算符的两侧各加一个空白符，否则会产生解析错误！这是为了向前兼容：未来，在calc()内部可能会允许使用关键字，而这些关键字可能会包含连字符（即`-`）。  
当然calc()还能进行乘法与除法。

#### 条纹背景
**渐变需要注意的点**
渐变指定的位置是色标的位置，色标之间的距离是过渡的空间。

**渐变的角度**
![此处输入图片的描述][1]

> 如果某个色标的位置值比整个列表中在它之前的色标的位置值都要小，则该色标的位置值会被设置为它前面所有色标位置值的最大值。

根据上述引用，我可以在构建条纹时（无过渡）时，把第二个坐标（即后续坐标）的位置值设置为0，那它的位置就总是会被浏览器调整为前一个色标的位置值。  
这可以避免每次改动条纹宽度时都要修改两个数字。
```css
background: linear-gradient(#fb3 30%, #58a 0);
```

#### 复杂的背景
对于现代浏览器，我们可以把SVG文件以data URI的方式嵌入到样式表中，甚至不需要用base64或URIencode来对其编码：
```css
background: url("data:image/svg+xml, \
            <svg> \
            ... \
            </svg>");
```
注意，如果出于可读性的考虑，需要把一句CSS代码（如background-image的字符串值）打断为多行，可用反斜杠（`\`）来转义每行末尾的换行。

#### 伪随机背景
与box-shadow一样，多图背景的位置（默认）都是从最左侧开始的。

#### 连续的图像边框
MDN：浏览器应用了border-image，则不再应用border-style。另外，若border-image-source的值为none或图片不能显示，则将应用border-style。

## 第三章 形状
### 梯形标签页
为了更好地实现回退方案，可同样通过不兼容的属性进行设置，如：变形后需要上移的情况，此时，可用translateY，而不是margin-top或postion。

### 简单的饼图
一个负的延时值（animation-delay）是合法的。它会跳过指定时间而从中间开始播放。

通过设置animation-play-state: paused，可让动画暂停。默认值是`running`。

**如何改变伪元素的样式？**

 - 通过继承，改变宿主元素
 - 通过animation，设置好一系列帧，然后设置延时值，以跳到某帧的样式（前提是处于暂停状态）。

## 第四章 视觉效果
### 单侧投影
border-radius：2px 3px 4px rgba(0,0,0,.5);  
使用高斯模糊（或类似算法）将它进行4px的模糊处理。这在本质上表示在阴影边缘发生**阴影色**与**纯透明色**之间的颜色过渡，长度近似于模糊半径的两倍（比如这里是`8px`）。  
模糊后的矩形**与原始元素的交集部分会被切除掉**。因此，如果元素是半透明时，也会看不到它下层的投影。这点与text-shadow不同，文字下层的投影不会被裁切。

### 不规则投影
box-shadow会忽略元素的伪元素或半透明部分（只对整体进行投影），这类情况包括：

 - 半透明图像、背景图像、或者border-image；
 - 元素设置了点状、虚线或半透明的边框，但没有背景（或者当background-clip不是border-box）时；
 - 对话气泡，它的小尾巴通常是通过伪元素生成；
 - ...

filter: drop-shadow()与box-shadow的参数一样，但不包括扩张半径和inset关键字，也不支持逗号分隔符的多层投影。
```css
filter: drop-shadow(2px 2px 2px rgba(0,0,0,.5));
```
该属性能对很好地解决上述问题（伪元素、虚线边框、半透明背景等）。

当然，它们的模糊算法可能不同，需要进行调整。

### 毛玻璃效果
filter: blur(); 是对自身进行模糊，但不透明。

background-attach: fixed; 表示背景相对于视口固定（以视口的左上角为背景定位origin）。即使一个元素拥有滚动机制，背景也不会随着元素的内容滚动。

## 第六章 用户体验
### 自定义复选框
新伪类 `:checked`，该伪类只在复选框（也适用于单选框）被选择时才会匹配，不论这个选择状态是由用户交互触发，还是由脚本触发。

> 属性选择符[checked]与它不同，该选择符不会根据用户的交互行为进行更新，因为用户的交互并不会影响到HTML标签上的属性。

由于没有多少样式能够对复选框（由于是替换元素）起作用。不过，可基于复选框的勾选状态借助组合选择符给其他元素（label）设置样式。

> 替换元素：其内容超出了CSS格式化模型的范畴，比如图片、嵌入的文档等。原则上我们无法为替换元素添加生成性内容，尽管某些浏览器可能会允许这样做。

```html
<input type="checkbox" id="awesome" />
<label for="awesome">Awesome!</label>
```
```css
input[type="checkbox"] + label::before{
    ...
}
// 选中状态
input[type="checkbox"]:checked + label::before{
    ...
}

```
当label元素与复选框关联后，也可以起到触发开关的作用。

### 交互式的图片对比控件
即使不设置 pointer-events: none;的情况下，伪元素（覆盖在它之上）也不会干扰调节手柄（resize:both）的功能。 


## 第七章 结构与布局
min-content：该关键字将解析为这个容器内部最大的不可断行元素的宽度（即最宽单词、图片或具有固定宽度的盒元素）。
```css
width: min-content;
```

还有max-content与fit-content关键字。

### 精确控制表格列宽
table默认情况下：

 - 不指定单元格的宽度，则这些单元格就会根据它们内容的长短来分配。
 - 当然，表格的每一行都会参与到列宽的计算中，而不仅是第一行
 - 即时我们为单元格指定了宽度（第一列1000px，第二列2000px），由于外层容器所能提供的宽度不足3000px，则这两个单元格的宽度会按比列缩小，分别得到33.3% 和 66.6%。
 - 如果禁止文本折行，那么表格宽度可能会超出其容器宽度（设置text-overflow:ellipsis;也没用（也不会显示...））。
 - 大图片也会导致上一点同样的问题。

而table-layout
```css
table{
    table-layout: fixed;
    width: 100%;
}
```

 - 如果不指定任何宽度，则各列宽度将是平均分配；
 - 后续的表格行并不会影响列宽；
 - 给单元格指定很大的宽度也能生效，并不会自动缩小；
 - overflow和text-overflow属性都正常生效；
 - 如果overflow的值是visible，则**单元格的内容**可能会溢出。

### 根据兄弟元素的数量来设置样式
```css
li:first-child:nth-last-child(4), /* 由于`~`选择符不包括自身 */
li:first-child:nth-last-child(4) ~ li{
    ...
}
```
该选择符 `li:first-child:nth-last-child(4)` 表示：**一个正好有四个列表项的列表中的第一个列表项**。

根据上述选择符，再通过 `~` 兄弟选择符，即可实现根据数量来设置样式（这里是只有li为4个时）。

根据兄弟元素的数量范围来匹配元素
:nth-child()选择符，它的参数可以是简单数字，也可以像 `an+b` 这样的表达式，`n` 表示0到 `正无穷`。 

 - :nth-child()的值是从1开始，即nth-child(1)对应第一个元素。
 - :nth-child(n+b)：则会选中除第一、二、三个子元素之外的所有子元素。

结合之前的【根据兄弟元素的数量来设置样式】，下面这个选择器会选择：**仅当列表中有4个或更少的列表项时**，才能匹配。
```css
li:first-child:nth-last-child(-n+4), 
li:first-child:nth-last-child(-n+4) ~ li{
    ...
}
```
 
而希望匹配列表包含2~6个列表项时：
```css
li:first-child:nth-last-child(n+2):nth-last-child(-n+6), 
li:first-child:nth-last-child(n+2):nth-last-child(-n+6) ~ li{
    ...
}
```
有上述选择符可看出，同一个选择符中，n的值并不一致，

### 垂直居中
在某些浏览器中，transform可能会导致元素的显示有一些模糊，因为元素可能被放置在半个像素上。这个问题可以用tranform-style: preserve-3d 来修复。

当使用Flexbox时，margin:auto可在水平和垂直方向上居中。另外，我们甚至不需要指定任何宽度：这个居中元素分配到的宽度等于 max-content。

## 第八章 过渡与动画
每个transform-origin都可以被两个translate()模拟出来。


完！

  [1]: http://7xq7nb.com1.z0.glb.clouddn.com/%E6%B8%90%E5%8F%98%E8%A7%92%E5%BA%A6.jpg
