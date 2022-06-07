```
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

我们来看SpringBootApplication的注解

SpringBootConfiguration其实就是@Configuration，

EnableAutoConfiguration是我们实现自动装配的

ComponentScan

