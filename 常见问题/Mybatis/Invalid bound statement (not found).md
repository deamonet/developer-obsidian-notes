含义就是 mapper 接口 跟 xml文件没有匹配上。原因可能如下，逐一排查，尤其是做了某些更改之后，很容易出现这种错误：
1. mapper xml 中是否写对了 namespace
2. mybatis 配置中是否写对了 xml 文件的位置 (classpath)
3. mapper 接口中的方法与 xml 中的方法能否一一对应
4. xml 没有正确配置 resultmap resultetype
5. 检查编译输出文件中是否有对应xml文件
6. 检查xml文件开头meta信息是否正确


另外看到一个说法是：mapper 接口名字必须跟 xml 文件名一致。这应该是错误的。