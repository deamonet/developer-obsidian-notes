[java.lang.ClassNotFoundException com.fasterxml.jackson.annotation.JsonInclude$Value](https://stackoverflow.com/posts/46406070/timeline)

Jackson marshalling/unmarshalling requires following jar files of same version.

1.  jackson-core
    
2.  jackson-databind
    
3.  jackson-annotations
    
    Make sure that you have added all these with same version in your classpath. In your case **jackson-annotations** is missing in classpath.


总的来说，就是要保证jackson所有相关的库，都得是同一个版本的。不然很容易出错。