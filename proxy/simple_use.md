#静态代理、JDK和Cglib动态代理的简单使用

前段时间看了下 Mybatis , 顺便梳理下关于代理这一块的知识。

这里自己简单想了一个近期关于火车票的实例，可能很多地方还有不当之处，还请谅解。

接口，包括两个方法，查票和卖票

```java
//卖火车票接口
public interface TicketSeller {
    void query();
    void sell();
}
```

```java
//12306 被代理方
public class Seller12306 implements TicketSeller {

    @Override
    public void query() {
        System.out.println("您好，您回家的票还有……1张");
    }

    @Override
    public void sell() {
        System.out.println("您已购买回家的票！");
    }

}
```

下面分别用3种代理来实现同样的功能。


### 一、静态代理

第三方代理售票，比如美团、铁友啊

````java
//第三方代理售票，加了漂亮的头和尾。。。>..
public class MeiTuanSeller implements TicketSeller {
    private Seller12306 seller12306;

    public MeiTuanSeller(Seller12306 seller12306) {
        this.seller12306 = seller12306;
    }

    @Override
    public void query() {
        System.out.println("***********before***********");
        seller12306.query();
        System.out.println("***********after************");
    }

    @Override
    public void sell() {
        System.out.println("***********before***********");
        seller12306.sell();
        System.out.println("***********after************");
    }
}
````
使用
````java
//调用
TrainTicketSeller seller = new MeiTuanSeller(new Seller12306());
seller.query();
seller.sell();

/**输出结果：
***********before***********
您好，您回家的票还有……1张
***********after************
***********before***********
您已购买回家的票！
***********after************
**/
````


>**问题1，不用代理可以么？**
>
>​	当然是可以的，就想买个票，直接上12306就行，不嫌弃美丑难用。
>



>**问题2，代理中新加的功能，不能直接修改被代理方或者在被代理方增加么？**
>
>​	当然也是可以的，但是。。为什么12306不改呢？就光卖个票都被喷成啥样了，还有心思至于网站好不好看交互怎么样？几亿人用了这么多年这个样式，说改就改貌似不太合适吧？ 好，就算你改了。改完，立马被喷了大家都用不惯，那是回到原来的样式啊，还是再换个新的呢？不管怎么样还是得再改一次。（修改有成本和风险，甚至可能影响核心系统稳定）
>
>​	但是如果第三方售票来做这个事情，第一12306不用动，第二如果我觉得第三方也不好看不好用，我还是能继续用原来的12306，第三如果美团做的不好看，旁边360、铁友貌似还挺好看的，我去试试。

​	个人总结起代理的优点，本质上还是符合**单一职责原则和开闭原则(**对扩展开放，对修改封闭)。核心职责和非核心**职责**的的划分清晰，保持原系统的**稳定性**的同时增加了**扩展性**。

​	可能会存在的缺点呢？如果被代理类功能繁杂多样，代理类只会更**复杂臃肿**。另一个问题是，代理类的新功能代码比较**难复用**。这里就可以引出动态代理了。



### 二、动态代理

​	动态代理好处？本质上还是代码复用的问题。代理功能只用写一遍啊。Java 动态代理一般有两种，一种是通过接口来实现（Jdk动态代理），一种是通过继承来实现（cglib）。

#### 1、JDK 动态代理

​	用 JDK 动态代理实现上面一样的功能。

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
    ProxyUtils.generateClassFile(seller.getClass(),"ReadProxy");
}

/**输出结果：
***********before***********
您好，您回家的票还有……1张
***********after************
***********before***********
您已购买回家的票！
***********after************
**/
````

​	假如现在另一个地方也需要此代理提供的功能，我的代理不需要做任何改动，只需要在调用的时候，更改bind()参数和强转类型即可。

````java
ISubject subject = (ISubject) new MtDyProxy().bind(new ReadSubject());
subject.do();
````

​	由于代理类是动态生成的，所以调用链不够清晰，不能一眼看出一步步是怎么走的。其实这点，可以将动态代理的生成的Class输出到本地，再反编译出来就OK了。这个点下次来说明。

​	动态代理最大的缺点是，必须依赖接口。生成的代理类中只会有实现接口的方法。如果需要被代理的方法，没有实现接口，那很遗憾就无法使用 Jdk动态代理。这个时候，就需要 Cglib 代理了。



#### 2、CGLIB 动态代理

````Java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class CglibProxy implements MethodInterceptor {

    /**
     * 被代理对象，供代理方法中进行真正的业务方法调用
     */
    private Object target;

    //相当于JDK动态代理中的绑定
    @SuppressWarnings("unchecked")
    public Object getInstance(Object target) {
        this.target = target;
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(this.target.getClass());
        //设置回调：对于代理类上所有方法的调用，都会调用CallBack，而Callback则需要实现intercept()方法进行拦
        enhancer.setCallback(this);
        // 创建动态代理类对象并返回
        return enhancer.create();
    }

    // 实现回调方法
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        doBefore();
        //调用业务类（父类中）的方法
        Object result = proxy.invokeSuper(obj, args);
        doAfter();
        return result;
    }

    private void doAfter() {
        System.out.println("***********before***********");
    }

    private void doBefore() {
        System.out.println("**********after************");
    }

}
````

使用

````java
@Test
public void cglibProxy() throws Exception {
    CglibProxy cglib = new CglibProxy();
    Seller12306 seller = (Seller12306) cglib.getInstance(new Seller12306());
    seller.query();
    seller.sell();
}

/**输出结果：
**********before************
您好，您回家的票还有……1张
***********after***********
**********before************
您已购买回家的票！
***********after***********
**/
````

​	这里只是简单了做了下使用说明，比较深的东西希望自己能在下次给出来。

###三、总结

​	现在已经很少看到单独使用静态代理的了。。。

​	前面也有提到，JDK动态动态代理是通过接口来实现的，而 Cglib  是通过继承来实现的。这也是 Cglib 相对于前者的一个优势。

### 四、补充

* 代理模式和装饰器模式

  ​	今天也看了不少有讨论这这两者区别异同的文章，一些统一的说法就是装饰为了增强，代理更偏控制。

  ​	说说自己的看法，两者确实十分相似，但是其实没有必要说把两个东西完全区分开，本质上都是为了解耦易扩展。装饰也好代理也好，只是文字上的不同而已，不必要纠结许多年前的理论文字了。

  ​	实际应用中，我相信没人说会在用的时候去纠结要用装饰还是代理。我们应该在意的点是，怎么更好的解决问题。设计模式只是一些经典的思想，思想应当是一直在进步的。况且现在 80% 的应用粗制滥造的程度，还达不到纠结设计模式的时候。