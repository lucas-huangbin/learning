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
![[Pasted image 20240806104727.png]]
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
