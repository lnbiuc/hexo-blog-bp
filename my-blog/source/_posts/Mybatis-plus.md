---
title: Mybatis-plus
date: 2022/5/3 23:00:16
updated: 2023/3/23 22:56:33
categories:
- BackEnd
tags:
- MyBatisPlus
---

## 基本环境

```xml
		<dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.5.1</version>
        </dependency>
```



```java
@Repository
public interface UserMapper extends BaseMapper<User> {

}
```

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    
    private Long id;
    private String name;
    private Integer age;
    private String email;
    
}

```

```java
@SpringBootApplication
@MapperScan("com.horizon.mapper")
public class MybatisPlusApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusApplication.class, args);
    }
    
}
```

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis_plus
    username: root
    password: Dd112211
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

## 自带id算法

启用id算法

` @TableId(type = IdType.ASSIGN_UUID)`主键

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    
    @TableId(type = IdType.ASSIGN_UUID)
    private String id;
    private String name;
    private Integer age;
    private String email;
    
}
```

![image-20220503025416700](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220503025416700.png)



## 查

通过列表查询多个

```java
@Test
    public void testCase4(){
        List<User> userList = userMapper.selectBatchIds(Arrays.asList("1", "2", "3"));
        userList.forEach(System.out::println);
    }
```

通过map实现多条件查询

```java
@Test
    public void testCase5(){
        Map<String, Object> map = new HashMap<>();
        map.put("age", 20);
        map.put("name", "Jack");
        List<User> userList = userMapper.selectByMap(map);
        userList.forEach(System.out::println);
    }
```

## 增

null id会根据实体类上id直接自动填充

```java
public void testCase1(){
        User user = new User(null,"ayanamirei",12,"email@e.cn");
        userMapper.insert(user);
    }
```

## 改

自动动态拼接sql

```java
public void testCase2(){
    User user = new User("eeeb0b9ce1f6011b515df210d945ccef","root",20,null);
    userMapper.updateById(user);
}
```

## 删

Easy

## 逻辑删除

增加一个字段deleted = 0 => deleted = 1

配置之后 deleted不会执行删除，会执行更新操作，将deleted值更新

查询会自动过滤逻辑删除的数据(会自动拼接deleted)

```java
@TableLogic
    private Integer deleted;
```

添加逻辑删除插件

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mybatis_plus
    username: root
    password: Dd112211
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      logic-delete-field: deleted # 逻辑删除的字段名
      logic-delete-value: 1 # 已删除的是1
      logic-not-delete-value: 0 # 未删除的是0
```



## 对于自动填充的日期类型处理

   ` @TableField(fill = FieldFill.INSERT)`

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    
    @TableId(type = IdType.ASSIGN_UUID)
    private String id;
    private String name;
    private Integer age;
    private String email;
    @TableField(fill = FieldFill.INSERT)
    private Date creationDate;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updatedDate;
    
}
```

![image-20220503031134137](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220503031134137.png)

```java
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    /**
     * 插入新数据时 自动填充creationDate，updatedDate 内容为当前时间
     * @param metaObject metaObject
     */
    @Override
    public void insertFill(MetaObject metaObject) {
        this.strictInsertFill(metaObject,"createTime", Date.class,new Date());
        this.strictUpdateFill(metaObject,"updataTime", Date.class,new Date());
    
    }
    
    /**
     * 更新数据时 自动更新updatedDate 内容为更新时间
     * @param metaObject metaObject
     */
    @Override
    public void updateFill(MetaObject metaObject) {
        this.strictUpdateFill(metaObject,"updataTime", Date.class,new Date());
    }
}
```

## 乐观锁and悲观锁

在配置类中开启mybatis plus的乐观锁

防止并发操作下，多个线程对同一个字段操作造成的冲突问题

> 乐观锁：默认不上锁，在执行更新期间，判断别线程是否修改了数据，如果别线程修改了数据则放弃操作，否次继续操作
>
> 悲观锁：默认上锁，在执行更新期间默认上锁，上锁期间别的线程不可以修改数据，直到操作完成才会释放锁

```java
@Configuration
@MapperScan("com.horizon.mapper")
public class MybatisPlusConfig {
    
    /*
    注册乐观锁插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
```

数据库表添加version字段，默认值设置为1

![image-20220503215939066](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220503215939066.png)

实体类上增加version类型和version注解

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@TableName(value = "user")
public class User {
    
    @TableId(type = IdType.ASSIGN_UUID)
    private String id;
    private String name;
    private Integer age;
    private String email;
    @Version
    private Integer version;
    @TableField(fill = FieldFill.INSERT)
    private Date createTime;
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private Date updataTime;
    
}
```



```java
@Test
    public void testCase3(){
        //线程1
        User user = userMapper.selectById("1");
        user.setAge(30);
        user.setEmail("newemail");
        
        //线程2
        User user2 = userMapper.selectById("1");
        user2.setAge(40);
        user2.setName("User2");
        userMapper.updateById(user);
        
        //线程1最终更新操作
        userMapper.updateById(user);
        
        //此时线程2的更新不会生效，因为有乐观锁的存在
    }
```

## 分页查询

在拦截器中添加分页插件

```java
@Configuration
@MapperScan("com.horizon.mapper")
public class MybatisPlusConfig {
    
    /*
    创建新的拦截器
    */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(){
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        //配置乐观锁插件
        mybatisPlusInterceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        //配置分页插件
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return mybatisPlusInterceptor;
    
    }
}
```

分页 `Page<User> pages = new Page<>(1,2);`第几页 要几个

```java
@Test
    public void testCase6(){
        //第1页 每一页2个
        Page<User> pages = new Page<>(1,2);
    
        Page<User> page = userMapper.selectPage(pages, null);
        page.getRecords().forEach(System.out::println);
    }
```

![image-20220503223129438](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220503223129438.png)

## 性能分析插件

找出比较耗时的查询，超过一定时间不查了

新版本mybatisplus移除了这个插件

## 条件构造器wrapper

```java
@Test
    public void testCase8(){
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.isNotNull("name").
                isNotNull("email").
                ge("age", 10);
        userMapper.selectList(wrapper);
    }
```

通过.来添加查询条件

![image-20220503224827537](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220503224827537.png)

|    allEq    |   全等于   |
| :---------: | :--------: |
|     eq      |    等于    |
|     ne      |   不等于   |
|     gt      |    大于    |
|     ge      |  大于等于  |
|     lt      |    小于    |
|     le      |  小于等于  |
|   between   |  在--之间  |
| notBetween  | 不在--之间 |
|    like     |            |
|   notLike   |            |
|    inSql    |            |
|  notInSql   |            |
|   groupBy   |            |
| orderByAsc  |            |
| orderByDesc |            |
|   having    |            |
|    func     |            |






