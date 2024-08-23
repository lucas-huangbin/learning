### 组件
#### 组件剖析
每个组件必须包含：
- 一个具有行为的TS类，例如处理瀛湖输入和从服务器获取数据
- 控制将渲染到DOM中的HTML模板
- 定义组件在HTML中如何使用的CSS选择器
通过在TS类的顶部添加一个@Component装饰器，为组件提供Angular特定信息：
`@Component({`
  `selector: 'profile-photo',`
  `template: <img src="profile-photo.jpg" alt="Your profile photo">,`
`})`
`export class ProfilePhoto { }`

Angular会为遇到的每个匹配的HTML元素创建组件的一个实例。与组件选择器匹配的DOM元素称为该组件的宿主元素。组件模板的内容将呈现在其宿主元素内部。

由组件渲染的DOM，对应于该组件的模板，称为该组件的视图。

通过这种方式组合组件，你可以将你的Angular应用程序视为一个组件树。

![[./Image/Pasted image 20240819104631.png]]


#### 选择器
**Angular在编译时静态匹配选择器**。在运行时改变DOM，无论是通过Angular绑定还是DOM API，都不会影响渲染的组件。

**一个元素只能匹配一个组件选择器**，多个组件选择器匹配一个元素，Angular会报错。
**组件选择器区分大小写**

##### 选择器类型
![[Pasted image 20240819105632.png]]
对于属性值，Angular支持使用等号（=）运算符匹配确切的属性值。Angular不支持其它属性值运算符。

Angular组件选择器不支持组合器，包括后代组合器或子组合器。

Angular组件选择器不支持指定命名空间。

###### :not伪类
angular支持伪类:not。你可以将这个伪类附加到任何其他选择器上，以缩小组件选择器匹配的元素范围。
例如，你可以定义一个`[dropzone]`属性选择器并阻止匹配textarea元素：
`@Component({`
  `selector: '[dropzone]:not(textarea)',`
  `...`
`})`
`export class DropZone { }`

Angular不支持组件选择器中的其它伪类或为元素。

###### 组合选择器
例如：指定匹配`type="reset"`的 `<button>`元素
`@Component({`
  `selector: 'button[type="reset"]',`
  `...`
`})`
`export class ResetButton { }`
例如：定义多个选择器：
`@Component({`
  `selector: 'drop-zone, [dropzone]',`
  `...`
`})`
`export class DropZone { }`

##### 选择选择器

##### 何时使用属性选择器
当希望在标准原生元素上创建组件时，可以考虑使用属性选择器。例如，如果你想创建一个自定义按钮组件，可以通过使用属性选择器利用标准`<button>`元素：
`@Component({`
  `selector: 'button[yt-upload]',`
   `...`
`})`
`export class YouTubeUploadButton { }`

#### 指定样式

##### 视图封装
在Angular中，组件样式可以封装在组件的宿主元素中，这样它们就不会影响应用程序的其余部分。
Component的装饰器提供了encapsulation选项，可以用来控制如何基于每个组件应用视图封装。

`@Component({`
  `...,`
  `encapsulation: ViewEncapsulation.None,`
`})`
`export class ProfilePhoto { }`

**ViewEncapsulation.ShadowDom**
这种模式会将组件的视图封装在一个shadow root内，作为组件的宿主元素，并以隔离的方式应用提供的样式。
影子树种的样式和全局样式不会互相影响。
然而，启用ShadowDom封装不仅仅是影响样式作用域。在shadow树种渲染组件会影响事件传播、与`<slot>`的交互，以及浏览器开发者工具宣誓元素的方式。在启用此选项前，务必理解在你的应用种使用ShadowDom的全部影响。

**ViewEncapsulation.Emulated**
默认情况下，Angular使用模拟封装，框架为每个组件实例生成一个唯一的HTML属性，将该属性添加到组件模板中的元素，
Angular修改组件的CSS选择器，将该属性插入到组件样式定义的CSS选择器种，以便它们仅应用于组件的视图，不影响应用程序种的其他元素，从而模拟了ShadowDOM的行为。
这种模式确保了组件的样式不会泄露并影响其它组件，然而，定义在组件外部的全局样式仍可能影响带有模拟封装的组件内部的元素。

::ng-deep
Angular的模拟封装模式支持自动逸伪类::ng-deep。将此伪类应用于CSS规则会禁用该规则的封装，实际上将其变为全局样式。

**ViewEncapsulation.None**
这种模式会禁用组件的所有样式封装，任何与组件关联的样式都会作为全局样式来处理。

##### 查看生成的CSS
使用模拟视图封装是，Angular会预处理所有组件的样式，以便它们仅应用于组件的视图。
在正运行的Angular应用程序的DOM中，使用模拟视图封装模式的组件所在的元素附加了一些额外的属性：
`<hero-details _nghost-pmm-5>`
  `<h2 _ngcontent-pmm-5>Mister Fantastic</h2>`
  `<hero-team _ngcontent-pmm-5 _nghost-pmm-6>`
    `<h3 _ngcontent-pmm-6>Team</h3>`
  `</hero-team>`
`</hero-details>`

**\_nghost**
添加到百位组件视图的元素中，这在本地Shadow DOM封装中将是ShadowRoots。这通常适用于组件的宿主元素。
**\_ngcontent**
添加到组件视图中的子元素，这些元素用于将元素与它们相应的模拟ShadowRoots（具有匹配_nghost属性的宿主元素）进行匹配

##### 混合封装
https://angular.cn/guide/view-encapsulation#mixing-encapsulation-modes



#### 输入
@Input装饰器：
`@Component({...})`
`export class CustomSlider {`
  `@Input() value = 0;`
`}`

在扩展组件类时，输入会被子类继承。

##### 自定义输入
@Input装饰器接受一个配置对象，允许你改变输入的工作方式。

**required属性**
强制指定输入必须始终具有值
`@Component({...})`
`export class CustomSlider {`
  `@Input({required: true}) value = 0;`
`}`

**输入转换**
你可以指定一个transform函数来在Angular设置输入时更改输入的值。

`@Component({`
  `selector: 'custom-slider',`
  `...`
`})`
`export class CustomSlider {`
  `@Input({transform: trimString}) label = '';`
`}`
`function trimString(value: string | undefined) {`
  `return value?.trim() ?? '';`
`}`

输入转换函数必须在构建时静态分析。不能有条件的设置转换函数，也不能作为表达式求值的结果。
输入转换函数应始终是纯函数。依赖于转换函数外部状态会导致行为不可预测。


**内置转换**
Angular包含两个内置转换函数，用于两种最常见的情况：将值强制转换为布尔值和数字。

`import {Component, Input, booleanAttribute, numberAttribute} from '@angular/core';`
`@Component({...})`
`export class CustomSlider {`
  `@Input({transform: booleanAttribute}) disabled = false;`
  `@Input({transform: numberAttribute}) number = 0;`
`}`

booleanAttribute模拟了标准HTML布尔属性的行为，属性的 *存在* 表示“true”值。然而，Angular的booleanAttribute将字面字符串“false”视为布尔值false。

numberAttribute尝试将给定值解析为数字，如果解析失败则产生NAN。

**输入别名**

`@Component({...})`
`export class CustomSlider {`
  `@Input({alias: 'sliderValue'}) value = 0;`
`}`

`<custom-slider [sliderValue]="50" />`

##### 带有getter和setter的输入
使用getter和setter(通过一个中间值做转换)实现的属性可以作为输入：
`export class CustomSlider {`
  `@Input()`
  `get value(): number {`
    `return this.internalValue;`
  `}`
  `set value(newValue: number) {`
    `this.internalValue = newValue;`
  `}`
  `private internalValue = 0;`
`}`

你甚至可以通过仅定义公共setter来创建一个只写输入：
`export class CustomSlider {`
  `@Input()`
  `set value(newValue: number) {`
    `this.internalValue = newValue;`
  `}`
  `private internalValue = 0;`
`}`

如果可能的话，最好使用输入转换，而不是使用getter和setter。

避免复杂或昂贵的getter和setter。Angular可能会多次调用输入的setter，如果setter执行任何昂贵的行为，比如DOM操作，可能会对应用性能产生负面影响。


##### 在@Component装饰器种指定输入
除了使用@Input装饰器外，你还可以在@Component装饰器种使用inputs属性指定组件的输入。当组件从积累继承属性时，这可能很有用：

`// 'CustomSlider' inherits the disabled property from 'BaseSlider'.`
`@Component({`
  `...,`
  `inputs: ['disabled'],`
`})`
`export class CustomSlider extends BaseSlider { }`

你还可以通过在字符串种的冒号后方式别名来在inputs列表中指定输入别名：
`// 'CustomSlider' inherits the disabled property from 'BaseSlider'.`
`@Component({`
  `...,`
  `inputs: ['disabled: sliderDisabled'],`
`})`
`export class CustomSlider extends BaseSlider { }`



#### 输出
@Output装饰器

Angular自定义事件不会冒泡至DOM
在扩展组件类时，输出会被子类继承

##### 自定义输出名称
@Output接受一个参数，让你在模板中指定事件的不同名称（别名）：
`@Component({...})`
`export class CustomSlider {`
  `@Output('valueChanged') changed = new EventEmitter<number>();`
`}`

`<custom-slider (valueChanged)="saveVolume()" />`

##### 在@Component装饰器中指定输出
除了@Output装饰器外，你还可以在@Component装饰器中的outputs属性指定组件的输出。这在组件从基类继承属性时非常有用:
`// CustomSlider inherits the valueChanged property from BaseSlider.`
`@Component({`
  `...,`
  `outputs: ['valueChanged'],`
`})`
`export class CustomSlider extends BaseSlider {}`

别名：
`// CustomSlider inherits the valueChanged property from BaseSlider.`
`@Component({`
  `...,`
  `outputs: ['valueChanged: volumeChanged'],`
`})`
`export class CustomSlider extends BaseSlider {}`


#### output()函数
output()函数声明指令或组件中的一个输出。输出允许你向父组件发出值。

例子：
`import {Component, output} from '@angular/core';`
`@Component({...})`
`export class MyComp {`
  `onNameChange = output<string>()    // OutputEmitterRef<string>`
  `setNewName(newName: string) {`
    `this.onNameChange.emit(newName);`
  `}`
`}`

`<my-comp (onNameChange)="showNewName($event)" />`

别名：
`class MyComp {`
  `onNameChange = output({alias: 'ngxNameChange'});`
`}`


##### 编程订阅
消费者可以使用对ComponentRef的引用动态创建你的组件。在这些情况下，父组件可以通过直接访问类型为OutputRef的属性来订阅输出。
`const myComp = viewContainerRef.createComponent(...);`
`myComp.instance.onNameChange.subscribe(newName => {`
  `console.log(newName);`
`});`

当销毁myComp时，Angular会自动清理订阅。或者，会返回一个显示取消订阅的函数的对象，以便更早地取消订阅。


##### 为什么应该使用output()而不是基于装饰器的@Output()?
output()函数相比基于装饰器的@Output和EventEmitter提供了许多好处：
1. 更简单的思维模型和API：
   - 没有错误通道、完成通道或其它来自RxJS的API概念。
   - 输出是简单的发射器。你可以使用.emit函数发射值
2. 更准确的类型：
   - OutputEmitterRef.emit(value)现在拥有正确的类型，而EventEmitter具有破损的类型，可能导致运行时错误。


#### 宿主元素
Angular会为每个与组件选择器匹配的HTML元素创建一个组件实例。与组件选择器匹配的DOM元素是该组件的宿主元素。组件模板的内容在其宿主元素内部渲染。


##### 绑定到宿主元素
组件可以将属性（Properties）、属性（Attributes）和事件绑定到其宿主元素。这与在模板内的元素上定义绑定完全相同，但需要在@Component装饰器中的host属性中定义：
`@Component({`
  `...,`
  `host: {`
    `'role': 'slider',`
    `'[attr.aria-valuenow]': 'value',`
    `'[tabIndex]': 'disabled ? -1 : 0',`
    `'(keydown)': 'updateValue($event)',`
  `},`
`})`
`export class CustomSlider {`
  `value: number = 0;`
  `disabled: boolean = false;`
  `updateValue(event: KeyboardEvent) { /* ... */ }`
  `/* ... */`
`}`


##### @HostBinding与@HostListener装饰器
你可以通过将@HostBinding和@HostListener装饰器应用于类成员来替代绑定到宿主元素。

@HostBinding允许你将宿主属性和属性绑定到属性和方法：
`@Component({`
  `/* ... */`
`})`
`export class CustomSlider {`
  `@HostBinding('attr.aria-valuenow')`
  `value: number = 0;`
  `@HostBinding('tabIndex')`
  `getTabIndex() {`
    `return this.disabled ? -1 : 0;`
  `}`
  `/* ... */`
`}`

@HostListener允许你将事件监听器绑定到宿主元素。该装饰器接受一个使事件名称和一个可选的参数数组：
`export class CustomSlider {`
  `@HostListener('keydown', ['$event'])`
  `updateValue(event: KeyboardEvent) {`
    `/* ... */`
  `}`
`}`

注：
始终优秀使用host属性，而不是@HostBinding和@HostListener

##### 绑定冲突
https://angular.cn/guide/components/host-elements#binding-collisions



#### 生命周期
https://angular.cn/guide/components/lifecycle



#### 使用查询引用组件的子元素

##### 视图查询
@ViewChild装饰器：
`@Component({`
  `selector: 'custom-card-header',`
  `...`
`})`
`export class CustomCardHeader {`
  `text: string;`
`}`
`@Component({`
  `selector: 'custom-card',`
  `template: '<custom-card-header>Visit sunny California!</custom-card-header>',`
`})`
`export class CustomCard {`
  `@ViewChild(CustomCardHeader) header: CustomCardHeader;`
  `ngAfterViewInit() {`
   `console.log(this.header.text);`
  `}`
`}`

在这个示例中， `CustomCard` 组件查询子元素 `CustomCardHeader` 并在 `ngAfterViewInit` 中访问结果。

如果查询没有找到结果，它的值为 `undefined`。这可能发生在目标元素被 `NgIf` 隐藏时。Angular 会随着应用状态的变化更新 `@ViewChild` 的结果

**视图查询结果在 `ngAfterViewInit` 生命周期方法中可用**。在此之前，值为 `undefined`

`你也可以使用 @ViewChildren 装饰器查询多个结果`

`@Component({`
  `selector: 'custom-card-action',`
  `...,`
`})`
`export class CustomCardAction {`
  `text: string;`
`}`
`@Component({`
  `selector: 'custom-card',`
  `template: `
    `<custom-card-action>Save</custom-card-action>`
    `<custom-card-action>Cancel</custom-card-action>`
  `,`
`})`
`export class CustomCard {`
  `@ViewChildren(CustomCardAction) actions: QueryList<CustomCardAction>;`
  `ngAfterViewInit() {`
    `this.actions.forEach(action => {`
      `console.log(action.text);`
    `});`
  `}`
`}`

`@ViewChildren` 创建一个包含查询结果的 `QueryList` 对象。你可以通过 `changes` 属性订阅查询结果的变化

**查询永远不会穿过组件边界。** 视图查询只能从组件模板中检索结果。


##### 内容查询
内容查询从组件的 *内容* 中的元素中检索结果 — 即在组件模板中使用的嵌套在其中的元素。你可以使用 `@ContentChild` 装饰器查询单个结果。

`@Component({`
  `selector: 'custom-toggle',`
  `...`
`})`
`export class CustomToggle {`
  `text: string;`
`}`
`@Component({`
  `selector: 'custom-expando',`
  `...`
`})`
`export class CustomExpando {`
  `@ContentChild(CustomToggle) toggle: CustomToggle;`
  `ngAfterContentInit() {`
    `console.log(this.toggle.text);`
  `}`
`}`
`@Component({`
  `selector: 'user-profile',`
  `template: `
    `<custom-expando>`
      `<custom-toggle>Show</custom-toggle>`
    `</custom-expando>`
  ``
`})`


在这个示例中， `CustomExpando` 组件查询子元素 `CustomToggle` 并在 `ngAfterContentInit` 中访问结果。

如果查询没有找到结果，它的值为 `undefined`。这可能发生在目标元素不存在或被 `NgIf` 隐藏时。Angular 会随着应用状态的变化更新 `@ContentChild` 的结果。

默认情况下，内容查询仅查找组件的 _直接_子元素，不会遍历到后代元素。

**内容查询结果在 `ngAfterContentInit` 生命周期方法中可用**。在此之前，值为 `undefined`。请参阅 [生命周期](https://angular.cn/guide/components/lifecycle) 部分，了解更多关于组件生命周期的细节。

你也可以使用 `@ContentChildren` 装饰器查询多个结果。

`@Component({`
  `selector: 'custom-menu-item',`
  `...`
`})`
`export class CustomMenuItem {`
  `text: string;`
`}`
`@Component({`
  `selector: 'custom-menu',`
  `...,`
`})`
`export class CustomMenu {`
  `@ContentChildren(CustomMenuItem) items: QueryList<CustomMenuItem>;`
  `ngAfterContentInit() {`
    `this.items.forEach(item => {`
      `console.log(item.text);`
    `});`
  `}`
`}`
`@Component({`
  `selector: 'user-profile',`
  `template: ’
    `<custom-menu>`
      `<custom-menu-item>Cheese</custom-menu-item>`
      `<custom-menu-item>Tomato</custom-menu-item>`
    `</custom-menu>`
  `‘
`})`

`@ContentChildren` 创建一个包含查询结果的 `QueryList` 对象。你可以通过 `changes` 属性订阅查询结果的变化。

**查询永远不会跨越组件边界。** 内容查询只能从与组件本身相同的模板中检索结果。


##### 查询定位器
每个查询装饰器的第一个参数是它的定位器。
大多数情况下，你想要使用组件或指令作为你的定位器。
你也可以指定一个字符串定位器，对应于模板引用变量。

`@Component({`
  `...,`
  `template: `
    `<button #save>Save</button>`
    `<button #cancel>Cancel</button>`
  ``
`})`
`export class ActionBar {`
  `@ViewChild('save') saveButton: ElementRef<HTMLButtonElement>;`
`}`

如果有多个元素定义了相同的模板引用变量，查询会检索第一个匹配的元素。

Angular 不支持 CSS 选择器作为查询定位器。


##### 查询和注入器树
对于高级的情况，你可以使用任何ProviderToken作为定位器。这让你能够基于组件和指令提供者定位元素。

`const SUB_ITEM = new InjectionToken<string>('sub-item');`
`@Component({`
  `...,`
  `providers: [{provide: SUB_ITEM, useValue: 'special-item'}],`
`})`
`export class SpecialItem { }`
`@Component({...})`
`export class CustomList {`
  `@ContentChild(SUB_ITEM) subItemType: string;`
`}`

上面的示例使用了一个 `InjectionToken` 作为定位器，但你可以使用任何 `ProviderToken` 来定位特定元素。

##### 查询选项
所有查询装饰器都接受一个选项对象作为第二个参数。这些选项控制查询如何查找其结果。


**静态查询**
@ViewChild和@ContentChild查询接受static选项。

`@Component({`
  `selector: 'custom-card',`
  `template: '<custom-card-header>Visit sunny California!</custom-card-header>',`
`})`
`export class CustomCard {`
  `@ViewChild(CustomCardHeader, {static: true}) header: CustomCardHeader;`
  `ngOnInit() {`
    `console.log(this.header.text);`
  `}`
`}`

通过设置static: true，你可以向Angular保证此查询的目标 总是存在且不是有条件渲染的。这使得结果在ngOnInit声明周期方法中更早可用。
静态查询结果在初始化后不会更新。
@ViewChildren和@ContentChildren查询不支持static选项


**内容后代**
默认情况下，内容查询进查找组件的直接子元素，不会遍历到后代元素。
`@Component({`
  `selector: 'custom-expando',`
  `...`
`})`
`export class CustomExpando {`
  `@ContentChild(CustomToggle) toggle: CustomToggle;`
`}`
`@Component({`
  `selector: 'user-profile',`
  `template: `
    `<custom-expando>`
      `<some-other-component>`
        `<!-- custom-toggle will not be found! -->`
        `<custom-toggle>Show</custom-toggle>`
      `</some-other-component>`
    `</custom-expando>`
  ``
`})`

在上述示例中，CustomExpando无法找到`<custom-toggle>`，因为它不是`<custom-expando>`的直接子元素。通过设置descendants: true，你可以配置查询以遍历统一模板中的所有后代。然而，查询永远不会穿透组件以遍历其他模板中的元素。

视图查询没有此选项，因为它们总是遍历后代。


**从元素注入器中读取特定值**
默认情况下，查询定位器指示了你要查找的元素和检索到的值。你也可以指定read选项，以从定位器匹配的元素中检索不同的值。

`@Component({...})`
`export class CustomExpando {`
  `@ContentChild(ExpandoContent, {read: TemplateRef}) toggle: TemplateRef;`
`}`

上面的示例定位了一个带有指令ExpandoContent的元素，并检索与该元素关联的TemplateRef。
开发者最常用read来检索ElementRef和TemplateRef。


##### 使用QueryList
`@ViewChildren` 和 `@ContentChildren` 都提供了一个包含结果列表的 `QueryList` 对象。

`QueryList` 提供了许多方便的 API，以数组方式处理结果，如 `map`、 `reduce` 和 `forEach`。你可以通过调用 `toArray` 来获得当前结果的数组。

你可以订阅 `changes` 属性，在结果发生变化时执行某些操作。



#### 使用DOM API
Angular为你处理了大部分的DOM创建、更新和移除操作。然而，你可能会偶尔需要直接与某个组件的DOM交互。组件可以注入ElementRef以获取该组件宿主元素的引用：
`@Component({...})`
`export class ProfilePhoto {`
  `constructor(elementRef: ElementRef) {`
    `console.log(elementRef.nativeElement);`
  `}`
`}`

nativeElement属性引用了宿主Element实例。

你可以使用Angular的afterRender和afterNextRender函数来注册一个渲染回调，该回调会在Angular完成页面渲染后运行。

`@Component({...})`
`export class ProfilePhoto {`
  `constructor(elementRef: ElementRef) {`
    `afterRender(() => {`
      `// Focus the first input element in this component.`
      `elementRef.nativeElement.querySelector('input')?.focus();`
    `});`
  `}`
`}`

afterRender和afterNextRender必须在注入上下文中调用，通常是在组件的构造函数中。
**尽量避免直接操作DOM**。始终优先在组件模板中表达你的DOM结构，并使用绑定来更新该DOM。

在服务器端渲染或构建时预渲染期间，渲染回调永远不会运行。

**绝不要在其它Angular生命周期钩子中直接操作DOM**。Angular不保证组件的DOM在任何时候（除了渲染回调中）是完全渲染的。此外，在其他生命周期钩子中读取或修改DOM会通过引起局部抖动，对页面性能产生负面影响。


##### 使用组件的渲染器
组件可以注入Renderer2的实例以执行某些其它Angular特性相关的DOM操作。

由组件的Renderer2创建的任何DOM元素都会参与该组件的样式封装。

某些Renderer2API也与Angular的动画系统相关。你可以使用setProperty方法来更新合成动画属性，并使用listen方法为合成动画事件添加事件监听器。

除了这两个狭隘的用例之外，使用Renderer2与使用原生DOM API没区别。Renderer2 API不支持在服务器端渲染或构建时预渲染上下文中进行DOM操作。

![[Pasted image 20240820105532.png]]



#### 继承
Angular组件是TypeScript类，并参与标准JavaScript继承语义。
组件可以扩展任何基类：

`export class ListboxBase {`
  `value: string;`
`}`
`@Component({ ... })`
`export class CustomListbox extends ListboxBase {`
  `// CustomListbox inherits the value property.`
`}`

##### 扩展其他组件和指令
当组件扩展另一个组件或指令时，它继承了基类装饰器中定义的所有元数据以及基类装饰器的装饰成员。这包括选择器、模板、样式、宿主绑定、输入、输出、生命周期方法和任何其他设置。

`@Component({`
  `selector: 'base-listbox',`
  `template: `
    `...`
  `,`
  `host: {`
    `'(keydown)': 'handleKey($event)',`
  `},`
`})`
`export class ListboxBase {`
  `@Input() value: string;`
  `handleKey(event: KeyboardEvent) {`
    `/* ... */`
  `}`
`}`
`@Component({`
  `selector: 'custom-listbox',`
  `template: `
    `...`
  `,`
  `host: {`
    `'(click)': 'focusActiveOption()',`
  `},`
`})`
`export class CustomListbox extends ListboxBase {`
  `@Input() disabled = false;`
  `focusActiveOption() {`
    `/* ... */`
  `}`
`}`

CustomListBox继承了与ListBoxBase相关的所有信息，用自己的值覆盖了选择器和模板。CustomListBox有两个输入（value和disabled）和两个事件监听器（keydown和click）。

子类最终拥有所有祖先的输入、输出和宿主绑定以及自己的并集。


##### 转发注入的依赖项
如果基类依赖于依赖注入，子类必须显式的将这些依赖项传递给super。

`@Component({ ... })`
`export class ListboxBase {`
  `constructor(private element: ElementRef) { }`
`}`
`@Component({ ... })`
`export class CustomListbox extends ListboxBase {`
  `constructor(element: ElementRef) {`
    `super(element);`
  `}`
`}`


##### 重写生命周期方法
如果基类定义了一个生命周期方法，比如ngOnInit，子类也是先了ngOnInit，则子类覆盖基类的实现。如果要保留基类的生命周期方法，请显式使用super调用该方法:
`@Component({ ... })`
`export class ListboxBase {`
  `protected isInitialized = false;`
  `ngOnInit() {`
    `this.isInitialized = true;`
  `}`
`}`
`@Component({ ... })`
`export class CustomListbox extends ListboxBase {`
  `override ngOnInit() {`
    `super.ngOnInit();`
    `/* ... */`
  `}`
`}`



#### 以编程方式渲染组件
除了在模板中直接使用组件外，你还可以动态渲染组件。有两种方式可以动态渲染组件：在模板中使用NgComponentOutlet，或者在你的TypeScript代码中使用ViewContainerRef。

##### 使用NgComponentOutlet

NgComponentOutlet是一个结构型指令，可以在模板中动态渲染给定的组件。
`@Component({ ... })`
`export class AdminBio { /* ... */ }`
`@Component({ ... })`
`export class StandardBio { /* ... */ }`
`@Component({`
  `...,`
  `template: `
    `<p>Profile for {{user.name}}</p>`
    `<ng-container *ngComponentOutlet="getBioComponent()" /> `
`})`
`export class CustomDialog {`
  `@Input() user: User;`
  `getBioComponent() {`
    `return this.user.isAdmin ? AdminBio : StandardBio;`
  `}`
`}`

##### 使用ViewContainerRef
视图容器是Angular组件树中可以包含内容的节点。任何组件或指令都可以注入ViewContainerRef以获取与该组件或指令在DOM中位置对应的视图容器的引用。
你可以在ViewContainerRef上使用createComponent方法来动态创建和渲染组件。当你使用ViewContainerRef创建新组件时，Angular会将其**附加到注入ViewContainerRef的组件或指令的下一个同级位置**。

`@Component({`
  `selector: 'leaf-content',`
  `template: `
    `This is the leaf content`
  `,`
`})`
`export class LeafContent {}`
`@Component({`
  `selector: 'outer-container',`
  `template: `
    `<p>This is the start of the outer container</p>`
    `<inner-item />`
    `<p>This is the end of the outer container</p>`
  `,`
`})`
`export class OuterContainer {}`
`@Component({`
  `selector: 'inner-item',`
  `template: `
    `<button (click)="loadContent()">Load content</button>`
  `,`
`})`
`export class InnerItem {`
  `constructor(private viewContainer: ViewContainerRef) {}`
  `loadContent() {`
    `this.viewContainer.createComponent(LeafContent);`
  `}`
`}`

在上面示例中，点击“加载内容”按钮会导致以下DOM结构：

`<outer-container>`
  `<p>This is the start of the outer container</p>`
  `<inner-item>`
    `<button>Load content</button>`
  `</inner-item>`
  `<leaf-content>This is the leaf content</leaf-content>`
  `<p>This is the end of the outer container</p>`
`</outer-container>`

##### 惰性加载组件
你可以使用上述描述的两种方法，NgComponentOutlet和ViewContainerRef，来使用标准的JavaScript动态导入来惰性加载组件。

`@Component({`
  `...,`
  `template: `
    `<section>`
      `<h2>Basic settings</h2>`
      `<basic-settings />`
    `</section>`
    `<section>`
      `<h2>Advanced settings</h2>`
      `<button (click)="loadAdvanced()" *ngIf="!advancedSettings">`
        `Load advanced settings`
      `</button>`
      `<ng-container *ngComponentOutlet="advancedSettings" />`
    `</section>`
`})`
`export class AdminSettings {`
  `advancedSettings: {new(): AdminSettings} | undefined;`
  `async loadAdvanced() {`
    `this.advancedSettings = await import('path/to/advanced_settings.js');`
  `}`
`}`

上面的示例在接收到按钮点击时加载并显示AdvancedSettings。



#### 高级配置


#### 自定义查询
https://angular.cn/guide/elements