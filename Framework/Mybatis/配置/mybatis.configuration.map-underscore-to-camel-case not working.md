mybatis.configuration.map-underscore-to-camel-case 这个属性是用来设置是否将数据库中的下划线风格的字段映射为驼峰式的 Java 对象属性的。如果这个属性没有生效，可能是因为你在使用 MyBatis 时使用了某些配置，导致了这个属性的设置被覆盖。

具体来说，有以下几种情况可能会导致这个属性失效：

-   在 MyBatis 的配置文件中，对 resultMap 元素使用了 resultMap 属性或 resultType 属性，并在 result 元素中使用了 column 属性或 property 属性
-   在代码中使用了 ResultHandler 接口处理查询结果

如果你的代码中存在以上情况，建议你在相应的配置或代码中使用驼峰式的属性名，或者手动转换数据库中的下划线风格的字段名。

https://juejin.cn/s/mybatis.configuration.map-underscore-to-camel-case%20not%20working