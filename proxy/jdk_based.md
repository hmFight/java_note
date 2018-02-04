Jdk-动态代理原理



参考文章

> [JDK动态代理3----WeakCache缓存的实现机制](http://www.cnblogs.com/liuyun1995/p/8144676.html)
>
> [代理4 动态代理的缓存机制](https://www.jianshu.com/p/9f5566b5e7fb)
>
> [引用](http://blog.csdn.net/stridebin/article/details/73639378)
>
> [ Java Reference 源码分析](https://www.cnblogs.com/jabnih/p/6580665.html)
>
> [Java引用类型](http://www.importnew.com/20468.html)

[上篇](https://hmfight.gitbooks.io/java_note/content/proxy/simple_use.html) 中简单介绍了关于 Java 中 3中代理的使用。其中静态代理模式比较简单直白，但是缺点也很明显，相比较而言 Jdk 动态代理和 Cglib 动态代理实用性更强。

但是由于两种动态代理模式中代理类都是由动态生成，导致整个调用链不够直白清晰。使用者不容易弄清楚发生了什么。接下来会通过实例和源码分析更深入的梳理下 JDK 动态代理。

一、实例 code

接口

````java
//卖火车票接口
public interface TrainTicketSeller {
    void query();
    void sell();
}

````

被代理

````java
//12306 被代理方
public class Seller12306 implements TrainTicketSeller {

    //实际业务
    @Override
    public void query() {
        System.out.println("您好，您回家的票还有……0张");
    }

    //实际业务
    @Override
    public void sell() {
        System.out.println("您已购买回家的票！");
    }

}
````

代理

````java
//代理类不跟任何具体被代理有强绑定关系，理论上所有需要 doAfter() 和 doAfter()的都可以使用此代理
public class JdkBasedProxy implements InvocationHandler {
    /**
     * 被代理对象
     */
    private Object target;

    public Object bind(Object target) {
        this.target = target;
        return Proxy.newProxyInstance(target.getClass().getClassLoader(),
                target.getClass().getInterfaces(), this);
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        doBefore();
        Object invokeRes = method.invoke(target, args);
        doAfter();
        return invokeRes;
    }

    private void doBefore() {
        System.out.println("***********before***********");
    }

    private void doAfter() {
        System.out.println("**********after************");
    }

}
````

**使用**

````java
@Test
public void testJdkBasedProxy() throws Exception {
    TicketSeller seller = (TicketSeller) new JdkBasedProxy().bind(new Seller12306());
    seller.query();
    seller.sell();
}
/**输出
**********before************
您好，您回家的票还有……1张
***********after***********
**********before************
您已购买回家的票！
***********after***********
**/
````

**问题点：这个 seller 是如何调用到 JdkBasedProxy 中的 doBefore() 和 doAfter()的？**

其实





### WeakCache 源码分析

1、二级缓存

2、弱引用

3、