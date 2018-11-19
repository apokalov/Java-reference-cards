## Spring Core annotations

#### @Configuration
`@Configuration` is used on classes which define beans.
Java class annotated with `@Configuration` is a configuration by itself and will have methods to instantiate and configure the dependencies.

Here is an example:

``` java
@Configuration
public class DataConfig{ 
  @Bean
  public DataSource source(){
    DataSource source = new OracleDataSource();
    source.setURL();
    source.setUser();
    return source;
  }
  @Bean
  public PlatformTransactionManager manager(){
    PlatformTransactionManager manager = new BasicDataSourceTransactionManager();
    manager.setDataSource(source());
    return manager;
  }
}
```

#### @Bean
`@Bean` is used  inside the `@Configuration` class on methods that instantiate and configure beans. The method annotated with this annotation works as bean ID.

```java
@Configuration
public class AppConfig{
  @Bean
  public Person person(){
    return new Person(address());
  }
}
```

#### @ComponentScan
This annotation is used with `@Configuration` annotation to allow Spring to know the packages to scan for annotated components. `@ComponentScan` is also used to specify base packages using `basePackageClasses` or `basePackage` attributes to scan. If specific packages are not defined, scanning will occur from the package of the class that declares this annotation.

#### @Autowired (JSR-330 @Inject)
This annotation is applied on fields, setter methods, and constructors. The `@Autowired` annotation injects object dependency implicitly.

When you use `@Autowired` on setter methods, Spring tries to perform the by Type autowiring on the method.

When you use `@Autowired` on a constructor, constructor injection happens at the time of object creation. It indicates the constructor to autowire when used as a bean. One thing to note here is that only one constructor of any bean class can carry the `@Autowired`  annotation.

NOTE: As of Spring 4.3, `@Autowired`  became optional on classes with a single constructor.

#### @Required
The @Required annotation applies to bean property setter methods, as in the following example:
```java
public class SimpleMovieLister {
    private MovieFinder movieFinder;

    @Required
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```
This annotation indicates that the affected bean property must be populated at configuration time, through an explicit property value in a bean definition or through autowiring. The container throws an exception if the affected bean property has not been populated.

#### @Qualifier
This annotation is used along with `@Autowired` annotation. This annotation is used to avoid confusion which occurs when you create more than one bean of the same type and want to wire only one of them with a property:
```java

@Configuration
public class AwsSNSConfiguration {

    @Bean
    public AmazonSNS notificationsSns(@Value("${aws.default.region}") String awsRegion){
        return AmazonSNSClientBuilder
                .standard()
                .withRegion(awsRegion)
                .build();
    }
}

@Component
public class SMSGateway {
    private final AmazonSNS amazonSNS;

    public SMSGateway(@Qualifier("notificationsSns") AmazonSNS amazonSNS) {
        this.amazonSNS = amazonSNS;
    }
}
```

#### @Value
This annotation is used at the field, constructor parameter, and method parameter level. The `@Value` annotation indicates a default value expression for the field or parameter to initialize the property with. As the `@Autowired` annotation tells Spring to inject object into another when it loads your application context, you can also use @Value annotation to inject values from a property file into a bean’s attribute. 

It supports both `#{...}` and `${...}` placeholders.
