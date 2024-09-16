Asked 6 years, 5 months ago

Modified [3 months ago](https://stackoverflow.com/questions/40418441/spring-security-cors-filter?lastactivity "2022-12-17 18:01:54Z")

Viewed 226k times

63

[](https://stackoverflow.com/posts/40418441/timeline)

We added `Spring Security` to our existing project.

From this moment on we get a 401 `No 'Access-Control-Allow-Origin' header is present on the requested resource` error from the our server.

That's because no `Access-Control-Allow-Origin` header is attached to the response. To fix this we added our own filter which is in the `Filter` chain before the logout filter, but the filter does not apply for our requests.

Our Error:

> XMLHttpRequest cannot load `http://localhost:8080/getKunden`. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin `http://localhost:3000` is therefore not allowed access. The response had HTTP status code 401.

Our Security configuration:

```java
@EnableWebSecurity
@Configuration
@ComponentScan("com.company.praktikant")
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

@Autowired
private MyFilter filter;

@Override
public void configure(HttpSecurity http) throws Exception {
    final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    final CorsConfiguration config = new CorsConfiguration();

    config.addAllowedOrigin("*");
    config.addAllowedHeader("*");
    config.addAllowedMethod("GET");
    config.addAllowedMethod("PUT");
    config.addAllowedMethod("POST");
    source.registerCorsConfiguration("/**", config);
    http.addFilterBefore(new MyFilter(), LogoutFilter.class).authorizeRequests()
            .antMatchers(HttpMethod.OPTIONS, "/*").permitAll();
}

@Autowired
public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
}
}
```

Our filter

```java
@Component
public class MyFilter extends OncePerRequestFilter {

@Override
public void destroy() {

}

private String getAllowedDomainsRegex() {
    return "individual / customized Regex";
}

@Override
protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
        throws ServletException, IOException {

    final String origin = "http://localhost:3000";

    response.addHeader("Access-Control-Allow-Origin", origin);
    response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS");
    response.setHeader("Access-Control-Allow-Credentials", "true");
    response.setHeader("Access-Control-Allow-Headers",
            "content-type, x-gwt-module-base, x-gwt-permutation, clientid, longpush");

    filterChain.doFilter(request, response);

}
}
```

Our Application

```java
@SpringBootApplication
public class Application {
public static void main(String[] args) {
    final ApplicationContext ctx = SpringApplication.run(Application.class, args);
    final AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext();
    annotationConfigApplicationContext.register(CORSConfig.class);
    annotationConfigApplicationContext.refresh();
}
}
```

Our filter is registered from spring-boot:

> 2016-11-04 09:19:51.494 INFO 9704 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean : Mapping filter: 'myFilter' to: [/*]

Our generated filterchain:

> 2016-11-04 09:19:52.729 INFO 9704 --- [ost-startStop-1] o.s.s.web.DefaultSecurityFilterChain : Creating filter chain: org.springframework.security.web.util.matcher.AnyRequestMatcher@1, [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@5d8c5a8a, org.springframework.security.web.context.SecurityContextPersistenceFilter@7d6938f, org.springframework.security.web.header.HeaderWriterFilter@72aa89c, org.springframework.security.web.csrf.CsrfFilter@4af4df11, com.company.praktikant.MyFilter@5ba65db2, org.springframework.security.web.authentication.logout.LogoutFilter@2330834f, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@396532d1, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@4fc0f1a2, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@2357120f, org.springframework.security.web.session.SessionManagementFilter@10867bfb, org.springframework.security.web.access.ExceptionTranslationFilter@4b8bf1fb, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@42063cf1]

The Response: [Response headers](https://i.stack.imgur.com/tw8pd.png)

We tried the solution from spring as well but it didn't work! The annotation @CrossOrigin in our controller didn't help either.

# Edit 1:

Tried the solution from @Piotr Sołtysiak. The cors filter isn't listed in the generated filter chain and we still get the same error.

> 2016-11-04 10:22:49.881 INFO 8820 --- [ost-startStop-1] o.s.s.web.DefaultSecurityFilterChain : Creating filter chain: org.springframework.security.web.util.matcher.AnyRequestMatcher@1, [org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter@4c191377, org.springframework.security.web.context.SecurityContextPersistenceFilter@28bad32a, org.springframework.security.web.header.HeaderWriterFilter@3c3ec668, org.springframework.security.web.csrf.CsrfFilter@288460dd, org.springframework.security.web.authentication.logout.LogoutFilter@1c9cd096, org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter@3990c331, org.springframework.security.web.authentication.ui.DefaultLoginPageGeneratingFilter@1e8d4ac1, org.springframework.security.web.authentication.www.BasicAuthenticationFilter@2d61d2a4, org.springframework.security.web.savedrequest.RequestCacheAwareFilter@380d9a9b, org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter@abf2de3, org.springframework.security.web.authentication.AnonymousAuthenticationFilter@2a5c161b, org.springframework.security.web.session.SessionManagementFilter@3c1fd3e5, org.springframework.security.web.access.ExceptionTranslationFilter@3d7055ef, org.springframework.security.web.access.intercept.FilterSecurityInterceptor@5d27725a]

Btw we are using spring-security version 4.1.3.!

-   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
-   [spring](https://stackoverflow.com/questions/tagged/spring "show questions tagged 'spring'")
-   [spring-security](https://stackoverflow.com/questions/tagged/spring-security)
-   [spring-boot](https://stackoverflow.com/questions/tagged/spring-boot "show questions tagged 'spring-boot'")
-   [cors](https://stackoverflow.com/questions/tagged/cors "show questions tagged 'cors'")

[Share](https://stackoverflow.com/q/40418441 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/40418441/edit)

Follow

[edited Jun 20, 2020 at 9:12](https://stackoverflow.com/posts/40418441/revisions "show all edits to this post")

[

![Community's user avatar](https://www.gravatar.com/avatar/a007be5a61f6aa8f3e85ae2fc18dd66e?s=64&d=identicon&r=PG)

](https://stackoverflow.com/users/-1/community)

[Community](https://stackoverflow.com/users/-1/community)Bot

111 silver badge

asked Nov 4, 2016 at 8:46

[

![Mace's user avatar](https://www.gravatar.com/avatar/20f463a38b6c76cf96ec5393d2399fa3?s=64&d=identicon&r=PG&f=1)

](https://stackoverflow.com/users/7113955/mace)

[Mace](https://stackoverflow.com/users/7113955/mace)

1,01911 gold badge1212 silver badges1414 bronze badges

-   1
    
    There is an issue with Chrome it does not support localhost to go through the Access-Control-Allow-Origin. Try with another browser
    
    – [Issam El-atif](https://stackoverflow.com/users/2998222/issam-el-atif "2,266 reputation")
    
    [Nov 4, 2016 at 9:35](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment68087372_40418441)
    
-   We tried with Edge and it is working... but firefox isn't working as well.
    
    – [Mace](https://stackoverflow.com/users/7113955/mace "1,019 reputation")
    
    [Nov 4, 2016 at 9:40](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment68087548_40418441)
    
-   I was having the same issue i solve it by adding `127.0.0.1 localhost local.net` to `/etc/hosts` then call [local.net:8080/getKunden](http://local.net:8080/getKunden)
    
    – [Issam El-atif](https://stackoverflow.com/users/2998222/issam-el-atif "2,266 reputation")
    
    [Nov 4, 2016 at 9:46](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment68087739_40418441)
    
-   see [stackoverflow.com/questions/28547288/…](http://stackoverflow.com/questions/28547288/no-access-control-allow-origin-header-is-present-on-the-requested-resource-err "no access control allow origin header is present on the requested resource err") it could help
    
    – [Issam El-atif](https://stackoverflow.com/users/2998222/issam-el-atif "2,266 reputation")
    
    [Nov 4, 2016 at 10:07](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment68088545_40418441)
    

[Add a comment](https://stackoverflow.com/questions/40418441/spring-security-cors-filter# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 17 Answers

Sorted by:

119

[](https://stackoverflow.com/posts/43559288/timeline)

Since Spring Security 4.1, this is the proper way to make Spring Security support CORS (also needed in Spring Boot 1.4/1.5):

```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedMethods("HEAD", "GET", "PUT", "POST", "DELETE", "PATCH");
    }
}
```

and:

```java
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
//        http.csrf().disable();
        http.cors();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        final CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(ImmutableList.of("*"));
        configuration.setAllowedMethods(ImmutableList.of("HEAD",
                "GET", "POST", "PUT", "DELETE", "PATCH"));
        // setAllowCredentials(true) is important, otherwise:
        // The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is 'include'.
        configuration.setAllowCredentials(true);
        // setAllowedHeaders is important! Without it, OPTIONS preflight request
        // will fail with 403 Invalid CORS request
        configuration.setAllowedHeaders(ImmutableList.of("Authorization", "Cache-Control", "Content-Type"));
        final UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

Do _not_ do any of below, which are the wrong way to attempt solving the problem:

-   `http.authorizeRequests().antMatchers(HttpMethod.OPTIONS, "/**").permitAll();`
-   `web.ignoring().antMatchers(HttpMethod.OPTIONS);`

Reference: [http://docs.spring.io/spring-security/site/docs/4.2.x/reference/html/cors.html](http://docs.spring.io/spring-security/site/docs/4.2.x/reference/html/cors.html)

[Share](https://stackoverflow.com/a/43559288 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/43559288/edit)

Follow

[edited Apr 22, 2017 at 13:02](https://stackoverflow.com/posts/43559288/revisions "show all edits to this post")

answered Apr 22, 2017 at 12:35

[

![Hendy Irawan's user avatar](https://www.gravatar.com/avatar/7ca11f2b2afc7a7392f5cc2713d8ecd4?s=64&d=identicon&r=PG)

](https://stackoverflow.com/users/122441/hendy-irawan)

[Hendy Irawan](https://stackoverflow.com/users/122441/hendy-irawan)

20.2k1010 gold badges102102 silver badges113113 bronze badges

-   5
    
    If you aren't using Guava, you can always do this: `Collections.unmodifiableList(Arrays.asList("HEAD",...))`
    
    – [Brian](https://stackoverflow.com/users/4614870/brian "4,902 reputation")
    
    [Dec 11, 2017 at 11:45](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment82464692_43559288)
    
-   8
    
    Please note that WebMvcConfigurerAdapter is deprecated now.
    
    – [priteshbaviskar](https://stackoverflow.com/users/807210/priteshbaviskar "2,070 reputation")
    
    [Aug 5, 2018 at 9:10](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment90347324_43559288)
    
-   5
    
    **Don't** remove the CSRF without knowing what you are doing! [owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF)](https://www.owasp.org/index.php/Cross-Site_Request_Forgery_(CSRF))
    
    – [Eran Medan](https://stackoverflow.com/users/239168/eran-medan "44,175 reputation")
    
    [Feb 19, 2019 at 3:26](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment96298102_43559288)
    

-   2
    
    Still doesn't work with this solution, cause preflight requests are blocked.
    
    – [Vortilion](https://stackoverflow.com/users/1263901/vortilion "396 reputation")
    
    [Mar 13, 2019 at 8:11](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment97014508_43559288)
    
-   6
    
    The type `WebMvcConfigurerAdapter` is deprecated. Since Spring 5 you just need to implement the interface `WebMvcConfigurer`
    
    – [Filomat](https://stackoverflow.com/users/5822240/filomat "723 reputation")
    
    [Jan 19, 2020 at 12:10](https://stackoverflow.com/questions/40418441/spring-security-cors-filter#comment105758787_43559288)
    

[Show **5** more comments](https://stackoverflow.com/questions/40418441/spring-security-cors-filter# "Expand to show all comments on this post")

30

[](https://stackoverflow.com/posts/40419969/timeline)

Ok, after over 2 days of searching we finally fixed the problem. We deleted all our filter and configurations and instead used this 5 lines of code in the application class.

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        final ApplicationContext ctx = SpringApplication.run(Application.class, args);
    }

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**").allowedOrigins("http://localhost:3000");
            }
        };
    }
}
```