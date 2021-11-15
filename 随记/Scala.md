# 模式匹配
## 模式匹配相比于 Java Switch 的优势
> Scala的match表达式可以计算出一个值,而Java的switch却不能
> 模式匹配的表达式永远不会穿透到下一个case表达式中, java的switch中没有break的话,则会穿透到下一个分支
> 模式匹配中如果没有任何的模式被匹配到,则抛出MatchError异常
- 比如http请求返回体为json字符串,我们往往只需要获取json中的部分字段值。
- https://www.cnblogs.com/barrywxx/p/10850502.html
> Java 必须知道所有json中所有字段及类型,并定义对应的JavaBean,将返回的json字符串先转换为对应的JavaBean,再通过javaBean获取想要的字段信息。
> Scala 先定义一个unapplySeq()实现json的模式匹配,然后直接取出

## SealedClass 
> 可以在case class的父类上加一个sealed的关键字,这个sealed会帮我们做两件事情：
> 1：在我们定义模式匹配的时候,如果没有考虑全场景,编译器会报警告
> 2：被sealed修饰的类,不能被在其他文件中的case class继承

