# 目录

[TOC]



# 前言



# JSON



# JACKSON

**本文档以2.12.1版本为基准。**

Jackson 的核心模块由三部分组成。

- jackson-core，核心包，提供基于"流模式"解析的相关 API，它包括 JsonPaser 和 JsonGenerator。Jackson 内部实现正是通过高性能的流模式 API 的 JsonGenerator 和 JsonParser 来生成和解析 json。
- jackson-annotations，注解包，提供标准注解功能；
- jackson-databind ，数据绑定包， 提供基于"对象绑定" 解析的相关 API （ ObjectMapper ） 和"树模型" 解析的相关 API （JsonNode）；基于"对象绑定" 解析的 API 和"树模型"解析的 API 依赖基于"流模式"解析的 API。



## 序列化和反序列化

JackSon 默认不是所有的属性都可以被序列化和反序列化。默认的属性可视化的规则如下：

- 若该属性修饰符是 public，该属性可序列化和反序列化。
- 若属性的修饰符不是 public，但是它的 getter 方法和 setter 方法是 public，该属性可序列化和反序列化。因为 getter 方法用于序列化， 而 setter 方法用于反序列化。
- 若属性只有 public 的 setter 方法，而无 public 的 getter 方 法，该属性只能用于反序列化。

若想更改默认的属性可视化的规则，需要调用 ObjectMapper 的方法 `setVisibility`。



### 序列化特性

通过设置`SerializationFeature`枚举类来配置序列化特性。

| SerializationFeature                 | 描述                         |
| ------------------------------------ | ---------------------------- |
| WRAP_ROOT_VALUE                      |                              |
| INDENT_OUTPUT                        |                              |
| FAIL_ON_EMPTY_BEANS                  |                              |
| FAIL_ON_SELF_REFERENCES              |                              |
| WRAP_EXCEPTIONS                      |                              |
| FAIL_ON_UNWRAPPED_TYPE_IDENTIFIERS   |                              |
| WRITE_SELF_REFERENCES_AS_NULL        |                              |
| CLOSE_CLOSEABLE                      |                              |
| FLUSH_AFTER_WRITE_VALUE              |                              |
| WRITE_DATES_AS_TIMESTAMPS            | 将日期类型序列化为毫秒时间戳 |
| WRITE_DATE_KEYS_AS_TIMESTAMPS        |                              |
| WRITE_DATES_WITH_ZONE_ID             |                              |
| WRITE_DURATIONS_AS_TIMESTAMPS        |                              |
| WRITE_CHAR_ARRAYS_AS_JSON_ARRAYS     |                              |
| WRITE_ENUMS_USING_TO_STRING          |                              |
| WRITE_ENUMS_USING_INDEX              |                              |
| WRITE_ENUM_KEYS_USING_INDEX          |                              |
| WRITE_SINGLE_ELEM_ARRAYS_UNWRAPPED   |                              |
| WRITE_DATE_TIMESTAMPS_AS_NANOSECONDS | 将日期类型序列化为纳秒时间戳 |
| ORDER_MAP_ENTRIES_BY_KEYS            |                              |
| EAGER_SERIALIZER_FETCH               |                              |
| USE_EQUALITY_FOR_OBJECT_ID           |                              |



### 反序列化特性

通过设置`DeserializationFeature`枚举类来配置反序列化特性。

| DeserializationFeature                       | 描述                   |
| -------------------------------------------- | ---------------------- |
| USE_BIG_DECIMAL_FOR_FLOATS                   |                        |
| USE_BIG_INTEGER_FOR_INTS                     |                        |
| USE_LONG_FOR_INTS                            |                        |
| USE_JAVA_ARRAY_FOR_JSON_ARRAY                |                        |
| FAIL_ON_UNKNOWN_PROPERTIES                   | 遇到未知属性则抛出异常 |
| FAIL_ON_NULL_FOR_PRIMITIVES                  |                        |
| FAIL_ON_NUMBERS_FOR_ENUMS                    |                        |
| FAIL_ON_INVALID_SUBTYPE                      |                        |
| FAIL_ON_READING_DUP_TREE_KEY                 |                        |
| FAIL_ON_IGNORED_PROPERTIES                   |                        |
| FAIL_ON_UNRESOLVED_OBJECT_IDS                |                        |
| FAIL_ON_MISSING_CREATOR_PROPERTIES           |                        |
| FAIL_ON_NULL_CREATOR_PROPERTIES              |                        |
| FAIL_ON_MISSING_EXTERNAL_TYPE_ID_PROPERTY    |                        |
| FAIL_ON_TRAILING_TOKENS                      |                        |
| WRAP_EXCEPTIONS                              |                        |
| ACCEPT_SINGLE_VALUE_AS_ARRAY                 |                        |
| UNWRAP_SINGLE_VALUE_ARRAYS                   |                        |
| UNWRAP_ROOT_VALUE                            |                        |
| ACCEPT_EMPTY_STRING_AS_NULL_OBJECT           |                        |
| ACCEPT_EMPTY_ARRAY_AS_NULL_OBJECT            |                        |
| ACCEPT_FLOAT_AS_INT                          |                        |
| READ_ENUMS_USING_TO_STRING                   |                        |
| READ_UNKNOWN_ENUM_VALUES_AS_NULL             |                        |
| READ_UNKNOWN_ENUM_VALUES_USING_DEFAULT_VALUE |                        |
| READ_DATE_TIMESTAMPS_AS_NANOSECONDS          |                        |
| ADJUST_DATES_TO_CONTEXT_TIME_ZONE            |                        |
| EAGER_DESERIALIZER_FETCH                     |                        |



### 特殊属性的序列化和反序列化

#### 普通日期类型

对于日期类型为 `java.util.Calendar`, `java.util.GregorianCalendar`, `java.sql.Date`, `java.util.Date`, `java.sql.Timestamp`，若不指定格式，在 json 文件中将序列化为 `long` 类型的数据。

- 使用 `@JsonFormat` 注解指定日期格式。
- 调用 ObjectMapper 的方法 `setDateFormat`，将序列化为指定格式的 string 类型的数据。



#### Local日期类型

对于日期类型为 java.time.LocalDate, java.time.LocalDateTime，还需要添加代码 `mapper.registerModule(new JavaTimeModule())`，同时添加相应的依赖 jar 包。

```xml
<dependency>
    <groupId>com.fasterxml.jackson.datatype</groupId>
    <artifactId>jackson-datatype-jsr310</artifactId>
</dependency>
```

```java
JavaTimeModule javaTimeModule = new JavaTimeModule();
javaTimeModule.addSerializer(LocalDateTime.class,
    new LocalDateTimeSerializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
javaTimeModule.addDeserializer(LocalDateTime.class,
    new LocalDateTimeDeserializer(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));
objectMapper.registerModule(javaTimeModule);
```



#### List类型



#### Map类型



## 注解

### @JsonCreator

用于构造方法，和 @JsonProperty 配合使用，适用有参数的构造方法。



### @JsonFormat

用于属性、方法，把属性的格式序列化时转换成指定的格式。



### @JsonIgnore

用于字段、getter/setter、构造函数参数上，对相应的字段产生影响。使相应字段不参与序列化和反序列化。



### @JsonInclude



### @JsonIgnoreProperties

用于类，使指定属性不参与序列化和反序列化。



### @JsonNaming

用于类，序列化和反序列化时指定属性命名规则。



### @JsonProperty

用于属性，把属性的名称序列化和反序列化时转换为另外一个名称。

```java
@JsonProperty("string")
private String stringValue;
```



### @JsonPropertyOrder

用于类， 和 @JsonProperty 的index属性类似，指定属性在序列化时 json 中的顺序。



### @JsonRootName

用于类，需开启`SerializationFeature.WRAP_ROOT_VALUE`，用于序列化时输出带有根属性名称的 JSON 串。但不支持该 JSON 串反序列化。

```java
@Data
@JsonRootName(value = "jsonObject")
public class JsonObject {
    ...
}
```


```json
{
  "jsonObject" : {
	
  }
}
```

### @JsonSerialize
```java
@Target({ElementType.ANNOTATION_TYPE, ElementType.METHOD, ElementType.FIELD, ElementType.TYPE, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
@com.fasterxml.jackson.annotation.JacksonAnnotation
public @interface JsonSerialize
{
    // // // Annotations for explicitly specifying deserializer
    
    @SuppressWarnings("rawtypes") // to work around JDK8 bug wrt Class-valued annotation properties
    public Class<? extends JsonSerializer> using() default JsonSerializer.None.class;
    
    @SuppressWarnings("rawtypes") // to work around JDK8 bug wrt Class-valued annotation properties
    public Class<? extends JsonSerializer> contentUsing()
        default JsonSerializer.None.class;
    
    @SuppressWarnings("rawtypes") // to work around JDK8 bug wrt Class-valued annotation properties
    public Class<? extends JsonSerializer> keyUsing()
        default JsonSerializer.None.class;
    
    @SuppressWarnings("rawtypes") // to work around JDK8 bug wrt Class-valued annotation properties
    public Class<? extends JsonSerializer> nullsUsing()
        default JsonSerializer.None.class;

    // // // Annotations for type handling, explicit declaration
    // // // (type used for choosing deserializer, if not explicitly
    // // // specified)
    
    public Class<?> as() default Void.class;
    
    public Class<?> keyAs() default Void.class;
    
    public Class<?> contentAs() default Void.class;
    
    public com.fasterxml.jackson.databind.annotation.JsonSerialize.Typing typing() default com.fasterxml.jackson.databind.annotation.JsonSerialize.Typing.DEFAULT_TYPING;

    // // // Annotations for specifying intermediate Converters (2.2+)
    
    @SuppressWarnings("rawtypes") // to work around JDK8 bug wrt Class-valued annotation properties
    public Class<? extends Converter> converter() default Converter.None.class;
    
    @SuppressWarnings("rawtypes") // to work around JDK8 bug wrt Class-valued annotation properties
    public Class<? extends Converter> contentConverter() default Converter.None.class;

    // // // Annotation(s) for inclusion criteria
    
    @Deprecated
    public com.fasterxml.jackson.databind.annotation.JsonSerialize.Inclusion include() default com.fasterxml.jackson.databind.annotation.JsonSerialize.Inclusion.DEFAULT_INCLUSION;

    @Deprecated // since 2.0, marked deprecated in 2.6
    public enum Inclusion
    {
        ALWAYS,
        NON_NULL,
        NON_DEFAULT,
        NON_EMPTY,
        DEFAULT_INCLUSION
        ;
    }
   
    public enum Typing
    {
        DYNAMIC,
        STATIC,
        DEFAULT_TYPING
        ;
    }
}
```

```java
public class Json {
    /**
     * 设置属性的序列化方式
     */
    @JsonSerialize(using = NumberSerializers.IntLikeSerializer.class)
    private Integer group;

    /**
     * 设置list中元素的序列化方式
     */
    @JsonSerialize(contentUsing = NumberSerializers.IntLikeSerializer.class)
    private List<Integer> groups;
}
```

## 功能

### 属性可视化

### 属性过滤

### 自定义序列化

### 树模型处理

## 参考文档

- [史上最全的Jackson框架使用教程](https://my.oschina.net/u/4606167/blog/4518138)



# FASTJSON



