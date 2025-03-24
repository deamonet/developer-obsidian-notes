springmvc中使用thymeleaf乱码完整方案

    在springmvc项目中，使用thymeleaf视图，出现中文乱码，尤其是message.properties文件中文，多次调试并总结如下。本文统一采用java config方式说明，如果你仍然是xml配置，相应修改xml配置即可。

thymeleaf模板文件

    <!DOCTYPE html>

    <html xmlns:th="http://www.thymeleaf.org">

    <head>

        <meta charset="UTF-8">

        <title th:text="#{app.title}">Home</title>

    </head>

上面<meta charset="UTF-8">行，声明最终文件的编码。

配置请求编码

在springmvc的启动配置中增加请求配置。

    // 设置请求编码

    @Override

    protected Filter[] getServletFilters() {

        final CharacterEncodingFilter encodingFilter = new CharacterEncodingFilter();

        encodingFilter.setEncoding(CHARACTER_ENCODING);

        encodingFilter.setForceEncoding(true);

        return new Filter[] { encodingFilter };

}

一般java config配置方式继承AbstractAnnotationConfigDispatcherServletInitializer类，实现三个默认方法。这里增加getServletFilters方法配置请求编码。

视图配置

配置thymeleaf时，有两个地方涉及编码，配置代码如下：

    配置SpringResourceTemplateResolver

    @Bean

    public SpringResourceTemplateResolver templateResolver(){

        SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();

        templateResolver.setPrefix("classpath:/templates/");

        templateResolver.setSuffix(".html");

        templateResolver.setCharacterEncoding("UTF-8");

        templateResolver.setTemplateMode(TemplateMode.HTML);

        templateResolver.setCacheable(true);

        return templateResolver;

    }

上面代码中templateResolver.setCharacterEncoding("utf8");行，配置解析编码。

    配置 ThymeleafViewResolver

    @Bean

    public ThymeleafViewResolver viewResolver(){

        ThymeleafViewResolver viewResolver = new ThymeleafViewResolver();

        viewResolver.setTemplateEngine(templateEngine());

        viewResolver.setCharacterEncoding("UTF-8");

        viewResolver.setOrder(1);

        return viewResolver;

    }

上面代码中viewResolver.setCharacterEncoding("UTF-8");行，配置ThymeleafViewResolver的解析编码。

配置国际化文件编码

首先确保在ide中的文件编码为UTF-8方式。然后在配置ResourceBundleMessageSource时需指定编码：

    @Bean

    public ResourceBundleMessageSource messageSource() {

        ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();

        messageSource.setBasename("Messages");

        messageSource.setDefaultEncoding("UTF-8");

        return messageSource;

    }

上述代码中messageSource.setDefaultEncoding("UTF-8");设定解析编码。

总结

上述情况完整说明了thymeleaf视图项目的编码问题，如果还不能解决你的问题，可能需确定数据库相关编码问题，在此不再说明。