# MP速通

如何使用？

1引入起步依赖，可以删除mybatis的了

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>
```

2继承BaseMapper接口，记得指定泛型，和数据库对应的实体类型



![image-20250902171800813](E:\Typora picture\image-20250902171800813.png)



![](E:\Typora picture\image-20250902171303963.png)

MP的常用配置，只需要记得一个就行，其他的走默认

```xml
mybatis-plus:
  type-aliases-package: com.itheima.mp.domain.po
```

存放的是和数据库对应的实体类

条件构造器

![](E:\Typora picture\image-20250903171422232.png)

自定义SQL

![](E:\Typora picture\image-20250903172949064.png)

MP中Service接口的使用

![image-20250903174504138](E:\Typora picture\image-20250903174504138.png)

IService处理批处理

![image-20250903203212840](E:\Typora picture\image-20250903203212840.png)

使用MybatisPlus中的分页

第一步：配置

```java
@Configuration
public class MyBatisConfig {

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 1.创建分页插件
        PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);
        paginationInnerInterceptor.setMaxLimit(1000L);
        // 2.添加分页插件
        interceptor.addInnerInterceptor(paginationInnerInterceptor);
        return interceptor;
    }
}
```

第二步：使用

![image-20250903215539463](E:\Typora picture\image-20250903215539463.png)