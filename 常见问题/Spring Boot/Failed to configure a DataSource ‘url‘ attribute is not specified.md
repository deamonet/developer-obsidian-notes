具体报错信息有：
1. Failed to configure a DataSource ‘url‘ attribute is not specified
2. Reason: Failed to determine a suitable driver class

如果没有配置，当然会报这种错误；配置的话也要注意URL中参数是否正确。比如SSL一项，如果没有加密不可设置成true。如果使用的H2数据库，也要注意。

如果不想配置，可以在application中或者springbootapplication注解中加入autoconfiguration 排除，具体看[[Disable Spring Data Auto Configuration]]

可以尝试的解决办法
1. 换一个合适的JDBC驱动，可能缺一个依赖spring-boot-starter-jdbc，或者jdbc connector的依赖。
2. 执行 MAVEN CLEAN （可加上 -U 参数）
3. 执行 MAVEN INSTALL
4. 检查target是否存在resources下的文件，如果没有，说明maven项目在编译的过程中，没有包括这部分文件，就是pom.xml中存在错误，查看build packaging resouces等标签。

packaging 标签中如果是 pom，说明这个项目只是单纯的提供依赖管理的作用，打包结果只有这个pom文件

build 标签详细描述的编译的过程

resources 指明打包过程中需要或者不需要的文件，例如：
```xml
<resources>
	<resource>
		<directory>src/main/resources</directory>
		<includes>
			<include>**/*.xml</include>
			<include>**/*.properties</include>
		</includes>
		<filtering>true</filtering>
	</resource>
</resources>
```

以上配置直接过滤掉了src/main/resources下包括子目录和子目录的子目录等所有的xml和properties文件。