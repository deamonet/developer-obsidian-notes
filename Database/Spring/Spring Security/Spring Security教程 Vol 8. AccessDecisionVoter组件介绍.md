[![](media/169b46cc8fe5df08~tplv-t2oaga2asx-no-mark!100!100!100!100.awebp.webp)](https://juejin.cn/user/3175045313342974)

[废柴大叔阿基拉 ![lv-3](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3f5a3e7550645a08184e5c4247cc3d4~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp "创作等级")](https://juejin.cn/user/3175045313342974) 

2019年04月20日 13:50 ·  阅读 2985

# 第八期 AccessDecisionVoter组件介绍

这一期主要我们将介绍访问控制三剑客负责对授权规则做角色的组件——`AccessDecisionVoter`接口。以及对Spring Security默认提供的几个基础`AccessDecisionVoter`实现类做一个详细的说明，最后我们将会客制化一个基于时间的`AccessDecisionVoter`实现用于实战说明。

-   `AccessDecisionVoter`接口说明
-   Spring Security的`AccessDecisionVoter`们
-   客制化实例：基于时间的`AccessDecisionVoter`

# 一、AccessDecisionVoter接口说明

![AccessDecisionVoter接口说明](media/AccessDecisionVoter接口说明.png)

`AccessDecisionVoter`主要的职责就是对它所对应的访问规则作出判断，当前的访问规则是否可以得到授权。 `AccessDecisionVoter`接口的主要方法其实与之前的`AuthenticationProvider`非常的相似。

```arduino
	boolean supports(ConfigAttribute attribute);

	int vote(Authentication authentication, S object,
			Collection<ConfigAttribute> attributes);
复制代码
```

-   supports方法用于判断对于当前`ConfigAttribute`访问规则是否支持；
-   如果支持的情况下，vote方法对其进行判断投票返回对应的授权结果。 最终的授权结果一共有三种,分别是同意、弃权和反对。说实话这个规则和联合国安理会投票差不多性质。当前一个访问可能存在多个规则的情况下，每一个`AccessDecisionVoter`投出自己的那一票，最终的投票结果是还是要看当前的投票规则，比如是超过1/3还是要过半数。而投票规则的判断则是被放置了在了`AccessDecisionManager`进行完成。

```ini
	int ACCESS_GRANTED = 1;
	int ACCESS_ABSTAIN = 0;
	int ACCESS_DENIED = -1;
复制代码
```

# 二、 Spring Security的`AccessDecisionVoter`们

通过上面对于`AccessDecisionVoter`的基本介绍，我们得知了一个设计上的大原则：`AccessDecisionVoter`的实现是为了满足对应规则`ConfigAttribute`。大体上来说`AccessDecisionVoter`是与`ConfigAttribute`一一对应的。 让我们回一下在上一期我们介绍的主要的几种`ConfigAttribute`实现：

-   基于Web表达式的WebExpressionConfigAttribute
-   基于@Secured注解的SecurityConfig
-   基于@Pre-@Post注解的PostInvocationExpressionAttribute 我们可以在下图中轻松的找到他门对应的`AccessDecisionVoter`。
    
    ![主要的AccessDecisionVoter](media/主要的AccessDecisionVoter.png)
    
    这边我们重点说一下在客制化场景下被利用的SecurityConfig配置和他默认的两个`AccessDecisionVoter`:
-   RoleVoter
-   AuthenticatedVoter 首先，我们来回忆下SecurityConfig的使用形式，即利用@Secured注解编写一个表达式：

```less
@Secured("ROLE_USER")
@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
复制代码
```

我们了解到了`AccessDecisionVoter`和`ConfigAttribute`的关联关系是通过supports方法进行判断，我们分别对`RoleVoter`和`AuthenticatedVoter`的supports方法进行浏览：

**RoleVoter** `RoleVoter`是Spring Security中默认基于角色规则的核心组件。在UserDetailsService中创建用户我们都会需要设置对用用户的角色信息。在默认配置下用户的角色信息都是以"ROLE_"+角色名的形式存储的。 对应的在`RoleVoter`的supports方法中会对表达式是否以'ROLE_'开始作为对应启用规则的判断。如果规则表达式是以ROLE_开始的，`RoleVoter`则会去遍历对用Authentication是否存在对应的角色，如果存在则返回通过，如果不存在则返回拒绝。

```typescript
public class RoleVoter implements AccessDecisionVoter<Object> {
	// ~ Instance fields
	// ================================================================================================

	private String rolePrefix = "ROLE_";

	// ~ Methods
	// ========================================================================================================

	public String getRolePrefix() {
		return rolePrefix;
	}

	/**
	 * Allows the default role prefix of <code>ROLE_</code> to be overridden. May be set
	 * to an empty value, although this is usually not desirable.
	 *
	 * @param rolePrefix the new prefix
	 */
	public void setRolePrefix(String rolePrefix) {
		this.rolePrefix = rolePrefix;
	}

	public boolean supports(ConfigAttribute attribute) {
		if ((attribute.getAttribute() != null)
				&& attribute.getAttribute().startsWith(getRolePrefix())) {
			return true;
		}
		else {
			return false;
		}
	}
}
复制代码
```

**AuthenticatedVoter** `AuthenticatedVoter`的使用场景就比较特殊，他并不是一个基于身份信息的访问控制，而是对于对应`Auhentication`的认证形式的一个判断。在之前的身份验证部分我们有了解过，在Spring Security设计中，我们可以铜鼓RememberMeService的方式不使用用户名和密码，而是通过存储于Cookie的信息进行授权登录。在日常工程中，对于一些敏感操作，我们要求当前的用户并不是一个基于历史进行授权认证的用户，比如在进行支付的情况下，如果我们希望用户是在本次访问中是通过用户名和密码进行登录展开的会话操作，而不是一个基于一个月前cookies进行登录都有用户。在这个场景下我们需要便可以使用`@Secured("IS_AUTHENTICATED_FULLY")`去限定用户是一个通过完全验证的用户，而不是通过RememberMe方式认证的用户。 在`AuthenticatedVoter`的supports方法中，便会判断当前的表达式是为他所支持的三种认证方法的访问控制：

-   IS_AUTHENTICATED_FULLY
-   IS_AUTHENTICATED_REMEMBERED
-   IS_AUTHENTICATED_ANONYMOUSLY 如果完全匹配，则会当前的`Authentication`对象的授权模式进行判断，返回相应的投票结果。

```java
public class AuthenticatedVoter implements AccessDecisionVoter<Object> {
	// ~ Static fields/initializers
	// =====================================================================================

	public static final String IS_AUTHENTICATED_FULLY = "IS_AUTHENTICATED_FULLY";
	public static final String IS_AUTHENTICATED_REMEMBERED = "IS_AUTHENTICATED_REMEMBERED";
	public static final String IS_AUTHENTICATED_ANONYMOUSLY = "IS_AUTHENTICATED_ANONYMOUSLY";
	// ~ Instance fields
	// ================================================================================================

	private AuthenticationTrustResolver authenticationTrustResolver = new AuthenticationTrustResolverImpl();

	// ~ Methods
	// ========================================================================================================

	private boolean isFullyAuthenticated(Authentication authentication) {
		return (!authenticationTrustResolver.isAnonymous(authentication) && !authenticationTrustResolver
				.isRememberMe(authentication));
	}

	public boolean supports(ConfigAttribute attribute) {
		if ((attribute.getAttribute() != null)
				&& (IS_AUTHENTICATED_FULLY.equals(attribute.getAttribute())
						|| IS_AUTHENTICATED_REMEMBERED.equals(attribute.getAttribute()) || IS_AUTHENTICATED_ANONYMOUSLY
							.equals(attribute.getAttribute()))) {
			return true;
		}
		else {
			return false;
		}
	}
}
复制代码
```

# 三、 客制化实例：基于时间的AccessDecisionVoter

对于`AccessDecisionVoter`结构、责任和Spring Security中提供的实现类有了一个基础的了解后。我们通过一个客制化的实例来加强这部分的理解。 我们将客制化一个基于时间的访问控制，在系统时间的分钟数是奇数的情况下才可以被访问，比如10点01分可以访问，但是10点02分则不可以被访问。

## 设计规则

首先，我们对访问规则进行设计。我们如同`RoleVoter`与`AuthenticatedVoter`一样基于@Secured注解的表达式进行扩展。我们拟定的规则名为"MINUTE_ODD"，当方法级被注解了@Secured("MINUTE_ODD")情况下，表示当前方法只有在满足系统时间的分钟数为奇数下才可以被访问。

## 客制化MinuteBasedVoter

接下来，我们编写一个`MinuteBasedVoter`扩展`AuthenticatedVoter`。

```java
public class MinuteBasedVoter implements AccessDecisionVoter {
}
复制代码
```

然后，我们实现对应的suppors方法用于完成我们对我们拟定的规则的判断。当入参ConfigAttribute 的表达式属性与我们预设的"MINUTE_ODD"一致时，那么我们便返回true告知框架，`MinuteBasedVoter`需要对该规则进行vote的投票操作。

```typescript
public class MinuteBasedVoter implements AccessDecisionVoter {
    public static final String IS_MINUTE_ODD= "MINUTE_ODD";

    @Override
    public boolean supports(ConfigAttribute attribute) {
        if ((attribute.getAttribute() != null)
                && attribute.getAttribute().equals(IS_MINUTE_ODD)) {
            return true;
        }
        else {
            return false;
        }
    }


    @Override
    public boolean supports(Class clazz) {
        return true;
    }
}
复制代码
```

最后，我们将vote的投票核心业务逻辑完成：当时间为奇数的时候则投赞同票，而在时间为偶数的时候则投一张明确的**反对票**。

```typescript
    @Override
    public int vote(Authentication authentication, Object object, Collection collection) {
        if(LocalDateTime.now().getMinute() % 2 != 0){
            return ACCESS_GRANTED;

        }else{
            return ACCESS_DENIED;
        }
    }
复制代码
```

## Java Config配置

最后，说一下如何将新的`AccessDecisionVoter`添加到现有的`AccessDecisionManager`中。我自己也百度了一下了中文世界和英文世界关于这方便的示例已经官方文档，真的是五花八门都有。最常见的是重新组织了一个AccessDecisionManager注入回Spring Security中，我很不推荐自己在方法中去new一个AccessDecisionManager。因为AccessDecisionManager的初始化过程中涉及的不只是`AccessDecisionVoter`，一不小心可能因为少设置什么组件就导致一部分默认行为没被正确的配置上去。 我推荐初学者方法是对于扩展Secured这类基于方法级的注解，单独新建一个Java Config类，然后重写原有框架中初始化`AccessDecisionManager`的方法：

```scala
@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
@Configuration
public class MethodSecurityConfiguration extends GlobalMethodSecurityConfiguration {
    @Override
    protected AccessDecisionManager accessDecisionManager() {
        AffirmativeBased affirmativeBased = (AffirmativeBased) super.accessDecisionManager();
        affirmativeBased.getDecisionVoters().add(new MinuteBasedVoter());
        return affirmativeBased;
    }
}
复制代码
```

虽然代码可能丑、有对类型强转，相对来说好理解控制很多。 在添加了MethodSecurityConfiguration的Java Config之后，我们在对受到@Secured("MINUTE_ODD")注解限制的controller方式时便会看到以下的投票日志:

```yaml
Secure object: ReflectiveMethodInvocation: public java.lang.String Attributes: [MINUTE_ODD]
Voter: org.springframework.security.access.prepost.PreInvocationAuthorizationAdviceVoter@456f4439, returned: 0
Voter: org.springframework.security.access.vote.RoleVoter@38b13fa8, returned: 0
Voter: org.springframework.security.access.vote.AuthenticatedVoter@590fa701, returned: 0
Voter: com.newnil.demo.security.MinuteBasedVoter@135c04e9, returned: 1
Authorization successful
复制代码
```

`AccessDecisionVoter`组件们依次投票，而因为当前时间是奇数，所以我们的MinuteBasedVoter投出一票值为1的赞同票。

# 结尾

这一期详细介绍了`AccessDecisionVoter`这一为访问控制提供核心判断及投票的组件。同时也通过框架默认提供与客制化实现了解了其工作原理。 下一期我们将最后一个核心组件`AccessDecisionManager`是如何对所有`AccessDecisionVoter`的投票结果进行汇总，以及如何以什么评价规则告知框架最终的授权结果进行说明。 我们下期再见。