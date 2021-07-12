---
layout: post
title: "SpringSecurity系列(三)"
categories: [SpringSecurity]

---

练习代码同步到github：SpringSecuritySamples/verify_code

## 自定义认证逻辑——以验证码为例

SpringSecurity框架的核心机制就是过滤器链，具体原理暂且不研究，只知大概和使用方法。

这次要在之前登录功能的基础上，添加比较常见的登录验证码功能，设计思路如下

> 登录请求是调用 AbstractUserDetailsAuthenticationProvider#authenticate 方法进行认证的，在该方法中，又会调用到 DaoAuthenticationProvider#additionalAuthenticationChecks 方法做进一步的校验，去校验用户登录密码。我们可以自定义一个 AuthenticationProvider 代替 DaoAuthenticationProvider，并重写它里边的 additionalAuthenticationChecks 方法，在重写的过程中，加入验证码的校验逻辑即可。

这样的好处是不会破坏原来的过滤器链，并且完成了想要实现的自定义功能。

首先引入一个现成的验证码库kaptcha。

```xml
<dependency>
    <groupId>com.github.penggle</groupId>
    <artifactId>kaptcha</artifactId>
    <version>2.3.2</version>
</dependency>
```

接着围绕验证码进行一系列设置，首先在SecurityConfig里提供一个验证码的实体类，设置验证码图片的基本属性

```JAVA
    @Bean
    DefaultKaptcha verifyCode() {
        Properties properties = new Properties();
        properties.setProperty("kaptcha.image.width", "150");
        properties.setProperty("kaptcha.image.height", "50");
        properties.setProperty("kaptcha.textproducer.char.string", "0123456789");
        properties.setProperty("kaptcha.textproducer.char.length", "4");
        Config config = new Config(properties);
        DefaultKaptcha defaultKaptcha = new DefaultKaptcha();
        defaultKaptcha.setConfig(config);
        return defaultKaptcha;
    }
```

编写返回验证码图片的接口

```JAVA
@RestController
public class VerifyCodeController {
    @Autowired
    Producer producer;
    
    /**
    * @Description: 返回验证码图片的接口
    * @Param: [resp, session]
    * @return: void
    * @Date: 2021/4/2
    */
    @GetMapping("/vc.jpg")
    public void getVerifyCode(HttpServletResponse resp, HttpSession session) throws IOException {
        resp.setContentType("image/jpeg");
        String text = producer.createText();
        session.setAttribute("verify_code", text);
        BufferedImage image = producer.createImage(text);
        try(ServletOutputStream out = resp.getOutputStream()) {
            ImageIO.write(image, "jpg", out);
        }
    }
```

重点就是自定义一个MyAuthenticationProvider类，重写additionalAuthenticationChecks方法，从而在过滤器链中实现对验证码的验证功能

```JAVA
public class MyAuthenticationProvider extends DaoAuthenticationProvider {

    @Override
    protected void additionalAuthenticationChecks(UserDetails userDetails, UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
        //获取当前请求
        HttpServletRequest req = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();
        //拿到用户传来的验证码
        String code = req.getParameter("code");
        //session中拿到生成的验证码字符串
        String verify_code = (String) req.getSession().getAttribute("verify_code");
        //比较二者
        if (code == null || verify_code == null || !code.equals(verify_code)) {
            throw new AuthenticationServiceException("验证码错误");
        }
        //调用父类的这个方法，其中进行密码的校验
        super.additionalAuthenticationChecks(userDetails, authentication);
    }
}
```

但是只定义了没有用，所有的 AuthenticationProvider 都是放在 ProviderManager 中统一管理的，需要把自定义的MyAuthenticationProvider注入ProviderManager。

```JAVA
    /**
    * @Description: 提供自定义实例
    * @Param: []
    * @return: com.example.demo.config.MyAuthenticationProvider
    * @Date: 2021/4/2
    */
    @Bean
    MyAuthenticationProvider myAuthenticationProvider() {
        MyAuthenticationProvider myAuthenticationProvider = new MyAuthenticationProvider();
        myAuthenticationProvider.setPasswordEncoder(passwordEncoder());
        myAuthenticationProvider.setUserDetailsService(userDetailsService());
        return myAuthenticationProvider;
    }

    /**
    * @Description: 把自定义的myAuthenticationProvider注入ProviderManager
    * @Param: []
    * @return: org.springframework.security.authentication.AuthenticationManager
    * @Date: 2021/4/2
    */
    @Override
    @Bean
    protected AuthenticationManager authenticationManager() throws Exception {
        ProviderManager manager = new ProviderManager(Arrays.asList(myAuthenticationProvider()));
        return manager;
    }
```

最后设置/vc.jpg所有人都可以访问，并且设置各种返回信息。

启动项目测试。

随便输一个验证码的话，提示错误

![image-20210402165940457](https://raw.githubusercontent.com/n1cef1sh/PhotoForBlog/main/img/image-20210402165940457.png)

然后get到/vc.jpg查看验证码

![image-20210402170008954](https://raw.githubusercontent.com/n1cef1sh/PhotoForBlog/main/img/image-20210402170008954.png)

输入正确的验证码登录

![image-20210402170044537](https://raw.githubusercontent.com/n1cef1sh/PhotoForBlog/main/img/image-20210402170044537.png)



## 关于登录用户详细信息





## 参考

[江南一点雨](http://itboyhub.com/category/springsecurity/)