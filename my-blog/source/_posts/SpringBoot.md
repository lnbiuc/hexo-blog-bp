---
title: SpringBoot
date: 2022/4/30 01:00:23
updated: 2023/3/23 22:58:39
categories:
- BackEnd
tags:
- SpringBoot
---

## yaml

使用yaml给实体类赋值



```json
{"Students":{"number":1,"name":"Alex","dead":"Mon Jan 01 00:00:00 CST 2001","isBoy":null,"list":[one, two, three],"map":{k1=v1, k2=v2, k3=v3},"personal":{"Personal":{"name":"steven","age":10}}}}
```

- 实体类必须包含getter setter 有参无参构造方法

pojo

```java
@Component
@ConfigurationProperties(prefix = "students")
//ConfigurationProperties获取yaml文件中值 从而为实体类设定初值
public class Students {
    private Integer number;
    private String name;
    private Date dead;
    private Boolean isBoy;
    private List<Object> list;
    private Map<Object, Object> map;
    private Personal personal;
```

yaml（注意日期格式）

```yaml
students:
  number: 001
  name: Alex
  dead: 2001/01/01
  isBoy: true
  list:
    - one
    - two
    - three
  map: {k1: v1,k2: v2,k3: v3}
  personal:
    name: steven
    age: 10
```

test

```java
@SpringBootTest
class SpringBootLearnApplicationTests {
    
    @Autowired
    private Personal personal;
    @Autowired
    private Students students;
    @Test
     void contextLoads() {
//        System.out.println(personal);
        System.out.println(students);
    }   
}
```

## jsr303校验

```java
@Component
@ConfigurationProperties(prefix = "students")
//ConfigurationProperties获取yaml文件中值 从而为实体类设定初值
@Validated
//自动数据校验
public class Students {
    
    private Integer number;
    private String name;
    private Date dead;
    private Boolean isBoy;
    private List<Object> list;
    private Map<Object, Object> map;
    private Personal personal;
```

## 静态资源放在maven/resources下

> 在resources中的 resources、static、public文件夹中
>
> 其中，访问优先级 resources > static > public
>
> 例如 访问maven/resources/resources/index.html
>
> 直接访问 localhost:8080/即可找到index.html
>
> index.html会被自动识别加载为首页

约定  public放静态资源 js jq css 等

​	 static放 图片等

​     resources 放客户端上传的文件

templates目录下的只能通过controller跳转来访问

## thymeleaf

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>templates</title>
</head>
<body>
<h1>templates</h1>
<h1 th:text="${msg}"></h1>
</body>
</html>
```

## Mybatis

![image-20220430235321089](https://typora-1308549476.cos.ap-nanjing.myqcloud.com/img/image-20220430235321089.png)

```java
@Controller
@RequestMapping("/user")
public class UserController {
    
    @Autowired
    private UserMapper userMapper;
    
    @RequestMapping("/getList")
    @ResponseBody
    public User queryUserList() {
        return userMapper.queryUserById(1);
    }
}

```

```java
@Mapper
//mapper注解指定mybatis mapper接口
@Repository
public interface UserMapper {
    
    List<User> queryUserList();
    
    User queryUserById(@Param("id") int id);
    
    int addUser(User user);
    
    int deleteUser(int id);
    
    int updateUser(User user);
}
```

## 资源过滤

新建配置类

```java
@Configuration
public class InterceptorConfig extends WebMvcConfigurationSupport {
    /*
    设置资源过滤 不过滤静态资源
     */
    @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/")
                .addResourceLocations("classpath:/static/fonts");
    }
}

```

Pom.xml

```xml
<build>
        <resources>
            <resource>
                <directory>src/main/java</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                </includes>
                <filtering>false</filtering>
            </resource>
            <resource>
                <directory>src/main/resources</directory>
                <includes>
                    <include>**/*.properties</include>
                    <include>**/*.xml</include>
                    <include>**/*.yml</include>
                    <include>**/*.html</include>
                    <include>**/*.css</include>
                    <include>**/*.js</include>
                    <include>**/*.woff</include>
                    <include>**/*.woff2</include>
                    <include>**/*.ttf</include>
                    <include>**/*.eot</include>
                    <include>**/*.svg</include>
                </includes>
                <filtering>false</filtering>
            </resource>
        </resources>
</build>
```

```properties
spring.datasource.username=root
spring.datasource.password=Dd112211
spring.datasource.url=jdbc:mysql://localhost:3306
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
mybatis.type-aliases-package=com.horizon.pojo
mybatis.mapper-locations=classpath:mapper/*.xml
spring.devtools.restart.enabled=true
spring.mvc.static-path-pattern=classpath:


```

```yaml
thymeleaf:
  prefix: classpath:/templates/  #prefix：指定模板所在的目录
  check-template-location: true  #check-tempate-location: 检查模板路径是否存在
  cache: false  #cache: 是否缓存，开发模式下设置为false，避免改了模板还要重启服务器，线上设置为true，可以提高性能。
  suffix:  .html
  #encoding: UTF-8
  #content-type: text/html
  mode: HTML5

```

## 文件提交

```html
<form id="fileForm" action="./upload" method="post" enctype="multipart/form-data" target="stop">
            <input multiple type="file" name="file">
            <button type="submit" value="upload">button</button>
            <input type="submit" value="input">
        </form>
        <iframe  name="stop" style="display:none;"></iframe>
```



测试远程修改