[![](media/170b937fbad8b9b0~tplv-t2oaga2asx-no-mark!100!100!100!100.awebp.webp)](https://juejin.cn/user/2999123453681159)

[Hugh的白板 ![lv-3](https://lf3-cdn-tos.bytescm.com/obj/static/xitu_juejin_web/img/lv-3.7938ebc.png "创作等级")](https://juejin.cn/user/2999123453681159) 

2021年06月21日 22:28 ·  阅读 10997

## 一、主要内容

-   spring boot 引入Elasticsearch
-   ElasticsearchTemplate的使用
-   ElasticsearchRepository的使用

## 二、环境整合

### 创建Elasticsearch工程，引入依赖

一般情况下，都会单独创建一个工程，用于操作es。

```pom
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-elasticsearch</artifactId>
	<version>2.5.1</version>
</dependency>
复制代码
```

### 版本统一

目前`springboot-data-elasticsearch`最新版本是2.5.1，这个版本使用的es版本为7.12.1，需要注意的是，我们这里引入的版本需要与服务器安装的版本一致。

可以通过Maven仓库进行查看版本信息，地址：[mvnrepository.com/artifact/or…](https://link.juejin.cn?target=https%3A%2F%2Fmvnrepository.com%2Fartifact%2Forg.springframework.boot%2Fspring-boot-starter-data-elasticsearch "https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-elasticsearch")

### 修改配置文件

```yml
spring: 
 data: 
  elasticsearch: 
   cluster-name: es-cluster
   cluster-nodes: 10.133.28.55:9300
复制代码
```

## 三、ElasticsearchTemplate的使用

ElasticsearchTemplate类似于RedisTemplate，是spring提供的较为底层的与Elasticserch交互的类。

### 常用注解

#### @Document

注解作用在类上，标记实体类为文档对象，指定实体类与索引对应关系。常用配置项有：

-   indexName：索引名称
-   type: 索引类型，默认为空
-   shards: 主分片数量，默认5
-   replicas：复制分片数量，默认1
-   createIndex：创建索引，默认为true

#### @Id

指定文档ID，加上这个注解，文档的`_id`会与我们的数据ID是一致的，否则在不给定默认值的情况下，es会自动创建。

#### @Field

指定普通属性，标明这个是文档中的一个字段。常用的配置项有：

-   type： 对应Elasticsearch中属性类型。默认自动检测。使用FiledType枚举可以快速获取

```java
public enum FieldType {
	Text,//会进行分词并建立索引的字符类型
	Integer,
	Long,
	Date,
	Float,
	Double,
	Boolean,
	Object,
	Auto,//自动判断字段类型
	Nested,//嵌套对象类型
	Ip,
	Attachment,
	Keyword//不会进行分词建立索引的类型
}
复制代码
```

-   index： 是否创建倒排索引，一般不需要分词的属性不需要创建索引
    
-   analyzer：指定索引类型。
    
-   store：是否进行存储，默认不进行存储。
    
    > 其实不管我们将store值设置为true或false，elasticsearch都会将该字段存储到Field域中；但是他们的区别是什么？
    > 
    > 1.  store = false时，默认设置；那么给字段只存储在"_source"的Field域中；
    >     
    > 2.  store = true时，该字段的value会存储在一个跟_source平级的独立Field域中；同时也会存储在_source中，所以有两份拷贝。
    >     
    > 
    > 那么我们在什么样的业务场景下使用store field功能？
    > 
    > 1.  _source field在索引的mapping 中disable了。这种情况下，如果不将某个field定义成store=true，那些将无法在返回的查询结果中看到这个field。
    > 2.  _source的内容非常大。这时候如果我们想要在返回的_source document中解释出某个field的值的话，开销会很大（当然你也可以定义source filtering将减少network overhead），比例某个document中保存的是一本书，所以document中可能有这些field: title, date, content。假如我们只是想查询书的title 跟date信息，而不需要解释整个_source（非常大），这个时候我们可以考虑将title, date这些field设置成store=true。
    

### 创建实体

```java
// 关联的索引是item，类型是_doc，直接使用而不创建索引
@Document(indexName = "item",type = "_doc",createIndex = "false")
public class Item {
    @Id
    private Long id;
    // title使用ik进行分词
    @Field(type = FieldType.Text,analyzer = "ik_max_word")
    private String title;
    // brand 不被分词
    @Field(type=FieldType.Keyword)
    private String brand;
    @Field(type=FieldType.Double)
    private Double price;
    // brand 不被分词，且不创建索引
    @Field(index = false,type = FieldType.Keyword)
    private String images;
}
复制代码
```

### 索引管理

ElasticsearchTemplate提供了创建索引的方法，但是不建议使用 ElasticsearchTemplate 对索引进行管理（创建索引，更新映射，删除索引）。

索引就像是数据库或者数据库中的表，我们平时是不会是通过java代码频繁的去创建修改删除数据库或者表的相关信息，我们只会针对数据做CRUD的操作。

```java
// 创建索引
elasticsearchTemplate.createIndex(Class<T> clazz);
// 删除索引,有好几个方法，有兴趣的同学可以自行翻阅源码
elasticsearchTemplate.deleteIndex(Class<T> clazz);
复制代码
```

### 创建文档

#### 新增单条文档

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ESTest {
    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    @Test
    public void insertItemDoc(){
        Item item = new Item(1001L,"XXX1","XXX1","XXX1");
        IndexQuery indexQuery = new IndexQueryBuilder().withObject(item).build();
        elasticsearchTemplate.index(indexQuery);
    }
}
复制代码
```

#### 批量新增文档

```java
@RunWith(SpringRunner.class)
@SpringBootTest(classes = Application.class)
public class ESTest {
    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    @Test
    public void insertItemDocBulk() {
    	List<Item> list = new ArrayList<>();
        list.add(new IndexQueryBuilder().withObject(new Item(1001L,"XXX1","XXX1","XXX1")).build())
        list.add(new IndexQueryBuilder().withObject(new Item(1002L,"XXX2","XXX2","XXX2")).build())
        IndexQuery indexQuery = new IndexQueryBuilder().withObject(item).build();
        elasticsearchTemplate.index(indexQuery);
    }
}
复制代码
```

### 删除文档

删除都是根据主键进行删除，提供了两个方法：

-   delete(String indexName,String typeName,String id); 通过字符串指定索引，类型和id值；
-   delete(Class,String id) 第一个参数传递实体类类类型，建议使用此方法，减少索引名和类型名由于手动编写出现错误的概率。

代码比较简单，不再示例。

### 修改文档

```java
@Test
public void updateItemDoc() {
    Map<String, Object> sourceMap = new HashMap<>();
    sourceMap.put("title", "YYY");

    IndexRequest indexRequest = new IndexRequest();
    indexRequest.source(sourceMap);

    UpdateQuery updateQuery = new UpdateQueryBuilder()
        .withClass(Stu.class)
        .withId("1001")
        .withIndexRequest(indexRequest)
        .build();
    elasticsearchTemplate.update(updateQuery);
}
复制代码
```

### 查询文档

#### 模糊查询

使用查询条件与所有的field进行匹配，进行查询

> 对于一些入参是不是看着很陌生？感觉头大是不是，别急，最后有解释。

```java
@Test
public void queryItemDoc() {
    // 少年去和所有field进行匹配
    QueryStringQueryBuilder queryStringQueryBuilder = QueryBuilders.queryStringQuery("少年");
    // 查询条件SearchQuery是接口，只能实例化实现类。
    SearchQuery searchQuery = new NativeSearchQuery(queryStringQueryBuilder);
    List<Item> list = elasticsearchTemplate.queryForList(searchQuery, Item.class);
    for(Item item : list){
        System.out.println(item);
    }
}
复制代码
```

#### 使用match_all查询所有文档

```java
@Test
public void matchAllItemDoc() {
   SearchQuery searchQuery = new NativeSearchQuery(QueryBuilders.matchAllQuery());
    List<Item> list = elasticsearchTemplate.queryForList(searchQuery, Item.class);
    for(Item item : list){
        System.out.println(item);
    }
}
复制代码
```

#### 使用match查询分页文档并排序

```java
@Test
public void matchItemDoc() {
    // 构造分页信息：第一个参数是页码，从0算起。第二个参数是每页显示的条数
   	Pageable pageable = PageRequest.of(0, 2);
    // 构造排序信息
    SortBuilder sortBuilder = new FieldSortBuilder("price")
                .order(SortOrder.DESC);
    SearchQuery query = new NativeSearchQueryBuilder()
        .withQuery(QueryBuilders.matchQuery("title", "english"))
        .withPageable(pageable)
        .withSort(sortBuilder)
        .build();
    AggregatedPage<Item> pagedItem = elasticsearchTemplate.queryForPage(query, Item.class);
    System.out.println("查询后的总分页数目为：" + pagedItem.getTotalPages());
    List<Item> list = pagedItem.getContent();
    for (Item item : list) {
        System.out.println(item);
    }
}
复制代码
```

#### 多条件查询

```java
 @Test
 public void mustShouldItemDoc(){
     BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
     List<QueryBuilder> listQuery = new ArrayList<>();
     listQuery.add(QueryBuilders.matchPhraseQuery("title","XX"));
     listQuery.add(QueryBuilders.rangeQuery("price").gte(10).lte(100));
     //boolQueryBuilder.should().addAll(listQuery); // 逻辑或
     boolQueryBuilder.must().addAll(listQuery);  // 逻辑与
     SearchQuery searchQuery = new NativeSearchQuery(boolQueryBuilder);
     List<Item> list = elasticsearchTemplate.queryForList(searchQuery, Item.class);
     for(Item item : list){
         System.out.println(item);
     }
 }
复制代码
```

#### 高亮查询

```java
@Test
public void highlightStuDoc(){
    // 指定高亮的信息用什么标签包裹，默认使用<em>
    String preTag = "<font color='red'>";
    String postTag = "</font>";

    Pageable pageable = PageRequest.of(0, 10);

    SortBuilder sortBuilder = new FieldSortBuilder("price")
        .order(SortOrder.DESC);
	// 构造查询规则
    SearchQuery query = new NativeSearchQueryBuilder()
        .withQuery(QueryBuilders.matchQuery("title", "XXX"))
        .withHighlightFields(new HighlightBuilder.Field("title")
                             .preTags(preTag)
                             .postTags(postTag))
        .withSort(sortBuilder)
        .withPageable(pageable)
        .build();
    AggregatedPage<Item> paged = esTemplate.queryForPage(query, Item.class, new SearchResultMapper() {
        @Override
        public <T> AggregatedPage<T> mapResults(SearchResponse response, Class<T> clazz, Pageable pageable) {
			// 定义一个list存放最终返回的数据
            List<Item> listHighlight = new ArrayList<>();
			// 取出命中的信息
            SearchHits hits = response.getHits();
            for (SearchHit h : hits) {
                // 取出高亮的数据
                HighlightField highlightField = h.getHighlightFields().get("title");
                String title = highlightField.getFragments()[0].toString();
				// 取出其他数据
                Object id = (Object)h.getSourceAsMap().get("id");
                String brand = (String)h.getSourceAsMap().get("brand");
                Object price = (Object)h.getSourceAsMap().get("price");
                String images = (String)h.getSourceAsMap().get("images");
				// 处理数据
                Item itemHL = new Item();
                item.setId(Long.valueOf(id.toString()));
                item.setTitle(title);
                item.setBrand(brand);
                item.setPrice(Double.valueof(price.toString));
                item.setImages(images);

                listHighlight.add(itemHL);
            }

            if (listHighlight.size() > 0) {
                return new AggregatedPageImpl<>((List<T>)listHighlight);
            }
            return null;
        }
    });
    System.out.println("查询后的总分页数目为：" + paged.getTotalPages());
    List<Item> list = paged.getContent();
    for (Item item : list) {
        System.out.println(item);
    }
}
复制代码
```

## 四、ElasticsearchRepository的使用

ElasticsearchRepository是基于ElasticsearchTemplate封装的一个接口用于操作Elasticsearch。

它主要干了啥呢？我们能通过它干些什么呢？

-   它主要帮我们封装了一些常用的方法。如下图：
    
    ![image-20210617133707448](media/image-20210617133707448.webp)
    
-   我们还可以通过使用`Not` `Add` `Like` `Or` `Between`等关键词自动创建查询语句。它会自动帮你完成，无需写实现类。
    
    比如：你的方法名叫做：findByTitle，那么它就知道你是根据title查询，然后自动帮你完成，无需写实现类。
    
    【自定义方法命名约定】：
    
    ![image-20210617134220264](media/image-20210617134220264.webp)
    

### 自定义Repository接口

这个是遵循SpringData的规范，在自定义接口中直接指定查询方法名称便可查询，无需进行实现。在idea里面也会提示ES里面有的字段，写起来挺方便的。

```java
/**
 * 需要继承ElasticsearchRepository接口
 */
public interface ItemRepository extends ElasticsearchRepository<Item,Long> {
    /**
     * 方法名必须遵守SpringData的规范
     * 价格区间查询
     */
    List<Item> findByPriceBetween(double price1, double price2);
}
复制代码
```

### 创建文档

#### 新增单条文档

可以直接调用`ItemRepository`的`save()`方法。

```java
@Autowired
private ItemRepository itemRepository;
@Test
public void insert(){
    Item item = new Item(1L, "小米手机7", "小米", 3499.00, "http://xxx/xxx.jpg");
    itemRepository.save(item);
}
复制代码
```

#### 批量新增文档

插入多条文档数据使用的`saveAll()`方法

```java
@Autowired
private ItemRepository itemRepository;
@Test
public void indexList() {
    List<Item> list = new ArrayList<>();
    list.add(new Item(1L, "小米手机7", "小米", 3299.00, "http://xxx/1.jpg"));
    list.add(new Item(2L, "坚果手机R1", "锤子", 3699.00, "http://xxx/1.jpg"));
    list.add(new Item(3L, "华为META10", "华为", 4499.00, "http://xxx/1.jpg"));
    list.add(new Item(4L, "小米Mix2S", "小米", 4299.00, "http://xxx/1.jpg"));
    list.add(new Item(5L, "荣耀V10", "华为", 2799.00, "http://xxx/1.jpg"));
    // 接收对象集合，实现批量新增
    itemRepository.saveAll(list);
}
复制代码
```

### 修改文档

修改文档数据也是使用的`save()`方法。

### 查询参数介绍

我们通过上面的方法截图，可以看出来`ElasticsearchRepository`提供了一些特殊的`search()`方法，用来构建一些查询。这几个方法里面的参数主要是`SearchQuery`和`QueryBuilder`，所以要完成一些特殊查询主要就是构建这两个参数。

**SearchQuery**

通过源码我们可以看到，`SearchQuery`是一个接口，有一个实现类叫`NativeSearchQuery`，实际使用中，我们的主要任务就是构建`NativeSearchQuery`来完成一些复杂的查询的。

![image-20210618093342034](media/image-20210618093342034.webp)

我们可以看到`NativeSearchQuery`的主要入参是`QueryBuilder`、`SortBuilder`、`HighlightBuilder`

一般情况下，我们不是直接是`new NativeSearchQuery`，而是使用`NativeSearchQueryBuilder`来完成NativeSearchQuery的构建。

```scss
NativeSearchQueryBuilder		
    .withQuery(QueryBuilder1)
    .withFilter(QueryBuilder2)
    .withSort(SortBuilder1)
    .withXXXX()
    .build();
复制代码
```

**QueryBuilder**

QueryBuilder是一个接口，它有非常多的实现类，可以用于不同的查询条件构建。

要构建QueryBuilder，我们可以使用工具类QueryBuilders，里面有大量的方法用来完成各种各样的QueryBuilder的构建，字符串的、Boolean型的、match的、地理范围的等等。

```java
// 构建一个普通的查询
SearchQuery searchQuery = new NativeSearchQueryBuilder()
    .withQuery(QueryBuilders.queryStringQuery("spring boot OR 书籍"))
    .build();
复制代码
```

查询方法跟上面介绍的`ElasticsearchTemplate`差不多，就不再单独举例了。

综上，我们在一般的场景下使用`ElasticsearchRepository`就可以满足我们的需要，在一些复杂的查询场景下，可以配合`ElasticsearchTemplate`来使用。

最后，欢迎关注「Hugh的白板」公号，私信我一起学习，一起成长！ 最最后，如果对你有一点帮助，可否给一个赞？非常感谢哦~~~