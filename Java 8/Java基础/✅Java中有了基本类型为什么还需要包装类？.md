# 典型回答
Java中有8种基本数据类型，这些基本类型又都有对应的包装类。

| 分类 | 基本数据类型 | 包装类 | 长度 | 表示范围 |
| --- | --- | --- | --- | --- |
| 布尔型 | boolean | Boolean | / | / |
| 整型 | byte | Byte | 1字节 | -128 到 127 |
|  | short | Short | 2字节 | -32,768 到 32,767 |
|  | int | Integer | 4字节 | -2,147,483,648 到 2,147,483,647 |
|  | long | Long | 8字节 | -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807 |
| 字符型 | char | Character | 2字节 | Unicode字符集中的任何字符 |
| 浮点型 | float | Float | 4字节 | 约 -3.4E38 到 3.4E38 |
|  | double | Double | 8字节 | 约 -1.7E308 到 1.7E308 |


**因为Java是一种面向对象语言，很多地方都需要使用对象而不是基本数据类型**。比如，在集合类中，我们是无法将int 、double等类型放进去的。因为集合的容器要求元素是Object类型。

为了让基本类型也具有对象的特征，就出现了包装类型，它相当于将基本类型“包装起来”，使得它具有了对象的性质，并且为其添加了属性和方法，丰富了基本类型的操作。

# 知识扩展

## 基本类型和包装类型的区别

1. 默认值不同，基本类型的默认值为0, false或\u0000等，包装类默认为null
2. 初始化方式不同，一个需要new，一个不需要
3. 存储方式不同，基本类型保存在栈上，包装类对象保存在堆上（成员变量的话，在不考虑JIT优化的栈上分配时，都是随着对象一起保存在堆上的）

[✅简单介绍一下JIT优化技术？](https://www.yuque.com/hollis666/fo22bm/nkr4ge?view=doc_embed)

## 接口定义中应该用什么？

[✅RPC接口返回中，使用基本类型还是包装类？](https://www.yuque.com/hollis666/fo22bm/hqm4f0?view=doc_embed)
## 如何理解自动拆装箱
### 拆箱与装箱
包装类是对基本类型的包装，所以，把基本数据类型转换成包装类的过程就是**装箱**；反之，把包装类转换成基本数据类型的过程就是**拆箱**。
### 自动拆装箱
在Java SE5中，为了减少开发人员的工作，Java提供了自动拆箱与自动装箱功能。

自动装箱: 就是将基本数据类型自动转换成对应的包装类。

自动拆箱：就是将包装类自动转换成对应的基本数据类型。
```java
Integer i =10;  //自动装箱
int b= i;     //自动拆箱
```
### 自动拆装箱原理
自动装箱都是通过包装类的`valueOf()`方法来实现的.自动拆箱都是通过包装类对象的`xxxValue()`来实现的。

如：int的自动装箱都是通过`Integer.valueOf()`方法来实现的，Integer的自动拆箱都是通过`Integer.intValue()`来实现的。
## 哪些地方会自动拆装箱

我们了解过原理之后，再来看一下，什么情况下，Java会帮我们进行自动拆装箱。前面提到的变量的初始化和赋值的场景就不介绍了，那是最简单的也最容易理解的。

我们主要来看一下，那些可能被忽略的场景。

### 场景一、将基本数据类型放入集合类

我们知道，Java中的集合类只能接收对象类型，那么以下代码为什么会不报错呢？

```java
List<Integer> li = new ArrayList<>();
for (int i = 1; i < 50; i ++){
    li.add(i);
}
```

将上面代码进行反编译，可以得到以下代码：

```java
List<Integer> li = new ArrayList<>();
for (int i = 1; i < 50; i ++){
    li.add(Integer.valueOf(i));
}
```

以上，我们可以得出结论，当我们把基本数据类型放入集合类中的时候，会进行自动装箱。

### 场景二、包装类型和基本类型的大小比较

有没有人想过，当我们对Integer对象与基本类型进行大小比较的时候，实际上比较的是什么内容呢？看以下代码：

```java
Integer a=1;
System.out.println(a==1?"等于":"不等于");
Boolean bool=false;
System.out.println(bool?"真":"假");
```

对以上代码进行反编译，得到以下代码：

```java
Integer a=1;
System.out.println(a.intValue()==1?"等于":"不等于");
Boolean bool=false;
System.out.println(bool.booleanValue?"真":"假");
```

可以看到，包装类与基本数据类型进行比较运算，是先将包装类进行拆箱成基本数据类型，然后进行比较的。

### 场景三、包装类型的运算

有没有人想过，当我们对Integer对象进行四则运算的时候，是如何进行的呢？看以下代码：

```java
Integer i = 10;
Integer j = 20;

System.out.println(i+j);
```

反编译后代码如下：

```java
Integer i = Integer.valueOf(10);
Integer j = Integer.valueOf(20);
System.out.println(i.intValue() + j.intValue());
```

我们发现，两个包装类型之间的运算，会被自动拆箱成基本类型进行。

### 场景四、三目运算符的使用

这是很多人不知道的一个场景，作者也是一次线上的血淋淋的Bug发生后才了解到的一种案例。看一个简单的三目运算符的代码：

```java
boolean flag = true; //设置成true，保证条件表达式的表达式二一定可以执行
boolean simpleBoolean = false; //定义一个基本数据类型的boolean变量
Boolean nullBoolean = null;//定义一个包装类对象类型的Boolean变量，值为null

boolean x = flag ? nullBoolean : simpleBoolean; //使用三目运算符并给x变量赋值
```

很多人不知道，其实在`int k = flag ? i : j;`这一行，会发生自动拆箱。反编译后代码如下：

```java
boolean flag = true;
boolean simpleBoolean = false;
Boolean nullBoolean = null;
boolean x = flag ? nullBoolean.booleanValue() : simpleBoolean;
```

这其实是三目运算符的语法规范。当第二，第三位操作数分别为基本类型和对象时，其中的对象就会拆箱为基本类型进行操作。

可以看到，反编译后的代码的最后一行，编译器帮我们做了一次自动拆箱，而就是因为这次自动拆箱，导致代码出现对于一个null对象（nullBoolean.booleanValue()）的调用，导致了NPE。

### 场景五、函数参数与返回值

这个比较容易理解，直接上代码了：

```java
//自动拆箱
public int getNum1(Integer num) {
return num;
}
//自动装箱
public Integer getNum2(int num) {
return num;
}
```

## 自动拆装箱与缓存

Java SE的自动拆装箱还提供了一个和缓存有关的功能，我们先来看以下代码，猜测一下输出结果：

```java
public static void main(String... strings) {

    Integer integer1 = 3;
    Integer integer2 = 3;

    if (integer1 == integer2)
        System.out.println("integer1 == integer2");
    else
        System.out.println("integer1 != integer2");

    Integer integer3 = 300;
    Integer integer4 = 300;

    if (integer3 == integer4)
        System.out.println("integer3 == integer4");
    else
        System.out.println("integer3 != integer4");

}
```

我们普遍认为上面的两个判断的结果都是false。虽然比较的值是相等的，但是由于比较的是对象，而对象的引用不一样，所以会认为两个if判断都是false的。在Java中，==比较的是对象引用，而equals比较的是值。所以，在这个例子中，不同的对象有不同的引用，所以在进行比较的时候都将返回false。奇怪的是，这里两个类似的if条件判断返回不同的布尔值。

上面这段代码真正的输出结果：

```java
integer1 == integer2
integer3 != integer4
```

原因就和Integer中的缓存机制有关。在Java 5中，在Integer的操作上引入了一个新功能来节省内存和提高性能。整型对象通过使用相同的对象引用实现了缓存和重用。

> 适用于整数值区间-128 至 +127。
>  
> 只适用于自动装箱。使用构造函数创建对象不适用。


我们只需要知道，当需要进行自动装箱时，如果数字在-128至127之间时，会直接使用缓存中的对象，而不是重新创建一个对象。

其中的javadoc详细的说明了缓存支持-128到127之间的自动装箱过程。最大值127可以通过`-XX:AutoBoxCacheMax=size`修改。

实际上这个功能在Java 5中引入的时候,范围是固定的-128 至 +127。后来在Java 6中，可以通过`java.lang.Integer.IntegerCache.high`设置最大值。

这使我们可以根据应用程序的实际情况灵活地调整来提高性能。到底是什么原因选择这个-128到127范围呢？因为这个范围的数字是最被广泛使用的。 在程序中，第一次使用Integer的时候也需要一定的额外时间来初始化这个缓存。

在Boxing Conversion部分的Java语言规范(JLS)规定如下：

如果一个变量p的值是：

```java
-128至127之间的整数(§3.10.1)

true 和 false的布尔值 (§3.10.3)

‘\u0000’至 ‘\u007f’之间的字符(§3.10.4)
```

范围内的时，将p包装成a和b两个对象时，可以直接使用a==b判断a和b的值是否相等。

### <br /><br />
