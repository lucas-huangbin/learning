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
