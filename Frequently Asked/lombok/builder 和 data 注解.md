# 当@Data和@Builder 共同使用导致无参构造丢失

1.单独使用@Data注解，会生成无参数构造方法。

2.单独使用@Builder注解，生成全属性的构造方法，无无参构造方法。

3.@Data和@Builder一起用：没有了默认的构造方法。手动添加无参数构造方法或者用@NoArgsConstructor注解会报错

