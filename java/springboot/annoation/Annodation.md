springboot使用Annocatin代替原来的繁琐的xml配置

## @SpringBootApplication

用来表示一个spring boot的应用入口程序

```java
@SpringBootApplication
class VehicleFactoryApplication {
    public static void main(String[] args) {
        SpringApplication.run(VehicleFactoryApplication.class, args);
    }
}
```

SpringbootApplication其实是封装了@Configuration，@EnableAutoConfiguration，@ComponentScan的Annotation

## @EnableAutoConfiguation

表示用自动配置的Bean

```Java
@Configuration
@EnableAutoConfiguration
class VehicleFactoryConfig {}
```

在自动装配的bean中我们需要使用到

### @Configuration

```java
@Configuration
public class MySQLAutoconfiguration {
}
```

在默写情况下，我们需要生成bean需要一些先决条件，例如需要一些其他的class或者bean，这个时候我们就需要

### @ConditionalOnClass

```JAVA
@Configuration
@ConditionalOnClass(DataSource.class)
public class MySQLAutoconfiguration {
    //...
}
```

### @ConditionalOnBean

```JAVA
@Bean
@ConditionalOnBean(name = "dataSource")
@ConditionalOnMissingBean
public LocalContainerEntityManagerFactoryBean entityManagerFactory() {
    LocalContainerEntityManagerFactoryBean em
      = new LocalContainerEntityManagerFactoryBean();
    em.setDataSource(dataSource());
    em.setPackagesToScan("com.baeldung.autoconfiguration.example");
    em.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    if (additionalProperties() != null) {
        em.setJpaProperties(additionalProperties());
    }
    return em;
}
```

### @ConditionOnMissBean

表示如果bean没有情况下会执行

```JAVA
@Bean
@ConditionalOnMissingBean(type = "JpaTransactionManager")
JpaTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
    JpaTransactionManager transactionManager = new JpaTransactionManager();
    transactionManager.setEntityManagerFactory(entityManagerFactory);
    return transactionManager;
}
```

当然我们生成bean的时候有的时候需要一些前置的配置文件如果没有这些配置文件则不需要产生bean

### @PropertySource

```JAVA
@PropertySource("classpath:mysql.properties")
public class MySQLAutoconfiguration {
    //...
}
```

如果有配置文件但是我们需要更确切的那些key是有值的情况才能生成bean

```JAVA
@Bean
@ConditionalOnProperty(name = "usemysql", havingValue = "local")
@ConditionalOnMissingBean
public DataSource dataSource() {
    DriverManagerDataSource dataSource = new DriverManagerDataSource();
    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
    dataSource.setUrl("jdbc:mysql://localhost:3306/myDb?createDatabaseIfNotExist=true");
    dataSource.setUsername("mysqluser");
    dataSource.setPassword("mysqlpass");
    return dataSource;
}
```

如果我们不想让某些路径下的类自动加载bean，可以使用@EnableAutoConfiguration(exclude=)

```java
@Configuration
@EnableAutoConfiguration(
  exclude={MySQLAutoconfiguration.class})
public class AutoconfigurationApplication {
    //...
}
```

## @ConditionalOnWebApplication

表示当前非webapplication，才能运行

```JAVA
@ConditionalOnWebApplication
HealthCheckController healthCheckController() {
    // ...
}
```

### @ConditionalOnNotWebApplication

这个就不解释

### @ConditionalExpression

```JAVA
@Bean
@ConditionalOnExpression("${usemysql} && ${mysqlserver == 'local'}")
DataSource dataSource() {
    // ...
}
```

### @Conditional

```JAVA
@Conditional(HibernateCondition.class)
Properties additionalProperties() {
    //...
}
```

