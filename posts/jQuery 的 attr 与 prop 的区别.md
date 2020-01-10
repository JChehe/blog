## jQuery 的 attr 与 prop 的区别

标签： JavaScript jQuery

---

先提出问题：对于 checked 这类值是 true/false 的属性，用 jQuery 的 attr 或 prop 方法进行 读取或设置值是有区别的。

在看 jQuery 文档前，我们先看看 attribute 与 property 是什么：

### 先搞懂 attribute 与 property

当编写 HTML 源码时，你能在 HTML 元素里定义 attributes。然后，一旦浏览器解析你的代码，该 HTML 元素相应的 DOM 节点就会被创建。该节点是一个对象，因此它就拥有 properties。
因此，我们知道：attributes 是 HTML 元素（标签）的属性，而 properties 是 DOM 对象的属性。

例如，下面这个 HTML 元素：

    <input type="text" value="Name:">

拥有两个 attributes。

一旦浏览器解析该代码，HTMLInputElement 对象就会被创建，并且该对象会拥有很多 properties，如：accept、accessKey、align、alt、attributes、autofocus、baseURI、checked、childElementCount、ChildNodes、children、classList、className、clientHeight 等等。

对于某个 DOM 节点对象，properties 是该对象的所有属性，而 attributes 是该对象对应元素(标签)的属性。

当一个 DOM 节点为某个 HTML 元素所创建时，其大多数 properties 与对应 attributes 拥有相同或相近的名字，但这并不是一对一的关系。例如，下面这个 HTML 元素：

    <input id="the-input" type="text" value="Name:">
    
其对应 DOM 节点会拥有如下properties： id、type 和 value：

 - `id` property是 `id` attribute 的映射：获取该 property 即等于读取其对应的 attribute 值，而设置该 property 即为 attribute 赋值。`id` 是一个纯粹的映射 property，它不会修改或限制其值。
 - `type` property 是 `type` attribute 的映射：获取该 property 即等于读取其对应的 attribute 值，而设置该 property 即为 attribute 赋值。但 `type` 并不是一个纯粹的映射 property，因为它的值被限制在已知值（即 input 的合法类型，如：text、password）。如果你有 `<input type="foo">`，然后 `theInput.getAttribute("type")` 会返回 `"foo"`，而 `theInput.type` 会返回 `"text"`。
 - 相比之下，`value` property 并不会映射 `value` attribute。取而代之的是 input 的当前值。当用户手动更改输入框的值，`value` property 会反映该改变。所以，如果用户在 input 输入 `John`，然后：

    theInput.value // 返回 "John"
然而：
    theInput.getAttribute('value') // 返回 "Name:"

`value` property 反映了 input 的**当前**文本内容，而 `value` attribute 则是在 HTML 源码 value 属性所指定的**初始**文本内容。

因此，如果你想知道文本框的当前值，则读取 property。而如果你想知道文本框的初始值，则读取 attribute。或者你也可以利用 defaultValue property，它是 value attribute 的纯粹映射。

    theInput.value                 // returns "John"
    theInput.getAttribute('value') // returns "Name:"
    theInput.defaultValue          // returns "Name:"
    
有几个 properties 是直接反映它们 attribute（rel、id），而有一些则用稍微不同的名字进行直接映射（`htmlFor` 映射 `for` attribute，`className` 映射 `class` attribute）。很多 property 所映射的 attribute 是带有限制/变动的（src、href、disabled、multiple）。该 [规范][1] 涵盖了各种各样的映射。


### 再看看 attr() 与 prop() 的区别


上述能让我们理清了 attribute 与 property 之间的区别，下面根据 [jQuery 文档][2] 对 attr() 与 prop() 方法进行比较：

自 jQuery 1.6 版本起，`attr()` 方法对于未设置的 attributes （即标签中没写该 attributes）都会返回 `undefined`。对于检索和改变 DOM 的 properties，如表单元素的 checked、selected 或 disabled 状态，应使用 `.prop()` 方法。

Attributes vs. Properties

attributes 与 properties 之间的差异在特定情况下会变得尤为重要。在 jQuery 1.6 前，`.attr()` 方法在检索一些 attributes 时，有时会把 property 考虑进去，这会导致不一致的行为。在 jQuery 1.6 版本之后，`.prop()` 方法提供了一种明确检索 property 值的方式，而 `.attr` 只会检索 attributes。

例如，selectedIndex、tagName、nodeName、nodeType、ownerDocument、defaultChecked 和 defaultSelected 能被 `.prop()` 检索与设置。在 jQuery 1.6 之前，这些 properties 都是通过 `.attr()` 检索的，但检索这些属性并不应属于 attr 方法职责内 。这些属性并没有对应的 attributes，只有 properties 本身。

对于值为布尔值的 attributes ，考虑到一个 DOM 元素是通过 HTML 标签 `<input type="checkbox" checked="checked />` 定义的，并且假定它在 JavaScript 的变量名为 `elem`：

| 读取属性 	| 返回值 	| 描述 	|
|:---------------------------------------	|:--------------------	|:------------------------------------------------------------------------------------------------------------------------------------------------	|
| elem.checked 	| true (Boolean) 	| 会随着 checkbox 状态作出相应改变 	|
| $(elem).prop("checked") 	| true (Boolean) 	| 会随着 checkbox 状态作出相应改变 	|
| elem.getAttribute("checked") 	| "checked" (String) 	| checkbox 的初始状态；并且不会随着 checkbox 的状态而改变。 	|
| $(elem).attr("checked") 	| "checked" (String) 	| （与jQuery文档描述不一样，1.9.0 及之后的版本，都是返回 “checked”，并不会随着 checkbox 的改变而改变），即 1.9.0 之前的版本会随着 checkbox 的改变而改变。|

根据 W3C forms（表单） 规范，`checked` 是一个值为 boolean 的 attribute，这意味着当该 attribute 存在（无论值是什么），其对应的 property 都是 true。例如，该 attribute 没赋值或设为空字符串，甚至设为 `"false"`。这同样适用于所有值为 boolean 的 attributes。

然而，对于 `checked` attribute 最重要的概念是记住它并不是对应 `checked` property。该 attribute 实际上是对应 `defaultChecked` property，并仅在初次设置 checkbox 值时使用。`checked` attribute 的值并不会随着 checkbox 的状态而作出相应改变，而 `checked` property 会。因此，为了兼容不同浏览器，当判断一个 checkbox 是否被选择时应该使用 `property`：

    if (elem.checked)
    if ($(elem).prop("checked"))
    if ($(elem).is(":checked"))
    
这同样适用于其它动态 attributes，如 selected 和 value。

其他说明：
在 IE9 之前的版本，如果使用 `.prop()` 为 DOM 元素的 property 设置的值不是一个简单的原始值（number、string 或 boolean），且该 property 在 DOM 元素从 document 移除前未被移除（使用 .removeProp()），则会导致内存泄漏。为 DOM 对象设置值的安全做法（避免内存泄漏）是使用  `.data()`。


参考（翻译）：
jQuery API Documentation：http://api.jquery.com/prop/  
Properties and Attributrs in HTML：http://stackoverflow.com/questions/6003819/properties-and-attributes-in-html

------

Github 地址: [jQuery 的 attr 与 prop 的区别](https://github.com/JChehe/blog/blob/master/posts/jQuery%20%E7%9A%84%20attr%20%E4%B8%8E%20prop%20%E7%9A%84%E5%8C%BA%E5%88%AB.md)

  [1]: https://www.w3.org/TR/html5/infrastructure.html#reflect
  [2]: http://api.jquery.com/prop/
