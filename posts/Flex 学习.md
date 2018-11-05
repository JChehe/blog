# Flex 学习

之前以为Flexbox不成熟，兼容性不好，所以看了书本上的知识后就没有深入了解和实践。

在 [Can I use][1] 网站上可看到 flex 的兼容情况。在该网站可以看到flex有新版和旧版的规范。

先不管旧版，学习新版。

学习资料是来自阮一峰老师的两篇教程：《[Flex 布局教程：语法篇][2]》 和 《[Flex 布局教程：实例篇][3]》

其中：flex-basis、flex-grow与flex-shrink是学习《[深入理解css3中的flex-grow、flex-shrink、flex-basis][4]》。

### Flex属性速记
#### Flex容器属性表格
| Flex容器属性    | 定义                                                              | 取值（第一个是默认值）                                                  |
|-----------------|-------------------------------------------------------------------|-------------------------------------------------------------------------|
| flex-direction  | 决定主轴的方向（即项目的排列方向）。                              | row \| row-reverse \| column \| column-reverse;                            |
| flex-wrap       | flex-wrap属性定义，如果一条轴线排不下，如何换行（默认是不换行）。 | nowrap \| wrap \| wrap-reverse;                                           |
| flex-flow       | 是flex-direction属性和flex-wrap属性的简写形式                     | 默认值为row nowrap。                                                    |
| justify-content | 定义了项目在主轴上的对齐方式。                                    | flex-start \| flex-end \| center \| space-between \| space-around            |
| align-items     | 定义项目在交叉轴上如何对齐。                                      | flex-start \| flex-end \| center \| baseline \| stretch                     |
| align-content   | 定义了多根轴线（即项目也声明为Flex容器）的对齐方式。一根轴不起效  | stretch \| flex-end \| center \| space-between \| space-around \| flex-start |


#### Flex项目属性表格

| Flex项目属性 | 定义                                                                                                                         | 取值                                                                                             |
|--------------|------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------|
| order        | 定义项目的排列顺序。数值越小（负数有效），排列越靠前，默认为0。                                                              | integer 默认为0，负值有效。                                                                      |
| flex-grow    | 属性定义项目的放大比例，默认为0，即如果存在剩余空间，也不放大。                                                              | number 默认值是0                                                                                 |
| flex-shrink  | 定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。0代表在空间不足时也不收缩。负值对该属性无效（相当于默认值 1）。 | number 默认值是1                                                                                 |
| flex-basis   | 定义了在分配容器空间之前，项目占据的主轴空间（main size）。默认值为auto，即项目的本来大小。                                  | length | auto                                                                                    |
| flex         | 是flex-grow, flex-shrink 和 flex-basis的简写                                                                                 | 默认值为0 1 auto。后两个属性可选。还有两个关键字：`auto` (`1 1 auto`) 和 `none` (`0 0 auto`)     |
| align-self   | 允许单个项目有与其他项目不一样的对齐（交叉轴上）方式，即覆盖align-items属性。                                                | auto | flex-start | flex-end | center | baseline | stretch auto表示继承父元素的align-items属性。 |
本文是自己的快速记录，你们想学可以看阮一峰老师的教程。

### Flex布局是什么？
Flex是Flexible Box的缩写，意为"弹性布局"，用来为盒状模型提供最大的灵活性。

任何一个容器都可以指定为Flex布局。
```css
div{
    display: flex;
    /* 旧版浏览器需要前浏览器前缀，如：-webkit-flex */
}
```

行内元素也可以使用Flex布局。
```css
.box{
    display: inline-flex;
}
```

注意，设为Flex布局以后，子元素的float、clear和vertical-align属性将失效（已测试，的确是这样）。

### 基本概念
采用Flex布局的元素，称为Flex容器（flex container），简称"容器"。它的所有子元素自动成为容器成员，称为Flex项目（flex item），简称"项目"。当然，Flex项目也可以再次声明为Flex容器。

![此处输入图片的描述][5]  
容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做main start，结束位置叫做main end；交叉轴的开始位置叫做cross start，结束位置叫做cross end。

项目默认沿主轴排列。单个项目占据的主轴空间叫做main size，占据的交叉轴空间叫做cross size。

### 容器的属性

#### flex-direction
决定主轴的方向（即项目的排列方向）。
```css
.box {
    flex-direction: row | row-reverse | column | column-reverse;
}
```

#### flex-wrap
默认情况下，项目都排在一条线（又称"轴线"，**默认是不换行的**）上。flex-wrap属性定义，如果一条轴线排不下，如何换行。
```css
.box{
    flex-wrap: nowrap | wrap | wrap-reverse; 
    /* wrap-reverse 行之间的倒序 */
}
```

#### flex-flow
是flex-direction属性和flex-wrap属性的简写形式，默认值为row nowrap。

#### justify-content属性
定义了项目在**主轴**上的对齐方式。

![此处输入图片的描述][6]

它可能取5个值，具体对齐方式与轴的方向有关。下面假设主轴为从左到右。

 - flex-start（默认值）：左对齐
 - flex-end：右对齐
 - center： 居中
 - space-between：两端对齐，项目之间的间隔都相等。
 - space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

#### align-items属性
定义项目在交叉轴上如何对齐。

它可能取5个值。具体的对齐方式与交叉轴的方向有关，下面假设交叉轴从上到下。

 - flex-start：交叉轴的起点对齐。
 - flex-end：交叉轴的终点对齐。
 - center：交叉轴的中点对齐。
 - baseline： 项目的第一行文字的基线对齐。
 - stretch（默认值）：如果项目未设置高度或设为auto，将占满整个容器的高度。

#### align-content属性
定义了多根轴线（即项目也声明为Flex容器）的对齐方式。如果容器只有一根轴线，该属性不起作用。

![此处输入图片的描述][7]
 - flex-start：与交叉轴的起点对齐。
 - flex-end：与交叉轴的终点对齐。
 - center：与交叉轴的中点对齐。
 - space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
 - space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。
 - stretch（默认值）：轴线占满整个交叉轴。

### 项目的属性
#### order
定义项目的排列顺序。数值越小（负数有效），排列越靠前，默认为0。

#### flex-grow属性
属性定义项目的放大比例，**默认为0，即如果存在剩余空间，也不放大**。  

即：对于剩余空间（容器宽度-项目总占位），然后按照项目指定的大小（比例=当前项目值/总项目值）进行分配。

如果所有项目的flex-grow属性都为1，则它们将等分剩余空间（如果有的话）。如果一个项目的flex-grow属性为2，其他项目都为1，则前者占据的剩余空间将比其他项多一倍。

#### flex-shrink属性
定义了项目的缩小比例，**默认为1，即如果空间不足，该项目将缩小**。0代表在空间不足时也不收缩。

负值对该属性无效（相当于默认值 1）。

该属性的工作机制与`flex-grow`一样，对`项目总占位`超出容器的部分进行按比例分配，然后对各个 flex-shrink 不为 0 的项目按照分配到的宽度进行收缩（即项目原本宽度减去分配到的宽度）。

#### flex-basis属性
定义了在分配容器空间之前，项目占据的主轴空间（main size）。

它的默认值为auto，即项目的本来大小。
```css
.item {
    flex-basis: <length> | auto; /* default value */
}
```
> 如果同时设置flex-basis和width，那么width属性会被覆盖，也就是说flex-basis的优先级比width高。有一点需要注意的是：如果flex-basis和width其中有一个是auto，那么另外一个非auto的属性优先级会更高。


#### flex属性
flex属性是flex-grow, flex-shrink 和 flex-basis的简写，默认值为0 1 auto。后两个属性可选。
```css
.item {
    flex: none | [ <'flex-grow'> <'flex-shrink'>? || <'flex-basis'> ]
}
```

该属性有两个关键字：`auto` (`1 1 auto`) 和 `none` (`0 0 auto`)。

#### align-self属性
允许单个项目有与其他项目不一样的对齐（交叉轴上）方式，即覆盖align-items属性。默认值为auto，表示继承父元素的align-items属性，如果没有父元素，则等同于stretch。
```css
.item {
    align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```
除了auto，其余与align-items 一样。

希望能尽快应用到项目中。


  [1]: http://caniuse.com/#search=flex
  [2]: http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html
  [3]: http://www.ruanyifeng.com/blog/2015/07/flex-examples.html
  [4]: http://zhoon.github.io/css3/2014/08/23/flex.html
  [5]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/flex.png
  [6]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/justify-content.png
  [7]: https://blog-1251477229.cos.ap-chengdu.myqcloud.com/others/align-content.png
