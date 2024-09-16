Spring Security 中最基本的组件应该是SecurityContextHolder了。这是一个工具类，只提供一些静态方法。这个工具类的目的是用来保存应用程序中当前使用人的安全上下文。
SecurityContextHolder的工作原理
缺省工作模式 MODE_THREADLOCAL

我们知道，一个应用同时可能有多个使用者，每个使用者对应不同的安全上下文，那么SecurityContextHolder是怎么保存这些安全上下文的呢 ？缺省情况下，SecurityContextHolder使用了ThreadLocal机制来保存每个使用者的安全上下文。这意味着，只要针对某个使用者的逻辑执行都是在同一个线程中进行，即使不在各个方法之间以参数的形式传递其安全上下文，各个方法也能通过SecurityContextHolder工具获取到该安全上下文。只要在处理完当前使用者的请求之后注意清除ThreadLocal中的安全上下文，这种使用ThreadLocal的方式是很安全的。当然在Spring Security中，这些工作已经被Spring Security自动处理，开发人员不用担心这一点。

这里提到的SecurityContextHolder基于ThreadLocal的工作方式天然很适合Servlet Web应用，因为缺省情况下根据Servlet规范，一个Servlet request的处理不管经历了多少个Filter，自始至终都由同一个线程来完成。

    注意 : 这里讲的是一个Servlet request的处理不管经历了多少个Filter，自始至终都由同一个线程来完成;而对于同一个使用者的不同Servlet request,它们在服务端被处理时，使用的可不一定是同一个线程(存在由同一个线程处理的可能性但不确保)。

其他工作模式

有一些应用并不适合使用ThreadLocal模式，那么还能不能使用SecurityContextHolder了呢？答案是可以的。SecurityContextHolder还提供了其他工作模式。

比如有些应用，像Java Swing客户端应用，它就可能希望JVM中所有的线程使用同一个安全上下文。此时我们可以在启动阶段将SecurityContextHolder配置成全局策略MODE_GLOBAL。

还有其他的一些应用会有自己的线程创建，并且希望这些新建线程也能使用创建者的安全上下文。这种效果，可以通过将SecurityContextHolder配置成MODE_INHERITABLETHREADLOCAL策略达到。
使用SecurityContextHolder
获取当前用户信息

在SecurityContextHolder中保存的是当前访问者的信息。Spring Security使用一个Authentication对象来表示这个信息。一般情况下，我们都不需要创建这个对象，在登录过程中，Spring Security已经创建了该对象并帮我们放到了SecurityContextHolder中。从SecurityContextHolder中获取这个对象也是很简单的。比如，获取当前登录用户的用户名，可以这样 :


// 获取安全上下文对象，就是那个保存在 ThreadLocal 里面的安全上下文对象
// 总是不为null(如果不存在，则创建一个authentication属性为null的empty安全上下文对象)
SecurityContext securityContext = SecurityContextHolder.getContext();

// 获取当前认证了的 principal(当事人),或者 request token (令牌)
// 如果没有认证，会是 null,该例子是认证之后的情况
Authentication authentication = securityContext.getAuthentication()

// 获取当事人信息对象，返回结果是 Object 类型，但实际上可以是应用程序自定义的带有更多应用相关信息的某个类型。
// 很多情况下，该对象是 Spring Security 核心接口 UserDetails 的一个实现类，你可以把 UserDetails 想像
// 成我们数据库中保存的一个用户信息到 SecurityContextHolder 中 Spring Security 需要的用户信息格式的
// 一个适配器。
Object principal = authentication.getPrincipal();
if (principal instanceof UserDetails) {
	String username = ((UserDetails)principal).getUsername();
} else {
	String username = principal.toString();
}

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    18
    19
    20

修改SecurityContextHolder的工作模式

综上所述，SecurityContextHolder可以工作在以下三种模式之一:

    MODE_THREADLOCAL (缺省工作模式)
    MODE_GLOBAL
    MODE_INHERITABLETHREADLOCAL
    修改SecurityContextHolder的工作模式有两种方法 :
        设置一个系统属性(system.properties) : spring.security.strategy;

        SecurityContextHolder会自动从该系统属性中尝试获取被设定的工作模式

        调用SecurityContextHolder静态方法setStrategyName()

        程序化方式主动设置工作模式的方法

SecurityContextHolder源码

    本文源代码基于 Spring Security Core 4.x.x

package org.springframework.security.core.context;

import org.springframework.util.ReflectionUtils;
import org.springframework.util.StringUtils;

import java.lang.reflect.Constructor;

/**
 * 将一个给定的SecurityContext绑定到当前执行线程。
 * 
 * This class provides a series of static methods that delegate to an instance of
 * org.springframework.security.core.context.SecurityContextHolderStrategy. The
 * purpose of the class is to provide a convenient way to specify the strategy that should
 * be used for a given JVM. This is a JVM-wide setting, since everything in this class is
 * static to facilitate ease of use in calling code.
 * 
 * To specify which strategy should be used, you must provide a mode setting. A mode
 * setting is one of the three valid MODE_ settings defined as
 * static final fields, or a fully qualified classname to a concrete
 * implementation of
 * org.springframework.security.core.context.SecurityContextHolderStrategy that
 * provides a public no-argument constructor.
 * 
 * There are two ways to specify the desired strategy mode String. The first
 * is to specify it via the system property keyed on #SYSTEM_PROPERTY. The second
 * is to call #setStrategyName(String) before using the class. If neither approach
 * is used, the class will default to using #MODE_THREADLOCAL, which is backwards
 * compatible, has fewer JVM incompatibilities and is appropriate on servers (whereas
 * #MODE_GLOBAL is definitely inappropriate for server use).
 *
 * @author Ben Alex
 *
 */
public class SecurityContextHolder {
	// ~ Static fields/initializers
	// =====================================================================================

	// 三种工作模式的定义，每种工作模式对应一种策略
	public static final String MODE_THREADLOCAL = "MODE_THREADLOCAL";
	public static final String MODE_INHERITABLETHREADLOCAL = "MODE_INHERITABLETHREADLOCAL";
	public static final String MODE_GLOBAL = "MODE_GLOBAL";

	// 类加载时首先尝试从环境属性中获取所指定的工作模式
	public static final String SYSTEM_PROPERTY = "spring.security.strategy";	
	private static String strategyName = System.getProperty(SYSTEM_PROPERTY);
	private static SecurityContextHolderStrategy strategy;

	// 初始化计数器,初始为0,
	// 1. 类加载过程中会被初始化一次，此值变为1
	// 2. 此后每次调用 setStrategyName 会对新的策略对象执行一次初始化，相应的该值会增1
	private static int initializeCount = 0;

	static {
		initialize();
	}

	// ~ Methods
	// =====================================================================================

	/**
	 * Explicitly clears the context value from the current thread.
	 */
	public static void clearContext() {
		strategy.clearContext();
	}

	/**
	 * Obtain the current SecurityContext.
	 *
	 * @return the security context (never null)
	 */
	public static SecurityContext getContext() {
		return strategy.getContext();
	}

	/**
	 * Primarily for troubleshooting purposes, this method shows how many times the class
	 * has re-initialized its SecurityContextHolderStrategy.
	 *
	 * @return the count (should be one unless you've called
	 * #setStrategyName(String) to switch to an alternate strategy.
	 */
	public static int getInitializeCount() {
		return initializeCount;
	}

	private static void initialize() {
		if (!StringUtils.hasText(strategyName)) {
			// Set default, 设置缺省工作模式/策略 MODE_THREADLOCAL
			strategyName = MODE_THREADLOCAL;
		}

		if (strategyName.equals(MODE_THREADLOCAL)) {
			strategy = new ThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_INHERITABLETHREADLOCAL)) {
			strategy = new InheritableThreadLocalSecurityContextHolderStrategy();
		}
		else if (strategyName.equals(MODE_GLOBAL)) {
			strategy = new GlobalSecurityContextHolderStrategy();
		}
		else {
			// Try to load a custom strategy
			try {
				Class<?> clazz = Class.forName(strategyName);
				Constructor<?> customStrategy = clazz.getConstructor();
				strategy = (SecurityContextHolderStrategy) customStrategy.newInstance();
			}
			catch (Exception ex) {
				ReflectionUtils.handleReflectionException(ex);
			}
		}

		initializeCount++;
	}

	/**
	 * Associates a new SecurityContext with the current thread of execution.
	 *
	 * @param context the new SecurityContext (may not be null)
	 */
	public static void setContext(SecurityContext context) {
		strategy.setContext(context);
	}

	/**
	 * Changes the preferred strategy. Do NOT call this method more than once for
	 * a given JVM, as it will re-initialize the strategy and adversely affect any
	 * existing threads using the old strategy.
	 *
	 * @param strategyName the fully qualified class name of the strategy that should be
	 * used.
	 */
	public static void setStrategyName(String strategyName) {
		SecurityContextHolder.strategyName = strategyName;
		initialize();
	}

	/**
	 * Allows retrieval of the context strategy. See SEC-1188.
	 *
	 * @return the configured strategy for storing the security context.
	 */
	public static SecurityContextHolderStrategy getContextHolderStrategy() {
		return strategy;
	}

	/**
	 * Delegates the creation of a new, empty context to the configured strategy.
	 */
	public static SecurityContext createEmptyContext() {
		return strategy.createEmptyContext();
	}

	public String toString() {
		return "SecurityContextHolder[strategy='" + strategyName + "'; initializeCount="
				+ initializeCount + "]";
	}
}

    1
    2
    3
    4
    5
    6
    7
    8
    9
    10
    11
    12
    13
    14
    15
    16
    17
    18
    19
    20
    21
    22
    23
    24
    25
    26
    27
    28
    29
    30
    31
    32
    33
    34
    35
    36
    37
    38
    39
    40
    41
    42
    43
    44
    45
    46
    47
    48
    49
    50
    51
    52
    53
    54
    55
    56
    57
    58
    59
    60
    61
    62
    63
    64
    65
    66
    67
    68
    69
    70
    71
    72
    73
    74
    75
    76
    77
    78
    79
    80
    81
    82
    83
    84
    85
    86
    87
    88
    89
    90
    91
    92
    93
    94
    95
    96
    97
    98
    99
    100
    101
    102
    103
    104
    105
    106
    107
    108
    109
    110
    111
    112
    113
    114
    115
    116
    117
    118
    119
    120
    121
    122
    123
    124
    125
    126
    127
    128
    129
    130
    131
    132
    133
    134
    135
    136
    137
    138
    139
    140
    141
    142
    143
    144
    145
    146
    147
    148
    149
    150
    151
    152
    153
    154
    155
    156
    157
    158
    159
    160

参考文章

    Spring Security 中跨 request 请求保持 SecurityContext

文章知识点与官方知识档案匹配，可进一步学习相关知识
Java技能树首页概览109964 人正在系统学习中