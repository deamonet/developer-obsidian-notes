org.apache.ibatis.reflection.ReflectionException: There is no getter for property named 'phone_number' in 'class me.deamonet.nar.entity.User'
	at org.apache.ibatis.reflection.Reflector.getGetInvoker(Reflector.java:374) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.reflection.MetaClass.getGetInvoker(MetaClass.java:164) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.reflection.wrapper.BeanWrapper.getBeanProperty(BeanWrapper.java:162) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.reflection.wrapper.BeanWrapper.get(BeanWrapper.java:49) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.reflection.MetaObject.getValue(MetaObject.java:122) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.DynamicContext$ContextMap.get(DynamicContext.java:102) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.DynamicContext$ContextAccessor.getProperty(DynamicContext.java:113) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.OgnlRuntime.getProperty(OgnlRuntime.java:3356) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.ASTProperty.getValueBody(ASTProperty.java:121) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.SimpleNode.evaluateGetValueBody(SimpleNode.java:212) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.SimpleNode.getValue(SimpleNode.java:258) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.ASTNotEq.getValueBody(ASTNotEq.java:50) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.SimpleNode.evaluateGetValueBody(SimpleNode.java:212) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.SimpleNode.getValue(SimpleNode.java:258) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.Ognl.getValue(Ognl.java:586) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.ognl.Ognl.getValue(Ognl.java:550) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.OgnlCache.getValue(OgnlCache.java:46) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.ExpressionEvaluator.evaluateBoolean(ExpressionEvaluator.java:32) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.IfSqlNode.apply(IfSqlNode.java:34) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.MixedSqlNode.lambda$apply$0(MixedSqlNode.java:32) ~[mybatis-3.5.9.jar:3.5.9]
	at java.util.ArrayList.forEach(ArrayList.java:1249) ~[na:1.8.0_131]
	at org.apache.ibatis.scripting.xmltags.MixedSqlNode.apply(MixedSqlNode.java:32) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.TrimSqlNode.apply(TrimSqlNode.java:55) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.MixedSqlNode.lambda$apply$0(MixedSqlNode.java:32) ~[mybatis-3.5.9.jar:3.5.9]
	at java.util.ArrayList.forEach(ArrayList.java:1249) ~[na:1.8.0_131]
	at org.apache.ibatis.scripting.xmltags.MixedSqlNode.apply(MixedSqlNode.java:32) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.scripting.xmltags.DynamicSqlSource.getBoundSql(DynamicSqlSource.java:39) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.mapping.MappedStatement.getBoundSql(MappedStatement.java:305) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.executor.statement.BaseStatementHandler.<init>(BaseStatementHandler.java:64) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.executor.statement.PreparedStatementHandler.<init>(PreparedStatementHandler.java:41) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.executor.statement.RoutingStatementHandler.<init>(RoutingStatementHandler.java:46) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.session.Configuration.newStatementHandler(Configuration.java:681) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.executor.SimpleExecutor.doUpdate(SimpleExecutor.java:48) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.executor.BaseExecutor.update(BaseExecutor.java:117) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.executor.CachingExecutor.update(CachingExecutor.java:76) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.session.defaults.DefaultSqlSession.update(DefaultSqlSession.java:194) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.session.defaults.DefaultSqlSession.insert(DefaultSqlSession.java:181) ~[mybatis-3.5.9.jar:3.5.9]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_131]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_131]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_131]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_131]
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:427) ~[mybatis-spring-2.0.7.jar:2.0.7]
	at com.sun.proxy.$Proxy86.insert(Unknown Source) ~[na:na]
	at org.mybatis.spring.SqlSessionTemplate.insert(SqlSessionTemplate.java:272) ~[mybatis-spring-2.0.7.jar:2.0.7]
	at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:62) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.binding.MapperProxy$PlainMethodInvoker.invoke(MapperProxy.java:145) ~[mybatis-3.5.9.jar:3.5.9]
	at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:86) ~[mybatis-3.5.9.jar:3.5.9]
	at com.sun.proxy.$Proxy87.insert(Unknown Source) ~[na:na]
	at me.deamonet.nar.service.UserService.register(UserService.java:34) ~[classes/:na]
	at me.deamonet.nar.service.UserService$$FastClassBySpringCGLIB$$264da4a5.invoke(<generated>) ~[classes/:na]
	at org.springframework.cglib.proxy.MethodProxy.invoke(MethodProxy.java:218) ~[spring-core-5.3.20.jar:5.3.20]
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.invokeJoinpoint(CglibAopProxy.java:793) ~[spring-aop-5.3.20.jar:5.3.20]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:163) ~[spring-aop-5.3.20.jar:5.3.20]
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.20.jar:5.3.20]
	at org.springframework.transaction.interceptor.TransactionInterceptor$1.proceedWithInvocation(TransactionInterceptor.java:123) ~[spring-tx-5.3.20.jar:5.3.20]
	at org.springframework.transaction.interceptor.TransactionAspectSupport.invokeWithinTransaction(TransactionAspectSupport.java:388) ~[spring-tx-5.3.20.jar:5.3.20]
	at org.springframework.transaction.interceptor.TransactionInterceptor.invoke(TransactionInterceptor.java:119) ~[spring-tx-5.3.20.jar:5.3.20]
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186) ~[spring-aop-5.3.20.jar:5.3.20]
	at org.springframework.aop.framework.CglibAopProxy$CglibMethodInvocation.proceed(CglibAopProxy.java:763) ~[spring-aop-5.3.20.jar:5.3.20]
	at org.springframework.aop.framework.CglibAopProxy$DynamicAdvisedInterceptor.intercept(CglibAopProxy.java:708) ~[spring-aop-5.3.20.jar:5.3.20]
	at me.deamonet.nar.service.UserService$$EnhancerBySpringCGLIB$$f7232efa.register(<generated>) ~[classes/:na]
	at me.deamonet.nar.controller.RegisterController.registerUser(RegisterController.java:28) ~[classes/:na]
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.8.0_131]
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:1.8.0_131]
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.8.0_131]
	at java.lang.reflect.Method.invoke(Method.java:498) ~[na:1.8.0_131]
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:205) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:150) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:117) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:895) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1067) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:963) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:909) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:681) ~[tomcat-embed-core-9.0.63.jar:4.0.FR]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883) ~[spring-webmvc-5.3.20.jar:5.3.20]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:764) ~[tomcat-embed-core-9.0.63.jar:4.0.FR]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:227) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:53) ~[tomcat-embed-websocket-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:327) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.invoke(FilterSecurityInterceptor.java:115) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.access.intercept.FilterSecurityInterceptor.doFilter(FilterSecurityInterceptor.java:81) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:122) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.access.ExceptionTranslationFilter.doFilter(ExceptionTranslationFilter.java:116) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:126) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.session.SessionManagementFilter.doFilter(SessionManagementFilter.java:81) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.authentication.AnonymousAuthenticationFilter.doFilter(AnonymousAuthenticationFilter.java:109) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.servletapi.SecurityContextHolderAwareRequestFilter.doFilter(SecurityContextHolderAwareRequestFilter.java:149) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.savedrequest.RequestCacheAwareFilter.doFilter(RequestCacheAwareFilter.java:63) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.session.ConcurrentSessionFilter.doFilter(ConcurrentSessionFilter.java:147) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.session.ConcurrentSessionFilter.doFilter(ConcurrentSessionFilter.java:125) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:223) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.authentication.AbstractAuthenticationProcessingFilter.doFilter(AbstractAuthenticationProcessingFilter.java:217) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:103) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.authentication.logout.LogoutFilter.doFilter(LogoutFilter.java:89) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.web.filter.CorsFilter.doFilterInternal(CorsFilter.java:91) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.header.HeaderWriterFilter.doHeadersAfter(HeaderWriterFilter.java:90) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.header.HeaderWriterFilter.doFilterInternal(HeaderWriterFilter.java:75) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersistenceFilter.java:112) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.context.SecurityContextPersistenceFilter.doFilter(SecurityContextPersistenceFilter.java:82) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter.doFilterInternal(WebAsyncManagerIntegrationFilter.java:55) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.session.DisableEncodeUrlFilter.doFilterInternal(DisableEncodeUrlFilter.java:42) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.security.web.FilterChainProxy$VirtualFilterChain.doFilter(FilterChainProxy.java:336) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy.doFilterInternal(FilterChainProxy.java:211) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.security.web.FilterChainProxy.doFilter(FilterChainProxy.java:183) ~[spring-security-web-5.7.1.jar:5.7.1]
	at org.springframework.web.filter.DelegatingFilterProxy.invokeDelegate(DelegatingFilterProxy.java:354) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.web.filter.DelegatingFilterProxy.doFilter(DelegatingFilterProxy.java:267) ~[spring-web-5.3.20.jar:5.3.20]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:100) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) ~[spring-web-5.3.20.jar:5.3.20]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.springframework.web.filter.FormContentFilter.doFilterInternal(FormContentFilter.java:93) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) ~[spring-web-5.3.20.jar:5.3.20]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:201) ~[spring-web-5.3.20.jar:5.3.20]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:117) ~[spring-web-5.3.20.jar:5.3.20]
	at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:189) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:162) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:197) ~[tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:97) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:541) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:135) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:92) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:78) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:360) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.coyote.http11.Http11Processor.service(Http11Processor.java:399) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.coyote.AbstractProcessorLight.process(AbstractProcessorLight.java:65) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.coyote.AbstractProtocol$ConnectionHandler.process(AbstractProtocol.java:890) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.tomcat.util.net.NioEndpoint$SocketProcessor.doRun(NioEndpoint.java:1743) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.tomcat.util.net.SocketProcessorBase.run(SocketProcessorBase.java:49) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.tomcat.util.threads.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1191) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.tomcat.util.threads.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:659) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61) [tomcat-embed-core-9.0.63.jar:9.0.63]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_131]


看样子是因为 mybatis 里 if 中的 test 写了数据库的字段，而不是类的属性名。（下划线和驼峰）