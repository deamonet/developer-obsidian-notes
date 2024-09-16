Asked 3 years, 11 months ago

Modified [3 years, 3 months ago](https://stackoverflow.com/questions/53733312/where-is-jre-11?lastactivity "2019-08-10 08:00:11Z")

Viewed 173k times

76

[](https://stackoverflow.com/posts/53733312/timeline)

**This question already has answers here**:

[How can I get Java 11 run-time environment working since there is no more JRE 11 for download?](https://stackoverflow.com/questions/53111921/how-can-i-get-java-11-run-time-environment-working-since-there-is-no-more-jre-11) (4 answers)

Closed 3 years ago.

# UPDATE:

_(to be **more clear**)_

You can find [JRE 8](https://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html), [JRE 9](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase9-3934878.html) and [JRE 10](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase10-4425482.html) on Oracle's official website (click on each). _But where is **JRE 11**?!_

Also, JDK 11 doesn't include a JRE. I was expecting JRE to be installed with JDK.

_**Do final users of our apps need to install JDK?**_

---

# ORIGINAL version of the question:

I downloaded and installed Oracle JDK 11 from its [official site](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html). I installed both `..._linux-x64_bin.rpm` and `..._windows-x64_bin.exe` (first on a Linux machine and second on a Windows machine). But I saw an unexpected thing! Where is JRE?

This is a snapshot of installation path on CentOS 7. As you can see there is no `jre` folder:

```java
# ls /usr/java/jdk-11.0.1/
bin  conf  include  jmods  legal  lib  README.html  release
```

Same snapshot about Oracle JDK 8 (See `jre` folder specially):

```java
# ls /usr/java/jdk1.8.0_191-amd64/
bin             lib          src.zip
COPYRIGHT       LICENSE      THIRDPARTYLICENSEREADME-JAVAFX.txt
include         man          THIRDPARTYLICENSEREADME.txt
javafx-src.zip  README.html
jre             release
```

---

Same snapshots on Windows machine:

```java
> dir /b "C:\Program Files\Java\jdk-11.0.1" 
bin                                           
conf                                          
COPYRIGHT                                     
include                                       
jmods                                         
legal                                         
lib                                           
README.html                                   
release                                                                                     
           
> dir /b "C:\Program Files\Java\jdk1.8.0_181"  
bin                                           
COPYRIGHT                                     
include                                       
javafx-src.zip                                
jre                                           
lib                                           
LICENSE                                       
README.html                                   
release                                       
src.zip                                       
THIRDPARTYLICENSEREADME-JAVAFX.txt            
THIRDPARTYLICENSEREADME.txt 
```

---

On Windows machine, there are also two another differences between JDK 8 and JDK 11.

1.  A standalone `JRE` alongside `JDK` as you can see:
    
    ```java
    > dir /b "C:\Program Files\Java"            
    jdk-11.0.1   
    jdk1.8.0_181 
    jre1.8.0_181 
    ```
    
2.  In path `C:\Program Files (x86)\Common Files\Oracle\Java`:
    
    ```java
    > dir "C:\Program Files (x86)\Common Files\Oracle\Java"                                                                   
    ...                                                                                                                   
    ...                14 java.settings.cfg                                                                  
    ...    <JUNCTION>     javapath [C:\Program Files (x86)\Common Files\Oracle\Java\javapath_target_3015921] 
    ...    <DIR>          javapath_target_3015921 
    ...
    ```
    
    As you see `javapath` (that is in `PATH` environment variable) points to `javapath_target_3015921`. This folder contains 3 executables of JDK 8 (that aren't _link_s!):
    
    ```java
    > dir /b "C:\Program Files (x86)\Common Files\Oracle\Java\javapath" 
    java.exe                         
    javaw.exe                        
    javaws.exe 
    ```
    

---

Finally, I searched the web to find a standalone JRE and found out it doesn't exist!

_**Do final users of our programs need to install JDK?**_

-   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
-   [java-11](https://stackoverflow.com/questions/tagged/java-11 "show questions tagged 'java-11'")

[Share](https://stackoverflow.com/q/53733312 "Short permalink to this question")

Follow

[edited Jun 20, 2020 at 9:12](https://stackoverflow.com/posts/53733312/revisions "show all edits to this post")

[

![Community's user avatar](media/Community's_user_avatar.png)

](https://stackoverflow.com/users/-1/community)

[Community](https://stackoverflow.com/users/-1/community)Bot

111 silver badge

asked Dec 11, 2018 at 22:28

[

![Mir-Ismaili's user avatar](media/Mir-Ismaili's_user_avatar.jpg)

](https://stackoverflow.com/users/5318303/mir-ismaili)

[Mir-Ismaili](https://stackoverflow.com/users/5318303/mir-ismaili)

12.1k66 gold badges7474 silver badges9696 bronze badges

-   13
    
    You might be further surprised to know that the Oracle JDK isn't free anymore except for development/testing purposes. Better switch to OpenJDK. 
    
    – [rustyx](https://stackoverflow.com/users/485343/rustyx "76,046 reputation")
    
     [Dec 11, 2018 at 22:37](https://stackoverflow.com/questions/53733312/where-is-jre-11#comment94320625_53733312) 
    
-   @rustyx - Thanks for reminding this important note! 
    
    – [Mir-Ismaili](https://stackoverflow.com/users/5318303/mir-ismaili "12,123 reputation")
    
     [Dec 11, 2018 at 22:41](https://stackoverflow.com/questions/53733312/where-is-jre-11#comment94320690_53733312)
    
-   2
    
    On the upside though OpenJDK does about everything OracleJDK does. If you can come up with things that you can't do with OpenJDK. I'd be interested. JavaFX was one of the things you needed OracleJDK before. Now it's completely seperate from any JDK/JRE and you bundle it with your program with a built tool like maven or gradle. 
    
    – [Max](https://stackoverflow.com/users/9697266/max "1,248 reputation")
    
     [Dec 11, 2018 at 22:46](https://stackoverflow.com/questions/53733312/where-is-jre-11#comment94320763_53733312)
    
-   1
    
    Taking up @rustyx's helpful remark. The link to OpenJDK's JRE is [here](https://adoptopenjdk.net/releases.html) 
    
    – [questionto42standswithUkraine](https://stackoverflow.com/users/11154841/questionto42standswithukraine "5,218 reputation")
    
     [Nov 16, 2020 at 15:51](https://stackoverflow.com/questions/53733312/where-is-jre-11#comment114674961_53733312) 
    
-   1
    
    Till which version Oracle JDK was allowed for commercial use? I thought from starting it is only allowed for development & teasting 
    
    – [Satish Patro](https://stackoverflow.com/users/8609847/satish-patro "3,174 reputation")
    
     [Apr 28, 2021 at 6:09](https://stackoverflow.com/questions/53733312/where-is-jre-11#comment118947397_53733312)
    

[Show **2** more comments](https://stackoverflow.com/questions/53733312/where-is-jre-11# "Expand to show all comments on this post")

## 1 Answer

Sorted by:

                                              Highest score (default)                                                                   Trending (recent votes count more)                                                                   Date modified (newest first)                                                                   Date created (oldest first)                              

75

[](https://stackoverflow.com/posts/53733414/timeline)

The whole structure with Java 11 has changed. Java is now a modular platform, where you can create your own "JRE" distribution with specifically the modules that you need to run your application.

The release notes at [https://www.oracle.com/technetwork/java/javase/11-relnote-issues-5012449.html](https://www.oracle.com/technetwork/java/javase/11-relnote-issues-5012449.html) have the following sentence:

> In this release, the JRE or Server JRE is no longer offered. Only the JDK is offered. Users can use jlink to create smaller custom runtimes.

Documentation about jlink: [https://docs.oracle.com/en/java/javase/11/tools/jlink.html](https://docs.oracle.com/en/java/javase/11/tools/jlink.html)

And another article about it: [https://medium.com/codefx-weekly/is-jlink-the-future-1d8cb45f6306](https://medium.com/codefx-weekly/is-jlink-the-future-1d8cb45f6306)