作者：陈峻2021-10-29 08:00:00

[开发](https://www.51cto.com/developer)[后端](https://www.51cto.com/backend)

本文将通过一系列基准性的测试，和您讨论目前四种最为流行的Java JSON库--JSON.simple、GSON、Jackson、以及JSONP，在解析不同文件大小时的速度，以方便您能够做出明智的选择。

[![](media/40c92dcb83798ca6b247dec2bb6ca679.jpg)](https://s3.51cto.com/oss/202110/28/40c92dcb83798ca6b247dec2bb6ca679.jpg)

【51CTO.com快译】如今，JSON已经是公认的、服务器与Web应用之间数据传输的API标准。它通过各种代码库，可以在Hadoop或Spark集群中，方便那些基于微服务和分布式架构的数据处理应用程序，传输和解析不同类型与大小的文件。不过，并非所有的JSON库都会执行相同的操作方式。在高吞吐量的环境中，一旦出现了快速、连续、大量的小文件，它们的解析速度就会出现巨大的差别。

可见，为自己的应用环境选择合适的代码库，往往是至关重要的。为此，我准备了一系列基准性的测试，将和您讨论目前四种最为流行的Java JSON库--JSON.simple、GSON、Jackson、以及JSONP，在解析不同文件大小时的速度，以方便您能够做出明智的选择。

## 四大JSON库简介

由于这四种代码库都时常被广泛地用于Java环境的JSON处理过程，因此我们根据它们在Github项目中的受欢迎程度，进行逐一介绍：

-   [Yidong Fang的JSON.simple](https://github.com/fangyidong/json-simpleJSON.simple)：是一个可用于编、解码的JSON文本类Java工具包。它旨在成为一个简单、轻量级、且能够高速运行的代码库。
-   [Google的GSON](https://github.com/google/gson)：是一个能够实现Java对象与JSON格式相互转换的Java代码库。由于提供了对于Java泛型的完全支持，因此您既不需要对自己的类进行注释，又能够简化对于特权源代码的访问。
-   [FasterXML的Jackson项目](https://github.com/FasterXML/jackson)：是一组带有流式JSON解析器和生成器库的处理工具。虽然它是针对Java设计的，但是它可以处理其他非JSON类的编码。根据Github使用情况的调查结果，它目前是最受欢迎的JSON解析器之一。
-   [Oracle的JSONP](https://jsonp.java.net/)：JSON Processing是以Java API的形式，通过消费(consuming)和生成(producing)流式JSON文本，来提供对于JSON的处理。它属于开源参考JSR353的实现。

## 基准：2017

我们根据不同文件大小的处理需求，在不同的库环境中，测试性地解析了不同大小的文件。总的说来，我们将基准测试分为两个关键场景：

-   从https://github.com/zeMirco/sf-city-lots-json处获取190 MB的大文件，并测试其解析速度。
-   从http://www.json-generator.com/处随机生成1 KB的小文件，并测试其解析速度。

无论是大文件，还是小文件，我们让每个库都对每个文件运行10次。其中，对于大文件而言，每个库的每一次运行时，都会进行10次迭代;而对于小文件，它们会每次运行10,000次迭代。而且在小文件的测试中，我们不会在迭代的间隙中，将文件保留在内存里，而是直接在AWS上的c3.large实例中运行测试。

下图完整地显示了大文件的解析结果。为了节省空间，我对小文件的解析结果进行了平均。如果您想查看扩展的结果，请参见--https://docs.google.com/spreadsheets/d/1iOSAkqRwGGbLHRkcKSyHwpOWYdSrkuQKyUIZ6ef7N-I/edit?usp=sharing。而如果您想查看小文件或库的源代码，则请参见--https://github.com/terencetaih/aws-speed。

### 大文件结果：2017

[![](media/a6590d7017101d87ad37a44e0838ba3c.png)](https://s2.51cto.com/oss/202110/28/a6590d7017101d87ad37a44e0838ba3c.png)

从上表中，我们可以看出Jackson或JSON.simple在处理大文件上用时最少，而且Jackson在总体上会领先于JSON.simple。通过查看所有测试的平均运行结果可知，Jackson和JSON.simple遥遥领先，JSONP名列第三，GSON排在最后。下面，让我们以Jackson为基准，用百分比来表示各个JSON库的解析速度：

[![](media/96f66a1efb1d59046537e14564bff9f7.png)](https://s2.51cto.com/oss/202110/28/96f66a1efb1d59046537e14564bff9f7.png)

### 小文件结果：2017

[![](media/bfb70ff045fbe86af06f6f0d0ec8929c.png)](https://s5.51cto.com/oss/202110/28/bfb70ff045fbe86af06f6f0d0ec8929c.png)

上表显示了每个文件被运行10次的平均值，最下面一行是总的平均值。各个库所获得的文件数量分别为：

-   GSON - 14
-   JSONP - 5
-   Jackson – 1
-   JSON.simple – 0

由所有文件的平均测试结果可知：

-   GSON是其中的赢家。
-   尽管在任何单个文件上都不是最快的，但是JSON.simple在总体上名列第二。
-   尽管在少数文件上能够达到最快，但是JSONP在总体上仍排名第三。
-   虽然Jackson在所有文件解析中都表现得非常一致，但是与其他三个库相比，Jackson的运行速度是最慢的。

让我们同样以GSON为基准，用百分比来表示各个JSON库的解析速度：

[![](media/c85d12c854ec31b747ea6bf233d8e814.png)](https://s2.51cto.com/oss/202110/28/c85d12c854ec31b747ea6bf233d8e814.png)

## 结论：2017

在选择JSON库时，解析速度虽然并非唯一的考虑因素，但它的确是一个重要的参考指标。在运行上述基准测试时，我们发现没有一个库可以在所有文件大小、以及所有解析速度上，同时能够击败其他库。在大文件解析上表现最好的库，反而对小文件的表现并不佳，反之亦然。

因此，我们可以参照如下环境特征，根据各自的解析速度优势，选择合适的JSON库。

-   如果您的环境需要经常或主要处理大型的JSON文件，那么Jackson可以成为您的首选库，而GSON在处理此类大文件时会非常费劲。
-   如果您的环境主要处理大量的小型JSON请求，例如需要在微服务、或分布式架构中进行各种设置的话，那么GSON可以成为您的首选库，而Jackson在处理小文件方面会比较费劲。
-   如果您的环境不得不经常处理大小不同的文件，那么由于JSON.simple在上述两类测试中都排在第二位，因此可以成为适用于可变环境的实用工具(毕竟Jackson和GSON在不同文件大小上的表现各有利弊)。

JSONP在任何情况下，与其他三种JSON库相比，对于各种文件的解析速度性能都比较差，因此不太值得推荐。当然，据报道Java 9正在获得JSON的原生实现，那么我们可以预见到它将会在参考实现方面，有所改进。

## 基准：2021

下面，我们将使用最新的JSON库，并在Java 11上再次运行相同的基准测试。总体而言，它们会在性能上，比我们之前运行的基准测试，出现显著的提高。

### 大文件结果：2021

[![](media/c9608216914146d885ccb341bb455588.png)](https://s6.51cto.com/oss/202110/28/c9608216914146d885ccb341bb455588.png)

如上表所示，本轮基准测试的赢家是GSON，JSONP紧随其后，接着是Jackson，最后才是JSON.simple。那么，让我们以GSON为基准，用百分比来表示各个JSON库的解析速度：

[![](media/4fde48c6dac033ce80ececc2593226dd.png)](https://s2.51cto.com/oss/202110/28/4fde48c6dac033ce80ececc2593226dd.png)

### 小文件结果：2021

上表显示了小文件的测试结果，Gson以明显的速度优势，再次击败了其他三个JSON库，蝉联冠军。JSONP紧随其后，接着是JSON.simple，最后才是Jackson。

[![](media/851cd82649eb772eaf86c83f9e23b0a9.png)](https://s4.51cto.com/oss/202110/28/851cd82649eb772eaf86c83f9e23b0a9.png)

由表可知，Gson处理这些文件的速度比第二名快了近200多毫秒。

## 结论：2021

通过上述两大类测试与比较，我们不难发现，无论是Java本身，还是各个JSON库，都在速度上提升了不少。其中，GSON的进步最为明显，并且能够最终在大文件和小文件，两类基准测试中脱颖而出。

**原文标题：The UltimateJSONLibrary:JSON.simple vs. GSON vs. Jackson vs.JSONP，作者：Nick Andrews**

【51CTO译稿，合作站点转载请注明原文译者和出处为51CTO.com】