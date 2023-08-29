---
title: shiro
date: 2022/5/23 15:22:43
updated: 2023/3/23 23:02:19
categories:
- Middleware
tags:
- Shiro
---

# shiro

### Subject

**Subject**：主体，代表了当前 “用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是 Subject，如网络爬虫，机器人等；即一个抽象概念；所有 Subject 都绑定到 SecurityManager，与 Subject 的所有交互都会委托给 SecurityManager；可以把 Subject 认为是一个门面；SecurityManager 才是实际的执行者；

### SecurityManager

**SecurityManager**：安全管理器；即所有与安全有关的操作都会与 SecurityManager 交互；且它管理着所有 Subject；可以看出它是 Shiro 的核心，它负责与后边介绍的其他组件进行交互，如果学习过 SpringMVC，你可以把它看成 DispatcherServlet 前端控制器；

### Realm

**Realm**：域，Shiro 从 Realm 获取安全数据（如用户、角色、权限），就是说 SecurityManager 要验证用户身份，那么它需要从 Realm 获取相应的用户进行比较以确定用户身份是否合法；也需要从 Realm 得到用户相应的角色 / 权限进行验证用户是否能进行操作；可以把 Realm 看成 DataSource，即安全数据源。

- 创建ShiroConfig配置类，配置权限等级等
- 创建Realm类，用户对登陆的用户授权和认证

>  使用`Subject subject = SecurityUtils.getSubject();`获取当前登陆的用户

数据库permission规定了用户的权限例如`user:root`

### ShiroConfig

```java
@Configuration
public class ShiroConfig
{
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(@Qualifier("securityManager") DefaultWebSecurityManager defaultWebSecurityManager)
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
        Map<String, String> map = new LinkedHashMap<>();
        //设置index页面需要认证
        map.put("/index", "authc");
        //user1页面需要授权user:root才能访问
        map.put("/user/user1", "perms[user:root]");
        //user2页面需要授权user:admin才能访问
        map.put("/user/user2", "perms[user:admin]");
        bean.setFilterChainDefinitionMap(map);
        
        //设置默认跳转的登陆页面
        bean.setLoginUrl("/toLogin");
        //设置默认跳转的未授权页面
        bean.setUnauthorizedUrl("/noAuth");
        
        return bean;
    }
    
    @Bean(name = "securityManager")
    public DefaultWebSecurityManager defaultWebSecurityManager(@Qualifier("userRealm") UserRealm userRealm)
    {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(userRealm);
        return securityManager;
    }
    
    @Bean
    public UserRealm userRealm()
    {
        return new UserRealm();
    }
}

```

### UserRealm

```java
public class UserRealm extends AuthorizingRealm
{
    @Autowired
    private UserServiceImpl userService;
    
    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException
    {
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        User user = userService.selectUser("username", token.getUsername());
        if (user == null)
        {
            //return null抛出异常
            return null;
        }
        
        return new SimpleAuthenticationInfo(user, user.getPassword(), "");
    }
    
    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection)
    {
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();
        Set<String> permission = new HashSet<>();
        //获取当前用户
        Subject subject = SecurityUtils.getSubject();
        if (subject.getPrincipal() != null)
        {
            User currentUser = (User) subject.getPrincipal();
            permission.add(currentUser.getPermission());
            //设置权限
            info.setStringPermissions(permission);
        }
        return info;
    }
}
```

### UserController

```java
@Controller
public class UserController
{
    @PostMapping("login")
    @ResponseBody
    public String login(@RequestParam("username") String username,
                        @RequestParam("password") String password)
    {
        //获取当前的用户
        Subject subject = SecurityUtils.getSubject();
        //封装用户登陆数据
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        /*
        执行登陆操作
        调用UserRealm中认证方法
         */
        try
        {
            subject.login(token);
            return "登陆成功";
        } catch (UnknownAccountException e)
        {
            return "用户不存在";
        } catch (IncorrectCredentialsException e)
        {
            return "密码错误";
        }
    }
    
    @RequestMapping("logout")
    public String logout()
    {
        //获取当前的用户
        Subject subject = SecurityUtils.getSubject();
        if (subject.getPrincipal() != null)
        {
            subject.logout();
        }
        return "index";
    }
}
```

# 原理

## 运行流程

- 调用`Subject.login(token)`方法登陆
- login方法调用SecurityManager进行身份验证
- SecurityManager调用Authenticator进行身份验证
- Authenticator将token传入Realm，从Realm中获取身份验证信息，如果没有返回认证失败

