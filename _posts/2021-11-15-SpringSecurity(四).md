---
layout: post
title: "SpringSecurity系列(四)"
categories: [SpringSecurity]

---

练习代码同步到github：SpringSecuritySamples/spring-security和withJPA

要求是同一个用户不能同时在多个设备上登录系统。

如果已登录，再在另一个设备上登录系统时，有两种处理方式：

- 踢掉前一个已登录用户
- 禁止后来的登录请求

### 踢掉已登录用户

在之前的SecurityConfig基础配置上只需要增加一个配置。

```JAVA
@Override
    protected void configure(HttpSecurity http) throws Exception{
        http.authorizeRequests()
                .antMatchers("/admin/**").hasRole("admin")
                .antMatchers("/user/**").hasRole("user")
                .anyRequest().authenticated()
				........
                ........
            	//新增
                .and()
                .sessionManagement()
                .maximumSessions(1);
    }
```

也就是session管理，最大会话数设置为1。

在edge浏览器上登录成功后，访问/hello正常。

在ie浏览器上登陆成功后，访问/hello正常。

此时再在edge上访问/hello时，则会提示下图。

![](https://raw.githubusercontent.com/n1cef1sh/PhotoForBlog/main/img/20211115150600.png)



### 禁止后来者登录

这个配置也不复杂，还是修改SecurityConfig，添加maxSessionsPreventsLogin配置即可。

```JAVA
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .anyRequest().authenticated()
            .and()
            .formLogin()
            .loginPage("/login.html")
            .permitAll()
            .and()
            .csrf().disable()
            .sessionManagement()
            .maximumSessions(1)
            .maxSessionsPreventsLogin(true);
}
```

这样再从第二个设备登录系统时就会提示该用户会话数已达最大数，无法登录。

![](https://raw.githubusercontent.com/n1cef1sh/PhotoForBlog/main/img/20211115152446.png)

但是这样并不完善，如果用户在前一个登录设备上注销登录后，再从新设备登陆系统时，仍会出现同样的提示。哪怕是在同一个设备注销登录后重新登录，也是会出现无法登录的情况。

SpringSecurity是通过监听session的销毁事件，来清理session记录。上述问题说明注销登录后session并没有及时地清理掉，所以导致无法正常的登陆系统。

教程里解释说：

```
当用户注销登录之后，session 就会失效，但是默认的失效是通过调用 StandardSession#invalidate 方法来实现的，这一个失效事件无法被 Spring 容器感知到，进而导致当用户注销登录之后，Spring Security 没有及时清理会话信息表，以为用户还在线，进而导致用户无法重新登录进来
```

没有细究源码，大概意思应该是默认的处理方法和springsecurity的配合并不理想。

既然如此，就需要我们添加一个可以感应到session销毁事件的东西。

在SecurityConfig添加一个Bean，这个类实现了 HttpSessionListener 接口，可以将 session 创建以及销毁的事件及时感知到，并且调用 Spring 中的事件机制将相关的创建和销毁事件发布出去，进而被 Spring Security 感知到。

```JAVA
    @Bean
    HttpSessionEventPublisher httpSessionEventPublisher(){
        return new HttpSessionEventPublisher();
    }
```

这样当前一个用户注销登录后，session就会被及时清理，不影响后续的登录操作。



### 存到数据库的用户

上述测试使用基于内存的用户没有问题，但是如果使用存到数据库的用户时则无法实现上述效果。此处在之前withJPA基础上进行修改测试。

Spring Security 中通过 SessionRegistryImpl 类来实现对会话信息的统一管理。

```JAVA
	/** <principal:Object,SessionIdSet> */
	private final ConcurrentMap<Object, Set<String>> principals;
	/** <sessionId:Object,SessionInformation> */
	private final Map<String, SessionInformation> sessionIds;
```

第一个ConcurrentMap里的key也就是用户主体principal，这里存在一个问题。

那就是hashmap中如果**用对象做key**，需要注意什么？

关于这个问题，单独做一篇学习记录：[Hashmap中用对象作为key的几点问题 | n1cef1sh’s Blog (nicefish.xyz)](https://www.nicefish.xyz/posts/2021/11/16/Hashmap中用对象作为key的几点问题.html)

回到之前的问题，当使用基于内存的用户时，框架源码其实已经重写了这两个方法，所以不会出现问题。

```JAVA
public class User implements UserDetails, CredentialsContainer {
	private String password;
	private final String username;
	private final Set<GrantedAuthority> authorities;
	private final boolean accountNonExpired;
	private final boolean accountNonLocked;
	private final boolean credentialsNonExpired;
	private final boolean enabled;
	@Override
	public boolean equals(Object rhs) {
		if (rhs instanceof User) {
			return username.equals(((User) rhs).username);
		}
		return false;
	}
	@Override
	public int hashCode() {
		return username.hashCode();
	}
}
```

那我们就在自定义的User类里重写hashcode和equals方法，建议右键generate根据配置自动生成，自己写也容易漏写或写错。

![image-20211115193637860](C:\Users\pYq\AppData\Roaming\Typora\typora-user-images\image-20211115193637860.png)

```JAVA
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return accountNonExpired == user.accountNonExpired && accountNonLocked == user.accountNonLocked && credentialsNonExpired == user.credentialsNonExpired && enabled == user.enabled && Objects.equals(id, user.id) && Objects.equals(username, user.username) && Objects.equals(password, user.password) && Objects.equals(roles, user.roles);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, username, password, accountNonExpired, accountNonLocked, credentialsNonExpired, enabled, roles);
    }
```

完成这些配置后，再去测试多端登录，就可以实现之前的效果了。