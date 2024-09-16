[![](media/f5e3b19936b6984f142477be33be27aa~100x100.awebp.webp)](https://juejin.cn/user/4107431172378887)

[码农小胖哥 ![lv-5](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2fa3f08a7107485f81157b296fd9d41f~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp "创作等级")

![](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/ffdbad884aa0e7884cbcf924226df6ce.svg)](https://juejin.cn/user/4107431172378887) 

2019年12月31日 13:42 ·  阅读 24428


## 1. 概述

今天带你了解一下 **Spring** 框架中的 `@Qualifier` 注解，它解决了哪些问题，以及如何使用它。我们还将了解它与 `@Primary` 注解的不同之处。更多的技术解析请访问 [felord.cn](https://link.juejin.cn?target=https%3A%2F%2Ffelord.cn "https://felord.cn")

## 2. 痛点

使用 `@Autowired` 注解是 **Spring** 依赖注入的绝好方法。但是有些场景下仅仅靠这个注解不足以让Spring知道到底要注入哪个 **bean**。 默认情况下，`@Autowired` 按类型装配 **Spring Bean**。 如果容器中有多个相同类型的 **bean**，则框架将抛出 `NoUniqueBeanDefinitionException`， 以提示有多个满足条件的 **bean** 进行自动装配。程序无法正确做出判断使用哪一个，下面就是个鲜活的例子：

```typescript
    @Component("fooFormatter")
    public class FooFormatter implements Formatter {
        public String format() {
            return "foo";
        }
    }

    @Component("barFormatter")
    public class BarFormatter implements Formatter {
        public String format() {
            return "bar";
        }
    }

    @Component
    public class FooService {
        @Autowired
        private Formatter formatter;
        
        //todo 
    }复制代码
```

如果我们尝试将 `FooService` 加载到我们的上下文中，**Spring** 框架将抛出 `NoUniqueBeanDefinitionException`。这是因为 **Spring** 不知道要注入哪个 **bean**。为了避免这个问题，有几种解决方案。那么我们本文要讲解的 `@Qualifier` 注解就是其中之一。跟着小胖哥的节奏往下走。

## 3. @Qualifier

通过使用 `@Qualifier` 注解，我们可以消除需要注入哪个 **bean** 的问题。让我们重新回顾一下前面的例子，看看我们如何通过包含 `@Qualifier` 注释来指出我们想要使用哪个 **bean** 来解决问题：

```less
    @Component
    public class FooService {
        @Autowired
        @Qualifier("fooFormatter")
        private Formatter formatter;
        
        //todo 
    }复制代码
```

通过将 `@Qualifier` 注解与我们想要使用的特定 **Spring bean** 的名称一起进行装配，**Spring** 框架就能从多个相同类型并满足装配要求的 **bean** 中找到我们想要的，避免让Spring脑裂。我们需要做的是@Component或者@Bean注解中声明的value属性以确定名称。 其实我们也可以在 `Formatter` 实现类上使用 `@Qualifier` 注释，而不是在 `@Component` 或者 `@Bean` 中指定名称，也能达到相同的效果：

```typescript
     @Component
     @Qualifier("fooFormatter")
     public class FooFormatter implements Formatter {
         public String format() {
             return "foo";
         }
     }
 
     @Component
     @Qualifier("barFormatter")
     public class BarFormatter implements Formatter {
         public String format() {
             return "bar";
         }
     }复制代码
```

## 4. @Qualifier VS @Primary

还有另一个名为 `@Primary` 的注解，我们也可以用来发生依赖注入的歧义时决定要注入哪个 **bean**。当存在多个相同类型的 **bean** 时，此注解定义了首选项。除非另有说明，否则将使用与 `@Primary` 注释关联的 **bean** 。 我们来看一个例子：

```typescript
    @Bean
    public Employee tomEmployee() {
        return new Employee("Tom");
    }

    @Bean
    @Primary
    public Employee johnEmployee() {
        return new Employee("john");
    }复制代码
```

在此示例中，两个方法都返回相同的 `Employee`类型。**Spring** 将注入的 **bean** 是方法 `johnEmployee` 返回的 **bean**。这是因为它包含 `@Primary` 注解。当我们想要指定默认情况下应该注入特定类型的 **bean** 时，此注解很有用。 如果我们在某个注入点需要另一个 **bean**，我们需要专门指出它。我们可以通过 `@Qualifier` 注解来做到这一点。例如，我们可以通过使用 `@Qualifier` 注释来指定我们想要使用 `tomEmployee` 方法返回的 **bean** 。 值得注意的是，如果 `@Qualifier` 和 `@Primary` 注释都存在，那么 `@Qualifier` 注释将具有优先权。基本上，`@Primary` 是定义了默认值，而 `@Qualifier` 则非常具体。 当然`@Component` 也可以使用`@Primary` 注解，这次使用的还是上面3的示例：

```typescript
     @Component
     @Primary
     public class FooFormatter implements Formatter {
         public String format() {
             return "foo";
         }
     }
 
     @Component
     public class BarFormatter implements Formatter {
         public String format() {
             return "bar";
         }
     }复制代码
```

在这种情况下，`@Primary` 注解指定了默认注入的是 `FooFormatter`，消除了场景中的注入歧义。

## 5. 通过名称来自动注入

在使用 `@Autowired` 进行自动装配时，如果 **Spring** 没有其他提示，将会按照需要注入的变量名称来寻找合适的 **bean**。也可以解决依赖注入歧义的问题。让我们看一些基于我们最初的例子的代码：

```ruby
    @Component
    public class FooService {
        @Autowired
        private Formatter fooFormatter;
        
        //todo 
    }复制代码
```

在这种情况下，**Spring** 将确定要注入的 **bean** 是 `FooFormatter`，因为字段名称与我们在该 **bean** 的 `@Component`或者 `@Bean` 注解中使用的值(默认 `@Bean` 使用方法名)相匹配。

## 6. 总结

通过对 `@Qualifier` 的探讨，我们知道该注解是用来消除依赖注入冲突的。这种在日常开发，比如 **Rabbtimq** 的队列声明中很常见。小胖哥也通过该注解和其他上述注解的组合使用和对比中展示了一些常用的用法。这将有助于你对 **Spring** 的依赖注入机制的了解。

`关注公众号：Felordcn获取更多资讯`

[个人博客：https://felord.cn](https://link.juejin.cn?target=https%3A%2F%2Ffelord.cn "https://felord.cn")

![](media/16f5a799c7412665~tplv-t2oaga2asx-zoom-in-crop-mark!3024!0!0!0.awebp.webp)

分类：

[后端](https://juejin.cn/backend)

标签：

[Java](https://juejin.cn/tag/Java)[Spring Boot](https://juejin.cn/tag/Spring%20Boot)