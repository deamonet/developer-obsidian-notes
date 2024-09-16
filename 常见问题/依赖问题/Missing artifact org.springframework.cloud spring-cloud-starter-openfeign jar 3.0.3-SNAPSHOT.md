[](https://stackoverflow.com/posts/67689141/timeline)

_I am trying to add "spring-cloud-starter-openfeign" dependency to pom.xml_

```xml
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

_But i am getting below error._

```csharp
[INFO] Scanning for projects...
[INFO] 
[INFO] [1m---------------------------< [0;36mcom.cts:stock[0;1m >----------------------------[m
[INFO] [1mBuilding stock 0.0.1-SNAPSHOT[m
[INFO] [1m--------------------------------[ jar ]---------------------------------[m
[WARNING] The POM for org.springframework.cloud:spring-cloud-starter-openfeign:jar:3.0.3-SNAPSHOT is missing, no dependency information available
[INFO] [1m------------------------------------------------------------------------[m
[INFO] [1;31mBUILD FAILURE[m
[INFO] [1m------------------------------------------------------------------------[m
[INFO] Total time:  1.370 s
[INFO] Finished at: 2021-05-25T18:56:40+05:30
[INFO] [1m------------------------------------------------------------------------[m
[ERROR] Failed to execute goal on project [36mstock[m: [1;31mCould not resolve dependencies for project com.cts:stock:jar:0.0.1-SNAPSHOT: Could not find artifact org.springframework.cloud:spring-cloud-starter-openfeign:jar:3.0.3-SNAPSHOT[m -> [1m[Help 1][m
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the [1m-e[m switch.
[ERROR] Re-run Maven using the [1m-X[m switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [1m[Help 1][m http://cwiki.apache.org/confluence/display/MAVEN/DependencyResolutionException
```

_After some findings from google i have added version like below_

```xml
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-openfeign</artifactId>
        <version>3.0.3</version>
</dependency>
```

_But still the error is not resolved. Can some one help me with this ?_

_Below is my **pom.xml**_

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.cts</groupId>
    <artifactId>stock</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>stock</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>11</java.version>
        <spring-cloud.version>2020.0.3-SNAPSHOT</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>3.0.2</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <version>2.5.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>




    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```

I have added the pom file. I am trying to create a feign client on one of my services in micro services project. I can't able to resolve this pom because of **spring-cloud-starter-openfeign** dependency. I am recently learning microservices, so can some one help me to resolve this pom file

-   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
-   [spring](https://stackoverflow.com/questions/tagged/spring "show questions tagged 'spring'")
-   [spring-boot](https://stackoverflow.com/questions/tagged/spring-boot "show questions tagged 'spring-boot'")
-   [maven](https://stackoverflow.com/questions/tagged/maven "show questions tagged 'maven'")

[Share](https://stackoverflow.com/q/67689141 "Short permalink to this question")

[Improve this question](https://stackoverflow.com/posts/67689141/edit)

Follow

[edited May 25, 2021 at 14:03](https://stackoverflow.com/posts/67689141/revisions "show all edits to this post")

asked May 25, 2021 at 13:36

[

![jagan mohan Mylapuru's user avatar](media/jagan_mohan_Mylapuru's_user_avatar.png)

](https://stackoverflow.com/users/11973540/jagan-mohan-mylapuru)

[jagan mohan Mylapuru](https://stackoverflow.com/users/11973540/jagan-mohan-mylapuru)

13711 gold badge11 silver badge99 bronze badges

[Add a comment](https://stackoverflow.com/questions/67689141/missing-artifact-org-springframework-cloudspring-cloud-starter-openfeignjar3# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 2 Answers

Sorted by:

6

[](https://stackoverflow.com/posts/67690318/timeline)

_Hi I finally fixed this issue by adding **spring-cloud-openfeign-core** which is below_

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-openfeign-core</artifactId>
    <version>3.0.2</version>
</dependency>
```

**Resolution:**

1.  I found this solution as error came on 4th line of pom file saying _**"Missing artifact org.springframework.cloud:spring-cloud-openfeign-core:jar:3.0.3-SNAPSHOT"**_ after adding version for **spring-cloud-starter-openfeign** dependency.
    
2.  Means Sping is searching for **openfeign-core** dependency after adding **spring-cloud-starter-openfeign**
    

So below is updated **pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.5.0</version>
        <relativePath /> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.cts</groupId>
    <artifactId>stock</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>stock</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>11</java.version>
        <spring-cloud.version>2020.0.3-SNAPSHOT</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-eureka-client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>3.0.2</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-devtools -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <version>2.5.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
            <version>3.0.2</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-openfeign-core -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-openfeign-core</artifactId>
            <version>3.0.2</version>
        </dependency>





    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>


```

[Share](https://stackoverflow.com/a/67690318 "Short permalink to this answer")

[Improve this answer](https://stackoverflow.com/posts/67690318/edit)

Follow