有些文章说是在application.yml配置文件中没有指明hibernate.dialect，也就是没说是什么数据库，另外一些回答就说是关于代码的jpa的配置。

不过这个解决办法对我无效，因为我根本就没用jpa，只是ngbatis这个依赖引用了spring-boot-starter-data-jpa

我是把mariadb 10.3.38-MariaDB-0ubuntu0.20.04.1 换成 mysql 5.7 版本之后就解决了。原来大家都用低版本mysql 是真的很有道理的啊

从网上的文章来看，mysql 8 也有这种问题。