# Otto
>Otto官网: http://square.github.io/otto/

>github: https://github.com/square/otto

>JavaDoc: http://square.github.io/otto/1.x/otto/

>Otto是一个事件总线，设计的目的是让应用的不同模块解耦，同时仍然能让模块之间有效地通信。
## 用法
>推荐使用单例模式， `Bus bus = new Bus();`
### 发布
>事件发布：`bus.post(new AnswerAvailableEvent(42));`。发布一个事件是一个同步的过程，在程序执行时，保证所有的订阅者都被调用（广播）。
>如果发布的事件类没有人订阅，并且该事件还不是DeadEvent，那么它将被包装到`DeadEvent`中并重新发布。

>protected void dispatch(Object event, com.squareup.otto.EventHandler wrapper)

>protected void dispatchQueuedEvents()

>protected void enqueueEvent(Object event, com.squareup.otto.EventHandler handler)
### 订阅
>使用@Subcribe来注解一个方法来订阅一个事件。该方法必须只有一个参数，该参数就是post方法发布的参数类型。
```java
//方法名可以随便取，但是注解、参数、public是必须的
@Subscribe 
public void answerAvailable(AnswerAvailableEvent event) {
    // TODO: React to the event somehow!
}
```
>为了使得一个类(订阅者)能够接受到事件，必须在bus上注册该类的实例
`bus.register(this)`,在恰当的地方使用`unregister()`方法。

>与Guava事件总线的不同：Registering will only find methods on the immediate class type. Unlike the Guava event bus, Otto will not traverse the class hierarchy and add methods from base classes or interfaces that are annotated. This is an explicit design decision to improve performance of the library as well as keep your code simple and unambiguous.
### 生产
>Otto添加了“生产者”的概念，它在订阅者注册时立即向他们提供回调。使用@Produce来创建生产者：
```java
@Produce
public AnswerAvailableEvent produceAnswer() {
    // Assuming 'lastAnswer' exists.
    return new AnswerAvailableEvent(this.lastAnswer);
}
```
>生产者跟订阅者一样，也需要在总线上注册：`bus.register(this)`

>在注册生产者时，将为之前为同一类型注册的每个订阅者调用一次生产者方法。对于订阅相同类型事件的每个新方法(@Subscribe)，也将调用一次生产者方法。

>在总线上每次只能为每个事件类型注册一个生产者。
### Thread Enforcement
>Otto提供一种Enforcement机制来保证能够使用特定的线程。在默认情况下，所有的交互都在主线程上进行。
```java
// Both of these are functionally equivalent.
Bus bus1 = new Bus();
Bus bus2 = new Bus(ThreadEnforcer.MAIN);
```
>如果不关心在哪个线程上交互，那么可用`ThreadEnforcer.ANY`来实例化一个总线实例。可以实现ThreadEnforcer接口来添加额外的功能。