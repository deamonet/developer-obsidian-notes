
https://juejin.cn/post/6844903825359241229

2019年04月17日 22:29 ·  阅读 2405

# 第七期 访问规则ConfigAttribute

从这一期开始，我们将分别对访问控制模块主要的三个组件进行介绍。首先，我们将对访问控制的配置部分展开说明，在这一期我们将对以下内容进行讲解：

1.  ConfigAttribute的常用组件
2.  WebExpressionConfigAttribute
3.  SecurityConfig
4.  PostInvocationExpressionAttribute

# 一、ConfigAttribute的常用组件

ConfigAttribute作为访问控制模板用于“描述”规则的组件，最常见的方式主要是两种一种是基于注解的一种是基于JavaConfig在Web配置中进行配置的。 其中Spring Security对应方法级的注解主要又可以分为两类： 第一类 `@Secured` 注解 - secured-annotations 第二类 `@PreAuthorize, @PreFilter, @PostAuthorize and @PostFilter` - pre-post-annotations

![ConfigAttribute的常用组件](media/ConfigAttribute的常用组件.png)

## 1.1 WebExpressionConfigAttribute

WebExpressionConfigAttribute是我们最早也是最常见的访问控制方式。比如下面的代码形式

```scss
 http.authorizeRequests()
        .antMatchers("/").access("hasAnyRole('ANONYMOUS', 'USER')")
        .antMatchers("/login/*").access("hasAnyRole('ANONYMOUS', 'USER')")
        .antMatchers("/logout/*").access("hasAnyRole('ANONYMOUS', 'USER')")
        .antMatchers("/admin/*").access("hasRole('ADMIN')")
        .antMatchers("/events/").access("hasRole('ADMIN')")
        .antMatchers("/**").access("hasRole('USER')")
复制代码
```

基于Web表达是可以对目标URL的模式进行访问控制，而控制检查的规则最常见两种方式一种是基于角色（Role-Based），另一种是基于表达式（Expressions-Based）。 演示代码中使用的是access的表达式进行控制，同样的也可以直接使用下面的形式通过角色来达到同样的效果。

```arduino
        .antMatchers("/logout/*").hasAnyRole("USER","ANONYMOUS")
复制代码
```

可以说基于Web表达式对于Web资源的控制是我们最长见的方式。 除此以外，access还有另外一个很酷炫的功能就是通过表达式将判断的方法委托表达式内的方法进行判断。如果最终表达式返回false则会返回403禁止访问，true则表示授权访问。 举个例子，假设我们系统内针对/user路径需要对用户名进行判断，如果用户名不是‘user’则不可以访问。我们把对应判断的代码写在一个CustomWebSecurity的Bean中。

```java
@Component()
public class CustomWebSecurity {
        public boolean check(Authentication authentication) {
            return  (authentication.getName().equals("user"));
        }
}
复制代码
```

然后我们通过修改Web表达式我访问控制向/user的路径增加这样一个访问控制的表达式。

```java
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/user").access("@customWebSecurity.check(authentication)")
                .and()
                .formLogin();
    }
复制代码
```

最后，我们为了达到测试的目的对应的添加两个用户

```java
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().withUser(User.withUsername("user").password("{noop}password").roles("USER"));
        auth.inMemoryAuthentication().withUser(User.withUsername("test").password("{noop}password").roles("USER"));
    }
复制代码
```

整个测试的Security配置类如下：

```scala
@Configuration
@EnableWebSecurity
public class WebSecurityConfig  extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .authorizeRequests()
                .antMatchers("/user").access("@customWebSecurity.check(authentication)")
                .and()
                .formLogin();
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.inMemoryAuthentication().withUser(User.withUsername("user").password("{noop}password").roles("USER"));
        auth.inMemoryAuthentication().withUser(User.withUsername("test").password("{noop}password").roles("USER"));
    }
}
复制代码
```

然后，我们分别登录用户进行测试，首先我们登录user的账户。

![登录界面](media/登录界面.png)

输入user和password之后，我们可以正确的访问/user的页面。

![正确的/user页面](media/正确的!user页面.png)

但当我们使用test和password以test用户身份登录时，因为无法通过access中的check方法对用户名的检查，所以我们便会得到一个403禁止访问的错误。

![image.png](media/image-33.png)

## 1.2 SecurityConfig

说完了，最常用的WebExpressionConfigAttribute了解了基于角色与表达式对目标地址进行访问控制的配置之后，接下来我们分别对两个基于方法级进行控制的配置展开介绍。 首先我们需要在相关的JavaConfig中激活相关的方法级的验证。

```java
@EnableGlobalMethodSecurity(securedEnabled = true) // <--打开Secured注解配置
public class MethodSecurityConfig {
// ...
}
复制代码
```

之后我们便可以在我们需要进行权限验证的地方增加相关的注解和规则：

```kotlin
public interface BankService {

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account readAccount(Long id);

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account[] findAccounts();

@Secured("ROLE_TELLER")
public Account post(Account account, double amount);
}
复制代码
```

`@Secured("ROLE_TELLER")`中的表达式便是验证规则，所有的`AccessDecisionVoter`都会一次检查是否对该表达式支持，比如这个例子里，`RoleVoter`便会对该规则进行表决：如果当前访问的用户拥有`TELLER`的角色，那么就可以继续执行该方法；如果不没有对应的角色，那么就会返回一个403的错误。 同理`@Secured("IS_AUTHENTICATED_ANONYMOUSLY")`对应的便是`AuthenticatedVoter`。 利用这种特性，我们也可以通过自定义相关表达式与客制化对应的`AccessDecisionVoter`来完成特有的访问控制逻辑。

## 1.3 PostInvocationExpressionAttribute

最后则是利用切面进行访问控制逻辑的@Pre与@Post注解。同样的如果需要使用这种方法级的配置，也需要激活对应的配置。

```java
@EnableGlobalMethodSecurity(prePostEnabled = true) // <--打开Pre与Post注解配置
public class MethodSecurityConfig {
// ...
}
复制代码
```

与@Secured注解不同，如果激活了改配置，则可以使用以下四个注解对Java方法级进行访问控制和处理：

-   @PreAuthorize
-   @PreFilter
-   @PostAuthorize
-   @PostFilter

### @PreAuthorize

其中比较常用的可能是`@PreAuthorize`注解，`@PreAuthrize`在功能上与`@Secured`基本是一致的，便是在运行被注解的方法事前先对规则进行投票，如果通过之后则授权进行访问。相比`@Secured`而言`@PreAuthrize`并不是依靠不同的`AccessDecisionVoter`来完成，而是依靠其编写的各种表达式的值进行判断。 比如我们对方法入参的用户名进行判断，则可使用

```less
@PreAuthorize("#n == authentication.name")
Contact findContactByName(@Param("n") String name);
复制代码
```

或者

```java
@PreAuthorize("#contact.name == authentication.name")
public void doSomething(Contact contact);
复制代码
```

`@PreAuthorize`的表达式是非常强大工具，毕竟注入Authentication对象的方法在写测试用例的时候就非常的痛苦了……

```less
public void doSomething(Contact contact,@Autowired Authentication authentication){
    if (contact.name == authentication.name){   
    ///
   }
};
复制代码
```

相对先验的`@PreAuthorize`来说后验的`@PostAuthorize`注解的使用场景就基本很罕见了。是一个几乎可以忽略的注解。

### @PreFilter & @PostFilter

接下来说两个其他教程中都不太提到的Spring Security中的两个Filter注解。这两个注解的本质是通过在方法级编写了一个Spring-EL表达式对方法使用和返回的集合进行过滤。 比如我们最常见的场景便是不同用户可能生成的菜单可能不同，我们可能会给每一个菜单都赋予一个`Permission`权限。但是如果在这里展开怕是3000个字也说不清楚，有机会单独在ACL部分做一个实战型的说明。 这边举个简单的例子，假设我们现在对用户使用的一个集合进行过滤，如果规则是只可访问奇数ID的对象。换言之，过滤的规则则是“当ID为偶数”。

```typescript
   @PreFilter(filterTarget="ids", value="filterObject%2==0")
 
   public void doSomething(List<Integer> ids) {
 
   }
复制代码
```

如果是对方法返回值的集合进行过滤，则需要使用`@PostFilter`

```sql
   @PostFilter("filterObject.id%2==0"）
  public List<User> findAll() {

      List<User> userList = new ArrayList<User>();

      User user;

      for (int i=0; i<10; i++) {

         user = new User();

         user.setId(i);

         userList.add(user);

      }

      return userList;
   }
复制代码
```

在SpringSecurity中也内建了一部分表达式规则，如基于Permission对相应对象做权限验证过滤：

```java
@PostFilter("hasPermission(filterObject, 'read') or hasPermission(filterObject, 'admin')")
public List<Contact> getAll();
复制代码
```

而这一部分便设计了`PermissionEvaluator`权限评估器还完成相应进行目标领域对象操作所需要的权限逻辑。而这一部分则是在ACL客制化的重点。

![PermissionEvaluator接口](media/PermissionEvaluator接口.png)

# 结尾

这一期稍微详细的带大家走马观花般的了解了下Spring Security提供访问控制各种配置方法和其使用场景。 因为针对这一部分如果过度展开脱离实战场景也非常难掌握，所以这一期的真的就只是让大家了解Spring Security针对不同的访问控制颗粒细度应该怎么配置，比如URL级、代码方法级、领域集合级别的权限过滤或者是客制化对应的控制逻辑。 下一期我们将先对访问控制另外一个核心，即如何针对配置进行处理的`AccessDecisionVoter`接口组件进行说明。 我们下期再见。