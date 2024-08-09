https://developer.aliyun.com/article/1341087?spm=a2c6h.14164896.0.0.2e6d47c5WUwud9&scm=20140722.S_community@@%E6%96%87%E7%AB%A0@@1341087._.ID_1341087-RL_%E5%89%8D%E7%AB%AFrxjs-LOC_search~UND~community~UND~item-OR_ser-V_3-P0_0

RxJS是兼具函数式和响应式两种先进变成风格的框架。
RxJS是一个组织异步逻辑的库，它有很多operator，可以极大的简化异步逻辑的编写。
RxJS中解决异步事件管理的基本概念是：
·Observable（可观察对象）：表示一个可调用的未来值或事件的集合的概念。
·Observer（观察者）：是回调的集合，知道如何监听Observable传递的值。
·Subscription（订阅）：表示Observable的执行，主要用于取消执行。
·Operators（操作符）：采用函数式编程风格的纯函数，支持使用map、filter、concat、reduce等操作处理集合。
·Subject（主体）：相当于一个EventEmitter，是将一个值或事件多播给多个Observers的唯一路径。
·Schedulers（调度器）：是控制并发的集中式调度程序，允许我们在计算发生时进行协调，例如setTimeout或requestAnimationFrame或其它。

#### Observable
被观察者，用来产生消息/数据。
![[./Image/Pasted image 20240806104727.png]]
Observable是多个值的惰性推送集合。本质其实就是一个随事件不断产生数据的一个集合，称之为流更容易理解。

示例：
`var observable = Rx.Observable.create(function (observer) {`
  `observer.next(1);`
  `observer.next(2);`
  `observer.next(3);`
  `setTimeout(() => {`
    `observer.next(4);`
    `observer.complete();`
  `}, 1000);`
`});`

要调用Observable并看到这些值，我们需要订阅Observable：
`var observable = Rx.Observable.create(function (observer) {`
  `observer.next(1);`
  `observer.next(2);`
  `observer.next(3);`
  `setTimeout(() => {`
    `observer.next(4);`
    `observer.complete();`
  `}, 1000);`
`});`
  
`console.log('just before subscribe');`
`observable.subscribe({`
  `next: x => console.log('got value ' + x),`
  `error: err => console.error('something wrong occurred: ' + err),`
  `complete: () => console.log('done'),`
`});`
`console.log('just after subscribe');`

执行结果：
`just before subscribe`
`got value 1`
`got value 2`
`got value 3`
`just after subscribe`
`got value 4`
`done`

##### Pull & Push
什么是Pull：在拉取体系中，由消费者（主动的）来决定何时从生产者那里接收数据。生产者（被动的）本身不知道数据是何时交付到消费者手中的。
什么是Push：在推送体系中，由生产者（主动的）来决定何时把数据发送给消费者。消费者（被动的）本身不知道何时会接受到数据。

##### Observable剖析

**创建Observables**
Rx.Obsercable.create是Observable构造函数的别名，它接受一个参数：subscribe函数。
示例：
`var observable = Rx.Observable.create(function subscribe(observer) {`
  `var id = setInterval(() => {`
    `observer.next('hi')`
  `}, 1000);`
`});`

注：Observables可以使用create来创建，但通常我们使用所谓的创建操作符，像of、from、interval、等等。

**订阅Observables**
示例中的Observable对象observable可以订阅，像这样：
`observable.subscribe(x => console.log(x));`

`observable.subscribe` 和 `Observable.create(function subscribe(observer) {...})` 中的 `subscribe`有着同样的名字，这并不是一个巧合。在库中，它们是不同的，但从实际出发，你可以认为在概念上它们是等同的。
这表明subscribe调用在同一Observable的多个观察者之间是不共享的。当使用一个观察者调用observable.subscribe时，`Observable.create(function subscribe(observer) {...})` 中的 `subscribe` 函数只服务于给定的观察者。对observable.subscribe的每次调用都会触发针对给定观察者的独立设置。

注：订阅Observable像是调用函数，并提供接收数据的回调函数。

**执行Observables**
`Observable.create(function subscribe(observer) {...})` 中`...`的代码表示 “Observable 执行”，它是惰性运算，只有在每个观察者订阅后才会执行。随着时间的推移，执行会以同步或异步的方式产生多个值。

Observable 执行可以传递三种类型的值：
- "Next" 通知： 发送一个值，比如数字、字符串、对象，等等。
- "Error" 通知： 发送一个 JavaScript 错误 或 异常。
- "Complete" 通知： 不再发送任何值。

**清理Observable执行**
当调用了observable.subscribe，观察者会被附加到新创建的Observable执行中。这个调用还返回了一个对象，即Subscription（订阅）：
`var subscription = observable.subscribe(x => console.log(x));`
取消执行：
`var observable = Rx.Observable.from([10, 20, 30]);` 
`var subscription = observable.subscribe(x => console.log(x));` 
`// 稍后：` 
`subscription.unsubscribe();`

当我们使用 `create()` 方法创建 Observable 时，Observable 必须定义如何清理执行的资源。你可以通过在 `function subscribe()` 中返回一个自定义的 `unsubscribe` 函数。
举例来说，这是我们如何清理使用了 `setInterval` 的 interval 执行集合：
`var observable = Rx.Observable.create(function subscribe(observer) {`
  `// 追踪 interval 资源`
  `var intervalID = setInterval(() => {`
    `observer.next('hi');`
  `}, 1000);`

  `// 提供取消和清理 interval 资源的方法`
  `return function unsubscribe() {`
    `clearInterval(intervalID);`
  `};`
`});`

#### Observer(观察者)
观察者是由Observable发送的值的消费者。观察者只是一组回调函数的集合，每个回调函数对应一种Observable发送的通知类型：next、error、和complete。下面的示例是一个典型的观察者对象：
`var observer = {`
  `next: x => console.log('Observer got a next value: ' + x),`
  `error: err => console.error('Observer got an error: ' + err),`
  `complete: () => console.log('Observer got a complete notification'),`
`};`

要使用观察者，需要把它提供给Observable的subscribe方法：
`observable.subscribe(observer);`

#### Subscription(订阅)
Subscription是表示可清理资源的对象，通常是Observable的执行。Subscription有一个重要的方法，即unsubscribe，它不需要任何参数，只是用来清理由Subscription占用的资源。在上一版本的rxjs中，Subscription叫做“Disposable”（可清理对象）。

Subscription还可以合在一起，这样一个Subscription调用unsubscribe()方法，可能有多个Subscription取消订阅。
`var observable1 = Rx.Observable.interval(400);`
`var observable2 = Rx.Observable.interval(300);`
  
`var subscription = observable1.subscribe(x => console.log('first: ' + x));`
`var childSubscription = observable2.subscribe(x => console.log('second: ' + x));`

`subscription.add(childSubscription);`
`setTimeout(() => {`
  `// subscription 和 childSubscription 都会取消订阅`
  `subscription.unsubscribe();`
`}, 1000);`

Subscription还有一个remove(otherSubscription)方法，用来撤销一个已添加的子Subscription。

#### Subject(主体)
Subject是一种特殊类型的Observable，它允许将值多播给多个观察者，所以Subject是多播的，而普通的Observables是单播的（每个已订阅的观察者都拥有Observable的独立执行）。
**每个Subject都是Observable** - 对于Subject你可以提供一个观察者并使用subscribe方法，就可以开始正常接受值。在Subject内部，subscribe不会调用发送值的新执行。它只是将给定的观察者注册到观察者列表中，类似于其他库或语言中的addListener的工作方式（只会接收观察者注册监听之后的值，注册监听之前的值不接收）。
**每个Subject都是观察者** - Subject是一个有有如下方法的对象：next(v)、error(e)和complete()。要给Subject提供新值，只要调用next(theValue)，它会将值多播给已注册监听该Subject的观察者们。
`var subject = new Rx.Subject();`

`subject.subscribe({`
  `next: (v) => console.log('observerA: ' + v)`
`});`
`subject.subscribe({`
  `next: (v) => console.log('observerB: ' + v)`
`});`

`subject.next(1);`
`subject.next(2);`

因为Subject是观察者，所以你也可以把Subject作为参数传给任何Observable的subscribe方法，示例：
`var subject = new Rx.Subject();`

`subject.subscribe({`
  `next: (v) => console.log('observerA: ' + v)`
`});`
`subject.subscribe({`
  `next: (v) => console.log('observerB: ' + v)`
`});`
  
`var observable = Rx.Observable.from([1, 2, 3]);`
  
`observable.subscribe(subject); // 你可以提供一个 Subject 进行订阅`

使用上面的方法，我们只是通过Subject将单播的Observable执行转换为多播的（**Subject本身可以发送和接收，将Subject作为观察者提供给Observable，Observable再通过Subject发送，那么监听Subject就可以接收到Observable的值**）。这也说明Subjects是将任意Observable执行共享给多个观察者的唯一方式。

##### 多播的Observables
“多播Observable”通过Subject来发送通知，这个Subject可能有多个订阅者，然而普通的“单播Observable”只发送给单个观察者。
在底层，这就是multicast操作符的工作原理：观察者订阅一个基础的Subject，然后Subject订阅源Observable。下面的示例与前面使用 `observable.subscribe(subject)` 的示例类似：

`var source = Rx.Observable.from([1, 2, 3]);`
`var subject = new Rx.Subject();`
`var multicasted = source.multicast(subject);`
  
`// 在底层使用了 subject.subscribe({...}):`
`multicasted.subscribe({`
  `next: (v) => console.log('observerA: ' + v)`
`});`
`multicasted.subscribe({`
  `next: (v) => console.log('observerB: ' + v)`
`});`
  
`// 在底层使用了 source.subscribe(subject):`
`multicasted.connect();`
