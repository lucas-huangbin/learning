#### 属性绑定（Property）

##### 了解数据流
属性绑定在单一方向上将值从组件的属性送到目标元素的属性。
目标属性是你要赋值的DOM属性。

##### 绑定到属性
要绑定到元素的属性，请将其放在方括号`[]`中，这回将此属性标识为目标属性。

`<div>`
  `<h1>Property Binding with Angular</h1>`
  `<h2>Binding the src property of an image:</h2>`
  `<img alt="item" [src]="itemImageUrl">`
  `<hr />`
  `<h2>Binding to the colSpan property</h2>`
  `<table border=1>`
    `<tr><td>Column 1</td><td>Column 2</td></tr>`
    `<!-- Notice the colSpan property is camel case -->`
    `<tr><td [colSpan]="2">Span 2 columns</td></tr>`
  `</table>`
  `<hr />`
  `<h2>Button disabled state bound to isUnchanged property:</h2>`
  `<!-- Bind button disabled state to `isUnchanged` property -->`
  `<button type="button" [disabled]="isUnchanged">Disabled Button</button>`
  `<hr />`
  `<h2>Binding to a property of a directive</h2>`
  `<p [ngClass]="classes">[ngClass] binding to the classes property making this blue</p>`
  `<hr />`
  `<h2>Model property of a custom component:</h2>`
  `<app-item-detail [childItem]="parentItem"></app-item-detail>`
  `<app-item-detail childItem="parentItem"></app-item-detail>`
  `<h3>Pass objects:</h3>`
  `<app-item-list [items]="currentItems"></app-item-list>`
  `<hr />`
  `<h2>Property binding and interpolation</h2>`
  `<p><img alt="Interpolated item" src="{{ itemImageUrl }}"> is the <em>interpolated</em> image.</p>`
  `<p><img alt="Property Bound item" [src]="itemImageUrl"> is the <em>property bound</em> image.</p>`
  `<p><span>"{{ interpolationTitle }}" is the <em>interpolated</em> title.</span></p>`
  `<p>"<span [innerHTML]="propertyTitle"></span>" is the <em>property bound</em> title.</p>`
  `<hr />`
  `<h2>Malicious content</h2>`
  `<p><span>"{{ evilTitle }}" is the <em>interpolated</em> evil title.</span></p>`
  `<!--`
  `Angular generates a warning for the following line as it sanitizes them`
  `WARNING: sanitizing HTML stripped some content (see https://g.co/ng/security#xss).`
 `-->`
  `<p>"<span [innerHTML]="evilTitle"></span>" is the <em>property bound</em> evil title.</p>`
`</div>`

#### 属性绑定（Attribute）
Angular中的Attribute绑定可帮助你直接设置Attribute值。使用Attribute绑定，你可以提升无障碍性、动态设置应用程序样式以及同时管理多个CSS类或样式。

##### 语法
属性绑定语法类似于属性绑定，但在方括号之间不是元素属性，而是使用前缀`attr`后跟一个点的属性名称。然后，你可以使用解析为字符串的表达式设置属性值。
`<p [attr.attribute-you-are-targeting]="expression"></p>`

注：当表达式解析为null或undefined时，Angular会完全移除属性。

##### 绑定ARIA Attribute
Attribute绑定的主要用例之一是设置ARIA Attribute。
`<h1>Attribute, class, and style bindings</h1>`
`<h2>Attribute binding</h2>`
`<table border=1>`
  `<!--  expression calculates colspan=2 -->`
  `<tr><td [attr.colspan]="1 + 1">One-Two</td></tr>`
  `<!-- ERROR: There is no 'colspan' property to set!`
    `<tr><td colspan="{{ 1 + 1 }}">Three-Four</td></tr>`
  `-->`
  `<!-- Notice the colSpan property is camel case -->`
  `<tr><td [colSpan]="1 + 1">Three-Four</td></tr>`
  `<tr><td>Five</td><td>Six</td></tr>`
`</table>`
`<div>`
  `<!-- create and set an aria attribute for assistive technology -->`
  `<button type="button" [attr.aria-label]="actionName">{{ actionName }} with Aria</button>`
`</div>`
`<hr />`
`<h2>Styling precedence</h2>`
`<h3>Basic specificity</h3>`
`<!-- The 'class.special' binding overrides any value for the 'special' class in 'classExpression'.  -->`
`<div [class.special]="isSpecial" [class]="classExpression">Some text.</div>`
`<!-- The 'style.border' binding overrides any value for the 'border' property in 'styleExpression'.  -->`
`<div [style.border]="border" [style]="styleExpression">Some text.</div>`
`<h3>Source specificity</h3>`
`<!-- The 'class.special' template binding overrides any host binding to the 'special' class set by 'dirWithClassBinding' or 'comp-with-host-binding'.-->`
`<comp-with-host-binding [class.special]="isSpecial" dirWithClassBinding></comp-with-host-binding>`
`<!-- The 'style.color' template binding overrides any host binding to the 'color' property set by 'dirWithStyleBinding' or 'comp-with-host-binding'. -->`
`<div>`
  `<comp-with-host-binding [style.color]="color" dirWithStyleBinding></comp-with-host-binding>`
`</div>`
`<h3>Dynamic vs static</h3>`
`<!-- If '[class.special]' equals false, this value overrides the 'class="special"' below -->`
`<div class="special" [class.special]="false">Some text.</div>`
`<!-- If 'styleExpression' has a value for the 'border' property, this value overrides the 'style="border: dotted darkblue 3px"' below -->`
`<div style="border: dotted darkblue 3px" [style]="styleExpression">Some text.</div>`
`<div class="readability">`
  `<comp-with-host-binding dirWithHostBinding></comp-with-host-binding>`
`</div>`
`<app-my-input-with-attribute-decorator type="number"></app-my-input-with-attribute-decorator>`

注：有时属性名称和属性之间存在差异
`colspan` 是 `<td>` 的 Attibute，而带有大写 "S" 的 `colSpan` 是 Property。使用 Attribute 绑定时，请使用带有小写“s”的 `colspan`。



#### 类绑定与样式绑定

##### 绑定到单个CSS class

`[class.sale]="onSale"`


##### 绑定到多个CSS 类
`[class]="classExpression"`

表达式可以是以下之一：
- 用空格分隔的类名字字符串。
- 以类名作为键名并将真或假表达式作为值的对象。
- 类名的数组

注：
对于任何类似对象的表达式，比如 `object`、 `Array`、 `Map` 或 `Set`，对象的标识必须改变才能使 Angular 更新类列表。更新属性而不改变对象标识没有效果。（需要改变对象本身才能响应）

如果存在多个绑定到相同类名的绑定，Angular使用样式优先级确定要使用的绑定。

##### 绑定到单个样式
要创建单个样式绑定，请使用前缀style，后跟点和CSS样式的名称。
`[style.width]="width"`
Angular将该属性设置为绑定表达式的值，这通常是一个字符串。（可选）你可以添加单位扩展名，比如em或%，这需要数字类型。
1. 要以中线格式（dash-case）编写样式：
   `<nav [style.background-color]="expression"></nav>`
2. 要以驼峰格式（camelCase）编写样式：
   `<nav [style.backgroundColor]="expression"></nav>`


##### 绑定到多个样式
`[style]="styleExpression"`
styleExpression可以是以下之一：
- 样式的字符串列表：
  `"width: 100px; height: 100px; background-color: cornflowerblue;"`
- 一个对象，其键名是样式名，其值是样式值，比如：
  `{width: '100px', height: '100px', backgroundColor: 'cornflowerblue'}`


##### 绑定到键盘事件
![[Pasted image 20240820175953.png]]


##### 绑定到被动事件
![[Pasted image 20240820180339.png]]