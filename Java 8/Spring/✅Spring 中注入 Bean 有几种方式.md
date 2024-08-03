# 典型回答

注入 Bean，就是说已经有一个 Bean 了，我想在其他的 Bean 中使用它，就要注入。比如场景的我们在 Service 中注入Dao。

在 Spring 中，Bean 的注入也是非常常见的操作了，有以下几种形式。

### 使用`@Autowired` 注解

@Autowired 注解是Spring框架的一部分，用于自动装配（自动注入）Spring Bean到其他Bean中。

他可以标注在字段（成员变量）、构造器、以及 setter 方法上。

#### 字段注入

```
@Component
public class HollisService {
    @Autowired
    private HollisRepository hollisRepository;
}

```

[✅为什么Spring不建议使用基于字段的依赖注入？](https://www.yuque.com/hollis666/fo22bm/lbst9ffoy74od6kr?view=doc_embed)
#### 构造器注入

```
@Component
public class HollisService {

    private HollisRepository hollisRepository;
    
    @Autowired
    public HollisService(HollisRepository hollisRepository){
        this.hollisRepository = hollisRepository;
    }
}

```

#### setter 注入

```
@Component
public class HollisService {

    private HollisRepository hollisRepository;
    
    @Autowired
    public void setHollisRepository(HollisRepository hollisRepository){
        this.hollisRepository = hollisRepository;
    }
}

```

### 使用 `@Resource` 和 `@Inject` 注解

Bean的注入，除了用 Spring 中提供的注解以外，还可以用 JDK 给我们提供的注解，包括了`@Resource` 和 `@Inject` 。

```
@Component
public class HollisService {
    @Resource
    private HollisRepository hollisRepository;
}

@Component
public class HollisService {
    @Inject
    private HollisRepository hollisRepository;
}
```

[✅Autowired和Resource的关系？](https://www.yuque.com/hollis666/fo22bm/gai6a9?view=doc_embed)

### 使用XML 配置文件

除了用注解以外，还可以用传统的XML 文件的配置方式，也能实现 Bean 的注入。

```
<beans>
    <bean id="hollisRepository" class="com.java.bagu.demo.HollisRepository"/>
    <bean id="hollisService" class="com.java.bagu.demo.HollisService">
        <property name="hollisRepository" ref="hollisRepository"/>
    </bean>
</beans>

```


### 构造器自动注入

除了以上几种情况意外，还有一个可能很少有人注意到，其实从 Spring 4.3 开始，除非一个类中声明了至少两个构造函数，否则不需要用 `@Autowired` 标注构造函数，这个构造函数也能直接注入 Bean：


```
@Component
public class HollisService {

    private HollisRepository hollisRepository;
    
    public HollisService(HollisRepository hollisRepository){
        this.hollisRepository = hollisRepository;
    }
}

```


### FactoryBean

说 FactoryBean 你可能不理解，看以下代码：

```
@Configuration
public class HollisService {

    @DubboReference(version = "1.0.0")
    private HollisRemoteFacadeService hollisRemoteFacadeService;
}
```

这里用到了@DubboReference把HollisRemoteFacadeService这个 bean给注入进来了。@DubboReference其实是RPC 框架Dubbo 提供的一个注解，他的实现原理其实就是基于 FactoryBean来创建并配置Dubbo服务的代理对象的。

具体的实现原理及过程可以参考：

[✅BeanFactory和FactroyBean的关系？](https://www.yuque.com/hollis666/fo22bm/cnhqfg?view=doc_embed)
