---
title: SpringBoot 分布式工程--多数据源的解决方案
date: 2021-08-21 15:49:44.233
updated: 2022-04-27 16:35:24.287
url: /archives/springboot分布式工程--多数据源的解决方案
categories: 
- 数据库
tags: 
- Spring | 数据库
---



## SpringBoot 多數據源配置

核心问题：开发时候如果碰到需要在一个工程下调用多台服务器上的数据库服务 该如何解决？

解决方案：pringBoot2.x整合多数据源，基于注解动态切换数据源,主从复制，读写分离，多数据源的事务处理
这里介绍的是MYSQL的主从复制实现及其原理，数据源分为主从，主数据源用于写操作，从数据源用于读操作，实现了读写分离
[TOC]
### 一、  启动类文件配置数据库信息



```

spring.datasource.dynamic.datasource.saas.jdbcUrl=jdbc:mysql://数据库发布地址:3306/saas-risk-gateway?useUnicode=true&characterEncoding=utf8
spring.datasource.dynamic.datasource.saas.username=用户名
spring.datasource.dynamic.datasource.saas.password=密码
```

###  二、自定义切换数据源注解类

```
@Retention(RetentionPolicy.RUNTIME)
public @interface SwitchDataSource {

    @AliasFor("dataSource")
    String value() default DataSourceConstants.ORDER;

    @AliasFor("value")
    String dataSource() default DataSourceConstants.ORDER;
}
```

### 三、利用 AOP 实现注解拦截

```
@Slf4j
@Component
@Aspect
public class DataSourceSwitchAspect {

    @Pointcut("@annotation(com.dycjr.xiakuan.report.config.helper.SwitchDataSource)")
    private void pointCut(){

    }


    @Around(value = "pointCut() && @annotation(source)")
    public Object switchDataSource(ProceedingJoinPoint joinPoint, SwitchDataSource source) throws Throwable {
        // 切换数据源
        DynamicDataSourceContextHolder.setContextKey(source.dataSource());

        System.out.println("------------aspect-------------");

        try {
            return joinPoint.proceed(joinPoint.getArgs());
        } catch (Throwable throwable) {
            throw throwable;
        }finally {
            // 切回默认数据源
            DynamicDataSourceContextHolder.removeContextKey();
        }
    }
}
```

### 四、 数据源配置类

##### 4.1 配置数据源

    @Bean(value = DataSourceConstants.REPORT)
    // prefix 中为配置文件中 的 数据源信息 （发布地址、用户名、密码 其中驱动会有Spring Boot根据发布地址的url自动识别无需配置）前缀
    @ConfigurationProperties(prefix = "spring.datasource.dynamic.datasource.report")
    public DataSource dataSourceReport() {
        return DataSourceBuilder.create().build();
    }

##### 4.2 配置动态数据源

    @Bean
    @Primary
    public DataSource dynamicDataSource() {
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put(DataSourceConstants.REPORT, dataSourceReport());
        dataSourceMap.put(DataSourceConstants.ORDER, dataSourceOrder());
        dataSourceMap.put(DataSourceConstants.DEALER, dataSourceDealer());
        dataSourceMap.put(DataSourceConstants.SAAS, dataSourceSaas());
        //设置动态数据源
        // DynamicDataSource 为实现 了SpringFrameWork JDBC 包下的动态路由数据源自定义对象
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        dynamicDataSource.setTargetDataSources(dataSourceMap);
        dynamicDataSource.setDefaultTargetDataSource(dataSourceOrder());
    
        return dynamicDataSource;
    }



##### 4.3 配置 Mybatis 的SqlSessionFactory

    @Primary
    @Bean("sqlSessionFactoryReport")
    public SqlSessionFactory sqlSessionFactoryReport() throws Exception {
        MybatisSqlSessionFactoryBean factory = new MybatisSqlSessionFactoryBean();
        factory.setDataSource(dataSourceReport());
        factory.setVfs(SpringBootVFS.class);
        this.getBeanThen(TransactionFactory.class, factory::setTransactionFactory);
        MybatisConfiguration mybatisConfiguration = new MybatisConfiguration();
        mybatisConfiguration.addInterceptor(mybatisPlusInterceptor());
        factory.setConfiguration(mybatisConfiguration);
        return factory.getObject();
    }



##### 4.4配置MP 的分页拦截器

```
//    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return mybatisPlusInterceptor;
    }
```



##### 4.5 Mybatis SqlSessionTemplate 事务管理

    /**
     * 通过自定义
     * 允许在transactional注解的方法上，
     * 使用com.dycjr.xiakuan.report.config.helper.SwitchDataSource切换数据源
     */
    public class CustomSqlSessionTemplate extends SqlSessionTemplate {
        @Getter
        @Setter
        private Map<String, SqlSessionFactory> sessionFactoryMap;
    
        public CustomSqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
            super(sqlSessionFactory);
        }
    
        @Override
        public SqlSessionFactory getSqlSessionFactory() {
            SqlSessionFactory factory = this.sessionFactoryMap.get(DynamicDataSourceContextHolder.getContextKey());
            return factory == null ? super.getSqlSessionFactory() : factory;
        }
    }

##### 4.6 多数据源配置类总览

```
@Configuration
@MapperScan(value = "com.dycjr.xiakuan.report.mapper")
public class DataSourceConfig {
    private final ApplicationContext applicationContext;

    public DataSourceConfig(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
    }

    @Bean(value = DataSourceConstants.REPORT)
    @ConfigurationProperties(prefix = "spring.datasource.dynamic.datasource.report")
    public DataSource dataSourceReport() {
        return DataSourceBuilder.create().build();
    }

    @Bean(value = DataSourceConstants.ORDER)
    @ConfigurationProperties(prefix = "spring.datasource.dynamic.datasource.order")
    public DataSource dataSourceOrder() {
        return DataSourceBuilder.create().build();
    }

    @Bean(value = DataSourceConstants.DEALER)
    @ConfigurationProperties(prefix = "spring.datasource.dynamic.datasource.dealer")
    public DataSource dataSourceDealer() {
        return DataSourceBuilder.create().build();
    }
    //addby xk-342 小易分月度数据统计 wangpeng start
// 配置Saas 数据源
    @Bean(value = DataSourceConstants.SAAS)
    @ConfigurationProperties(prefix = "spring.datasource.dynamic.datasource.saas")
    public DataSource dataSourceSaas() {
        return DataSourceBuilder.create().build();
    }
    //addby xk-342 小易分月度数据统计 wangpeng end
    @Bean
    @Primary
    public DataSource dynamicDataSource() {
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put(DataSourceConstants.REPORT, dataSourceReport());
        dataSourceMap.put(DataSourceConstants.ORDER, dataSourceOrder());
        dataSourceMap.put(DataSourceConstants.DEALER, dataSourceDealer());
        dataSourceMap.put(DataSourceConstants.SAAS, dataSourceSaas());
        //设置动态数据源
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        dynamicDataSource.setTargetDataSources(dataSourceMap);
        dynamicDataSource.setDefaultTargetDataSource(dataSourceOrder());

        return dynamicDataSource;
    }

//    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
        mybatisPlusInterceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return mybatisPlusInterceptor;
    }

    public SqlSessionTemplate sqlSessionTemplate() throws Exception {
        HashMap<String, SqlSessionFactory> map = new HashMap<>(3);
        map.put(DataSourceConstants.ORDER, sqlSessionFactoryOrder());
        map.put(DataSourceConstants.DEALER, sqlSessionFactoryDealer());
        map.put(DataSourceConstants.REPORT, sqlSessionFactoryReport());
        map.put(DataSourceConstants.SAAS, sqlSessionFactorySaas());

        CustomSqlSessionTemplate template = new CustomSqlSessionTemplate(sqlSessionFactoryOrder());
        template.setSessionFactoryMap(map);

        return template;
    }

    @Primary
    @Bean("sqlSessionFactoryReport")
    public SqlSessionFactory sqlSessionFactoryReport() throws Exception {
        MybatisSqlSessionFactoryBean factory = new MybatisSqlSessionFactoryBean();
        factory.setDataSource(dataSourceReport());
        factory.setVfs(SpringBootVFS.class);
        this.getBeanThen(TransactionFactory.class, factory::setTransactionFactory);
        MybatisConfiguration mybatisConfiguration = new MybatisConfiguration();
        mybatisConfiguration.addInterceptor(mybatisPlusInterceptor());
        factory.setConfiguration(mybatisConfiguration);
        return factory.getObject();
    }

    private <T> void getBeanThen(Class<T> clazz, Consumer<T> consumer) {
        if (this.applicationContext.getBeanNamesForType(clazz, false, false).length > 0) {
            consumer.accept(this.applicationContext.getBean(clazz));
        }

    }

    @Bean("sqlSessionFactoryOrder")
    public SqlSessionFactory sqlSessionFactoryOrder() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceOrder());
        return factoryBean.getObject();
    }

    @Bean("sqlSessionFactoryDealer")
    public SqlSessionFactory sqlSessionFactoryDealer() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceDealer());
        return factoryBean.getObject();
    }
//addby xk-342 小易分月度数据统计 wangpeng start
// 配置Saas 数据源
    @Bean("sqlSessionFactorySaas")
    public SqlSessionFactory sqlSessionFactorySaas() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceSaas());
        return factoryBean.getObject();
    }
//addby xk-342 小易分月度数据统计 wangpeng end

    /**
     * 通过自定义
     * 允许在transactional注解的方法上，
     * 使用com.dycjr.xiakuan.report.config.helper.SwitchDataSource切换数据源
     */
    public class CustomSqlSessionTemplate extends SqlSessionTemplate {
        @Getter
        @Setter
        private Map<String, SqlSessionFactory> sessionFactoryMap;

        public CustomSqlSessionTemplate(SqlSessionFactory sqlSessionFactory) {
            super(sqlSessionFactory);
        }

        @Override
        public SqlSessionFactory getSqlSessionFactory() {
            SqlSessionFactory factory = this.sessionFactoryMap.get(DynamicDataSourceContextHolder.getContextKey());
            return factory == null ? super.getSqlSessionFactory() : factory;
        }
    }

}
```

### 五、设定数据源枚举类

需要切换到的数据源枚举  

也可以使用枚举类来实现

```
public class DataSourceConstants {

    public static final String ORDER = "order";

    public static final String DEALER = "dealer";

    public static final String REPORT = "report";

    public static final String SAAS = "saas";
}
```

### 六、 配置动态数据源路由

集成SpringBoot jdbc包下的动态路由数据源类

```
public class DynamicDataSource extends AbstractRoutingDataSource {

    @Override protected Object determineCurrentLookupKey() {
        return DynamicDataSourceContextHolder.getContextKey();
    }
}
```

### 七、使用 ThreadLocal 配置数据源的上下文环境

需要实现的功能

##### 7.1 切换当前 Serivce 线程的数据源

##### 7.2 设定默认的线程数据源

##### 7.3 清除当前线程的数据源

```
public class DynamicDataSourceContextHolder {

    /**
     * 动态数据源名称上下文
     */
    private static final ThreadLocal<String> DATASOURCE_CONTEXT_KEY_HOLDER = new ThreadLocal<>();

    /**
     * 设置/切换数据源
     */
    public static void setContextKey(String key) {
        DATASOURCE_CONTEXT_KEY_HOLDER.set(key);
    }

    /**
     * 获取数据源名称
     * 默认： order
     */
    public static String getContextKey() {
        String key = DATASOURCE_CONTEXT_KEY_HOLDER.get();
        return key == null ? DataSourceConstants.ORDER : key;
    }

    /**
     * 删除当前数据源名称
     */
    public static void removeContextKey() {
        DATASOURCE_CONTEXT_KEY_HOLDER.remove();
    }
}
```