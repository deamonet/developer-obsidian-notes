13

[](https://stackoverflow.com/posts/65696133/timeline)

**What does properties tag mean ?**

From Official Maven Docs :

> Properties are the last required piece to understand POM basics. Maven properties are value placeholders, like properties in Ant. Their values are accessible anywhere within a POM by using the notation ${X}, where X is the property. Or they can be used by plugins as default values, for example:

In your case you have defined properties as version of java.

Now this property(`java.version`) can be reused later in maven pom file.

From Official Maven Docs :

> They come in five different styles:
> 
> -   env.X: Prefixing a variable with "env." will return the shell's environment variable. For example, ${env.PATH} contains the PATH environment variable. Note: While environment variables themselves are case-insensitive on Windows, lookup of properties is case-sensitive. In other words, while the Windows shell returns the same value for %PATH% and %Path%, Maven distinguishes between ${env.PATH} and ${env.Path}. The names of environment variables are normalized to all upper-case for the sake of reliability.
>     
> -   project.x: A dot (.) notated path in the POM will contain the corresponding element's value. For example: 1.0 is accessible via ${project.version}.
>     
> -   settings.x: A dot (.) notated path in the settings.xml will contain the corresponding element's value. For example: false is accessible via ${settings.offline}.
>     
> -   Java System Properties: All properties accessible via java.lang.System.getProperties() a-re available as POM properties, such as ${java.home}.
>     
> -   x: Set within a element in the POM. The value of value may be used as ${someVar}.
>     

**What can i add also in properties tag ?**

You can add all the variables which you need to reuse later in your maven pom file.

For e.g. Below POM snippet reuses jackson.version 4 times.

```xml
<properties>
    <jackson.version>2.10.2</jackson.version>
    <dropwizard.version>2.0.1</dropwizard.version>
    <websocket.version>1.4.0</websocket.version>
    <apachehttp.version>4.5.10</apachehttp.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
        <version>${apachehttp.version}</version>
    </dependency>
    <dependency>
        <groupId>org.java-websocket</groupId>
        <artifactId>Java-WebSocket</artifactId>
        <version>${websocket.version}</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-core</artifactId>
        <version>${jackson.version}</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-annotations</artifactId>
        <version>${jackson.version}</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>${jackson.version}</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.dataformat</groupId>
        <artifactId>jackson-dataformat-yaml</artifactId>
        <version>${jackson.version}</version>
    </dependency>
 <dependencies>
```

References :

[Maven Pom Properties](https://maven.apache.org/pom.html#Properties)