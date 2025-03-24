# [What is "pom" packaging in maven?](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven)

Asked 11 years, 5 months ago

Modified [2 months ago](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven?lastactivity "2023-01-27 01:32:12Z")

Viewed 173k times

207

[](https://stackoverflow.com/posts/7692161/timeline)

I was given a maven project to compile and get deployed on a tomcat server. I have never used maven before today, but I have been googling quite a bit. It seems like the top level `pom.xml` files in this project have the packaging type set as `pom`.

What am I supposed to do after `mvn install` to get this application deployed? I was expecting to be able to find a `war` file somewhere or something, but I guess I am looking in the wrong place or missing a step.

-   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
-   [maven](https://stackoverflow.com/questions/tagged/maven "show questions tagged 'maven'")

[Share](https://stackoverflow.com/q/7692161 "Short permalink to this question")

Follow

[edited Jan 27 at 1:32](https://stackoverflow.com/posts/7692161/revisions "show all edits to this post")

[

![Ben Creasy's user avatar](media/Ben_Creasy's_user_avatar.jpg)

](https://stackoverflow.com/users/4200039/ben-creasy)

[Ben Creasy](https://stackoverflow.com/users/4200039/ben-creasy)

3,72344 gold badges4141 silver badges5050 bronze badges

asked Oct 7, 2011 at 19:45

[

![Derek's user avatar](media/Derek's_user_avatar.png)

](https://stackoverflow.com/users/229072/derek)

[Derek](https://stackoverflow.com/users/229072/derek)

11.7k3131 gold badges124124 silver badges227227 bronze badges

-   3
    
    mvn install - this is used for install your artifact (jar, war, ear) in your local repository (usually it'll be ~/.m2/repository direcotry)
    
    – [Łukasz Siwiński](https://stackoverflow.com/users/235973/%c5%81ukasz-siwi%c5%84ski "330 reputation")
    
    [Oct 23, 2013 at 11:06](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven#comment28990895_7692161)
    

[Add a comment](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven# "Use comments to ask for more information or suggest improvements. Avoid answering questions in comments.")

## 10 Answers

Sorted by:

169

[](https://stackoverflow.com/posts/7692214/timeline)

`pom` is basically a container of submodules, each submodule is represented by a subdirectory in the same directory as `pom.xml` with `pom` packaging.

Somewhere, nested within the project structure you will find artifacts (modules) with `war` packaging. Maven generally builds everything into `/target` subdirectories of each module. So after `mvn install` look into `target` subdirectory in a module with `war` packaging.

Of course:

```lua
$ find . -iname "*.war"
```

works equally well ;-).

[Share](https://stackoverflow.com/a/7692214 "Short permalink to this answer")

Follow

answered Oct 7, 2011 at 19:51

[

![Tomasz Nurkiewicz's user avatar](media/Tomasz_Nurkiewicz's_user_avatar.jpg)

](https://stackoverflow.com/users/605744/tomasz-nurkiewicz)

[Tomasz Nurkiewicz](https://stackoverflow.com/users/605744/tomasz-nurkiewicz)

332k6868 gold badges697697 silver badges671671 bronze badges

-   Just to add a quick comment, there are other packaging types that can exist other than "war". A very common packaging type that could be use in place of "war" is "jar". If you replace every instance of "war" in this answer with "jar", you will get the gist. Read more about packaging types here: [baeldung.com/maven-packaging-types](https://www.baeldung.com/maven-packaging-types)
    
    – [Matt C.](https://stackoverflow.com/users/4052745/matt-c "1,850 reputation")
    
    [Jul 26, 2021 at 14:02](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven#comment121114301_7692214)
    

[Add a comment](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

56

[](https://stackoverflow.com/posts/11525305/timeline)

pom packaging is simply a specification that states the primary artifact is not a war or jar, but the pom.xml itself.

Often it is used in conjunction with "modules" which are typically contained in sub-directories of the project in question; however, it may also be used in certain scenarios where no primary binary was meant to be built, all the other important artifacts have been declared as secondary artifacts

Think of a "documentation" project, the primary artifact might be a PDF, but it's already built, and the work to declare it as a secondary artifact might be desired over the configuration to tell maven how to build a PDF that doesn't need compiled.

[Share](https://stackoverflow.com/a/11525305 "Short permalink to this answer")

Follow

answered Jul 17, 2012 at 14:57

[

![Edwin Buck's user avatar](media/Edwin_Buck's_user_avatar.png)

](https://stackoverflow.com/users/302139/edwin-buck)

[Edwin Buck](https://stackoverflow.com/users/302139/edwin-buck)

68.7k77 gold badges100100 silver badges134134 bronze badges

-   Other example which I can think of in similar lines as pom project ==> abstract class and module(s) inside it ==> concrete class(s).
    
    – [bharatj](https://stackoverflow.com/users/1504824/bharatj "2,237 reputation")
    
    [Jul 13, 2018 at 17:46](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven#comment89636481_11525305)
    
-   POM stands for Project Object Model. It is fundamental unit of work in Maven. It is an XML file that resides in the base directory of the project as pom.
    
    – [Jerry An](https://stackoverflow.com/users/10153574/jerry-an "987 reputation")
    
    [Feb 27, 2021 at 9:24](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven#comment117383106_11525305)
    

[Add a comment](https://stackoverflow.com/questions/7692161/what-is-pom-packaging-in-maven# "Use comments to ask for more information or suggest improvements. Avoid comments like “+1” or “thanks”.")

27

[](https://stackoverflow.com/posts/7692220/timeline)

Packaging of `pom` is used in projects that aggregate other projects, and in projects whose only useful output is an attached artifact from some plugin. In your case, I'd **guess** that your top-level pom includes `<modules>...</modules>` to aggregate other directories, and the actual output is the result of one of the other (probably sub-) directories. It will, if coded sensibly for this purpose, have a packaging of `war`.