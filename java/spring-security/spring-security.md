### 参考文档

[Spring Security 入门原理及实战 - 逃离沙漠 - 博客园](https://www.cnblogs.com/demingblog/p/10874753.html)

### 遇到的问题

springboot中导入spring-boot-starter-security包，会自动启动认证框架配置，自己覆盖spring security配置才能让访问成功，否则默认的配置让客户端无法通过认证？



@Autowired自动注入是如何按类型注入的？容器中有每个bean实现的接口或者说继承的父类名字，按那个名字来的吗？

bean实现的接口或者继承的类就是该bean的类型，一个bean可以对应多个类型。通过beanFactory的getBean(Class ...)可以获取指定类型的实现或子类，**判断大致过程就是遍历容器中的bean,检测是否是指定类型的子类**



@ConditionalOnBean中源码

```java
	/**
	这里的The class types of beans指的是继承的类型或者接口，
	 * The class types of beans that should be checked. The condition matches when no bean
	 * of each class specified is contained in the {@link BeanFactory}.
	 * @return the class types of beans to check
	 */
	Class<?>[] value() default {};
```

`@ConditionalOnMissingBean(WebSecurityConfigurerAdapter.class)`，如果自己写代码继承了`WebSecurityConfigurerAdapter`类并且加入到容器中，如上注解的类就不会被加载到容器中。通过该类型查找其子类或实现类是否在容器中就能判断容器是否有该类型bean





### 该技术是什么？

安全认证框架

### 该技术有什么特点(使用注意)：



### 该技术怎么使用。demo

**环境：**springboot 2.3.x ,直接导入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

#### 默认启动的过程

直接启动系统，springboot自动创建如下类

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(AuthenticationManager.class)
@ConditionalOnBean(ObjectPostProcessor.class)
@ConditionalOnMissingBean(
		value = { AuthenticationManager.class, AuthenticationProvider.class, UserDetailsService.class },
		type = { "org.springframework.security.oauth2.jwt.JwtDecoder",
				"org.springframework.security.oauth2.server.resource.introspection.OpaqueTokenIntrospector" })
public class UserDetailsServiceAutoConfiguration {

	private static final String NOOP_PASSWORD_PREFIX = "{noop}";

	private static final Pattern PASSWORD_ALGORITHM_PATTERN = Pattern.compile("^\\{.+}.*$");

	private static final Log logger = LogFactory.getLog(UserDetailsServiceAutoConfiguration.class);

	@Bean
	@ConditionalOnMissingBean(
			type = "org.springframework.security.oauth2.client.registration.ClientRegistrationRepository")
	@Lazy
	public InMemoryUserDetailsManager inMemoryUserDetailsManager(SecurityProperties properties,
			ObjectProvider<PasswordEncoder> passwordEncoder) {
		SecurityProperties.User user = properties.getUser();
		List<String> roles = user.getRoles();
		return new InMemoryUserDetailsManager(
            //这里的User是框架认证需要的类，实现了UserDetails接口
				User.withUsername(user.getName()).password(getOrDeducePassword(user, passwordEncoder.getIfAvailable()))
						.roles(StringUtils.toStringArray(roles)).build());
	}

	private String getOrDeducePassword(SecurityProperties.User user, PasswordEncoder encoder) {
		String password = user.getPassword();
		if (user.isPasswordGenerated()) {
            //控制台打印密码
			logger.info(String.format("%n%nUsing generated security password: %s%n", user.getPassword()));
		}
		if (encoder != null || PASSWORD_ALGORITHM_PATTERN.matcher(password).matches()) {
			return password;
		}
		return NOOP_PASSWORD_PREFIX + password;
	}

}
```



可以配置默认的user属性，如用户名，密码，角色

```java
@ConfigurationProperties(prefix = "spring.security")
public class SecurityProperties {
	//...其他代码省略
    
    
    public static class User {

		/**
		 * Default user name.
		 */
		private String name = "user";

		/**
		 * Password for the default user name.
		 * 随机生成uuid作为密码
		 */
		private String password = UUID.randomUUID().toString();

		/**
		 * Granted roles for the default user name.
		 */
		private List<String> roles = new ArrayList<>();

		private boolean passwordGenerated = true;

		public String getName() {
			return this.name;
		}

		public void setName(String name) {
			this.name = name;
		}

		public String getPassword() {
			return this.password;
		}

		public void setPassword(String password) {
			if (!StringUtils.hasLength(password)) {
				return;
			}
			this.passwordGenerated = false;
			this.password = password;
		}

		public List<String> getRoles() {
			return this.roles;
		}

		public void setRoles(List<String> roles) {
			this.roles = new ArrayList<>(roles);
		}

		public boolean isPasswordGenerated() {
			return this.passwordGenerated;
		}

	}
}
```

**注意**：这里的User是配置类内部定义的，用来读取配置的User信息，而spring security框架中定义的User继承UserDetails，是认证框架需要的类。

> If you define a `@Configuration` with a `WebSecurityConfigurerAdapter` in your application, it switches off the default webapp security settings in Spring Boot.
>
> 如果你没有配置该类`WebSecurityConfigurerAdapter`，springboot会自动配置，否则会关闭springboot的默认配置
>
> If you provide a `@Bean` of type `AuthenticationManager`, `AuthenticationProvider`, or `UserDetailsService`, the default `@Bean` for `InMemoryUserDetailsManager` is not created. 

#### 如何完成认证？流程？







### 该技术什么时候用？test。