[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)是一个 [MyBatis](http://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

```xml
<configuration>
  <settings>
    ...
    <setting name="logImpl" value="LOG4J"/>
    ...
  </settings>
</configuration>
```



## 特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑

- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作

- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求

- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错

- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题

- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作

- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）

- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用

- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询

- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库

- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询

- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

  

|      |      |      |
| ---- | ---- | ---- |
|      |      |      |
|      |      |      |
|      |      |      |



```json
{
    "id": "1041",
    "isUndrag": 0,
    "openCer": 1,
    "cerId": "123",
    "projectRuleCer": {
        "openMaterialLiveRule": 1,
        "materialLiveCompleteNum": 1,
        "openExamRule": 1,
        "examCompleteNum": 0,
        "deliverType": 2,
        "deliverTime": "2020-12-04 17:02:28"
    }
}
```



```java
public class Disk{
    void startup() {
        System.out.println("disk startup!");
    }
    void shutdown() {
        System.out.println("disk shutdown!");
    }
}
```

# 参考资料
- []()
- []()
- [MyBatis && MyBatisPlus 源码分析](http://chenzz.me/15159417270086.html#toc_30)
- [mybatis plus源码解析(一) ---基于springboot配置加载和SqlSessionFactory的构造](https://juejin.cn/post/6844903601740087304)
- [MybatisPlus源码详解](https://juejin.cn/post/6844904142658338829)
- [MyBatis-plus 源码解析](https://blog.csdn.net/weixin_45505313/article/details/104855453)
- [Mybatis-Plus使用全解](https://www.cnblogs.com/jpfss/p/11375500.html)
- [MyBatis Plus 教程](https://www.hxstrive.com/subject/mybatis_plus.htm?id=301)
- [MPP-MyBatisPlus-Plus](https://github.com/jeffreyning/mybatisplus-plus)
- [Mybatis plus多筛选条件批量更新](https://blog.csdn.net/tcctcszhanghao/article/details/107604799)
- [Mybatis实体属性与数据库表列名映射的四种方法](https://www.kancloud.cn/tuna_dai_/day01/488641)
- [看云](https://www.kancloud.cn/explore)


```java
    @Override
    public void batchUpdateMoveCourseCategoriesChildren(List<CourseCategories> modifyCourseCategories) {
        String sqlStatement = sqlStatement(SqlMethod.UPDATE);
        try (SqlSession batchSqlSession = SqlHelper.sqlSessionBatch(entityClass)) {
            int i = 0;
            for (CourseCategories category : modifyCourseCategories) {
                MapperMethod.ParamMap<Object> param = new MapperMethod.ParamMap<>();
                param.put(Constants.ENTITY, null);
                param.put(Constants.WRAPPER, Wrappers.<CourseCategories>lambdaUpdate()
                    .set(CourseCategories::getParentSort, category.getSort())
                    .eq(CourseCategories::getParent, category.getCCategoryId())
                    .eq(CourseCategories::getOwnerType, CourseCategoryConstant.OWNER_TYPE_COMPANY)
                    .eq(CourseCategories::getOwner, LoginUtil.getCompanyId()));
                batchSqlSession.update(sqlStatement, param);
                if (i >= 1 && i % 500 == 0) {
                    batchSqlSession.flushStatements();
                }
                i++;
            }
            batchSqlSession.flushStatements();
        }
    }
```