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

**应用举例**
1. 模块内部一些公共配置的共享，如：权限前缀

#### 2.@Injectable & providers

**@Injectable（可被注入的）:** service的装饰器，用于标记这个class具有DI的特性，只有service会打上这个装饰器
**providers（提供者）** ：是ngModule、Component的传入参数，传入模组作为该模组的公用函数，每一次注入都会**重新**实例化service

**@Injectable**
注入根目录，
`@Injectable({
	providedIn: 'root'
})`

