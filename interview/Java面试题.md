#### 什么是面向对象？面向对象3大要素

````
封装 - 隐藏/屏蔽内部细节；
继承 - 复用，实现类的，并可以添加新的特性；
多态 - 允许不同类的对象对同一消息做出响应。即同一消息可以根据发送对象的不同而采用多种不同的行为方式。（发送消息就是函数调用）
````

#### 面向接口编程？

````
接口是企鹅却
````

#### 访问修饰符？

| 访问权限 | 类   | 包   | 子类 | 其他包 |                                    |
| -------- | ---- | ---- | ---- | ------ | ---------------------------------- |
| public   | √    | √    | √    | √      | 对任何人都是可用的                 |
| protect  | √    | √    | √    | ×      | 子类和同一包的类可以访问           |
| default  | √    | √    | ×    | ×      | 包访问权限，即在整个包内均可被访问 |
| private  | √    | ×    | ×    | ×      | 只有当前类可以访问                 |

#### 3、基本数据类型有哪些？位? 字节? 

````
byte：Java中最小的数据类型，在内存中占8位(bit)，即1个字节，取值范围-128~127，默认值0
short：短整型，在内存中占16位，即2个字节，取值范围-32768~32717，默认值0
int：整型，用于存储整数，在内在中占32位，即4个字节，取值范围-2147483648~2147483647，默认值0
long：长整型，在内存中占64位，即8个字节-2^63~2^63-1，默认值0L

float：浮点型，在内存中占32位，即4个字节，用于存储带小数点的数字（与double的区别在于float类型有效小数点只有6~7位），默认值0.0f
double：双精度浮点型，用于存储带有小数点的数字，在内存中占64位，即8个字节，默认值0.0

char：字符型，用于存储单个字符，占16位，即2个字节，取值范围0~65535，默认值为空
boolean：布尔类型，占1个字节，用于判断真或假（仅有两个值，即true、false），默认值false
````

#### 4、float 和 short 类型变量声明 

**float f=3.4;是否正确？**

````
long a = 231L;
float a = 5.2f;
long c = 10000000000000000L;
long d = 35;  //放大，自动类型转换

````

#### 5、基本类型和包装类型* 

java1.5后引入 自动装箱和自动拆箱

````java
// Integer i = 100 编译后反编译回来代码是什么？或者说这段代码实际有哪些操作？
Integer i = Integer.valueOf(100);
````
````java
//这段代码的输出是什么？为什么？
@Test
public void testFloat() throws Exception {
    Integer i = -100;
    Integer i2 = -100;
    Integer i3 = -200;
    Integer i4 = -200;

    Integer i5 = 100;
    Integer i6 = 100;
    Integer i7 = 200;
    Integer i8 = 200;

    System.out.println(i == i2);  //ture
    System.out.println(i3 == i4); //false
    System.out.println(i5 == i6); //true
    System.out.println(i7 == i8); //false
}

解释：Integer 类中 静态内部类 IntegerCache 会对 -127~128 做缓存，当 i 的值在这个范围的时候，Integer 不会 new 新的对象，而是直接引用缓存中的 Integer 对象。
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
````



多线程、类加载、JVM、内存模型、静态代理和动态代理

读写分离、

*CAP* 定理