# [Jackson Vs. Gson [closed]](https://stackoverflow.com/questions/2378402/jackson-vs-gson)

[Ask Question](https://stackoverflow.com/questions/ask)

Asked 12 years, 9 months ago

Modified [7 years, 11 months ago](https://stackoverflow.com/questions/2378402/jackson-vs-gson?lastactivity "2014-12-30 17:58:41Z")

Viewed 196k times

395

[](https://stackoverflow.com/posts/2378402/timeline)

**Closed**. This question needs to be more [focused](https://stackoverflow.com/help/closed-questions). It is not currently accepting answers.

---

**Want to improve this question?** Update the question so it focuses on one problem only by [editing this post](https://stackoverflow.com/posts/2378402/edit).

Closed 8 years ago.

The community reviewed whether to reopen this question last month and left it closed:

> **Opinion-based** Update the question so it can be answered with facts and citations by [editing this post](https://stackoverflow.com/posts/2378402/edit).

[Improve this question](https://stackoverflow.com/posts/2378402/edit)

After searching through some existing libraries for JSON, I have finally ended up with these two:

-   Jackson
-   Google GSon

I am a bit partial towards GSON, but word on the net is that GSon suffers from a certain celestial performance [issue](http://www.cowtowncoder.com/blog/archives/2009/09/entry_326.html) (as of Sept 2009).

I am continuing my comparison; in the meantime, I'm looking for help to make up my mind.

-   [java](https://stackoverflow.com/questions/tagged/java "show questions tagged 'java'")
-   [json](https://stackoverflow.com/questions/tagged/json "show questions tagged 'json'")
-   [comparison](https://stackoverflow.com/questions/tagged/comparison "show questions tagged 'comparison'")
-   [gson](https://stackoverflow.com/questions/tagged/gson "show questions tagged 'gson'")
-   [jackson](https://stackoverflow.com/questions/tagged/jackson "show questions tagged 'jackson'")

[Share](https://stackoverflow.com/q/2378402 "Short permalink to this question")

Follow

[edited Aug 22, 2013 at 21:23](https://stackoverflow.com/posts/2378402/revisions "show all edits to this post")

[

![Lance Roberts's user avatar](media/Lance_Roberts's_user_avatar.jpg)

](https://stackoverflow.com/users/13295/lance-roberts)

[Lance Roberts](https://stackoverflow.com/users/13295/lance-roberts)

22k3131 gold badges110110 silver badges129129 bronze badges

asked Mar 4, 2010 at 10:20

[

![Suraj Chandran's user avatar](media/Suraj_Chandran's_user_avatar.png)

](https://stackoverflow.com/users/119772/suraj-chandran)

[Suraj Chandran](https://stackoverflow.com/users/119772/suraj-chandran)

24.3k1212 gold badges6262 silver badges9494 bronze badges

-   3
    
    Also, for Android usage, latest performance benchmark I have seen is this: [martinadamek.com/2011/02/04/…](http://www.martinadamek.com/2011/02/04/json-parsers-performance-on-android-with-warmup-and-multiple-iterations/) 
    
    – [StaxMan](https://stackoverflow.com/users/59501/staxman "110,726 reputation")
    
     [Feb 15, 2011 at 22:19](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment5600620_2378402)
    
-   1
    
    [Latest CowTalk performnce benchmark.](http://www.cowtowncoder.com/blog/archives/2011/01/entry_438.html#links) - January, 08, 2011 
    
    – [Iogui](https://stackoverflow.com/users/618499/iogui "1,191 reputation")
    
     [Feb 24, 2011 at 3:42](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment5716764_2378402) 
    
-   8
    
    One quick note: anyone choosing GSon should make sure to use 2.1 -- its performance is finally measurably better than earlier versions. 
    
    – [StaxMan](https://stackoverflow.com/users/59501/staxman "110,726 reputation")
    
     [Feb 11, 2012 at 21:07](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment11645452_2378402)
    
-   64
    
    With 74 up-votes as of now, this question obviously has some valuable answers. Good answers trump "not constructive" questions. Voting to reopen. 
    
    – [Nicholas](https://stackoverflow.com/users/43786/nicholas "15,779 reputation")
    
     [Aug 22, 2013 at 21:03](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment27009499_2378402) 
    
-   1
    
    Jackson's documentation is getting ridiculously complex now . . . 
    
    – [dongshengcn](https://stackoverflow.com/users/428024/dongshengcn "6,354 reputation")
    
     [Sep 17, 2013 at 19:18](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment27826433_2378402)
    

[Show **5** more comments](https://stackoverflow.com/questions/2378402/jackson-vs-gson# "Expand to show all comments on this post")

## 5 Answers

Sorted by:

                                              Highest score (default)                                                                   Trending (recent votes count more)                                                                   Date modified (newest first)                                                                   Date created (oldest first)                              

131

[](https://stackoverflow.com/posts/2378615/timeline)

I did this research the last week and I ended up with the same 2 libraries. As I'm using Spring 3 (that adopts Jackson in its default Json view '[JacksonJsonView](http://static.springsource.org/spring/docs/3.0.0.RC2/javadoc-api/org/springframework/web/servlet/view/json/MappingJacksonJsonView.html)') it was more natural for me to do the same. The 2 lib are pretty much the same... at the end they simply map to a json file! :)

Anyway as you said **Jackson** has a + in performance and that's very important for me. The project is also quite active as you can see from [their web page](https://github.com/FasterXML/jackson) and that's a very good sign as well.  

[Share](https://stackoverflow.com/a/2378615 "Short permalink to this answer")

Follow

[edited Dec 30, 2014 at 17:58](https://stackoverflow.com/posts/2378615/revisions "show all edits to this post")

[

![Hernán Eche's user avatar](media/Hernán_Eche's_user_avatar.jpg)

](https://stackoverflow.com/users/231382/hern%c3%a1n-eche)

[Hernán Eche](https://stackoverflow.com/users/231382/hern%c3%a1n-eche)

6,2461212 gold badges5151 silver badges7474 bronze badges

answered Mar 4, 2010 at 10:52

[

![mickthompson's user avatar](media/mickthompson's_user_avatar.png)

](https://stackoverflow.com/users/109970/mickthompson)

[mickthompson](https://stackoverflow.com/users/109970/mickthompson)

5,3721111 gold badges4545 silver badges5959 bronze badges

-   2
    
    Also, Google GSon does not yet suppor circular references. Does Jackson handle them ? 
    
    – [Guido](https://stackoverflow.com/users/12388/guido "45,611 reputation")
    
     [Mar 5, 2010 at 8:12](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment2363087_2378615)
    
-   1
    
    Circular References support... that should be a primary feature but I'm not sure if it does support them, I've never encountered a circular reference so far (even if they should be quite common I think, especially in the model). Here's another benchmark that can highlight how fast Jackson is if compared to GSon. It looks 100x faster in Serialization/Deserialization [code.google.com/p/thrift-protobuf-compare/wiki/Benchmarking](http://code.google.com/p/thrift-protobuf-compare/wiki/Benchmarking) 
    
    – [mickthompson](https://stackoverflow.com/users/109970/mickthompson "5,372 reputation")
    
     [Mar 5, 2010 at 8:45](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment2363230_2378615) 
    
-   1
    
    Jackson does not handle circular references currently. If that is important, XStream does; not sure if any native json package does (flex-json perhaps?) 
    
    – [StaxMan](https://stackoverflow.com/users/59501/staxman "110,726 reputation")
    
     [Mar 12, 2010 at 7:39](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment2415394_2378615)
    
-   13
    
    As of version 1.6, Jackson does support circular references. See [Handle bi-directional references using declarative methods](http://wiki.fasterxml.com/JacksonFeatureBiDirReferences) for reference. 
    
    – [Ophir Radnitz](https://stackoverflow.com/users/159452/ophir-radnitz "1,713 reputation")
    
     [Apr 10, 2011 at 20:41](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment6395783_2378615)
    
-   Jackson has more security issue, accodring to fortify 
    
    – [TuGordoBello](https://stackoverflow.com/users/3167016/tugordobello "4,134 reputation")
    
     [Apr 10, 2019 at 23:43](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment97938415_2378615)
    

[Show **1** more comment](https://stackoverflow.com/questions/2378402/jackson-vs-gson# "Expand to show all comments on this post")

88

[](https://stackoverflow.com/posts/2431212/timeline)

Jackson and Gson are the most complete Java JSON packages regarding actual data binding support; many other packages only provide primitive Map/List (or equivalent tree model) binding. Both have complete support for generic types, as well, as enough configurability for many common use cases.

Since I am more familiar with Jackson, here are some aspects where I think Jackson has more complete support than Gson (apologies if I miss a Gson feature):

-   Extensive annotation support; including full inheritance, and advanced "mix-in" annotations (associate annotations with a class for cases where you can not directly add them)
-   Streaming (incremental) reading, writing, for ultra-high performance (or memory-limited) use cases; can mix with data binding (bind sub-trees) -- **EDIT**: latest versions of Gson also include streaming reader
-   Tree model (DOM-like access); can convert between various models (tree <-> java object <-> stream)
-   Can use any constructors (or static factory methods), not just default constructor
-   Field and getter/setter access (earlier gson versions only used fields, this may have changed)
-   Out-of-box JAX-RS support
-   Interoperability: can also use JAXB annotations, has support/work-arounds for common packages (joda, ibatis, cglib), JVM languages (groovy, clojure, scala)
-   Ability to force static (declared) type handling for output
-   Support for deserializing polymorphic types (Jackson 1.5) -- can serialize AND deserialize things like List correctly (with additional type information)
-   Integrated support for binary content (base64 to/from JSON Strings)

[Share](https://stackoverflow.com/a/2431212 "Short permalink to this answer")

Follow

[edited May 3, 2012 at 1:07](https://stackoverflow.com/posts/2431212/revisions "show all edits to this post")

answered Mar 12, 2010 at 7:51

[

![StaxMan's user avatar](media/StaxMan's_user_avatar.png)

](https://stackoverflow.com/users/59501/staxman)

[StaxMan](https://stackoverflow.com/users/59501/staxman)

111k3434 gold badges204204 silver badges235235 bronze badges

-   7
    
    Actually, this post -- [cowtowncoder.com/blog/archives/2010/11/entry_434.html](http://www.cowtowncoder.com/blog/archives/2010/11/entry_434.html) -- summarizes many of Jackson features that are not found in other packages. 
    
    – [StaxMan](https://stackoverflow.com/users/59501/staxman "110,726 reputation")
    
     [Mar 2, 2011 at 23:02](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment5812860_2431212)
    
-   15
    
    I would consider not requiring annotations to be a feature of GSON, not a deficiency (which you have listed at least 3 times above). 
    
    – [orbfish](https://stackoverflow.com/users/407502/orbfish "7,101 reputation")
    
     [Jan 1, 2013 at 18:24](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment19530478_2431212)
    
-   8
    
    Neither Jackson nor Gson requires use of annotations. But having annotations as an option is a valuable feature in my opininion (esp. "mix-in annotations" which is additional processing option to allow associating external configuration). 
    
    – [StaxMan](https://stackoverflow.com/users/59501/staxman "110,726 reputation")
    
     [Jan 3, 2013 at 18:43](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment19589241_2431212)
    
-   3
    
    Gson allows you to register an InstanceCreator to specify an alternate way to construct an instance instead of using a default constructor. 
    
    – [inder](https://stackoverflow.com/users/154059/inder "1,756 reputation")
    
     [Mar 11, 2014 at 19:17](https://stackoverflow.com/questions/2378402/jackson-vs-gson#comment33942292_2431212)