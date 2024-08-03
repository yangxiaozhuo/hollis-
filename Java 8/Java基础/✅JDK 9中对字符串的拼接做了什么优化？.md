# 典型回答

在JDK 9之前，字符串拼接通常使用`+`进行（也有其他的，我们不做展开了），`+`的实现其实是基于`StringBuilder`的。具体参考：

[✅String、StringBuilder和StringBuffer的区别？](https://www.yuque.com/hollis666/fo22bm/pg23qhb7rgnuamd1?view=doc_embed&inner=rBBk1)

这个过程其实是比较低效的，因为整个过程包含了创建`StringBuilder`对象，通过调用`append`方法拼接字符串，最后通过`toString`方法转换成最终的字符串等多个操作。所以才有个规范说不要在 for 循环中用+来拼接字符串的。

但是，这个其实在 JDK 9中已经被修改了。JDK 9引入了`StringConcatFactory`。这玩意被推出的的主要目标是提供一种灵活且高效的方式来拼接字符串，代替之前的 `StringBuilder` 或 `StringBuffer` 的静态编译方法。

`StringConcatFactory`是基于`invokedynamic`指令实现的。

> 是 Java 7 中引入的一种动态类型指令，允许 JVM 在运行时动态解析和调用方法。


也就说，利用`invokedynamic`的特性，将字符串拼接的操作**延迟到运行时**，而不是在编译时固定使用`StringBuilder`。（前面的链接中我们做过反编译，可见+转成 StringBuilder是编译的时候就确定了的。）

这就使得，JVM可以在运行时根据实际的场景选择最优的拼接策略，可能是使用`StringBuilder`、`StringBuffer`、或者其他更高效的方法。

在 [JDK 9](https://github.com/zxiaofan/JDK/blob/master/JDK1.9/src/java.base/java/lang/invoke/StringConcatFactory.java#L126)中（后续版本会有所变化，1.9 看的比较清楚），支持的拼接策略有以下几个：

```
private enum Strategy {
    /**
     * 使用 StringBuilder 进行字符串拼接，但不预估所需的存储空间。
     */
    BC_SB,

    /**
     * 使用 StringBuilder 进行字符串拼接，同时尝试估计所需的存储空间，以优化性能。
     */
    BC_SB_SIZED,

    /**
     * 使用 StringBuilder 进行字符串拼接，且能够精确计算出所需的存储空间，以实现最高的效率。
     */
    BC_SB_SIZED_EXACT,

    /**
     * 基于 MethodHandle 技术，使用 StringBuilder 进行拼接，并尝试预估所需的存储空间。
     */
    MH_SB_SIZED,

    /**
     * 基于 MethodHandle 技术，使用 StringBuilder 进行拼接，并精确计算所需的存储空间。
     */
    MH_SB_SIZED_EXACT,

    /**
     * 通过 MethodHandle 技术，直接从输入参数构建一个字节数组，并准确计算出所需的存储空间，以实现高效的字符串拼接。
     */
    MH_INLINE_SIZED_EXACT
}
```

StringBuilder你不陌生，`MethodHandle` 是啥？他 Java 7 开始引入的特性，它提供了一种灵活且高效的方法来直接操作方法、构造函数和字段的调用。

它与反射相似，但提供了更高的性能和更低的使用限制。`MethodHandle` 是一种非常底层的机制，允许开发者在运行时动态查找和调用方法，无论方法的访问权限如何。

使用 `MethodHandles.lookup()` 获取一个 `Lookup` 实例，然后使用这个实例来查找特定的方法。

```
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;

public class MethodHandleExample {
    public static void main(String[] args) throws Throwable {
        MethodHandles.Lookup lookup = MethodHandles.lookup();
        MethodType methodType = MethodType.methodType(String.class, int.class, int.class);
        MethodHandle mh = lookup.findVirtual(String.class, "substring", methodType);
        
        String str = "Hello, World!";
        String result = (String) mh.invokeExact(str, 7, 12);
        System.out.println(result);  // Outputs "World"
    }
}
```

使用 `invoke`, `invokeExact`, 或者其他形式的 `invoke` 方法来调用 `MethodHandle`。
