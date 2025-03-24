[![](media/169b46cc8fe5df08~tplv-t2oaga2asx-no-mark!100!100!100!100.awebp.webp)](https://juejin.cn/user/3175045313342974)

[废柴大叔阿基拉 ![lv-3](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3f5a3e7550645a08184e5c4247cc3d4~tplv-k3u1fbpfcp-no-mark:0:0:0:0.awebp "创作等级")](https://juejin.cn/user/3175045313342974) 

2019年03月28日 22:44 ·  阅读 1601

# 前言

我们在某一期其实已经对`Authentication`身份验证中的主要组件进行过介绍，并且通过几期的分享让大家大致了解了Web应用大致利用核心进行身份验证的流程和相关的扩展点。 这一期我们用一期的篇章把关注点放在`Authentication`身份验证核心最主要的几个服务`AuthenticationMananger`、`AuthenticationProvider`和`UserDetailsService`，三个顶层接口进行展开。通过Spring Security对这些接口服务的实现进行说明讲解。目的是为了将来客制化扩展核心服务做好知识储备。

# 第五期 核心组件AuthenticationManager专题

## 本期的任务清单

1.  AuthenticationMananger与ProviderMananger
2.  Authentication与AuthenticationProvider
3.  UserDetailsService接口和它的实现类

# 零、整体概述

本期的重点是Authentication身份验证几个核心服务接口：

1.  AuthenticationMananger
2.  AuthenticationProvide
3.  UserDetailsServic

额外的还包括两个负责封装用户身份信息的接口与类：

1.  Authentication
2.  UserDetails

我们回顾下前几期我们在分享Spring Security核心组件时候曾用到过以下这张图比较重要的核心组件：

![Spring Security核心组件](media/Spring_Security核心组件.png)

这一期的重点显而易见便是红框中身份验证部分的三个核心组件以及其相关的组件。

# 一、AuthenticationMananger与ProviderMananger

`AuthenticationMananger`作为整个身份验证核心最外层的封装负责与外部使用者进行交互。 `AuthenticationMananger`接口有且仅有一个对外的服务便是“身份验证”。这样是整个身份验证服务对外提供的服务接口。

```java
	Authentication authenticate(Authentication authentication)
			throws AuthenticationException;
复制代码
```

外部使用者通过将身份验证的必要信息，比如用户名和密码封装一个Authentication传递、调用AuthenticationMananger的authenticate方法。如果没有返回异常和null值，那么验证服务便是完成。完成身份验证的Authentication不仅包含了用户的身份验证信息，比如用户名，额外还会将该用户身份下所有对应的权限列表也一并封装返回。

![AuthenticationMananger的authenticate是整个验证服务的入口](media/AuthenticationMananger的authenticate是整个验证服务的入口.png)

### 身份信息交互的纽带：Authentication

在整个与外部使用交互的过程中`Authentication`的职责有两个，第一个是封装了验证请求的参数，第二个便是封装了用户的权限信息。结合`Authentication`的接口设计便更加清晰了这样的设计意图：principal用于存放用户的身份标识信息，比如用户名，credentials用于存放用户的验证凭证比如密码，authorities用于存放用户的权限列表。而details则存放了除了用户名和密码其他可能会被用于身份验证的信息，比如应用限定用户的使用ip范围场景下，ip信息可能便会被存放在details做辅助的验证信息使用。

![Authentication设计](media/Authentication设计.png)

### 唯一的实现类：ProviderMananger

为了向外部提供身份验证服务，Spring Security中通过`ProviderMananger`实现了`AuthenticationManager`的身份验证接口。作为实现类`ProviderMananger`便不能和`AuthenticationManager`一样只关心唯一的抽象核心服务authenticate。在`ProviderMananger`为了管理外部输入与像外部返回`的Authentication`，`ProviderMananger`内部大致的工序如下：

1.  首先，寻找可以进行验证当前外部输入`Authentication`形式的`AuthenticationProvider`；如果自身的providers中无法处理验证并且当前层次的`Mananger`还有父级的`Mananger`则向上传递，交由父层`Mananger`进行处理；
2.  然后，因为details的信息是外部传入的，内部身份验证后的`Authentication`并不会从持久化或者其他数据源中携带，在返回前将details写入返回给外部的`Authentication`；
3.  最后，如果有必要则将外部身份验证请求中的敏感擦除，比如讲请求验证的密码置空。 了解了`ProviderMananger`完成的三件工作，大致明白了虽然整个验证框架只有一个`ProviderManager`暴露在外部，但是其内部可能是有多个`AuthenticationMananger`和`AuthenticationProvider`组成的网络，并且最终进行核心身份验证的还是`AuthenticationProvider`。核心在叶子节点中依次寻找对验证当前`Authentication`形式的`AuthenticationProvider`。如果存在支持便将验证请求的`Authentication`传递给`AuthenticationProvider`，委托其进行验证。在处理输入的验证请求`Authentication`，`ProviderMananger`并不对其进行任何的处理，而是指在处理完后进行必要的加工和处理。
    
    ![一个验证服务可能的组件结构](media/一个验证服务可能的组件结构.png)
    

# 二、 Authentication与AuthenticationProvider

相对AuthenticationMananger而言AuthenticationProvider的工作更加明确：针对特定的验证数据，提供特定的验证行为。在这个语境下，Authentication的设计目的是解决验证什么（What）的问题，而authenticate方法更像是在回答怎么验证的问题（How）。

## AuthenticationProvider视角中的Authentication

那么我们先对验证数据也就是Authentication的设计进行展开讨论。Authentication主要职责就是封装身份验证时候需要的信息数据，比如用户名场景下的用户名和密码，短信验证码下的手机号码和验证码，OAuth2场景下的ID和Code。总之每个不同验证协议使用的验证信息都需要被被封装成Authentication，更准确说在Spring Security把这种封装了用户身份验证信息的Authentication具体为了![\color{red}{AuthenticationToken}](https://juejin.cn/equation?tex=%5Ccolor%7Bred%7D%7BAuthenticationToken%7D)的概念，毕竟一说token更容易理解。所有Spring Security中提供的各种协议的身份验证数据的封装都继承![\color{red}{AbstractAuthenticationToken}](https://juejin.cn/equation?tex=%5Ccolor%7Bred%7D%7BAbstractAuthenticationToken%7D)，基于用户名和密码的UsernamePasswordAuthenticationToken，基于OAuth2的OAuth2AuthorizationCodeAuthenticationToken，基于CAS的CasAssertionAuthenticationToken。 通常我们使用用户名和密码的场景是最多，无论是使用基于数据库持久化的用户名密码方案还是基于LDAP的用户名和密码方法。虽然验证在验证协实现有细微差别，但是无论使用验证LDAP还是数据库进行身份验证比对，因为用户提交的验证身份信息几乎一致，我们便可以复用通用结构的AuthenticationToken——将username赋值到principal属性并将password赋值到cencredentials属性中。这样就意味着我们在AuthenticationToken设计上最需要考虑是数据的封装，而不是身份验证行为的实现。

![各式各样的AuthenticationToken](media/各式各样的AuthenticationToken.png)

我们已经解决了第一个问题，在AuthenticationProvider验证数据放都可以通过传入Authentication的各种实现类AuthenticationToken进行获取。下一个问题便是AuthenticationProvider是如何进行身份验证的。 我们假设的场景是需要对使用用户名和密码的UsernamePasswordAuthenticationToken进行验证。 在Spring Security针对UsernamePasswordAuthenticationToken进行身份验证的有主要有两个AuthenticationProvider：一个是基于Dao模型与数据层用户信息对比验证的DaoAuthenticationProvider，另外一个是虽然同样使用用户名和密码，但是验证流程更加复杂，且用户数据是通过与LDAP服务进行用户验证的LdapAuthenticationProvider。在这里使用最广泛使用的基于Dao的DaoAuthenticationProvider进行说明，如果有对Ldap实现有兴趣的相信在看完对DaoAuthenticationProvider的分析之后再阅读LdapAuthenticationProvider部分的代码就会轻松许多。

![DaoAuthenticationProvider与LdapAuthenticationProvider都基于UsernamePasswordAuthenticationToken的形式进行验证](media/DaoAuthenticationProvider与LdapAuthenticationProvider都基于UsernamePasswordAuthenticationToken的形式进行验证.png)

## DaoAuthenticationProvider

DaoAuthenticationProvider为了实现外部的验证请求便需要对外部传递身份信息——用户名和密码进行验证。我们把这个任务进一步分解成两个独立的任务：

1.  从数据层获取对应用户名在数据层的数据记录；
2.  对外部的用户名、密码与数据层的用户名、密码进行比对。

为什么在DaoAuthenticationProvider会将验证任务再分解成这两个独立任务，最大原因便是，这两个任务一个感知外部资源，另一个感知验证算法，两种都是不同用户可能存在不同的使用场景，框架并无法控制具体的实现。 第一个任务，从数据层获取对应用户名在数据层的数据记录，我们的目标是从数据层中查找到我们需要比对的用户身份数据，但是在这个场景下我们无非控制的是数据层的实现具体是什么？是通过JDBC访问Mysql还是通过JPA访问Oracle，更或是直接通过内存访问一个存储了用户信息键值对的Map？ 第二个任务，对外部的用户名、密码与数据层的用户名、密码进行比对，具体的加密算法是什么？如何实现的？ 这两个问题在DaoAuthenticationProvider中都无法给出明确的实现。Spring Security便将这两种在DaoAuthenticationProvider无法确定、存在变化的行为分别委托给了DaoAuthenticationProvider两个重要组件去完成：

1.  通过用户名返回数据层中的用户信息的UserDetailsService；
2.  通过特定加密算法处理用户密码的PasswordEncoder。
    
    ![DaoAuthenticationProvider中的两个组件](media/DaoAuthenticationProvider中的两个组件.png)
    

### 使用UserDetailsService获取内部用户身份信息

UserDetailsService接口定位从他的接口方法就可以明白，就是向核心组件们提供数据层的用户信息，而用户信息在这里被封装成了UserDetails。

![UserDetails与UserDetailsService](media/UserDetails与UserDetailsService.png)

UserDetailsService中只有一个方法便是loadUserByUsername方法，通过传入用户名返回数据层的用户身份记录。

```arduino
	UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;

复制代码
```

当我们确定我们获取用户身份信息的方法之后，我们便可以自行扩展UserDetailsService方法，告知框架如何获取用户身份信息。通常这个步骤是使用Spring Security中是必须完成的工作。 我们在第一个章节中曾经写过以下代码用于配置我们使用的UserDetailsService：

```scala
public class WebSecurityConfig  extends WebSecurityConfigurerAdapter {
    //注入新的UserDetailsServiceBean
    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        return manager;
    }
}

复制代码
```

那一次我们使用了基于内存键值对的形式来存储和获取用户信息记录。同样的我们也可以通过JDBC和JPA来获取用户身份信息，而Spring Security很贴心的已经提供了一个基于JDBC的UserServiceDetails实现和对应的模板DDL：

![JBCD的UserServiceDetails实现JdbcImpl](media/JBCD的UserServiceDetails实现JdbcImpl.png)

```sql
create table users(username varchar_ignorecase(50) not null primary key,password varchar_ignorecase(500) not null,enabled boolean not null);
create table authorities (username varchar_ignorecase(50) not null,authority varchar_ignorecase(50) not null,constraint fk_authorities_users foreign key(username) references users(username));
create unique index ix_auth_username on authorities (username,authority);
复制代码
```

而通过UserDetailsService返回的数据类型是UserDetails，UserDetails中封装了用户名、密码和授权信息同时还额外包括了一些过期和锁定的标识属性。我们不难发现UserDetails封装的数据和Authentication非常的相似。没错，在身份验证成功后，DaoAuthenticationProvider便会将内部的UserDetails抽离必要的数据对应赋值到UsernamePasswordAuthenticationToken最终返回给外部调用者进行使用。我们可以简单的把UserDetails理解为用户身份信息在数据层的封装。在客制化的过程中，如果用户信息的数据结构是比较特殊的结构，比如Ldap，那么便可以自行扩展UserDetails客制化一个特殊的结构用于获取用户数据记录。 说到这里基本上我们已经了解了DaoAuthenticationProvider两个组件中用于获取用户身份记录的UserDetailsService部分，下面我们介绍下处理密码验证的PasswordEncoder部分。

### 使用PasswordEncoder来进行密码的比对

我们继续追踪上面的场景来说下关于验证算法部分。DaoAuthenticationProvider收到了外部提交的用户名和密码，同样的DaoAuthenticationProvider也查找到了对应用户名在数据库中的用户名和密码。通常情况下虽然都是密码，数据库中存储的密码通常会进行过一定的加密。DaoAuthenticationProvider便需要将外部提交的用户名和密码进行一次加密流程并进行比对。举个例子我们当前使用的算法比如是MD5，加密明文的样本是用户的用户名拼接用户名。如有当前需要身份验证的请求中用户名是admin，密码是password。同样的在数据库中加密后的密码是9b02edfbc208a538。我们便需要对外部传递的用户名和密码做一个MD5("passwordadmin")得到9b02edfbc208a538，再与数据库中的password字段进行对比，如果一致则认定验证成功。 在Spring Security中这种针对处理称为![\color{red}{PasswordEncoder}](https://juejin.cn/equation?tex=%5Ccolor%7Bred%7D%7BPasswordEncoder%7D)，PasswordEncoder接口主要的作用就是对明文密码进行加密与比对。

![PasswordEncoder接口](media/PasswordEncoder接口.png)

![通过PasswordEncoder比对密码的示意图](media/通过PasswordEncoder比对密码的示意图.png)

Spring Security中默认向DaoAuthenticationProvider提供的PasswordEncoder是BCryptPasswordEncoder。如果对BCrypt可以额外通过谷歌去了解加密流程。 如果我们需要客制化自己的加密算法，只要实现PasswordEncoder接口，并重新通过Spring注入DaoAuthenticationProvider便可以了。

```typescript
@Bean
public PasswordEncoder passwordEncoder() {
    //通过修改注入的实例，客制化自己的PasswordEncoder
    return new BCryptPasswordEncoder();
}
复制代码
```

PasswordEncoder 部分的功能相对较少，通常情况下使用默认提供的BCryptPasswordEncoder就足够完成任务。

## 结尾

在本期我们花了很大篇幅介绍了身份验证核心中最主要个几个组件和基于一个使用用户名和密码验证场景下对应接口的实现类的具体职责：

1.  AuthenticationManager负责核心验证前后的处理，并且负责与外部调用者进行交互；
2.  AuthenticationProvider是验证服务的核心实现，其验证的数据形式是AuthenticationToken其中封装了验证使用的用户标识和用户验证凭证信息；
3.  AuthenticationProvider中获取内部用户身份信息是通过UserDetailsService完成的。如需要进行密码处理，则引入了PasswordEncoder；
4.  UserDetails封装了用户信息在内部的结构，在向外部返回Authentication之前，AuthenticationProvider通常会将UserDetails必要的数据复制到向外部返回的AuthenticationToken中。

通过几期的说明，整个Spring Security关于身份验证的组件、流程和特定场景的实现基本我们都了解了一遍。从下一期开始，我们会开始讲解访问控制部分、框架配置部分的设计与概念。同时也将不定期通过一些场景的实战强相关框架概念和设计理念。 谢谢大家，我们下期再见。