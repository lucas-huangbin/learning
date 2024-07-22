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

