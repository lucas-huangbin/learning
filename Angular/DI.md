### **DI（Dependency Import）**
**DI(依赖注入)：** 一种设计模式，允许我们将一个对象（依赖）注入另一个对象

#### 1. InjectionToken
注入的依赖项不是一个类的实例，而是一个配置项、字符串、或其它非类的值。

https://open.alipay.com/portal/forum/post/138401031

**主要作用：**
1）唯一性标识： InjectionToken是一个唯一的标识符，确保依赖唯一性
2）非类依赖注入
3）提供器配置：通过提供器配置，我们可以告诉Angular如何为`InjectionToken`提供依赖项的实例。这使得我们可以在不同的上下文中为`InjectionToken`提供不同的值。

**示例1：**

创建一个名为`LANG_CONFIG`的InjectionToken：
`export const LANG_CONFIG = new InjectionToken<string>('langConfig')`
在模块中提供这个依赖项的实例：
`@NgModule({`
	`providers: [`
		`{`
			`provide: LANG_CONFIG,`
			`useValue: 'en-US'    //默认语言配置`
		`}`
	`]`
`})`
组件中注入配置：
`constructor(@Inject(LANG_CONFIG) private langConfig: string)`

同时，在定义InjectionToken的时候还可以设置providedIn 和 factory：
`export const TOKEN_FACTORY = new InjectionToken('factory-token', {`
    `providedIn: 'root',`
    `factory: () => {`
        `return 'I am from InjectionToken factory';`
    `}`
`});`

**应用举例**
1. 模块内部一些公共配置的共享，如：权限前缀

#### 2.@Injectable & providers

**@Injectable（可被注入的）:** service的装饰器，用于标记这个class具有DI的特性，只有service会打上这个装饰器
**注入器（Injector）** ：会创建依赖、维护一个容器来管理这些依赖，并尽可能复用他们，是依赖注入的底层机制
**@Inject()**：类构造函数中依赖项参数上的参数装饰器，用于指定依赖项的自定义提供者，参数传入DI Token，映射到要注入的依赖项。
**providers（提供者）** ：是ngModule、Component的传入参数，传入模组作为该模组的公用函数，每一次注入都会**重新**实例化service

**@Injectable**
providedIn: 'root',表示在整个应用所有地方都可以注入，并全局唯一实例
`@Injectable({
	providedIn: 'root'
})

**providers**`
·告诉注入器如何提供依赖
·限制服务可使用的范围
属性：
·provide属性是依赖令牌，他作为一个key，在定义依赖值和配置注入器时使用，可以是一个类的类型、InjectionToken、或者字符串，甚至对象，但是不能是一个Interface、数字和布尔类型
·第二个属性是一个提供者定义对象，它告诉注入器要如何创建依赖值。提供者定义对象中的key可以是useClass，也可以是useExisting、useValue、或useFactory，每一个key都用于提供一种不同类型的依赖

**1）类提供者（TypeProvider和ClassPrvoder）**
`providers: [ Logger ]   // 简写`
`providers: [{ provide: Logger, useClass: Logger }]  // 全写`
·provide: Logger 表示是把类的类型作为DI Token（依赖令牌）
·useClass表示使用此类实例化作为依赖值，其实就是通过new Logger()返回依赖值

使用场景：
·所有class定义的服务默认都是用**类提供者**
·指定替代性的类提供者，替换原有服务的行为实现可扩展性，这样在使用的时候还是注入Logger，但是实际返回的对象是BetterLogger 示例：
`[{provide: Logger, useClass: BetterLogger}]`
`// 当是使用Logger令牌请求Logger时，返回一个BetterLogger`

**2）别名类提供者（ExistingProvider）**
要为类提供者设置别名，请在providers数组中使用useExisting属性指定别名和类提供者。
在下面的例子中，当组件请求新的或旧的记录时，注入器都会注入一个NewLogger的实例。通过这种方式，OldLogger就成了NewLogger的别名。
`[ NewLogger,`
  `// Alias OldLogger w/ reference to NewLogger`
  `{ provide: OldLogger, useExisting: NewLogger}]`
如果使用useClass来把OldLogger设为NewLogger的别名，这样就会创建两个不同的NewLogger实例

**3）对象提供者（ValueProvider）**
列子：
`// An object in the shape of the logger service`
`function silentLoggerFn() {}`

`export const silentLogger = {`
  `logs: ['Silent logger says "Shhhhh!". Provided via "useValue"'],`
  `log: silentLoggerFn`
`};`

`[{ provide: Logger, useValue: silentLogger }]`

使用场景：
通过对象提供者注入一个配置对象，一般推荐使用InjectionToken作为令牌
`// src/app/app.config.ts`
`import { InjectionToken } from '@angular/core';`

`export const APP_CONFIG = new InjectionToken<AppConfig>('app.config');`

`export const HERO_DI_CONFIG: AppConfig = {`
  `apiEndpoint: 'api.heroes.com',`
  `title: 'Dependency Injection'`
`};`

`// src/app/app.module.ts (providers)`
`providers: [`
  `{ provide: APP_CONFIG, useValue: HERO_DI_CONFIG }`
`],`
在使用InjectionToken作为令牌时，在组建或者服务中必须借助参数装饰器@Inject()，才可以把这个配置对象注入到构造函数中。
`// src/app/app.component.ts`
`constructor(@Inject(APP_CONFIG) config: AppConfig) {`
  `this.title = config.title;`
`}`

**4）工厂提供者**
根据运行前尚不可用的信息创建可变的依赖值，可以用工厂提供者。也就是完全自己决定如何创建依赖。
例如：你不希望直接将UserService注入到HeroService中，因为你不希望把这个服务与那些高敏感的信息牵扯到一起
`// src/app/heroes/hero.service.ts (excerpt)`
`constructor(`
  `private logger: Logger,`
  `private isAuthorized: boolean) { }`

`getHeroes() {`
  `const auth = this.isAuthorized ? 'authorized ' : 'unauthorized';`
  `this.logger.log(Getting heroes for ${auth} user.);`
  `return HEROES.filter(hero => this.isAuthorized || !hero.isSecret);`
`}`

`// src/app/heroes/hero.service.provider.ts (excerpt)`

`const heroServiceFactory = (logger: Logger, userService: UserService) => {`
  `return new HeroService(logger, userService.user.isAuthorized);`
`};`

`export const heroServiceProvider =`
  `{` 
    `provide: HeroService,`
    `useFactory: heroServiceFactory,`
    `deps: [Logger, UserService]`
  `};`

·useFactory：字段指定该提供者是一个工厂函数，其实现代码是heroServiceFactory
·deps ：是一个提供者令牌数组，Logger和UserService类都是类提供者的令牌。该注入器解析了这些令牌，并把相应的服务注入到heroServiceFactory工厂函数的参数中
#### 3.控制反转和依赖注入
https://segmentfault.com/a/1190000040807826
例子：通过构造函数注入
`class Logger {`
    `log(message: string) {}`
`}`
`class HeroesService {`
    `constructor(logger: Logger) {}`
`}`

`const logger = new Logger();`
`const heroesService = new HeroesService(logger);`


控制反转容器（IoC Container）：通过HeroesService的构造函数寻找Logger的以来并实例化。
`const heroesService = IocContainer.get(HeroesService);`

如果类很多，依赖层级比较深，IocContainer会统一管理依赖，IocContainer也叫**注入器（Injector）**，在Angular中叫Injector


#### 多级注入器
通过依赖注入的概念我们知道，创建实例的工作都交给了Ioc容器（也就是注入器），通过构造函数参数装饰器@Inject(DIToken)告诉注入器我们需要注入DIToken对应的依赖，注入器就会帮我们查找依赖并返回值，Angular应用启动会默认创建相关的注入器，而且Angular的注入器有层级的，类似于Dom树

**1）两个注入器层级**
Angular中有两个注入器层次结构
·ModuleInjector层次结构 -- 使用@NgModule()或@Injectable()装饰器在此层次结构中配置ModuleInjector
·ElementInjector层次结构 -- 在每个DOM元素上隐式创建。除非你再@Directive() 或@Component()的providers属性中进行配置，否则默认情况下，ElementInjector为空

**ModuleInjector**
通过以下两种方式之一配置ModuleInjector：
·使用@Injectable()的providedIn属性引用NgModuleType、root、platform或者any
·使用@NgModule()的providers数组

*Tree-shaking and @Injectable() - 摇树优化与 @Injectable()*  

使用 `@Injectable()` 的 `providedIn` 属性优于 `@NgModule()` 的 `providers` 数组，因为使用 `@Injectable()` 的 `providedIn` 时，优化工具可以进行摇树优化，从而删除你的应用程序中未使用的服务，以减小捆绑包尺寸。 摇树优化对于库特别有用，因为使用该库的应用程序不需要注入它。在 服务与依赖注入简介了解关于可摇树优化的提供者的更多信息。
**·特别需要注意**
·ModuleInjector 由 @NgModule.providers和NgModule.imports属性配置。ModuleInjector是可以通过NgModule.imports递归找到的所有数组的扁平化
·子ModuleInjector是在惰性加载其它@NgModules时创建的
第一点的意思是AppModule导入了FeatureAModule和FeatureBModule，那么Angular会根据模块树找到所有模块配置的providers并打平存放到一起，这就意味着整个应用程序所有地方都可以注入这些打平的providers，
这就是 Angular 模块下的服务和组件/指令/管道所不同的地方，组件/指令/管道在某个模块定义，只要没有导出，其他模块都无法使用，必须导出才可以被导入该模块的组件使用
第二点可以不严谨的认为整个应用程序只有一个根模块注入器，只有通过路由的懒加载才会创建子模块注入器

**ElementInjector**
除了模块注入器外，Angular会为每个DOM元素饮食创建ElementInjector
可以用@Component()装饰器中的providers或viewProviders属性来配置ElementInjector以提供服务
`@Component({`
  `...`
  `providers: [{ provide: ItemService, useValue: { name: 'lamp' } }]`
`})`
`export class TestComponent`

·在组件中提供服务时，可以通过ElementInjector在该组件以及子组件/指令处通过注入令牌使用该服务
·当组件实例被销毁时，该服务实例也被销毁
·组件是一种特殊类型的指令，这意味着@Directive()和@Component()都具有providers属性

#### 4.解析规则
前面已经已经介绍了Angular会有两个层级的注入器，那么当组件或指令解析令牌时，Angular分为两个阶段来解析它：
·针对ElementInjector层次结构（其父级）
·针对 `ModuleInjector` 层次结构（其父级）
Angular 会先从当前组件/指令的 `ElementInjector` ​查找令牌，找不到会去父组件中查找，直到根组件，如果根还找不到，就去当前组件所在的模块注入器中查找，如果不是懒加载那么就是根注入器，一步一步到最顶层的 `NullInjector` ​

#### 5.解析修饰符
默认情况下， `Angular` 始终从当前的 `Injector` 开始，并一直向上搜索，修饰符可以更改开始（默认是自己）或结束位置，从而达到一些高级的使用场景，Angular 中可以使用 `@Optional()` ， `@Self()` ， `@SkipSelf()` 和 `@Host()` 来修饰 Angular 的解析行为，每个修饰符的说明如下：

**@Optional**
@Optional允许Angular将你注入的服务视为可选服务。这样，如果无法在运行时解析它，Angular只会将服务解析为null，而不会抛出错误
`export class OptionalComponent {`
  `constructor(@Optional() public optional?: OptionalService) {}`
`}`

**@Self()**
使用@Self()让Angular仅查看当前组件或指令的**ElementInjector**。
`@Component({`
  `selector: 'app-self',`
  `templateUrl: './self.component.html',`
  `styleUrls: ['./self.component.css'],`
  `providers: [{ provide: FlowerService, useValue: { emoji: '🌼' } }]`

`})`
`export class SelfComponent {`
  `constructor(@Self() public flower: FlowerService) {}`
`}`

和@Optional()组合使用
`@Component({`
  `selector: 'app-self-no-data',`
  `templateUrl: './self-no-data.component.html',`
  `styleUrls: ['./self-no-data.component.css']`
`})`
`export class SelfNoDataComponent {`
  `constructor(@Self() @Optional() public flower?: FlowerService) { }`
`}`

**@SkipSelf()**
@SkipSelf()与@Self()相反，使用@SkipSelf()，Angular在父ElementInjector中开始搜索服务，而不是从当前ElementInjector中开始搜索服务。
`@Injectable({`
    `providedIn: 'root'`
`})`
`export class FlowerService {`
    `emoji = '🌿';`
    `constructor() {}`
`}`
`import { Component, OnInit, SkipSelf } from '@angular/core';`
`import { FlowerService } from '../flower.service';`

`@Component({`
    `selector: 'app-skipself',`
    `templateUrl: './skipself.component.html',`
    `styleUrls: ['./skipself.component.scss'],`
    `providers: [{ provide: FlowerService, useValue: { emoji: '🍁' } }]`
`})`
`export class SkipselfComponent implements OnInit {`
    `constructor(@SkipSelf() public flower: FlowerService) {}`

    `ngOnInit(): void {}`
`}`

上面的示例会得到 root 注入器中的 🌿，而不是组件所在的 `ElementInjector` ​中提供的 🍁。  
如果值为 null 可以同时使用 `@SkipSelf()` 和 `@Optional()` 来防止错误。
`class Person {`
  `constructor(@Optional() @SkipSelf() parent?: Person) {}`
`}`

**@Host()**
@Host()使你可以在搜索提供者时将当前组件指定为注入器树的最后一站，即使树的更上级有一个服务实例，Angular也不会继续寻找

`@Component({`
  `selector: 'app-host',`
  `templateUrl: './host.component.html',`
  `styleUrls: ['./host.component.css'],`
  `//  provide the service`
  `providers: [{ provide: FlowerService, useValue: { emoji: '🌼' } }]`
`})`
`export class HostComponent {`
  `// use @Host() in the constructor when injecting the service`
  `constructor(@Host() @Optional() public flower?: FlowerService) { }`
`}`
@Host 和@Self的区别
@Host属性装饰器会禁止在宿主组件以上的搜索，宿主组件通常就是请求依赖的那个组件，不过，当该组件投影进莫格组件时，那个父组件就会变成宿主，意思就是ng-content中的组件所在的宿主组件不是自己，而是ng-content提供的父组件。

**注：**
所有修饰符都可以组合使用，但是不能互斥，比如：@Self & @SkipSelf，@Host()和@Self()

**providedIn: 'any' | 'root' | 'platform' | NgModuleType**
·root 表示在根模块注入器提供依赖
·platform 表示在平台注入器提供依赖
·指定模块表示在特定的特性模块提供依赖（注意循环依赖）
·any 所有急性加载的模块都会共享同一个服务单例，惰性加载模块各自有它们自己独有的单例



#### 6.高级技巧

##### 轻量级注入令牌 - Lightweight injection tokens

在开发类库的时候，支持摇树优化是一个重要的特性，要减少体积，那么在angular类库中需要做以下几点：
·分模块打包和导入，按钮模块和模态框模块分别打包
·服务尽量使用@Injectable({ providedIn: 'root' | 'any' | })优先
·使用轻量级注入Token

**令牌什么时候会被保留**
那么在同一个组件模块中，提供了很多个组件，如果只想打包被使用的组件如何做呢？
比如我么定义如下的一个card组件，包含了header，同时card组件中需要获取header组件示例。
`<lib-card>`
  `<lib-header>...</lib-header>`
`</lib-card>`

`@Component({`
  `selector: 'lib-header',`
  `...,`
`})`
`class LibHeaderComponent {}`

`@Component({`
  `selector: 'lib-card',`
  `...,`
`})`
`class LibCardComponent {`
  `@ContentChild(LibHeaderComponent)`
  `header: LibHeaderComponent|null = null;`
`}`

因为`<lib-header>`是可选的，所以元素可以用最小化的形式<lib-card></lib-card>出现在模板中，在这个例子中，`<lib-header>`没有用过，你肯能期望它会被摇树优化掉。
但是因为代码中出现了如下的一段导致无法被优化：
@ContentChild(LibHeaderComponent) header: LibHeaderComponent;
·其中一个引用位于 类型位置 上，即，它把 LibHeaderComponent 用作类型：header：LibHeaderComponent
·另一个引用位于 值的位置 ，即，LibHeaderComponent是@ContentChild() 参数装饰器的值：@ContentChild(LibHeaderComponent)
编译器对这些位置的令牌引用的处理方式时不同的。

·编译器在从TypeScript转换完后会删除这些类型位置上的引用，所以它们对于摇树优化没有什么影响
·编译器必须在运行时保留值位置上的引用，这就会住址该组件被摇树优化掉。

**什么时候使用轻量级注入令牌模式**
当一个组件被用作注入令牌时，就会出现摇树优化的问题，有两种情况：
·令牌用在内容查询中值的位置上，也就是@ContentChild或者@ViewChild等查询装饰器
·该令牌用构造函数注入的类型说明符，@Inject(OtherComponent),下面的代码虽然没有出现@Inject(),通过前面的知识点，就可以知道这只是简写。

`class MyComponent {`
  `constructor(@Optional() other: OtherComponent) {}`

  `@ContentChild(OtherComponent)`
  `other: OtherComponent|null;`
`}`

**使用轻量级注入令牌**
轻量级注入令牌设计模式