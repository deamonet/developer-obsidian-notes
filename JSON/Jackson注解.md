 [jackson](https://www.xiehai.zone/tags.html?tag=jackson) [java](https://www.xiehai.zone/tags.html?tag=java) 248 2023-05-04
## 注解分类[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#%E6%B3%A8%E8%A7%A3%E5%88%86%E7%B1%BB)

Jackson的注解主要在[jackson-annotations](https://github.com/FasterXML/jackson-annotations) 及[jackson-databind](https://github.com/FasterXML/jackson-databind)中，主要分为以下几种类型

|类型|作用|示例|
|---|---|---|
|序列化、反序列化注解|同时影响序列化和反序列化过程|@JsonIgnore、@JsonProperty|
|仅序列化注解|仅影响对象序列化过程|@JsonGetter、@JsonValue|
|仅反序列化注解|仅影响json反序列化对象过程|@JonsSetter、@JsonCreator|
|元注解|用于将多个注解组合为一个容器注解的元注解或仅有标记作用的注解|@JacksonAnnotationsInside|

## 序列化、反序列化注解[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#%E5%BA%8F%E5%88%97%E5%8C%96%E3%80%81%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%B3%A8%E8%A7%A3)

序列化、反序列化注解有的是一个注解就同时影响序列化与反序列化，有的是一对注解分别来分别控制。

### @JsonAnyGetter/@JsonAnySetter[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonanygetter-jsonanysetter)

- [@JsonAnyGetter](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonAnyGetter.java)：将带有JsonAnyGetter注解的方法返回map序列化为json的其他属性(平铺)
    - 一个bean中`只能有一个属性/方法`带有@JsonAnyGetter注解，多个会报错
    - 方法`只能返回Map类型`
- [@JsonAnySetter](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonAnySetter.java)：将json中bean不能匹配的属性使用注解方法、pojo字段或map类型字段反序列化
    - 一个bean中`只能有一个属性/方法`带有@JsonAnySetter注解，多个会报错

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonAutoDetect[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonautodetect)

[@JsonAutoDetect](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonAutoDetect.java)可以在序列化、反序列化时包含不可访问的属性，即使没有getter、setter，**需要bean有默认构造方法**。

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonValue/@JsonCreator[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonvalue-jsoncreator)

- [@JsonValue](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonValue.java)：使用单个方法/字段序列化对象
    - 一个bean中`只能有一个属性/方法`带有@JsonValue注解，多个会报错
- [@JsonCreator](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonCreator.java)：指定反序列化使用的构造方法或工厂方法，如果是对象类型可以结合@JsonProperty使用

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonFormat[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonformat)

[@JsonFormat](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonFormat.java)用于指定任意类型序列化和反序列化格式，常用于日期(Date**不是java8的time**)、枚举。

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonGetter/@JsonSetter[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsongetter-jsonsetter)

- [@JsonGetter](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonGetter.java): 指定一个非静态、无参、返回值非void的方法为序列化的getter方法
- [@JsonSetter](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonSetter.java): 指定一个非静态、单个参数的方法为反序列化的setter方法

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonIgnore[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonignore)

[@JsonIgnore](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonIgnore.java)用于忽略序列化及反序列化，可将注解放在字段、setter、getter上都可以达到忽略的目的。

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonIgnoreProperties[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonignoreproperties)

[@JsonIgnoreProperties](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonIgnoreProperties.java)作用于class用于忽略多个字段的序列化与反序列化

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonIgnoreType[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonignoretype)

[@JsonIgnoreType](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonIgnoreType.java)用于标记某个类型忽略其序列化与反序列化

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonProperty[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonproperty)

[@JsonProperty](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonProperty.java) 用于标记非静态方法、字段作为json属性

- 标记属性时，表示序列化、反序列化都使用给定属性
- 标记getter方法时，表示序列化使用给定属性
- 标记setter方法时，表示反序列化使用给定属性
- 标记枚举实例时，表示序列化、反序列化枚举都使用给定属性

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonUnwrapped[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonunwrapped)

[@JsonUnwrapped](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonUnwrapped.java) 在序列化时将嵌套对象属性放到父对象，反序列化时将父对象属性解析到嵌套对象属性，支持属性的前缀、后缀，但扩展的前缀不支持驼峰

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonSerialize/@JsonDeserialize[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonserialize-jsondeserialize)

- [@JsonSerialize](https://github.com/FasterXML/jackson-databind/blob/2.16/src/main/java/com/fasterxml/jackson/databind/annotation/JsonSerialize.java)用于自定义序列化，针对特定类型作为属性、map的key、map的value定义
- [@JsonDeserialize](https://github.com/FasterXML/jackson-databind/blob/2.16/src/main/java/com/fasterxml/jackson/databind/annotation/JsonDeserialize.java)用于自定义反序列化，针对特定类型作为属性、map的key、map的value定义

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

## 仅序列化注解[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#%E4%BB%85%E5%BA%8F%E5%88%97%E5%8C%96%E6%B3%A8%E8%A7%A3)

### @JsonInclude[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsoninclude)

[@JsonInclude](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonInclude.java) 用与在序列化时排除一些具有`NULL`/`EMPTY`/`DEFAULT`/自定义条件的属性

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonIncludeProperties[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonincludeproperties)

[@JsonIncludeProperties](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonIncludeProperties.java) 用于一个类型序列化时仅包含给定的属性

- 如果使用了@JsonInclude，结果是@JsonIncludeProperties属性列表的子集

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  
  

### @JsonKey[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonkey)

[@JsonKey](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonKey.java) 用于当对象作为map的key时，使用标记的字段作为key

- 一个对象中，`只能有一个`@JsonKey注解在字段或者getter上，多个会报错。

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonPropertyOrder[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonpropertyorder)

[@JsonPropertyOrder](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonPropertyOrder.java) 用于控制序列化时属性顺序

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonRawValue[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonrawvalue)

[@JsonRawValue](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonRawValue.java) 按照属性原始值直接序列化，不做任何改变

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

## 仅反序列化注解[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#%E4%BB%85%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%B3%A8%E8%A7%A3)

### @JsonAlias[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonalias)

[@JsonAlias](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonAlias.java) 用于一个属性可以使用一个或多个别名反序列化，若json中存在多个别名属性，按照json读取顺序覆盖，即使是null

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonMerge[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonmerge)

[@JsonMerge](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonMerge.java) 用于在反序列化时，若属性本身非空，将json字符串的值和已有值做合并

- 基础类型不做合并
- 普通对象，按照属性依次合并
- map类型，按照entry依次合并
- 集合类型，追加输入元素合并
- 数组类型，创建新的数组追加输入元素合并

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  

### @JsonPOJOBuilder[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonpojobuilder)

[@JsonPOJOBuilder](https://github.com/FasterXML/jackson-databind/blob/2.16/src/main/java/com/fasterxml/jackson/databind/annotation/JsonPOJOBuilder.java) 用于序列化时使用构造器来实例化，lombok的[`@Jacksonized`](https://projectlombok.org/features/experimental/Jacksonized)注解就是通过这个注解实现的

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

## 特殊注解[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#%E7%89%B9%E6%AE%8A%E6%B3%A8%E8%A7%A3)

一般不常用或需要搭配[ObjectMapper](https://github.com/FasterXML/jackson-databind/blob/2.16/src/main/java/com/fasterxml/jackson/databind/ObjectMapper.java)配置使用，也可能需要多个注解搭配使用。

### @JsonEnumDefaultValue[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonenumdefaultvalue)

[@JsonEnumDefaultValue](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonEnumDefaultValue.java) 用于反序列化枚举时值未知时，使用标记的枚举实例作为默认值

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JacksonInject[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jacksoninject)

[@JacksonInject](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JacksonInject.java) 用于在反序列化时注入属性值，值来源并非json

- 若以class注入，则注入值名称为class的全名，否则名称就是注入名
- JacksonInject不指定名称，则默认为属性字段class为名称

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  

### @JsonManagedReference、@JsonBackReference[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonmanagedreference%E3%80%81-jsonbackreference)

用于解决属性与属性之间相互依赖，避免出现序列化死循环导致SOE(StackOverflowError)

- [@JsonManagedReference](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonManagedReference.java)用于标记引用父对象
- [@JsonBackReference](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonBackReference.java)用于标记引用子对象

如何区分属性是用@JsonManagedReference标记还是@JsonBackReference标记呢?取决于属性之间的归属关系，即按照序列化顺序最开始 出现的属性使用@JsonManagedReference标记，后出现的属性使用@JsonBackReference标记。

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonClassDescription、@JsonPropertyDescription[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonclassdescription%E3%80%81-jsonpropertydescription)

用于描述json schema，非用于json的序列化与反序列化

- [@JsonClassDescription](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonClassDescription.java)类型描述（字面意思，实际好像并未生效）
- [@JsonPropertyDescription](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonPropertyDescription.java)属性描述

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  

### @JsonFilter[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonfilter)

[@JsonFilter](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonFilter.java) 用于属性过滤自定义过滤器，参考[SimpleBeanPropertyFilter](https://github.com/FasterXML/jackson-databind/blob/2.16/src/main/java/com/fasterxml/jackson/databind/ser/impl/SimpleBeanPropertyFilter.java)

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  

### @JsonIdentityInfo、@JsonIdentityReference[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonidentityinfo%E3%80%81-jsonidentityreference)

若同一个对象重复出现时使用指定的属性来序列化对象，而不是序列化整个对象，一般需要两个注解配合使用。

当alwaysAsId为false时类似@JsonManagedReference、@JsonBackReference来解决循环依赖问题

当alwaysAsId为true时，类组合时只想序列化其id而不序列化整个对象，用来简化序列化的内容大小

- [@JsonIdentityInfo](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonIdentityInfo.java)指定序列化方式及属性名称
- [@JsonIdentityReference](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonIdentityReference.java)

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonRootName[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonrootname)

[@JsonRootName](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonRootName.java) 在序列化时添加根节点，只能在类型上添加。

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonAppend[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonappend)

[@JsonAppend](https://github.com/FasterXML/jackson-databind/blob/2.16/src/main/java/com/fasterxml/jackson/databind/annotation/JsonAppend.java) 在序列化时增加额外属性，可以自定义增加，或由mapper序列化时追加

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  

### @JsonNaming[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonnaming)

[@JsonNaming](https://github.com/FasterXML/jackson-databind/blob/2.16/src/main/java/com/fasterxml/jackson/databind/annotation/JsonNaming.java) 用于在序列化及反序列化时指定命名方式，支持LOWER_CAMEL_CASE、UPPER_CAMEL_CASE、SNAKE_CASE、LOWER_CASE、KEBAB_CASE、LOWER_DOT_CASE

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  

### @JsonTypeId、@JsonTypeInfo、@JsonSubTypes、@JsonTypeName[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsontypeid%E3%80%81-jsontypeinfo%E3%80%81-jsonsubtypes%E3%80%81-jsontypename)

实现反序列化时通过父类反序列化为子类即反序列化的多态，可以通过属性、包装属性或者推断的方式实现

- [@JsonTypeId](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonTypeId.java)标记某个字段为推断标识
- [@JsonTypeName](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonTypeName.java)用在子类上，标记子类推断标识
- [@JsonTypeInfo](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonTypeInfo.java)用于定义及声明推断子类的方式
- [@JsonSubTypes](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonSubTypes.java)用于定义支持推断的子类集合

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  
  
  
  
  

### @JsonView[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jsonview)

[@JsonView](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JsonView.java) 用于序列化分组，按照不同的组序列化不同结果，类似hibernate validator的group

代码示例

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  

  
  
  

## 元注解[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#%E5%85%83%E6%B3%A8%E8%A7%A3)

主要用于标记注解或类为jackson实现，或自定义组合注解

### @JacksonAnnotation[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jacksonannotation)

[@JacksonAnnotation](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JacksonAnnotation.java) 仅用来标记注解是jackson注解，将来可能会用来传递其他泛型注解配置，表名标记的注解是jackson注解的一部分

### @JacksonStdImpl[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jacksonstdimpl)

[@JacksonStdImpl](https://github.com/FasterXML/jackson-databind/blob/2.16/src/main/java/com/fasterxml/jackson/databind/annotation/JacksonStdImpl.java) 仅用来标记实现类(序列化器、反序列化器等)是jackson官方标准实现

### @JacksonAnnotationsInside[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#jacksonannotationsinside)

[@JacksonAnnotationsInside](https://github.com/FasterXML/jackson-annotations/blob/2.16/src/main/java/com/fasterxml/jackson/annotation/JacksonAnnotationsInside.java) 用于将多个jackson注解组合成一个jackson识别的用户自定义注解。

比如，现在我要通过jackson序列化实现一个关键字脱敏功能

代码示例

java

```
public class Test {
    @FieldDefaults(level = AccessLevel.PRIVATE, makeFinal = true)
    @RequiredArgsConstructor(access = AccessLevel.PRIVATE)
    public static class SensitiveDataSerializer extends JsonSerializer<String> implements ContextualSerializer {
        JsonSensitive sensitive;

        public SensitiveDataSerializer() {
            this.sensitive = null;
        }

        @Override
        public void serialize(String value, JsonGenerator gen, SerializerProvider provider) throws IOException {
            if (Objects.nonNull(this.sensitive)) {
                gen.writeString(this.sensitive.value().sensitize(value));
            }
            // 当脱敏注解为空时 兼容默认构造方法问题
        }

        @Override
        public JsonSerializer<?> createContextual(SerializerProvider provider, BeanProperty property)
            throws JsonMappingException {
            // 查找字段或者方法上是否存在注解
            JsonSensitive sensitive =
                Optional.ofNullable(property.getAnnotation(JsonSensitive.class))
                    .orElseGet(() -> property.getContextAnnotation(JsonSensitive.class));
            if (Objects.nonNull(sensitive)) {
                return new SensitiveDataSerializer(sensitive);
            }

            return provider.findValueSerializer(property.getType(), property);
        }
    }

    public enum SensitiveType {
        /**
         * 前缀姓名脱敏
         * 张三 *三
         * 王丽莎 *丽莎
         * 蒙娜丽莎 *丽莎
         */
        PREFIX_NAME {
            @Override
            protected String doSensitize(String s, int len) {
                char[] chars = s.toCharArray();
                chars[0] = SYMBOL;

                return new String(chars);
            }
        },
        /**
         * 中缀姓名脱敏
         * 张三 张*
         * 王丽莎 王*莎
         * 蒙娜丽莎 蒙*莎
         */
        INFIX_NAME {
            /**
             * 名字长度2
             */
            protected static final int NAME_2 = 2;

            @Override
            protected String doSensitize(String s, int len) {
                char[] chars = s.toCharArray();
                IntStream.range(1, len - 1).forEach(it -> chars[it] = SYMBOL);

                if (len == NAME_2) {
                    chars[1] = SYMBOL;
                }

                return new String(chars);
            }
        },
        /**
         * 后缀姓名脱敏
         * 张三 张*
         * 王丽莎 王**
         * 蒙娜丽莎 蒙**
         */
        SUFFIX_NAME {
            @Override
            protected String doSensitize(String s, int len) {
                char[] chars = s.toCharArray();
                IntStream.range(1, len).forEach(it -> chars[it] = SYMBOL);

                return new String(chars);
            }
        },
        /**
         * 身份证号码 脱敏出生年月日
         * 511123199901011234 511123********1234
         */
        IDENTITY_CARD_NUMBER {
            @Override
            protected String doSensitize(String s, int len) {
                // 身份证号码脱敏年月日长度8 开始脱敏位置为6
                return SensitiveType.sensitize(s, len, 8, 6);
            }
        },
        /**
         * 手机号码 脱敏中间4位
         * 13512345678 135****5678
         */
        MOBILE_NUMBER {
            @Override
            protected String doSensitize(String s, int len) {
                // 手机号脱敏中间4位 开始脱敏位置为3
                return SensitiveType.sensitize(s, len, 4, 3);
            }
        };

        /**
         * 脱敏符号
         */
        protected static final char SYMBOL = '*';

        /**
         * 数据脱敏
         * @param s   待脱敏数据
         * @param len 待脱敏数据长度
         * @return 脱敏后数据
         */
        protected abstract String doSensitize(String s, int len);

        /**
         * 数据脱敏
         * @param s 待脱敏数据
         * @return 脱敏后数据
         */
        String sensitize(String s) {
            // 空字符串不做脱敏
            if (StringUtils.isEmpty(s)) {
                return s;
            }

            return this.doSensitize(s, s.length());
        }

        /**
         * 连续脱敏工具方法
         * @param s              待脱敏字符串
         * @param len            字符串长度
         * @param sensitizeLen   脱敏长度
         * @param sensitizeStart 开始脱敏位置
         * @return 脱敏后字符串
         */
        static String sensitize(String s, int len, int sensitizeLen, int sensitizeStart) {
            // 小于等于脱敏长度 不做脱敏 兼容异常数据
            if (len <= sensitizeLen) {
                return s;
            }

            // 剩余不需要隐藏的长度
            int left = len - sensitizeLen;
            char[] chars = s.toCharArray();
            // 
            if (left <= sensitizeStart) {
                // 保证首尾不被脱敏
                int start = (left >> 1) + (len & 1);
                IntStream.range(0, sensitizeLen).forEach(it -> chars[start + it] = SYMBOL);
            } else {
                IntStream.range(0, sensitizeLen).forEach(it -> chars[sensitizeStart + it] = SYMBOL);
            }

            return new String(chars);
        }
    }

    @Target({ElementType.FIELD, ElementType.METHOD})
    @Retention(RetentionPolicy.RUNTIME)
    @JacksonAnnotationsInside
    @JsonSerialize(using = SensitiveDataSerializer.class)
    @Documented
    public @interface JsonSensitive {
        /**
         * 脱敏数据类型
         * @return {@link SensitiveType}
         */
        SensitiveType value();
    }


    @FieldDefaults(level = AccessLevel.PRIVATE)
    @Data
    static class Person {
        @JsonSensitive(SensitiveType.IDENTITY_CARD_NUMBER)
        String id;
        @JsonSensitive(SensitiveType.INFIX_NAME)
        String name;
        @JsonSensitive(SensitiveType.MOBILE_NUMBER)
        String phone;
    }

    public static void main(String[] args) throws JsonProcessingException {
        ObjectMapper mapper = new JsonMapper();
        Person person = new Person();
        person.setId("510101199001010681");
        person.setName("欧阳锋");
        person.setPhone("13888888888");

        System.out.println(mapper.writeValueAsString(person));
    }
}
```

1  
2  
3  
4  
5  
6  
7  
8  
9  
10  
11  
12  
13  
14  
15  
16  
17  
18  
19  
20  
21  
22  
23  
24  
25  
26  
27  
28  
29  
30  
31  
32  
33  
34  
35  
36  
37  
38  
39  
40  
41  
42  
43  
44  
45  
46  
47  
48  
49  
50  
51  
52  
53  
54  
55  
56  
57  
58  
59  
60  
61  
62  
63  
64  
65  
66  
67  
68  
69  
70  
71  
72  
73  
74  
75  
76  
77  
78  
79  
80  
81  
82  
83  
84  
85  
86  
87  
88  
89  
90  
91  
92  
93  
94  
95  
96  
97  
98  
99  
100  
101  
102  
103  
104  
105  
106  
107  
108  
109  
110  
111  
112  
113  
114  
115  
116  
117  
118  
119  
120  
121  
122  
123  
124  
125  
126  
127  
128  
129  
130  
131  
132  
133  
134  
135  
136  
137  
138  
139  
140  
141  
142  
143  
144  
145  
146  
147  
148  
149  
150  
151  
152  
153  
154  
155  
156  
157  
158  
159  
160  
161  
162  
163  
164  
165  
166  
167  
168  
169  
170  
171  
172  
173  
174  
175  
176  
177  
178  
179  
180  
181  
182  
183  
184  
185  
186  
187  
188  
189  
190  
191  
192  
193  
194  
195  
196  
197  
198  
199  
200  
201  
202  
203  

输出结果

console

```
{"id":"510101********0681","name":"欧*锋","phone":"138****8888"}
```

1  

## 总结[​](https://www.xiehai.zone/2023-05-04-jackson-annotations.html#%E6%80%BB%E7%BB%93)

以上列出了jackson的全部注解，工作中绝大部分可能使用不上，只是作为了解以及某些特殊情况下的json处理能够多一种快速解决方案。