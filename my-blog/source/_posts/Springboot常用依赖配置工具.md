---
title: Springboot常用依赖配置工具
date: 2022/10/6 22:39:10
updated: 2023/3/23 22:55:42
categories:
- BackEnd
tags:
- SpringBoot
---

# 常用依赖

## Mybatis-plus

```xml
<dependency>
	<groupId>com.baomidou</groupId>
	<artifactId>mybatis-plus-boot-starter</artifactId>
	<version>3.5.1</version>
</dependency>
```

## 参数校验

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## FastJson

```xml
<dependency>
	<groupId>com.alibaba</groupId>
	<artifactId>fastjson</artifactId>
	<version>2.0.3</version>
</dependency>
```

## jjwt Token

```xml
<dependency>
	<groupId>io.jsonwebtoken</groupId>
	<artifactId>jjwt</artifactId>
	<version>0.9.1</version>
</dependency>
```

## redis

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## jedis

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>lettuce-core</artifactId>
            <groupId>io.lettuce</groupId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
    <version>2.9.0</version>
</dependency>
```

## shiro

```xml
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-spring</artifactId>
	<version>1.9.0</version>
</dependency>
```

## 腾讯云、短信、cos上传

```xml
<dependency>
	<groupId>com.qcloud</groupId>
	<artifactId>cos_api</artifactId>
	<version>5.6.75</version>
</dependency>
<dependency>
	<groupId>com.qcloud</groupId>
	<artifactId>qcloud-java-sdk</artifactId>
	<version>2.0.7</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>com.tencent.cloud</groupId>
	<artifactId>cos-sts-java</artifactId>
	<version>3.0.8</version>
	<scope>test</scope>
</dependency>
<dependency>
	<groupId>com.tencentcloudapi</groupId>
	<artifactId>tencentcloud-sdk-java</artifactId>
	<version>3.1.270</version>
</dependency>
```

## pageHelper

```xml
<dependency>
	<groupId>com.github.pagehelper</groupId>
	<artifactId>pagehelper-spring-boot-starter</artifactId>
	<version>1.4.3</version>
</dependency>
```

## UserAgent解析

```xml
 <dependency>
	<groupId>eu.bitwalker</groupId>
	<artifactId>UserAgentUtils</artifactId>
	<version>1.20</version>
</dependency>
```

## 密码加解密

```xml
<dependency>
	<groupId>org.apache.commons</groupId>
	<artifactId>commons-lang3</artifactId>
</dependency>
<dependency>
	<groupId>commons-codec</groupId>
	<artifactId>commons-codec</artifactId>
</dependency>
<dependency>
	<groupId>commons-collections</groupId>
	<artifactId>commons-collections</artifactId>
	<version>3.2.2</version>
</dependency>
<dependency>
	<groupId>joda-time</groupId>
	<artifactId>joda-time</artifactId>
	<version>2.10.14</version>
</dependency>
```

# application.yaml

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/my_blog?useUnicode=true&useSSL=false&characterEncoding=utf8&serverTimezone=Asia/Shanghai
    username: root
    password: 123456
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    password: test
    jedis:
      pool:
        min-idle: 0
        max-active: 8
        max-idle: 8
        max-wait: -1ms
    connect-timeout: 30000ms
  cache:
    type: redis
    redis:
      time-to-live: 24h
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 20MB
  mvc:
    static-path-pattern: /static/**
mybatis-plus:
  #configuration:
  #log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
  global-config:
    db-config:
      cache-enabled: true
      table-prefix: blog_
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
logging:
  file:
    path: log
  logback:
    rollingpolicy:
      clean-history-on-start: true
      max-file-size: 5MB
server:
  port: 8888
  servlet:
    context-path: /api
```

# 配置类

## Redis

### Redis序列化及缓存配置

```java
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport
{
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory)
    {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        // 配置序列化（解决乱码的问题）
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(3))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(serializer()))
                .disableCachingNullValues();
        
        return RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
    }
    
    
    /**
     * RedisTemplate配置
     */
    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory)
    {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        // 用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值
        redisTemplate.setValueSerializer(serializer());
        
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        // 使用StringRedisSerializer来序列化和反序列化redis的key值
        redisTemplate.setKeySerializer(stringRedisSerializer);
        
        // hash的key也采用String的序列化方式
        redisTemplate.setHashKeySerializer(stringRedisSerializer);
        // hash的value序列化方式采用jackson
        redisTemplate.setHashValueSerializer(serializer());
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }
    
    private Jackson2JsonRedisSerializer<Object> serializer()
    {
        // 使用Jackson2JsonRedisSerializer来序列化和反序列化redis的value值
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper objectMapper = new ObjectMapper();
        
        // 指定要序列化的域，field,get和set,以及修饰符范围，ANY是都有包括private和public
        objectMapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        
        // 指定序列化输入的类型，类必须是非final修饰的，final修饰的类，比如String,Integer等会跑出异常
        objectMapper.activateDefaultTyping(LaissezFaireSubTypeValidator.instance, ObjectMapper.DefaultTyping.NON_FINAL);
        
        jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
        return jackson2JsonRedisSerializer;
    }
}
```

## 跨域配置

```java
@Component
@Slf4j
public class CORSFilter implements Filter
{
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException
    {
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        //放行所有,类似*,这里*无效
        response.setHeader("Access-Control-Allow-Origin", request.getHeader("Origin"));
        response.setHeader("Access-Control-Allow-Credentials", "true");
        //允许请求方式
        response.setHeader("Access-Control-Allow-Methods", "POST,PUT, GET, OPTIONS, DELETE");
        response.setHeader("Access-Control-Max-Age", "3600");
        //需要放行header头部字段 如需鉴权字段，自行添加，如Authorization
        response.setHeader("Access-Control-Allow-Headers",
                "content-type,x-requested-with,token,Authorization,authorization");
        try
        {
            filterChain.doFilter(request, response);
        } catch (Exception e)
        {
            log.error("CORS过滤器放行异常:", e);
        }
    }

    @Override
    public void init(FilterConfig filterConfig)
    {

    }

    @Override
    public void destroy()
    {
        Filter.super.destroy();
    }
}
```

## token拦截校验

```java
@Slf4j
@Component
public class AdminInterceptor implements HandlerInterceptor
{
    private final TokenService tokenService;

    public AdminInterceptor(TokenService tokenService)
    {
        this.tokenService = tokenService;
    }

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception
    {

        if (HttpMethod.OPTIONS.toString().equals(request.getMethod()))
        {
            return true;
        }
        String token = request.getHeader("token");
        if (token == null)
        {
            response.sendError(401);
            return false;
        }
        UserVo user = tokenService.getUserByToken(token);
        if (user == null)
        {
            response.sendError(401);
            return false;
        }
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception
    {
        HandlerInterceptor.super.postHandle(request, response, handler, modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception
    {
        HandlerInterceptor.super.afterCompletion(request, response, handler, ex);
    }
}
```

## Mybatis-plus分页

```java
@Configuration
@MapperScan("top.beyondhorizon.mapper")
public class MybatisConfig
{
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor()
    {
        //创建拦截器
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        //分页插件
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor());
        return mybatisPlusInterceptor;
    }
}
```

## shiro配置

```java
@Configuration
public class ShiroConfig
{
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(
            @Qualifier("securityManager")
            DefaultWebSecurityManager defaultWebSecurityManager
    )
    {
        ShiroFilterFactoryBean bean = new ShiroFilterFactoryBean();
        bean.setSecurityManager(defaultWebSecurityManager);
        
        /*
         anon：所有人可访问
         authc：认证才能访问
         user：必须有记住我功能才能访问
         perms：拥有对某个资源的权限才能访问
         roles：拥有某个角色权限才能访问
         */
//        Map<String, String> map = new LinkedHashMap<>();
//        map.put("/user/logout", "authc");
//        map.put("/comments/add", "authc");
//        map.put("/articles/publish", "authc");
//        map.put("/user/status", "authc");
//        map.put("/admin/**", "authc");
//        bean.setFilterChainDefinitionMap(map);

//        Map<String, Filter> filters = bean.getFilters();
//        filters.put("authc", new StatelessAuthcFilter());
//        bean.setFilters(filters);
        bean.setLoginUrl("/login");

        bean.setUnauthorizedUrl("/noPermission");
        return bean;
    }

    @Bean(name = "securityManager")
    public DefaultWebSecurityManager defaultWebSecurityManager(
            @Qualifier("userRealm")
            UserRealm userRealm
    )
    {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm);
        return securityManager;
    }

    @Bean
    public UserRealm userRealm(HashedCredentialsMatcher matcher)
    {
        UserRealm userRealm = new UserRealm();
        userRealm.setCredentialsMatcher(matcher);
        return userRealm;
    }

    @Bean
    public HashedCredentialsMatcher hashedCredentialsMatcher()
    {
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        hashedCredentialsMatcher.setHashAlgorithmName("md5");
        hashedCredentialsMatcher.setHashIterations(1);
        hashedCredentialsMatcher.setStoredCredentialsHexEncoded(true);
        return hashedCredentialsMatcher;
    }
}
```

```java
@Slf4j
public class UserRealm extends AuthorizingRealm
{
    @Autowired
    private UserService userService;

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken)
            throws AuthenticationException
    {
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        UserDto userDto = userService.getUserByUsername(token.getUsername());
        if (userDto == null)
        {
            return null;
        }
        String username = token.getUsername();
        String password = userDto.getPassword();
        String databaseSalt = userService.getSaltByUsername(username);
        ByteSource salt = ByteSource.Util.bytes(databaseSalt);
        String realmName = this.getName();
        return new SimpleAuthenticationInfo(userDto, password, salt, realmName);
    }

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection)
    {
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        Set<String> permission = new HashSet<>();
        Subject subject = SecurityUtils.getSubject();
        User currentUser = (User) subject.getPrincipal();
        permission.add(String.valueOf(currentUser.getPermission()));
        log.warn("currentUser.getPermission() === {}", currentUser.getPermission());
        //设置权限 默认普通用户为false
        info.setStringPermissions(permission);
        return info;
    }
}
```

## web拦截器

```java
@Configuration
public class WebMvcConfig implements WebMvcConfigurer
{

    @Autowired
    private AdminInterceptor adminInterceptor;

    @Override
    public void addCorsMappings(CorsRegistry registry)
    {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:8080")
                .allowedMethods("*")
                .allowedHeaders("*")
                .allowCredentials(true)
                .maxAge(3600);
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry)
    {
        registry.addInterceptor(adminInterceptor)
                .addPathPatterns("/admin/**");
    }
}
```


# 统一接口返回

## 状态码

```java
@Getter
@AllArgsConstructor
public enum RespStatus
{
    
    SUCCESS(200, "success"),
    FAIL(500, "failed"),
    
    HTTP_STATUS_200(200, "ok"),
    // 响应错误
    HTTP_STATUS_400(400, "request error"),
    // 身份认证失败
    HTTP_STATUS_401(401, "no authentication"),
    // 未授权
    HTTP_STATUS_403(403, "no authorities"),
    // 服务器错误
    HTTP_STATUS_500(500, "server error");
    
    private final Integer statusCode;
    
    private final String description;
    
}
```

## 返回数据

```java
@Data
@Builder
public class RespData<T>
{
    /*
    状态码
     */
    private Integer code;
    
    /*
    返回消息
     */
    private String message;
    
    /*
    时间标签
     */
    private Date timestamp;
    
    /*
    返回数据
     */
    private T data;
    
    /**
     * 请求成功无返回数据
     * @return null
     * @param <T> RespData.type
     */
    public static <T> RespData<T> success()
    {
        return success(null);
    }
    
    /**
     * 请求成功 返回数据
     * @param data 需要返回的数据
     * @return RespData
     * @param <T> RespData.type
     */
    public static <T> RespData<T> success(T data)
    {
        return RespData.<T>builder().data(data)
                .code(RespStatus.SUCCESS.getStatusCode())
                .message(RespStatus.SUCCESS.getDescription())
                .timestamp(new Date())
                .data(data)
                .build();
                
    }
    
    /**
     * 请求失败
     * @param message 需要返回失败消息
     * @return 返回失败消息
     * @param <T> RespData.type
     */
    public static <T extends Serializable> RespData<T> fail(String message)
    {
        return fail(null,message);
    }
    
    /**
     * 请求失败，返回数据
     * @param data 需要返回的数据
     * @param message 失败信息
     * @return 返回失败消息 + 数据
     * @param <T> RespData.type
     */
    public static <T> RespData<T> fail(T data,String message)
    {
        return RespData.<T>builder()
                .code(RespStatus.FAIL.getStatusCode())
                .message(message)
                .timestamp(new Date())
                .data(data)
                .build();
    }
}
```

# 参数校验

## 所需依赖

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

## 请求参数（@RequestBody）

> ReqParamsDemo

```java
@Data
public class ReqParamsDemo implements Serializable
{
    private int id;
    
    @NotEmpty(message = "Name不可为空")
    private String name;
    
    @NotEmpty(message = "phone不可为空")
    @Pattern(regexp = "/(13[0-9]|14[579]|15[0-3,5-9]|16[6]|17[0135678]|18[0-9]|19[89])\\d{8}/",message = "手机号格式有误")
    private String phoneNum;
    
    @NotEmpty(message = "email不可为空")
    @Email
    private String email;
    
    @Range(min = 0, max = 1, message = "只能是1或0")
    private int gender;
//
    @Max(value = 100,message = "最大支持年龄100")
    private int age;
}
```

> controller

```java
@RestController
@RequestMapping("test")
@Slf4j
public class TestController
{
    
    private final ArticleService articleService;
    
    public TestController(ArticleService articleService)
    {
        this.articleService = articleService;
    }
    
    @GetMapping("get")
    public RespData<List<Article>> get()
    {
        List<Article> articles = articleService.getAllArticles();
        return RespData.success(articles);
    }
    
    @PostMapping("add")
    public RespData<String> add(@Valid @RequestBody ReqParamsDemo params
            , BindingResult bindingResult)
    {
        if (bindingResult.hasErrors())
        {
            List<ObjectError> errors = bindingResult.getAllErrors();
            errors.forEach(p ->
            {
                FieldError fieldError = (FieldError) p;
                log.error("Invalid Parameter : object - {},field - {},errorMessage - {}", fieldError.getObjectName(), fieldError.getField(), fieldError.getDefaultMessage());
            });
            return RespData.fail("参数格式有误");
        }
        log.info(params.toString());
        return RespData.success();
    }
}
```

# 统一异常处理

```java
@Slf4j
@RestControllerAdvice
public class AllExceptionHandler
{
    
    /**
     * 对所有异常拦截并返回
     * @param e 异常
     * @return 返回异常描述信息
     */
    @ResponseBody
    @ExceptionHandler(Exception.class)
    public RespData<Exception> allException(Exception e)
    {
        log.error(e.getLocalizedMessage(), e);
        return RespData.fail(e, e.getMessage());
    }
    
    
    /**
     * 对于特定状态吗的异常拦截并返回
     * @param e 异常
     * @return 返回异常信息
     */
    @ResponseBody
    @ResponseStatus(code = HttpStatus.BAD_REQUEST)
    @ExceptionHandler(value = {BindException.class, ValidationException.class, MethodArgumentNotValidException.class})
    public RespData<Exception> badRequest(Exception e)
    {
        log.error(e.getLocalizedMessage(), e);
        return RespData.fail(e, e.getMessage());
    }
}
```

# 工具类

## 线程池创建

```java
@Configuration
//开启多线程
@EnableAsync
public class ThreadPoolConfig
{
    @Bean("taskExecutor")
    public Executor asyncServiceExecutor()
    {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        //核心线程数
        executor.setCorePoolSize(5);
        //最大线程数
        executor.setMaxPoolSize(20);
        //队列大小
        executor.setQueueCapacity(Integer.MAX_VALUE);
        //线程活跃时间（秒）
        executor.setKeepAliveSeconds(120);
        //默认线程名
        executor.setThreadNamePrefix("viewCountAsyncExecutor");
        //所有任务结束再关闭线程池 true
        executor.setWaitForTasksToCompleteOnShutdown(true);
        //执行初始化
        executor.initialize();
        return executor;
    }
}
```

## 请求Ip解析

```java
public class HttpContextUtils
{
    public static HttpServletRequest getHttpServletRequest()
    {
        return ((ServletRequestAttributes) Objects.requireNonNull(RequestContextHolder.getRequestAttributes())).getRequest();
    }
}
```

```java
@Slf4j
public class IpUtils
{
    /**
     * 获取IP地址
     * <p>
     * 使用Nginx等反向代理软件， 则不能通过request.getRemoteAddr()获取IP地址
     * 如果使用了多级反向代理的话，X-Forwarded-For的值并不止一个，而是一串IP地址，X-Forwarded-For中第一个非unknown的有效IP字符串，则为真实IP地址
     */
    public static String getIpAddr(HttpServletRequest request)
    {
        String ip = null, unknown = "unknown", seperator = ",";
        int maxLength = 15;
        try
        {
            ip = request.getHeader("x-forwarded-for");
            if (StringUtils.isEmpty(ip) || unknown.equalsIgnoreCase(ip))
            {
                ip = request.getHeader("Proxy-Client-IP");
            }
            if (StringUtils.isEmpty(ip) || ip.length() == 0 || unknown.equalsIgnoreCase(ip))
            {
                ip = request.getHeader("WL-Proxy-Client-IP");
            }
            if (StringUtils.isEmpty(ip) || unknown.equalsIgnoreCase(ip))
            {
                ip = request.getHeader("HTTP_CLIENT_IP");
            }
            if (StringUtils.isEmpty(ip) || unknown.equalsIgnoreCase(ip))
            {
                ip = request.getHeader("HTTP_X_FORWARDED_FOR");
            }
            if (StringUtils.isEmpty(ip) || unknown.equalsIgnoreCase(ip))
            {
                ip = request.getRemoteAddr();
            }
        } catch (Exception e)
        {
            log.error("IpUtils ERROR ", e);
        }
        
        // 使用代理，则获取第一个IP地址
        if (StringUtils.isEmpty(ip) && ip.length() > maxLength)
        {
            int idx = ip.indexOf(seperator);
            if (idx > 0)
            {
                ip = ip.substring(0, idx);
            }
        }
        
        return ip;
    }
    
    /**
     * 获取ip地址
     */
    public static String getIpAddr()
    {
        HttpServletRequest request = HttpContextUtils.getHttpServletRequest();
        return getIpAddr(request);
    }
}
```

## Jwt Token生成

```java
@Slf4j
@Component
public class JWTUtils
{
    //token加密密钥
    private static final String jwtToken = "加密密钥🔐";
    
    public static String createToken(String userId)
    {
        Map<String, Object> claims = new HashMap<>();
        claims.put("userId", userId);
        JwtBuilder jwtBuilder = Jwts.builder()
                .signWith(SignatureAlgorithm.HS256, jwtToken) // 签发算法，秘钥为jwtToken
                .setClaims(claims) // body数据，要唯一，自行设置
                .setIssuedAt(new Date()) // 设置签发时间
                .setExpiration(new Date(System.currentTimeMillis() + 168L * 60 * 60 * 60 * 1000));// 7天的有效时间
        return jwtBuilder.compact();
    }
    
    public static Map<String, Object> checkToken(String token)
    {
        try
        {
            Jwt parse = Jwts.parser().setSigningKey(jwtToken).parse(token);
            return (Map<String, Object>) parse.getBody();
        } catch (Exception e)
        {
            log.warn("JWT Token解析失败");
        }
        return null;
    }
}
```

## qq号解析头像昵称

```java
/**
 * 根据qq号获取头像 昵称
 * {"qq":"2857621299","nickname":"MyDearAyanamiRei","avatar":"https://q1.qlogo.cn/g?b=qq&nk=2857621299&s=40","email":"2857621299@qq.com","url":"https://user.qzone.qq.com/2857621299"}
 */
public class QqAvatar {
    
    public QqAvatarRespDto getInfoByQqNumber(String qqNumber) {
        String url = "https://api.lixingyong.com/api/qq?id=" + qqNumber;
        RestTemplate restTemplate = new RestTemplate();
        String object = restTemplate.getForObject(url, String.class);
        QqAvatarRespDto dto = JSONObject.parseObject(object, QqAvatarRespDto.class);
        return dto;
    }
}
```

## 腾讯云发短信

```java
@Slf4j
public class SendSmsUtil {
    private static final String SECRET_ID = "";
    private static final String SECRET_KEY = "";
    private static final String SDK_APPID = "";
    private static final String SIGN_NAME = "";
    private static final String TEMPLATE_ID = "";
    
    /**
     * @param code        发送的验证码
     * @param phoneNumber 手机号
     * @return 发送成功返回true 失败返回false
     */
    public static String SendSMS(String code, String phoneNumber) {
        try {
            Credential cred = new Credential(SECRET_ID, SECRET_KEY);
            HttpProfile httpProfile = new HttpProfile();
            httpProfile.setReqMethod("POST");
            httpProfile.setConnTimeout(60);
            httpProfile.setEndpoint("sms.tencentcloudapi.com");
            ClientProfile clientProfile = new ClientProfile();
            clientProfile.setSignMethod("HmacSHA256");
            clientProfile.setHttpProfile(httpProfile);
            SmsClient client = new SmsClient(cred, "ap-guangzhou", clientProfile);
            SendSmsRequest req = new SendSmsRequest();
            req.setSmsSdkAppid(SDK_APPID);
            req.setSign(SIGN_NAME);
            req.setTemplateParamSet(new String[]{code});
            String phoneNumberSet = "+86" + phoneNumber;
            req.setPhoneNumberSet(new String[]{phoneNumberSet});
            SendSmsResponse res = client.SendSms(req);
            String s = SendSmsResponse.toJsonString(res);
            JSONObject jo = JSON.parseObject(s);
            JSONArray jsarr = jo.getJSONArray("SendStatusSet");
            JSONObject ao = jsarr.getJSONObject(0);
            String sendCode = ao.getString("Code");
            log.warn("短信发送状态，{},{}", phoneNumber, sendCode);
            return sendCode;
        } catch (TencentCloudSDKException e) {
            e.printStackTrace();
        }
        return null;
    }
}
```

## 腾讯云cos文件上传

```java
public class TencentCOSUploadFileUtil {
    // 访问域名
    public static final String URL = "";
    // 存储桶名称
    private static final String BUCKET_NAME = "";
    //secretId 秘钥id
    private static final String SECRET_ID = "";
    //SecretKey 秘钥
    private static final String SECRET_KEY = "";
    // 腾讯云 自定义文件夹名称
    private static final String PREFIX = "";
    // 地址
    private static final String REGION = "";
    // 创建COS 凭证
    private static final COSCredentials credentials = new BasicCOSCredentials(SECRET_ID, SECRET_KEY);
    // 配置 COS 区域
    private static final ClientConfig clientConfig = new ClientConfig(new Region(REGION));
    
    public static String uploadfile(MultipartFile file) {
        // 创建 COS 客户端连接
        COSClient cosClient = new COSClient(credentials, clientConfig);
        String fileName = file.getOriginalFilename();
        try {
            assert fileName != null;
            String substring = fileName.substring(fileName.lastIndexOf("."));
            File localFile = File.createTempFile(String.valueOf(System.currentTimeMillis()), substring);
            file.transferTo(localFile);
            Random random = new Random();
            fileName = PREFIX + random.nextInt(10000) + System.currentTimeMillis() + substring;
            // 将 文件上传至 COS
            PutObjectRequest objectRequest = new PutObjectRequest(BUCKET_NAME, fileName, localFile);
            cosClient.putObject(objectRequest);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            cosClient.shutdown();
        }
        return URL + fileName;
    }
}
```

## 生成6位uuid

```java
public class UuidBuilder
{
    public static String[] chars = new String[]{"a", "b", "c", "d", "e", "f",
            "g", "h", "i", "j", "k", "l", "m", "n", "o", "p", "q", "r", "s",
            "t", "u", "v", "w", "x", "y", "z", "0", "1", "2", "3", "4", "5",
            "6", "7", "8", "9", "A", "B", "C", "D", "E", "F", "G", "H", "I",
            "J", "K", "L", "M", "N", "O", "P", "Q", "R", "S", "T", "U", "V",
            "W", "X", "Y", "Z"};
    
    
    public static String getUuid()
    {
        StringBuilder shortBuffer = new StringBuilder();
        String uuid = UUID.randomUUID().toString().replace("-", "");
        for (int i = 0; i < 8; i++)
        {
            String str = uuid.substring(i * 4, i * 4 + 4);
            int x = Integer.parseInt(str, 16);
            shortBuffer.append(chars[x % 0x3E]);
        }
        return shortBuffer.toString();
    }
}
```

## Aop日志记录

```java
@Component
@Slf4j
@Aspect
public class LogAspect {
    private final ThreadService threadService;
    
    public LogAspect(ThreadService threadService) {
        this.threadService = threadService;
    }
    
    @Pointcut("@annotation(top.beyondhorizon.aop.log.LogAnnotation)")
    public void pt() {
    }
    
    //环绕通知
    @Around("pt()")
    public Object log(ProceedingJoinPoint point) throws Throwable {
        long beginTime = System.currentTimeMillis();
        //执行方法
        Object result = point.proceed();
        //执行时长(毫秒)
        long time = System.currentTimeMillis() - beginTime;
        //保存日志
        recordLog(point, time);
        return result;
        
    }
    
    private void recordLog(ProceedingJoinPoint joinPoint, long time) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        LogAnnotation logAnnotation = method.getAnnotation(LogAnnotation.class);
        log.info("=====================log start================================");
        String module = logAnnotation.module();
        log.info("module:{}", module);
        String operation = logAnnotation.operation();
        log.info("operation:{}", operation);
        
        //请求的方法名
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = signature.getName();
        log.info("request method:{}", className + "." + methodName + "()");
//        //请求的参数
        Object[] args = joinPoint.getArgs();
        if (args.length > 0 && args.length < 255) {
            String params = JSON.toJSONString(args[0]);
            log.info("params:{}", params);
        }
        //获取request 设置IP地址
        HttpServletRequest request = HttpContextUtils.getHttpServletRequest();
        String ipAddr = IpUtils.getIpAddr(request);
        log.info("ip:{}", ipAddr);
        log.info("execute time : {} ms", time);
        log.info("=====================log end================================");
        Map<String, String> map = new HashMap<>();
        map.put("module", module);
        map.put("operation", operation);
        map.put("method", className + "." + methodName);
        map.put("ip", ipAddr);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("S");
        map.put("executeTime", simpleDateFormat.format(time));
        threadService.visitorInfo(map, request);
    }
}
```

## 实体类时间格式化

```java
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd HH:mm:ss", timezone = "Asia/Shanghai")
```




