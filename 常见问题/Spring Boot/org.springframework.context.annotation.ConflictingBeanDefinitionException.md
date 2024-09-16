Caused by: org.springframework.context.annotation.ConflictingBeanDefinitionException: Annotation-specified bean name '/user' for bean class [me.deamonet.nar.controller.UserController] conflicts with existing, non-compatible bean definition of same name and class [me.deamonet.nar.controller.NotebookController]

就是简简单单的Bean定义冲突，也就是定义了同名的两个Bean。检查一番即可。

而在我这个例子中，同名的Bean是两个Controller ，UserController 和 NotebookController。而这两个同名的原因竟然是 RestController 注解中还可以加上字符串，用来表示 Controller 的名字，而我把这俩的注解内容写成一样的。

而我这样做的原因是，我以为这是URL前缀的共同部分，也就是当成了RequestMapping。
