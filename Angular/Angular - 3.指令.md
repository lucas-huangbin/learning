![[Pasted image 20240823135525.png]]
#### 概述
##### 内置的属性型指令
属性型指令会监听并修改其它HTML元素和组件的行为、Attribute和Property。
最常见的属性型指令如下：
NgClass：添加和删除一组CSS类
NgStyle：添加和删除一组HTML样式
NgModel：为HTML表单元素添加双向数据绑定

##### 使用NgClass添加和移除类

###### 在组件中导入NgClass
要使用NgClass，请将其添加到组件的imports列表中。

`import {Component, OnInit} form '@angular/core';`

`@Component({`
  `standalone: true,`
  `selector: 'app-root',`
  `templateUrl: './app.component.html',`
  `styleUrls: ['./app.component.css'],`
  `imports: [`
  `NgClass， // <-- import into the component`
  `NgStyle, // <-- import into the component`
  `]`
`})`
`export class AppComponent implements OnInit {}`


###### 使用NgClass

例如：
`<a [NgClass]="isSpecial ? 'aClass' : 'bClass'"> </a>`


#### 属性型指令

##### 构建属性型指令
1. 要创建指令，请使用CLI命令 ng generate directive.
   `ng generate directive highlight`
   CLI会创建`src/app/highlight.directive.ts` 和对应的测试文件 `src/app/highlight.directive.spec.ts`。
   
   `import {Directive} from '@angular/core';`
	`@Directive({`
	  `standalone: true,`
	  `selector: '[appHighlight]',`
	`})`
	`export class HighlightDirective {}`
	
	@Directive()装饰器的配置属性指定了指令的CSS属性选择器，`[appHighlight]`。
2. 从 `@angular/core` 导入 `ElementRef`。 `ElementRef` 通过其 `nativeElement` 属性直接访问宿主 DOM 元素。
3. 在指令的 `constructor()` 中添加 `ElementRef`，以 [注入](https://angular.cn/guide/di)对宿主 DOM 元素的引用，即你应用 `appHighlight` 的元素。
4. 向`HighlightDirective`类中添加逻辑，将背景设置为黄色。
   `import {Directive, ElementRef} from '@angular/core';`
	`@Directive({`
	  `standalone: true,`
	  `selector: '[appHighlight]',`
	`})`
	`export class HighlightDirective {`
	  `constructor(private el: ElementRef) {`
		    `this.el.nativeElement.style.backgroundColor = 'yellow';`
	  `}`
	`}`

注： 指令不支持命名空间。
`<p app:Highlight>This is invalid</p>`

##### 应用属性型指令
`<p appHighlight>Highlight me!</p>`

angular会创建HighlightDirective类的实例，并将`<p>`元素的引用注入到该指令的构造函数中，它会将`<p>`元素的背景样式设置为黄色。


##### 处理用户事件
本节展示如何检测用户何时将鼠标移入或移除元素以及如何通过设置或清除突出显示颜色来进行相应。
1. 从'@angular/core'导入HostListener。
2. 添加两个事件处理器，当鼠标进入或离开时相应，每个处理器都有@HostListener()装饰器。

`import {Directive, ElementRef, HostListener} from '@angular/core';`
`import {Input} from '@angular/core';`
`@Directive({`
  `standalone: true,`
  `selector: '[appHighlight]',`
`})`
`export class HighlightDirective {`
  `constructor(private el: ElementRef) {}`
  `@HostListener('mouseenter') onMouseEnter() {`
    `this.highlight('yellow');`
  `}`
  `@HostListener('mouseleave') onMouseLeave() {`
    `this.highlight('');`
  `}`
  `private highlight(color: string) {`
    `this.el.nativeElement.style.backgroundColor = color;`
  `}`
`}`

使用 `@HostListener()` 装饰器订阅宿主属性型指令的 DOM 元素事件，在此例中为 `<p>`。


##### 将值传递给属性型指令

TS：
`import {Component} from '@angular/core';`
`import {HighlightDirective} from './highlight.directive';`
`@Component({`
  `standalone: true,`
  `selector: 'app-root',`
  `templateUrl: './app.component.1.html',`
  `imports: [HighlightDirective],`
`})`
`export class AppComponent {`
  `color = 'yellow';`
`}`

HTML：
`<h1>My First Attribute Directive</h1>`
`<h2>Pick a highlight color</h2>`
`<div>`
  `<input type="radio" name="colors" (click)="color='lightgreen'">Green`
  `<input type="radio" name="colors" (click)="color='yellow'">Yellow`
  `<input type="radio" name="colors" (click)="color='cyan'">Cyan`
`</div>`
`<p [appHighlight]="color">Highlight me!</p>`
`<p [appHighlight]="color" defaultColor="violet">`
  `Highlight me too!`
`</p>`
`<hr />`
`<h2>Mouse over the following lines to see fixed highlights</h2>`
`<p [appHighlight]="'yellow'">Highlighted in yellow</p>`
`<p appHighlight="orange">Highlighted in orange</p>`
`<hr />`
`<h2>ngNonBindable</h2>`
`<p>Use ngNonBindable to stop evaluation.</p>`
`<p ngNonBindable>This should not evaluate: {{ 1 + 1 }}</p>`
`<h3>ngNonBindable with a directive</h3>`
`<div ngNonBindable [appHighlight]="'yellow'">This should not evaluate: {{ 1 +1 }}, but will highlight yellow.`
`</div>`


##### 绑定到第二个属性

例如：
`import {Directive, ElementRef, HostListener, Input} from '@angular/core';`
`@Directive({`
  `standalone: true,`
  `selector: '[appHighlight]',`
`})`
`export class HighlightDirective {`
  `constructor(private el: ElementRef) {}`
  `@Input() defaultColor = '';`
  `@Input() appHighlight = '';`
  `@HostListener('mouseenter') onMouseEnter() {`
    `this.highlight(this.appHighlight || this.defaultColor || 'red');`
  `}`
  `@HostListener('mouseleave') onMouseLeave() {`
    `this.highlight('');`
  `}`
  `private highlight(color: string) {`
    `this.el.nativeElement.style.backgroundColor = color;`
  `}`
`}`

HTML:
`<p [appHighlight]="color" defaultColor="violet">`
  `Highlight me too!`
`</p>`

与组件一样，你可以将指令的多个属性绑定到宿主元素上。

##### 通过NgNonBindable停用Angular处理过程
要防止在浏览器中进行表达式求值，请将ngNonBindable添加到宿主元素。ngNonBindable会停用模板中的插值、指令和绑定。

在下面的示例中，表达式`{{1 + 1}}`的渲染方式会和在代码编辑器的一样，而不会显示2.

`<p>Use ngNonBindable to stop evaluation.</p>`
`<p ngNonBindable>This should not evaluate: {{ 1 + 1 }}</p>`

将ngNonBindable应用于元素将停止对该元素的子元素的绑定。但是，ngNonBindable仍然允许指令在应用ngNonBindable的元素上工作。在以下示例中，appHighlight指令仍然处于活跃状态，但Angular不会对表达式`{{1 + 1}}`求值。

`<h3>ngNonBindable with a directive</h3>`
`<div ngNonBindable [appHighlight]="'yellow'">This should not evaluate: {{ 1 +1 }}, but will highlight yellow.`
`</div>`

如果将ngNonBindable应用于父元素，则Angular会禁用该元素的子元素的任何插值和绑定，比如属性绑定或事件绑定。


#### 结构型指令
结构型指令是应用于`<ng-template>`元素的指令，这些指令有条件地或反复地渲染该`<ng-template>`的内容。

注：Angular的`<ng-template>`元素定义了一种默认情况下不渲染任何内容的模板，如果你只是将元素包裹在`<ng-template>`中而不引用结构型指令，这些元素将不会被渲染。

##### 结构型指令简写形式
Angular支持结构型指令的简写语法，这避免了显式编写`<ng-template>`元素的需要。

可以通过在指令属性选择器前加上星号（`*`）直接在元素上应用结构型指令，例如`*select`。Angular会将结构型指令前的幸好转换为包含该指令的`<ng-template>`，冰包围该元素及其后代。


##### 创建结构型指令
1. 生成指令
   `ng generate directive select`
2. 将此指令变为结构型指令
   导入TemplateRef和ViewContainerRef。在指令构造函数中将TemplateRef和ViewContainerRef注入为私有变量。
   `import {Directive, TemplateRef, ViewContainerRef} from '@angular/core';`
	`@Directive({`
	  `standalone: true,`
	  `selector: 'select',`
	`})`
	`export class SelectDirective {`
	  `constructor(private templateRef: TemplateRef, private ViewContainerRef: ViewContainerRef) {}`
	`}`
3. 添加selectForm输入属性
   `export class SelectDirective {`
	  `// ...`
	  `@Input({required: true}) selectFrom!: DataSource;`
	`}`
4. 添加业务逻辑
   现在SelectDirective已作为带有输入属性的结构型指令搭建好，你可以添加逻辑来获取数据并用它渲染模板：
   `export class SelectDirective {`
	  `// ...`
	  `async ngOnInit() {`
	    `const data = await this.selectFrom.load();`
	    `this.viewContainerRef.createEmbeddedView(this.templateRef, {`
	      `// Create the embedded view with a context object that contains`
	      `// the data via the key $implicit.`
	      `$implicit: data,`
	    `});`
	  `}`
	`}`

##### 结构型指令语法参考
当你编写自己的结构型指令时，请使用以下语法：
`*:prefix="( :let | :expression ) (';' | ',')? ( :let | :as | :keyExp )*"`

以下模式描述了结构型指令语法的每个部分：
`as = :export "as" :local ";"?`
`keyExp = :key ":"? :expression ("as" :local)? ";"?`
`let = "let" :local "=" :export ";"?`

![[Pasted image 20240823163652.png]]
![[Pasted image 20240823171621.png]]


#### 组合指令API

##### 向组件添加指令
你可以通过hostDirectives属性添加到组件的装饰器来将指令应用于组件。我们称这样的指令为宿主指令。

在此示例中，我们将指令MenuBehavior应用五AdminMenu的宿主元素。这类似于将MenuBehavior应用于模板的`<admin-menu>`元素。

`@Component({`
  `standalone: true,`
  `selector: 'admin-menu',`
  `template: 'admin-menu.html',`
  `hostDirectives: [MenuBehavior],`
`})`
`export class AdminMenu { }`


当框架渲染组件时，Angular还会创建每个宿主指令的实例。这些指令的宿主绑定应用于宿主元素。默认情况下，宿主指令的输入和输出不会作为组件公共API的一部分公开。

**Angular会在编译时静态应用宿主指令**。你不能在运行时动态添加指令。
**在hostDirectives中使用的指令必须是standalone：true**。

**Angular会忽略应用在hostDirectives属性中的指令selector**。




##### 包含输入属性和输出属性
默认情况下，当你将hostDirectives应用于组件时，宿主指令的输入属性和输出属性不会包含在组件的API中。你可以通过扩展hostDirectives中的条目来在组件的API中显示包含输入和输出：
`@Component({`
  `standalone: true,`
  `selector: 'admin-menu',`
  `template: 'admin-menu.html',`
  `hostDirectives: [{`
    `directive: MenuBehavior,`
    `inputs: ['menuId'],`
    `outputs: ['menuClosed'],`
  `}],`
`})`
`export class AdminMenu { }`

通过显式指定输入和输出，使用 `hostDirective` 的组件的使用者可以将它们绑定在模板中：

`<admin-menu menuId="top-menu" (menuClosed)="logMenuClosed()">`