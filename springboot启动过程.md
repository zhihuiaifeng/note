```
/**
springboot启动过程产生的６种事件.通过ApplicationReadyEvent事件可以实现系统启动完后做一些系统初始化的操作．接下来讲讲通过ApplicationRunner(CommandLineRunner也类似)这种方式也可以实现同样的功能．
 * Created by l2h on 18-4-16.
 * Desc: 系统启动完可以做一些业务操作
 * @author l2h
 */
@Component
//如果有多个runner需要指定一些顺序
@Order(1)
public class SimosApplicationRunner implements ApplicationRunner {
@Autowired
SystemInitService systemInitService;
@Override
public void run(ApplicationArguments args) throws Exception {
    systemInitService.systemInit();
}
}

@SpringBootApplication 
 2 public class XdclassApplication extends SpringBootServletInitializer {
 3 
 4     @Override
 5     protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
 6         return application.sources(XdclassApplication.class);
 7     }
 8 
 9     public static void main(String[] args) throws Exception {
10         SpringApplication.run(XdclassApplication.class, args);
11     }
12 
13 }
在pom.xml中将打包形式 jar 修改为war <packaging>war</packaging>

构建项目名称 <finalName>xdclass_springboot</finalName>
```