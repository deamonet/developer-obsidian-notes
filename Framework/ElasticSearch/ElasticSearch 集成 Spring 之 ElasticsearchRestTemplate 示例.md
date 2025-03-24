发表于 2021-02-07 分类于 [Database](https://jueee.github.io/categories/Database/) ， [ElasticSearch](https://jueee.github.io/categories/Database/ElasticSearch/) Valine： [0](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#valine-comments "valine")

### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#ElasticsearchRestTemplate "ElasticsearchRestTemplate")ElasticsearchRestTemplate

ElasticsearchRestTemplate 是 spring-data-elasticsearch 项目中的一个类，和其他 spring 项目中的 template 类似。

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#引入依赖 "引入依赖")引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#操作方法 "操作方法")操作方法

Spring Data ElasticSearch 有下边这几种方法操作 ElasticSearch：

-   ElasticsearchRepository（传统的方法，可以使用）
-   ElasticsearchRestTemplate（推荐使用。基于 RestHighLevelClient）
-   ElasticsearchTemplate（ES7 中废弃，不建议使用。基于 TransportClient）
-   RestHighLevelClient（推荐度低于 ElasticsearchRestTemplate，因为 API 不够高级）
-   TransportClient（ES7 中废弃，不建议使用）

### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#使用ElasticsearchRest "使用ElasticsearchRest")使用 ElasticsearchRest

1. 参数配置：
```java
elasticsearch.address=127.0.0.1:9200
```


2. 配置引入：
    
    ```java
    import org.elasticsearch.client.RestHighLevelClient;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.data.elasticsearch.client.ClientConfiguration;
    import org.springframework.data.elasticsearch.client.RestClients;
    import org.springframework.data.elasticsearch.core.ElasticsearchRestTemplate;
    
    @Configuration
    public class ElasticSearchConfig {
    
        @Value("${elasticsearch.address}")
        private String elasticSearchAddress;
    
        @Bean
        RestHighLevelClient elasticsearchClient() {
            final ClientConfiguration configuration = ClientConfiguration.builder()
                    .connectedTo(elasticSearchAddress)
                    .build();
            RestHighLevelClient client = RestClients.create(configuration).rest();
            return client;
        }
    
        @Bean
        ElasticsearchRestTemplate elasticsearchTemplate() {
            return new ElasticsearchRestTemplate(elasticsearchClient());
        }
    }
    ```
    
    或者：
    
    ```java
    @Bean
    RestHighLevelClient elasticsearchHighLevelClient() {
        RestHighLevelClient client=new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("127.0.0.1", 9200, "http")
                )
        );
        return client;
    }
    ```
    
3.  使用：
    
    ```java
    @Autowired
    ElasticsearchRestTemplate elasticsearchTemplate;
    ```
    

### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#索引示例 "索引示例")索引示例

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#创建索引 "创建索引")创建索引

```java
Map<String, Object> settings = new HashMap<>();
settings.put("index.number_of_replicas", "30");
boolean createResult = elasticsearchTemplate.createIndex(clazz,settings);
```

刷新 Mapping：

```java
elasticsearchTemplate.putMapping(clazz);
elasticsearchTemplate.refresh(clazz);
```

虽然可以正常创建索引，但会提示 `putMapping` 和 `refresh` 方法过期，此时可以使用：

```java
IndexOperations createResult = elasticsearchTemplate.indexOps(clazz);
boolean create = createResult.create();
boolean putMapping = createResult.putMapping();
```

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#删除索引 "删除索引")删除索引

```java
elasticsearchTemplate.deleteIndex(clazz);
```

或者：

```java
elasticsearchTemplate.indexOps(clazz).delete();
```

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#查看索引是否存在 "查看索引是否存在")查看索引是否存在

```none
elasticsearchTemplate.indexExists(indexName);
```

或者：

```none
elasticsearchTemplate.indexOps(clazz).exists();
```

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#获取索引Mapping "获取索引Mapping")获取索引 Mapping

```java
IndexOperations createResult = elasticsearchTemplate.indexOps(clazz);
return createResult.createMapping().toJson();
```

### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#数据示例 "数据示例")数据示例

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#新增数据 "新增数据")新增数据

```java
List<IndexQuery> queries = new ArrayList<IndexQuery>();
for(Person test:testList) {
    IndexQuery indexQuery = new IndexQueryBuilder().withId(test.getId()).withObject(test).build();
    queries.add(indexQuery);
}
elasticsearchTemplate.bulkIndex(queries, Person.class);
```

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#查询数据 "查询数据")查询数据

```java
GetQuery query = new GetQuery(id);
Person info = elasticsearchTemplate.queryForObject(query, Person.class);
```

#### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#聚合数据 "聚合数据")聚合数据

简化版：

```java
NativeSearchQuery query = new NativeSearchQueryBuilder()
        .addAggregation(AggregationBuilders.terms("agg_count").field("module.keyword"))
        .build();

SearchHits<PhishingLog> searchHits = elasticsearchTemplate.search(query, PhishingLog.class);
//取出聚合结果
Aggregations aggregations = searchHits.getAggregations();
Terms terms = (Terms) aggregations.asMap().get("agg_count");

for (Terms.Bucket bucket : terms.getBuckets()) {
    String keyAsString = bucket.getKeyAsString();   // 聚合字段列的值
    long docCount = bucket.getDocCount();           // 聚合字段对应的数量
    System.out.println(keyAsString + " " + docCount);
}
```

详细版：

需要格外注意类型转换，如 ParsedStringTerms、ParsedLongTerms 等。

```java
// 关键词条件筛选
BoolQueryBuilder bool = new BoolQueryBuilder();
bool.must(QueryBuilders.rangeQuery("@timestamp").from(start.getTime()));
bool.must(QueryBuilders.rangeQuery("@timestamp").to(end.getTime()));

// 分组。terms分组名称、field分组字段、size分组数量
TermsAggregationBuilder aggregationBuilderGroupBy = AggregationBuilders.terms("agg_count").field("module.keyword").size(200);

// 组合查询
NativeSearchQuery searchQuery = new NativeSearchQueryBuilder().addAggregation(aggregationBuilderGroupBy).withQuery(bool).build();

// 查询。实体类上需要有Doucment注解
SearchHits<PhishingLog> searchHits = elasticsearchTemplate.search(searchQuery, PhishingLog.class);

// 解析
Aggregations aggPage = searchHits.getAggregations();
Aggregation aggregation = aggPage.get("agg_count");
// 因为是利用String类型字段来进行的term聚合，所以结果要强转为 ParsedStringTerms 类型
List<? extends Terms.Bucket> buckets = ((ParsedStringTerms) aggregation).getBuckets();
int total = buckets.size();
for (int index = 0; index < total; index++) {
    Terms.Bucket bucket = buckets.get(index);
    ModuleDto dto = new ModuleDto();
    dto.setName(bucket.getKeyAsString());
    if(searchLevel) {
        ParsedStringTerms aggregationsTemp = bucket.getAggregations().get("get_level");
        ParsedLongTerms aggregationsTemp2 = bucket.getAggregations().get("get_time");
        for (Terms.Bucket levelBucket : aggregationsTemp.getBuckets()) {
            if("error".equalsIgnoreCase(levelBucket.getKeyAsString())) {
                // 这个 getDocCount 是每组的数量
                dto.setErrorNum((int)levelBucket.getDocCount());
            }
            if("info".equalsIgnoreCase(levelBucket.getKeyAsString())) {
                dto.setInfoNum((int)levelBucket.getDocCount());
            }
            if("debug".equalsIgnoreCase(levelBucket.getKeyAsString())) {
                dto.setDebugNum((int)levelBucket.getDocCount());
            }
        }
        for (Terms.Bucket levelBucket : aggregationsTemp2.getBuckets()) {
            dto.setUpdateTime(new DateTime(levelBucket.getKeyAsString()).toDate());
        }

    }
    list.add(dto);
}
```

### [](https://jueee.github.io/2021/02/2021-02-07-ElasticSearch%E9%9B%86%E6%88%90Spring%E4%B9%8BElasticsearchRestTemplate%E7%A4%BA%E4%BE%8B/#参考资料 "参考资料")参考资料

-   [https://blog.csdn.net/qq_45071180/article/details/122702830](https://blog.csdn.net/qq_45071180/article/details/122702830)

相关文章

-   [curl 命令行修改 ElasticSearch 配置](https://jueee.github.io/2020/12/2020-12-25-curl命令行修改ElasticSearch配置/)
    
-   [ElasticSearch Spring-Data Date format always is long](https://jueee.github.io/2021/02/2021-02-19-ElasticSearch Spring-Data Date format always is long/)
    
-   [SpringBoot 配置多个 Elasticsearch 集群](https://jueee.github.io/2022/01/2022-01-18-SpringBoot 配置多个 Elasticsearch 集群/)
    
-   [ElasticsearchRestTemplate 查询 API 示例](https://jueee.github.io/2021/02/2021-02-06-ElasticsearchRestTemplate查询API示例/)
    
-   [ElasticSearch 设置用户名密码](https://jueee.github.io/2022/04/2022-04-18-ElasticSearch设置用户名密码/)