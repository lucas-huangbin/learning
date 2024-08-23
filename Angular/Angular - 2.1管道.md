#### 在模板中使用管道
##### Pipe的附加参数
Pipe可以接受附加参数来配置转换。参数可以是可选的或必要的。
例子：
`<p>The hero's birthday is in {{ birthday | date:'yyyy' }}</p>`

Pipe也可以接收多个参数，你可以通过冒号（:）分隔这些参数来传递多个参数。
例子：
`<p>The current time is: {{ currentTime | date:'hh:mm':'UTC' }}</p>`


##### 链式调用Pipe
例子：
`<p>The hero's birthday is {{ birthday | date }}</p>`
`<p>The hero's birthday is in {{ birthday | date:'yyyy' | uppercase }}</p>`


#### 自定义管道

##### 将类标记为管道
要将类标记为pipe并提供配置元数据，请将@Pipe应用于该类

`import { Pipe } from '@angular/core';`
`@Pipe({`
  `name: 'greet',`
  `standalone: true,`
`})`
`export class GreetPipe {}`


##### 使用PipeTransform接口
在你的自定义管道类中实现PipeTransform接口以执行转换。

Angular调用transform方法，该方法使用绑定的值作为第一个参数，把其他任何参数都以列表的形式作为第二个参数，并返回转换后的值。
`import { Pipe, PipeTransform } from '@angular/core';`
`@Pipe({`
  `name: 'greet',`
  `standalone: true,`
`})`
`export class GreetPipe implements PipeTransform {`
  `transform(value: string, param1: boolean, param2: boolean): string {`
    `return Hello ${value};`
  `}`
`}`

##### 模板表达式中的管道优先级

**管道操作符优先级高于三元运算符**
例如：
`condition ? a : b | pipe`
解析为：
`condition ? a : (b | pipe)`

#### 管道的变更检测
Angular在每次DOM事件后（每次按键、鼠标移动、计时器滴答和服务器响应）会在变更检测过程中寻找数据绑定值的变化
纯管道检测不到引用类型内部的变化

##### 检测复合对象中的非纯变更
要在符合对象内部（例如数组的一个元素）发生变化后执行自定义管道，你需要将管道定义为impure以检测非纯变化。Angular每次检测到变化时（例如每次案件或鼠标事件）都会执行非纯管道。

注：虽然非纯管道可能很有用，但是一个长时间运行的非纯管道可能会显著拖慢你的应用。


`import {Pipe, PipeTransform} from '@angular/core';`
`import {Hero} from './heroes';`
`@Pipe({`
  `standalone: true,`
  `name: 'flyingHeroes',`
`})`
`export class FlyingHeroesPipe implements PipeTransform {`
  `transform(allHeroes: Hero[]) {`
    `return allHeroes.filter((hero) => hero.canFly);`
  `}`
`}`
`/////// Identical except for the pure flag`
`@Pipe({`
  `standalone: true,`
  `name: 'flyingHeroesImpure',`
  `pure: false,`
`})`
`export class FlyingHeroesImpurePipe extends FlyingHeroesPipe {}``



#### 从观察者中解包数据
使用内置的AsyncPipe接受一个可观察者作为输入并自动订阅该输入。如果没有这个管道，你的组件代码将不得不订阅可观察者以消费其值、提取求解后的值、将其暴露给绑定，并在可观察者销毁时取消订阅以防止内存泄漏。AsyncPipe是一个管道，它在你都组件中节省了样板代码，以维护订阅并在这些值到达时继续传递来自该可观察者的值。

例子：
`import {Component} from '@angular/core';`
`import {AsyncPipe} from '@angular/common';`
`import {Observable, interval} from 'rxjs';`
`import {map, startWith, take} from 'rxjs/operators';`
`@Component({`
  `standalone: true,`
  `selector: 'app-hero-async-message',`
  `template: `
    `<h2>Async Messages and AsyncPipe</h2>`
    `<p>{{ message$ | async }}</p>`
    `<button type="button" (click)="resend()">Resend Messages</button>,`
  `imports: [AsyncPipe],`
`})`
`export class HeroAsyncMessageComponent {`
  `message$: Observable<string>;`
  `private messages = ['You are my hero!', 'You are the best hero!', 'Will you be my hero?'];`
  `constructor() {`
    `this.message$ = this.getResendObservable();`
  `}`
  `resend() {`
    `this.message$ = this.getResendObservable();`
  `}`
  `private getResendObservable() {`
    `return interval(1000).pipe(`
      `map((i) => `Message #${i + 1}: ${this.messages[i]}`),`
      `take(this.messages.length),`
      `startWith('Waiting for messages...'),`
    `);`
  `}`
`}`
