Description:

An attempt was made to call a method that does not exist. The attempt was made from the following location:

    org.apache.catalina.authenticator.AuthenticatorBase.startInternal(AuthenticatorBase.java:1319)

The following method did not exist:

    javax.servlet.ServletContext.getVirtualServerName()Ljava/lang/String;

The calling method's class, org.apache.catalina.authenticator.AuthenticatorBase, was loaded from the following location:

    jar:file:/C:/Users/Auime/.m2/repository/org/apache/tomcat/embed/tomcat-embed-core/9.0.65/tomcat-embed-core-9.0.65.jar!/org/apache/catalina/authenticator/AuthenticatorBase.class

The called method's class, javax.servlet.ServletContext, is available from the following locations:

    jar:file:/C:/Users/Auime/.m2/repository/javax/servlet/servlet-api/2.5/servlet-api-2.5.jar!/javax/servlet/ServletContext.class
    jar:file:/C:/Users/Auime/.m2/repository/org/apache/tomcat/embed/tomcat-embed-core/9.0.65/tomcat-embed-core-9.0.65.jar!/javax/servlet/ServletContext.class

The called method's class hierarchy was loaded from the following locations:

    javax.servlet.ServletContext: file:/C:/Users/Auime/.m2/repository/javax/servlet/servlet-api/2.5/servlet-api-2.5.jar


看样子仍然是servlet-api这个依赖包的版本太老，导致找不到其中的方法，但是就算直接加入新版本的引用也不能排除掉原来的。

尝试：
1. 引入新版本的（大于3.1），删除上述提到的servlet-api.jar ，可以解决。但是再次刷新maven项目，此jar就会出现。导致项目失败。
2. 