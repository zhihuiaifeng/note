springboot 读取yam

```
这是一种，读取配置字段进行映射，微信商城在Javademo中
@Data
@ConfigurationProperties(prefix = "projectUrl")
@Component 不存在bean
public class ProjectUrlConfig {

    /**
     * 微信公众平台授权url
     */
    public String wechatMpAuthorize;

    /**
     * 微信开放平台授权url
     */
    public String wechatOpenAuthorize;

    /**
     * 点餐系统
     */
    public String sell;
}
```



读取德鲁伊数据源，miaosha

@Configuration
@ConfigurationProperties(prefix="spring.datasource")
public class DruidConfig {

	private String url;
	private String username;
	private String password;
	private String driverClassName;
	private String type;
	private String filters;
	private int maxActive;
	private int initialSize;
	private int minIdle;
	private long maxWait;
	private long timeBetweenEvictionRunsMillis;
	private long minEvictableIdleTimeMillis;
	private String validationQuery;
	private boolean testWhileIdle;
	private boolean testOnBorrow;
	private boolean testOnReturn;
	private boolean poolPreparedStatements;
	private int maxOpenPreparedStatements;
	
	@Bean
	public ServletRegistrationBean druidSverlet() {
		ServletRegistrationBean reg = new ServletRegistrationBean();
		reg.setServlet(new StatViewServlet());
		reg.addUrlMappings("/druid/*");
		reg.addInitParameter("loginUsername", "joshua");
		reg.addInitParameter("loginPassword", "123456");
		reg.addInitParameter("logSlowSql", "true");
		reg.addInitParameter("slowSqlMillis", "1000");
		return reg;
	}
	
	@Bean
	public DataSource druidDataSource() {
		 	DruidDataSource datasource = new DruidDataSource();
	        datasource.setUrl(url);
	        datasource.setUsername(username);
	        datasource.setPassword(password);
	        datasource.setDriverClassName(driverClassName);
	        datasource.setInitialSize(initialSize);
	        datasource.setMinIdle(minIdle);
	        datasource.setMaxActive(maxActive);
	        datasource.setMaxWait(maxWait);
	        datasource.setTimeBetweenEvictionRunsMillis(timeBetweenEvictionRunsMillis);
	        datasource.setMinEvictableIdleTimeMillis(minEvictableIdleTimeMillis);
	        datasource.setValidationQuery(validationQuery);
	        datasource.setTestWhileIdle(testWhileIdle);
	        datasource.setTestOnBorrow(testOnBorrow);
	        datasource.setTestOnReturn(testOnReturn);
	        datasource.setPoolPreparedStatements(poolPreparedStatements);
	        datasource.setMaxOpenPreparedStatements(maxOpenPreparedStatements);
	        try {
	            datasource.setFilters(filters);
	        } catch (SQLException e) {
	            e.printStackTrace();
	        }
	        return datasource;
	}
	}
```
如果使用Redis就不存在配置的问题了
	@Autowired
	RedisTemplate redisTemplate;


jwtshiro里面的 jedispool的配置。
Jedis配置，项目启动注入JedisPool
@Configuration
@EnableAutoConfiguration
@PropertySource("classpath:config.properties")
@ConfigurationProperties(prefix = "redis")
public class JedisConfig {

    /**
     * LOGGER
     */
    private static final Logger LOGGER = LoggerFactory.getLogger(JedisConfig.class);

    private String host;

    private int port;

    private String password;

    private int timeout;

    @Value("${redis.pool.max-active}")
    private int maxActive;

    @Value("${redis.pool.max-wait}")
    private int maxWait;

    @Value("${redis.pool.max-idle}")
    private int maxIdle;

    @Value("${redis.pool.min-idle}")
    private int minIdle;

    @Bean
    public JedisPool redisPoolFactory() {
        try {
            JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
            jedisPoolConfig.setMaxIdle(maxIdle);
            jedisPoolConfig.setMaxWaitMillis(maxWait);
            jedisPoolConfig.setMaxTotal(maxActive);
            jedisPoolConfig.setMinIdle(minIdle);
            // JedisPool jedisPool = new JedisPool(jedisPoolConfig, host, port, timeout, password);
            String pwd = StringUtil.isBlank(password) ? null : password;
            JedisPool jedisPool = new JedisPool(jedisPoolConfig, host, port, timeout, pwd);
            LOGGER.info("初始化Redis连接池JedisPool成功!地址: " + host + ":" + port);
            return jedisPool;
        } catch (Exception e) {
            LOGGER.error("初始化Redis连接池JedisPool异常:" + e.getMessage());
        }
        return null;
    }
```

这个也是这种配置不带bean的使用component的形式进行的

@ConfigurationProperties(prefix = "jwt")
@Component
public class JwtConfig {

    private long time;     // 5天(以秒s计)过期时间
    private String secret;// JWT密码
    private String prefix ;         // Token前缀
    private String header; // 存放Token的Header Key
    
    public long getTime() {
        return time;
    }
    
    public void setTime(long time) {
        this.time = time;
    }
    
    public String getSecret() {
        return secret;
    }
    
    public void setSecret(String secret) {
        this.secret = secret;
    }
    
    public String getPrefix() {
        return prefix;
    }
    
    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }
    
    public String getHeader() {
        return header;
    }
    
    public void setHeader(String header) {
        this.header = header;
    }
}

```
这个是热部署的方式进行的，在jwt-token这个框架里面使用的
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-devtools</artifactId>
   <optional>true</optional>
</dependency>
```