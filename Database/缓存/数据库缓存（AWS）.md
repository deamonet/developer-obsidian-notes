### 数据库的速度和吞吐量是影响应用程序整体性能的最有效因素。

[开始使用缓存](https://aws.amazon.com/elasticache/)

## 概览

内存数据缓存是提高应用程序整体性能以及降低数据库成本最为有效的战略之一。

缓存可应用到任何类型的数据库，包括诸如 [Amazon RDS](https://aws.amazon.com/rds/) 之类的关系数据库，或诸如 [Amazon DynamoDB](https://aws.amazon.com/dynamodb/)、[MongoDB](https://www.mongodb.com/) 和 [Apache Cassandra](http://cassandra.apache.org/) 之类的 [NoSQL 数据库](https://aws.amazon.com/nosql/)。缓存的最大优点在于对实施产生的影响最小，这样可以在规模和速度方面显著提高应用程序的性能。

您将在下面找到在解决基于磁盘的数据库的相关限制和挑战过程中，可以采用的部分缓存策略和实施方法。

延伸阅读：[数据库缓存策略之使用 Redis 技术白皮书](https://d0.awsstatic.com/whitepapers/Database/database-caching-strategies-using-redis.pdf)  

![](media/MLabib-ElastiCache-Webinar-03-2017.6c5ed477377ea1e63ed7e08aa8aea1cf92b4e164.PNG.png)

ElastiCache 最佳实践和使用模式 - 2017 年 3 月

### [返回缓存主页](https://aws.amazon.com/caching/)

## 数据库带来的挑战

构建需要低延迟和可扩展性的分布式应用程序时，基于磁盘的数据库可能会给应用程序带来许多挑战。以下是其中几个常见的挑战：

-   查询处理速度慢：尽管有许多查询优化技术和架构设计有助于提升查询性能，但从磁盘检索数据加上额外的查询处理时间往往需要至少数十毫秒的查询响应时间，这还是在假设您的负载稳定且数据库以最佳状态运行的情况下。

-   扩展成本：不管数据是通过基于磁盘的 NoSQL 数据库分发还是在关系数据库中垂直扩展，要扩展为具有极高的读取速率都需要付出高昂成本，需要许多数据库只读副本才能应对单个内存缓存节点每秒的请求量。

-   需要简化数据访问：尽管关系数据库为数据模型关系提供了极好的方法，但这些方法并不是用于数据访问的最佳方法。在某些情况下，应用程序可能需要访问特定结构或视图中的数据，以简化数据检索和提高应用程序性能。

实施数据库缓存之前，许多架构师和工程师竭尽全力以期发挥出数据库的最大性能。尽管预期是合理的，但如果使用错误的工具来解决问题，可能会适得其反。例如，假如您想降低数据库查询的延迟，带着这样的合理预期来操作当属明智，但如果违背了从磁盘检索数据的相关物理定律，那就是浪费时间。 

## 缓存如何发挥作用

数据库缓存消除了主数据库面临的不必要的压力（通常是频繁访问的读取数据），对其功能有所助益。缓存本身可存在于数据库、应用程序等许多区域中，也可作为独立层存在。

下面是三种最常用的数据库缓存类型：

-   数据库集成缓存：一些数据库（例如 Amazon Aurora）提供集成缓存，该缓存在数据库引擎中托管，具有内置直写功能。当数据库表中的底层数据更改时，数据库会自动更新其缓存，这种方式非常好。无需动用应用层内的任何资源就可以利用此缓存。集成缓存的不足之处在于规模和功能。集成缓存通常受限制于数据库实例分配给缓存的可用内存，也不能用于与其他实例共享数据等其他用途。

-   本地缓存：本地缓存可存储应用程序中经常使用的数据。这不仅加快了数据检索速度，而且消除了与之相关的网络流量，使其速度快于其他缓存架构。主要缺点在于：应用程序内的每个节点都有自己的驻留缓存，这些缓存彼此孤立，并不连贯。单个缓存节点内存储的信息无法与其他本地缓存共享，不管这些信息是数据库缓存数据、Web 会话还是用户购物车。这给信息共享对于支持可扩展动态环境至关重要的分布式环境带来了挑战。由于大多数应用程序利用多个应用程序服务器，因此如果每个服务器都有自己的缓存，那么在这些缓存之间协调值就成为了一个巨大的挑战。  
      
    此外，出现中断时，本地缓存中的数据会丢失，需要有效补充，这就会给缓存造成不利影响。利用远程缓存可克服其中的大多数缺点。远程缓存（或称为“端缓存”）是专门用于存储内存中缓存数据的单独实例（或多个实例）。  
      
    对于介意网络延迟的环境，可以采用同时利用本地缓存和远程缓存的双层缓存策略。我们不会详细介绍此策略，但一般只有在绝对必要时才会使用，因为这种策略会增加复杂性。假如远程缓存的请求一般能以亚毫秒级性能实现，对于大多数应用程序来说，使用远程缓存时增加的网络开销便不值一提。

-   远程缓存：远程缓存存储在专用服务器上，通常基于 [Redis](https://aws.amazon.com/redis/) 和 [Memcached](https://aws.amazon.com/memcached/) 等键/值 NoSQL 存储构建而成。其中的每个缓存节点每秒可提供数十万到百万条请求。诸如 [Amazon ElastiCache for Redis](https://aws.amazon.com/elasticache/redis/) 之类的许多解决方案也可为关键工作负载提供所需的高可用性。  
      
    另外，远程缓存的平均请求延迟实现了亚毫秒级，其传输数量级比基于磁盘的数据库更快。在这种速度下，极少需要使用本地缓存。由于远程缓存可用作供所有不同系统使用的连接集群，因此非常适用于分布式环境。

借助远程缓存，数据缓存和数据有效性管理之间的编排工作由利用远程缓存的应用程序和/或进程进行管理。缓存本身未直接连接到数据库，但使用时就好像与之相邻。我们重点关注如何利用远程缓存（尤其是 [Amazon ElastiCache for Redis](https://aws.amazon.com/elasticache/redis/)）来缓存关系数据库数据。

如需了解缓存模式的更多信息，请访问[实施注意事项页面](https://aws.amazon.com/caching/implementation-considerations/)。  

## 关系数据库缓存技术

我们要谈到的许多技术都可应用于任何类型的数据库。但我们重点介绍的是关系数据库，因为这是最常见的数据库缓存用例。

应用程序从关系数据库查询数据时，基本模式是执行 SQL 语句，然后遍历返回的 ResultSet 对象指针以检索数据库行。  如果需要缓存返回的数据，可采用一些方法，但最好选择能够简化数据访问模式和/或优化应用程序架构目标的方法。

为了更加直观地呈现，我们通过示例代码段来说明数据库缓存逻辑。尽管包含 [Lettuce](https://github.com/wg/lettuce) 和 [Redisson](https://github.com/redisson/redisson) 的任何 Java Redis 库都有效，但我们还是使用 [Jedis](https://github.com/xetorthio/jedis) Redis 客户端库来连接 Redis。同样值得注意的是，一些应用程序框架可能包含以下部分数据库缓存逻辑技术。尽管如此，我们仍然需要了解实施详情，这在未利用这些较高级别抽象选项的情况下尤为重要。

我们假设已经向客户数据库发出了针对特定记录的查询，然后从中了解可以利用的缓存策略。假设以下 SQL 查询返回了一条记录：

SELECT FIRST_NAME, LAST_NAME, EMAIL, CITY, STATE, ADDRESS, COUNTRY FROM CUSTOMERS WHERE CUSTOMER_ID = “1001”;

...

Statement stmt = connection.createStatement();  
ResultSet rs = stmt.executeQuery(query);  
while (rs.next()) {  
      Customer customer = new Customer();  
      customer.setFirstName(rs.getString("FIRST_NAME"));             
      customer.setLastName(rs.getString("LAST_NAME"));  
等等…  
}

...

遍历 ResultSet 指针可从数据库行中检索字段和值，应用程序可在此时选择在哪里利用此数据以及如何利用。由于我们在这里不是讨论应用程序设计，因此不重点解释代码，而是重点介绍缓存逻辑。

我们还假设您使用的不是可用于抽象化处理缓存实现的应用程序框架。在明确了上述各点以后，接下来的问题就是，如何最好地缓存返回的数据库数据？

在上面的情景中，您有许多选择，我们来评估其中的几种。 

### 缓存数据库 SQL ResultSet

缓存包含已提取的数据库行的序列化 ResultSet 对象。

-   优点：当数据检索逻辑经过抽象化处理（例如在[数据访问对象](http://www.oracle.com/technetwork/java/dataaccessobject-138824.html)或“DAO”层中）后，使用代码仅预计获取 ResultSet 对象，无需了解其来源。不管 ResultSet 源自数据库还是从缓存中反序列化，结果都是 ResultSet，并已准备好进行遍历，这显著减少了集成逻辑。这种模式也可应用于任何关系数据库。  
      
    
-   缺点：数据检索仍需从 ResultSet 对象指针提取值，并未进一步简化数据访问；只是减少了数据检索延迟。

请注意，缓存行时，务必确保此行已序列化。以下示例通过实施 CachedRowSet 来实现此目标。另外，使用 Redis 时，这会存储为字节数组值。  

... 

// rs 包含 ResultSet  
    if (rs != null) {  //直接写入缓存  
        CachedRowSet cachedRowSet = new CachedRowSetImpl();  
        cachedRowSet.populate(rs, 1);  
        ByteArrayOutputStream bos = new ByteArrayOutputStream();  
        ObjectOutput out = new ObjectOutputStream(bos);  
        out.writeObject(cachedRowSet);  
        byte[] redisRSValue = bos.toByteArray();  
        jedis.set(key.getBytes(), redisRSValue);  
        jedis.expire(key.getBytes(), ttl);  
    }

...  

以上代码段将 CachedRowSet 对象转换为字节数组，然后将该字节数组存储为 Redis 字节数组值。上面使用的键值是已转换为字节的实际 SQL 语句。这种方法将 SQL 语句存储为键的好处在于能够生成隐藏实施详情的透明缓存抽象层。另一个额外的好处是无需在自定义键 ID 和已执行的 SQL 语句之间创建任何附加映射。最后一个语句执行过期命令，以将 TTL 应用到已存储的键。此代码遵循我们的直写逻辑，即在查询数据库之后，立即存储缓存的值。

对于延迟总体，您需要在对数据库执行查询之前，先查询缓存。隐藏实施详情的一个好建议是利用 DAO 模式，展示一种通过应用程序检索数据的通用方法。例如，由于您的键是实际的 SQL 语句，方法签名可能类似以下内容：

public ResultSet getResultSet(String key);    // 键是 sql 语句 

调用（使用）此方法的代码仅预计获取 ResultSet 对象，无论此接口的底层实施详情如何。在后台，getResultSet 方法会对 SQL 键执行 GET，如果存在，则会将其反序列化并转换为 ResultSet。 

public ResultSet getResultSet(String key) {  
  byte[] redisResultSet = null;  
  redisResultSet = jedis.get(key.getBytes());  
  ResultSet rs = null;  
  if (redisResultSet != null) { //如果存在缓存值，将其反序列化并返回  
    try {  
          cachedRowSet = new CachedRowSetImpl();  
          ByteArrayInputStream bis = new         ByteArrayInputStream(redisResultSet);  
          ObjectInput in = new ObjectInputStream(bis);  
          cachedRowSet.populate((CachedRowSet) in.readObject());  
          rs = cachedRowSet;  
    }...  
  } else {  
  //从数据库获取 ResultSet，将其存储在 rs 对象中，然后进行缓存。

 ...

 }

...

return rs;  
}  

如果此数据不在缓存中，您可以在数据库中查询它，先将其缓存然后再返回。如上文所述，最好也在键上应用适当的 TTL。

对于下面的其他所有缓存技术，需要为 Redis 键确定命名约定。好的命名约定便于应用程序和开发人员轻松预测。用冒号分隔的层次结构是键的常见命名约定，例如 object:type:id

### 采用自定义格式缓存所选字段和值

将已提取的数据库行的子集缓存到可供应用程序使用的自定义结构中

-   优点：这种方法非常易于实施。您主要将检索的具体字段和值存储到诸如 JSON 或 XML 之类的结构中，然后使用 SET 命令将该结构设置为 Redis 字符串。您选择的格式应该符合应用程序数据访问模式。
-   缺点：您的应用程序在查询特定数据时采用了不同类型的对象（例如根据需要使用的 Redis 字符串和数据库结果）。此外，您需要解析整个结构以检索与之相关的各个属性。 

...

// rs 包含 ResultSet  
while (rs.next()) {  
              Customer customer = new Customer();            
              Gson gson = new Gson();           
              JsonObject customerJSON = new JsonObject();  
              customer.setFirstName(rs.getString("FIRST_NAME"));  
              customerJSON.add(“first_name”, gson.toJsonTree(customer.getFirstName() );     
              customer.setLastName(rs.getString("LAST_NAME"));  
              customerJSON.add(“last_name”, gson.toJsonTree(customer.getLastName() );  
              等等…  
              jedis.set(customer:id:"+customer.getCustomerID(), customerJSON.toString() );  
      }

...  

以上代码段将特定的客户属性存储在 JSON 对象中，然后将该 JSON 对象缓存在 Redis 字符串中。

对于数据检索，您可以实施接受客户键（例如 customer:id:1001）和 SQL 语句字符串参数的通用方法。它还可以返回应用程序所需的任何结构（JSON、XML 等），对底层详情进行抽象化处理，这和我之前提到的情况类似。

发出初始请求时，应用程序会对客户键执行 GET 操作，如果存在相应值，则返回此值并完成调用。如果不存在，则在数据库中查询此记录，将数据的 JSON 表示形式直写到缓存然后再返回。

### 将所选字段和值缓存到聚合 Redis 数据结构

将已提取的数据库行缓存到可简化应用程序数据访问的特定数据结构中

-   优点：将 ResultSet 转换为可简化访问的格式（例如 Redis 哈希）后，应用程序能够更有效地利用此数据。此技术无需遍历 ResultSet 或解析自定义结构（例如用字符串存储的 JSON 对象），从而简化了数据访问模式。此外，与 Redis 中 Lists、Sets 和 Hashes 等聚合数据结构配合使用，可采用原生应用程序数据结构的形式执行 SET/GET 操作。  
      
    
-   缺点：您的应用程序在查询特定数据时采用了不同类型的对象（例如根据需要使用的 Redis 哈希和数据库结果）。  

...

// rs 包含 ResultSet  
  while (rs.next()) {  
       Customer customer = new Customer();  
       Map<String, String> map = new HashMap<String, String>();  
       customer.setFirstName(rs.getString("FIRST_NAME"));  
       map.put("firstName", customer.getFirstName());                 
       customer.setLastName(rs.getString("LAST_NAME"));  
       map.put("lastName", customer.getLastName());  
       等等…  
       jedis.hmset(customer:id:"+customer.getCustomerID(), map);  
  }

...

上面的代码创建了 HashMap 对象来存储客户数据。该映射使用数据库数据进行填充，并用 SET 操作存储到 Redis 哈希。

对于数据检索，您可以实施接受客户 ID（键）和 SQL 语句参数的通用方法。它还会将 HashMap 返回给调用方。与其他示例一样，可写入这项方法实施以隐藏映射来源的详情。首先，您的应用程序可以使用客户 ID 键在缓存中查询客户数据。如果此数据不存在，将执行 SQL 语句并从数据库中检索此数据。在检索时，您还可以存储该客户的哈希表示形式以延迟加载。

与 JSON 不同，以哈希格式在 Redis 中存储数据的额外优点是可以查询其中的单个属性。对于一个给定请求，您不希望将所有数据存储在一个具体的客户映射中，而只希望存储客户名字，Redis 支持此功能，除此之外还支持诸如在映射中添加/删除单个属性等其他多种功能。

### 缓存已序列化的应用程序对象实体

将已提取的数据库行的子集缓存到可供应用程序使用的自定义结构中

-   优点：使用简单的序列化/反序列化技术以原生应用程序状态利用应用程序对象。这可以最大限度减少数据转换逻辑，从而迅速提高应用程序性能。  
      
    
-   缺点：适用于高级应用程序开发用例。

...

       Customer customer = (Customer) object;  
       ByteArrayOutputStream bos = new ByteArrayOutputStream();         
       ObjectOutput out = null;  
       try {  
             out = new ObjectOutputStream(bos);             
             out.writeObject(customer);             
             out.flush();             
             byte[] objectValue = bos.toByteArray();  
             jedis.set(key.getBytes(), objectValue);  
             jedis.expire(key.getBytes(), ttl);  
         }

...     

以上代码将客户对象转换为字节数组，然后将该值存储到 Redis 中。键可以是客户标识符或字节表示形式的键（例如 customer:id:1001）。

与其他示例一样，在实例化对象或为其补充状态时，您可以创建接受客户 ID（键）和 SQL 语句参数的通用方法。该方法还将返回客户对象。首先，您的应用程序可以使用客户 ID 在缓存中查询已序列化的客户对象。如果数据不存在，则可执行 SQL 语句，应用程序将使用此数据，补充客户实体对象，然后延迟在缓存中加载此数据的序列化表示形式。 

    public Customer getObject(String key) {  
    Customer customer = null;  
    byte[] redisObject = null;  
    redisObject = jedis.get(key.getBytes());  
    if (redisObject != null) {  
      try {  
          ByteArrayInputStream in = new ByteArrayInputStream(redisObject);  
          ObjectInputStream is = new ObjectInputStream(in);  
          customer = (Customer) is.readObject();  
       } ...

   } ...  
     return customer;  
}

## Amazon ElastiCache 与自主管理的 Redis 对比

Redis 是一种开源内存数据存储，已成为市场上最常用的键/值引擎。Redis 之所以受欢迎，很大程度上是因为支持多种数据结构以及包括 [Lua](https://www.lua.org/) 脚本支持和 Pub/Sub 消息收发功能在内的其他功能。其他额外优势包括具有高可用性的拓扑结构，该结构支持只读副本并且能够保存数据。

Amazon ElastiCache 为 Redis 提供了完全托管服务。这意味着与 Redis 集群相关的所有管理任务（包括监控、修补、备份和自动故障转移）都由 Amazon 负责管理。这样，您就能够集中注意力于业务和数据，而不是运营。

使用 Amazon ElastiCache for Redis 自助管理缓存环境的其他优势包括：

-   增强的 Redis 引擎不仅完全兼容开源版本，而且还新增了稳定性和稳健性。
-   提供诸如移出策略、缓冲限制等易于修改的参数
-   能够扩展集群规模以及将其调整为支持 TB 级数据
-   增强了安全性，支持在 Amazon VPC 内隔离集群

有关更多信息，请参阅以下资源：  
有关 Amazon ElastiCache for Redis 的更多信息，请[单击此处](https://aws.amazon.com/elasticache/redis/)。  
有关 Redis 的完整命令列表，请[单击此处](https://redis.io/commands)。

---

## 数据库缓存示意图

![](media/caching-database-caching-diagram.70a0c9d62877d7a32bf1024d00561eb5b560a45d.PNG.png)