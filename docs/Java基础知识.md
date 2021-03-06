# Java transient 关键字

## 1. transient的作用及使用方法

我们都知道一个对象只要实现了Serilizable接口，这个对象就可以被序列化，java的这种序列化模式为开发者提供了很多便利，我们可以不必关系具体序列化的过程，只要这个类实现了Serilizable接口，这个类的所有属性和方法都会自动序列化。

然而在实际开发过程中，我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，打个比方，如果一个用户有一些敏感信息（如密码，银行卡号等），为了安全起见，不希望在网络操作（主要涉及到序列化操作，本地序列化缓存也适用）中被传输，这些信息对应的变量就可以加上transient关键字。换句话说，这个字段的生命周期仅存于调用者的内存中而不会写到磁盘里持久化。

总之，java 的transient关键字为我们提供了便利，你只需要实现Serilizable接口，将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会序列化到指定的目的地中。

[Java transient 关键字 | 菜鸟教程 (runoob.com)](https://www.runoob.com/w3cnote/java-transient-keywords.html)

# Java finally语句

网上有很多人探讨Java中异常捕获机制try...catch...finally块中的finally语句是不是一定会被执行？很多人都说不是，当然他们的回答是正确的，经过我试验，**至少有两种情况下finally语句是不会被执行的：**

**（1）try语句没有被执行到，如在try语句之前就返回了，这样finally语句就不会执行，这也说明了finally语句被执行的必要而非充分条件是：相应的try语句一定被执行到。**

**（2）在try块中有System.exit(0);这样的语句，System.exit(0);是终止Java虚拟机JVM的，连JVM都停止了，所有都结束了，当然finally语句也不会被执行到。**

当然还有很多人探讨Finally语句的执行与return的关系，颇为让人迷惑，不知道finally语句是在try的return之前执行还是之后执行？我也是一头雾水，我觉得他们的说法都不正确，我觉得应该是：**finally语句是在try的return语句执行之后，return返回之前执行**。
