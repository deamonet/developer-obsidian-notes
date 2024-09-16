

2019年04月23日 09:26 ·  阅读 5047

# 第九期 AccessDecisionManager组件介绍

作为访问控制的最后一期，但确实整个章节部分里最简单的一部分。`ConfigAttribute`负责表述规则，`AccessDecisionVoter`负责为规则表决，但最终的访问授权是否通过是由`AccessDecisionManager`进行决策的。 这一期我们将主要介绍Spring Security中提供的三种主要决策模型。

# 一、AccessDecisionManager接口说明

`AccessDecisionManager`的接口表述非常的简单，简单来说就一个主要功——为当前的访问规则进行决策，是否给予访问的权限。无论是decide方法还是supports方法，`AccessDecisionManager`本身并不完成相关的逻辑，全部交由其管理的`AccessDecisionVoter`依次去判断与执行。而根据decide的逻辑规则不同，Spring Security中分别存在三种不同decide决策规则的`AccessDecisionManager`，它们分别是:

-   AffirmativeBased
-   UnanimousBased
-   ConsensusBased 在Spring Security默认设置中，使用的是`AffirmativeBased`。
    
    ![AccessDecisionManager接口](media/AccessDecisionManager接口.png)
    

在详细介绍三种`AccessDecisionManager`的实现类前，我们先再来梳理下`AccessDecisionManager`与`AccessDecisionVoter`的在决策框架中的关系。 在框架设计中`AccessDecisionManager`是`AccessDecisionVoter`的集合类，管理着对于不同规则进行判断与表决的`AccessDecisionVoter`们。 但不同的是，`AccessDecisionVoter`分别都只会对自己支持的规则进行表决，如一个资源的访问规则存在多个并行时，便不能以某一个`AccessDecisionVoter`的表决作为最终的访问授权结果。`AccessDecisionManager`的职责便是在这种场景下，汇总所有`AccessDecisionVoter`的表决结果后给出一个最终的决策。从而导致框架中预设了三种不同决策规则的`AccessDecisionManager`的实现类。

![image.png](media/image-32.png)

# 二、一票通过AffirmativeBased

第一个我们来介绍，Spring Security中默认提供的访问决策模型`AffirmativeBased`。一句话来说`AffirmativeBased`的逻辑就是**一票通过**——当前只要存在任何一个投了赞同表的`AccessDecisionVoter`便会最终给予相关授权。

> affirmative adj. 肯定的；积极的 n. 肯定语；赞成的一方

假设存在资源A，在`RoleVoter`中要求有Admin的角色，而在`MinutedOddVoter`中缺只要是奇数分钟则可以访问。那么在`AffirmativeBased`模型下，即时用于没有Admin的角色，只要满足奇数分钟的条件一样可以访问目标资源。

```less
    @Secured({"IS_AUTHENTICATED_FULLY","ROLE_USER","MINUTE_ODD"})
    @RequestMapping("/")
    public String root(@Autowired Authentication authentication) {
        return "index";
    }
复制代码
```

当我们奇数分钟数访问对应资源的时候：

```yaml
Voter: org.springframework.security.access.vote.RoleVoter@508280a4, returned: -1
Voter: org.springframework.security.access.vote.AuthenticatedVoter@2846f995, returned: -1
Voter: com.newnil.demo.security.MinuteBasedVoter@1ec9ec13, returned: 1
Authorization successful
复制代码
```

即使`RoleVoter`与`AuthenticatedVoter`存在明确的反对，但是因为`MinuteBasedVoter`满足了时间的要求，一样会得到一个肯定的结果。 这边有一个经验，在默认的`AffirmativeBased`的模型下客制化`AccessDecisionVoter`如果不是很决定性的规则，诸如一些辅助性的访问限制避免投出明确的赞同表，而是换个角度，投出明确的反对票，如不满足反对的情况可以投出弃权票。我们在上一期客制化的`MinuteBasedVoter`便是一个不好的反面教材^_^。

# 三、一票否决UnanimousBased

第二个，我们再来介绍一个备选的决策规则，即`UnanimousBased`所代表的一票否则制，所有人都没有反对意见。

> unanimous adj. 全体一致的；意见一致的；无异议的

其规则也十分容易懂，只要任意一个`AccessDecisionVoter`投出了反对票，则无论有多少个赞同票都无法授权访问权限。`UnanimousBased`代表了与`AffirmativeBased`完全对立的规则，有点类似五常表决的一票否则制，比如前几年著名的新闻“土耳其要求取消俄罗斯的一票否决票,这个提案被俄罗斯一票否决了”。 同样回到代码上来，我们通过调整Java Config配置代码将使用的`AccessDecisionManager`实现变更为`UnanimousBased`。

```scala
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
@Configuration
public class MethodSecurityConfiguration extends GlobalMethodSecurityConfiguration {
    @Override
    protected AccessDecisionManager accessDecisionManager() {
        List<AccessDecisionVoter<? extends Object>> decisionVoters = new ArrayList();
        ExpressionBasedPreInvocationAdvice expressionAdvice = new ExpressionBasedPreInvocationAdvice();
        expressionAdvice.setExpressionHandler(this.getExpressionHandler());
        decisionVoters.add(new RoleVoter());
        decisionVoters.add(new AuthenticatedVoter());
        decisionVoters.add(new MinuteBasedVoter());
        return new UnanimousBased(decisionVoters);
    }
}
复制代码
```

对于同样的场景下，如用户已经登录并拥有了对应的权限而因为当前时间不是偶数分钟，那么最终的决策结果因为有一票否决变为了**不可访问**

```yaml
 Voter: org.springframework.security.access.vote.RoleVoter@ddc490, returned: 0
 Voter: org.springframework.security.access.vote.AuthenticatedVoter@27f66035, returned: 1
 Voter: com.newnil.demo.security.MinuteBasedVoter@4f6b68aa, returned: -1
Access is denied (user is not anonymous);
复制代码
```

# 四、少数服从多数ConsensusBased

最后出场的`ConsensusBased`可能是三个规则里最“民主”，即少数服从多数制。

> consensus n. 一致；舆论；合意 `ConsensusBased`对所有投票的`AccessDecisionVoter`的意见进行汇总，以数量多那一方的结果为准。 但是存在一种特殊情况——平票：如果产生平票则根据配置`allowIfEqualGrantedDeniedDecisions`来判断是否通过，在默认情况下`allowIfEqualGrantedDeniedDecisions`值是true。

同样的我们修改Java Config来测试下`ConsensusBased`的行为:

```scala
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
@Configuration
public class MethodSecurityConfiguration extends GlobalMethodSecurityConfiguration {
    @Override
    protected AccessDecisionManager accessDecisionManager() {
        List<AccessDecisionVoter<? extends Object>> decisionVoters = new ArrayList();
        ExpressionBasedPreInvocationAdvice expressionAdvice = new ExpressionBasedPreInvocationAdvice();
        expressionAdvice.setExpressionHandler(this.getExpressionHandler());
        decisionVoters.add(new RoleVoter());
        decisionVoters.add(new AuthenticatedVoter());
        decisionVoters.add(new MinuteBasedVoter());
        ConsensusBased consensusBased = new ConsensusBased(decisionVoters);
        consensusBased.setAllowIfEqualGrantedDeniedDecisions(false);//可以调整平票逻辑
        return consensusBased;
    }
}
复制代码
```

我们同样在偶数分钟访问，在登录后访问受限制的资源:

```yaml
Voter: org.springframework.security.access.vote.RoleVoter@2c1ae72c, returned: 1
Voter: org.springframework.security.access.vote.AuthenticatedVoter@60d5234, returned: 1
Voter: com.newnil.demo.security.MinuteBasedVoter@6a34393c, returned: -1
Authorization successful
复制代码
```

与之前`UnanimousBased`的表现不同，因为赞同票大于否对票所以我们最终还是获取了访问的权限。

# 结尾

作为访问控制的最后一个组件，由于有了之前的铺垫和了解，三种决策规则相比之下会显得简单很多。并且在通常的情况下，对于`AccessDecisionManager`我们也不太会存在任何客制化的可能性。我们只需要了解如何选择合适的`AccessDecisionManager`与如何编写相关的Java配置代码即可。 从下一期开始，我们将进入新的主题开始介绍Spring Security中的config包下关于配置的一些内容。 我们下期再见。