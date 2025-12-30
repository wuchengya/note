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





总结：

更新删除用update(),查找用query()

```ABAP
query()
一般不使用，我们都使用带Lambda的，这样比较安全

lambdaQuery()
这个是上面的安全版本，可以满足很多查询要求了
LambdaQueryWrapper()
这个是为了满足剩下的10%的查询需要
LambdaQueryChainWrapper()
这个是上面的升级版，可以自己执行查询，上面只是构造一个构造器，不能自主执行


update()

lambdaUpdate()

LambdaUpdateWrapper()
相对于LambdaQueryWrapper()有特殊的方法，是关于set的，如.sql(),等

LambdaUpdateChainWrapper()
```

