https://juejin.cn/post/6844903811069263879

2019年04月01日 16:10 ·  阅读 1556

# 前言

从这期开始我们将主要对Spring Security的另一个领域Authority，通常被称为访问控制的部分进行说明。 相比较Authentication身份验证而言，客制化访问控制的可能性相对会低许多。所以我们对这部分主要是理解流程和分析设计上的一些动机为主。 管理访问控制对大多数读者来说都都不太陌生，Spring Security基于角色管理的访问控制学习起来也会较身份验证来的又有代入感。

# 第六期 初识访问控制

1.  访问控制的概述
2.  访问控制中主要的组件及实现介绍

# 一、关于整体结构

在之前的分享中，我们已经了解了在Spring Security中，对于访问的身份验证是通过WebFilter作为入口。然后调用`AuthenticationManager`暴露的验证服务接口进行验证的。 同样的，在验证身份之后，如访问受限资源，同样也会如果身份验证一样，被某个WebFilter作为入口。然后调用一个名为`AccessDecisionManager`的访问控制决策管理器进行验证。

![AccessDecisionManager的三种主要实现类](media/AccessDecisionManager的三种主要实现类.png)

在Spring Security中，主要关于访问控制的代码都被放在了`org.springframework.security.access.vote`包中，其中主要的接口分为三种:

-   `AccessDecisionManager`: 负责整个访问控制授权部分的投票策略和管理；
-   `AccessDecisionVoter`: 负责对访问控制的规则进行表决，是否授权用户访问目标资源；
-   `ConfigAttribute`: 用于保存相关的访问控制规则

![access中主要的接口](media/access中主要的接口.png)

用一句话描述他们的责任就是：`AccessDecisionVoter`负责对`ConfigAttribute`进行表决，`AccessDecsionManager`汇总表决，最终向框架返回最终的授权结果。

# 二、ConfigAttribute组件介绍

在一开始，我们先来回顾下，最早的应用中，我们是如何进行访问控制的:

```scss
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/").permitAll()
                .antMatchers("/user").hasRole("USER")
                .antMatchers("/user").denyAll()
                .and()
                .formLogin();
    }
复制代码
```

我们通过在configure方法中编写路径模式和相应的限定规则来配置整个应用的访问控制，基于Web表达式的访问控制规则，除了支持Spring Security中默认提供的一些限制方法以外，也可以额通过`.antMatchers("/user").access("hasRole('USER') or hasRole('ADMIN')")`的access方法加上表达式的形式进行扩展。表达式更是可以将表达判断委托给一个Java Class去完成。 比如下方示例代码就是讲表达式的判断委托给了名为webSecurity的Bean中的check方法去完成。

```perl
http
        .authorizeRequests()
                .antMatchers("/user/**").access("@webSecurity.check(authentication,request)")
                ...
复制代码
```

除了Web表达式以外，Spring Security还提供了几种进行访问控制配置的方式，其中最主要的一种便是通过注解在方法级对Controller和Service的方法进行对应的访问控制的设置。

![ConfigAttribute](media/ConfigAttribute.png)

其中Spring Security对应方法级的注解主要又可以分为两类：

-   第一类 `@Secured` 注解 - secured-annotations
-   第二类 `@PreAuthorize, @PreFilter, @PostAuthorize and @PostFilter` - pre-post-annotations

而在Spring Security中以上两种注解默认是禁用的，我们需要通过激活配置才可以进行使用。

```less
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
@EnableWebSecurity
public class WebSecurityConfig  extends WebSecurityConfigurerAdapter {
}
复制代码
```

> 关于相关访问控制规则的详细介绍和应用，我们将在以后的专题中进行逐一说明。当前只是为将来的分析打一下基础。

# 三、AccessDecisionVoter组件介绍

`AccessDecisionVoter`可是说是整个访问控制、授权业务的核心。每一个`AccessDecisionVoter`都需要对它所支持的访问控制规则进行投票。投票结果只有以下三种情况:

```ini
	int ACCESS_GRANTED = 1; // 赞成
	int ACCESS_ABSTAIN = 0;  // 弃权
	int ACCESS_DENIED = -1;  // 反对
复制代码
```

![AccessDecisionVoter](media/AccessDecisionVoter.png)

从`AccessDecisionVoter`的接口方法签名可以看出，其主要两个方法分别对应的职责一个是判断是否支持当前的访问控制规则，另一个便是对支持的访问规则进行投票。 在Spring Security提供的`AccessDecisionVoter`实现类主要有:

-   基于Web表达是配置的`WebExpressionVoter`;
-   基于角色名前缀'ROLE_'对角色限制判断的`RoleVoter`;
-   根据当前Authentication授权形式判断的`AuthenticationVoter`。 而三种Voter对应的访问控制规则又有些略微的不同，在身份验证模块我们了解到基本上每一个`AuthenticationProvider`都需要定制化一种`AuthenticationToken'。而在访问控制模块中，同样的每一个`AccessDecisionVoter`也可以定制化一种`ConfigAttribute`。例如，对于`WebExpressionVoter`其对应的便是之前提到的`WebExpressionConfigAttribute`。

> 框架提供的AccessDecisionVoter一般可以满足中小应用下的访问控制的基本场景。如果我们当前开发的大型应用有复杂的访问控制模型，那么`AccessDecisionVoter`的客制化便是我们一定会面对的问题，在将来的专题里我们会单独针对如何客制化`AccessDecisionVoter`进行说明。

# 四、AccessDecisionManager组件介绍

`AccessDecisionManager`对于访问控制业务的作用与`AuthenticationManager`对于身份验证的业务的作用差不多。其中一个便是对暴露了唯一的访问控制验证接口。而与`AuthenticationManager`不同的地方是，在Spring Security中针对`AuthenticationManager`只提供了一种`ProviderManager`实现类，而`AccessDecisionManager`缺因为有不同的表决制度分别提供了三种实现类：

-   `AffirmativeBased` 一票赞同制
-   `UnanimousBased` 一票否决制
-   `ConsensusBased` 少数服从多数

![AccessDecisionManager](media/AccessDecisionManager.png)

Spring Security在默认的配置下使用的`AccessDecisionManager`是`AffirmativeBased`。如果需要变更的，则可以通过配置文件修改注入的Bean即可，比如下面的Java Config形式。

```typescript
    @Bean
    public AccessDecisionManager accessDecisionManager() {
        List<AccessDecisionVoter<? extends Object>> decisionVoters
                = Arrays.asList(

                new WebExpressionVoter(),
                new RoleVoter(),
                new AuthenticatedVoter()
        );
        return  new UnanimousBased(decisionVoters);
    }
复制代码
```

> 对于`AccessDecisionManager`而言，框架提供的的三种表决制度在绝大多数情况都已经足够强大。我们对于其的了解便只需要集中在如何配置和管理使用的`AccessDecisionVoter`便可。

# 结尾

访问控制相关通常是我们使用Spring Security客制化扩展最频繁的模块。所以与别的模块介绍的流程不同，本次是先将整个访问控制模块的主要接口和组件进行基本的介绍。在后面几期中，我们将对每个组件和其应用进行展开讨论。