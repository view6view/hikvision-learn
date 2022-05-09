# 多数据源切换demo

- 导入依赖
  - 导入`spring-boot-configuration-processor`注解处理器依赖的主要功能主要是在编译阶段通过修改配置指定不同的数据源
  - ORM插件可选Mybatis-Plus和JPA，这里演示的是Mybatis-Plus，JPA需要修改对应的代码

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>

        <!--<dependency>-->
            <!--<groupId>org.springframework.boot</groupId>-->
            <!--<artifactId>spring-boot-starter-data-jpa</artifactId>-->
        <!--</dependency>-->

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>

        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.1.2</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.12</version>
        </dependency>
```

- 创建两个不同的数据库`test1`和`test2`，分别创建`user`表和`person`表

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `password` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=16 DEFAULT CHARSET=utf8

CREATE TABLE `person` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `password` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=16 DEFAULT CHARSET=utf8
```

- 编写配置文件，分别连接不同的数据库

```sql
spring.datasource1.jdbcurl=jdbc:mysql://YOUR_HOST:3306/test1?useSSL=false
spring.datasource1.username=YOUR_USERNAME
spring.datasource1.password=YOUR_PASSWORD
spring.datasource1.driver-class-name=com.mysql.cj.jdbc.Driver

spring.datasource2.jdbcurl=jdbc:mysql://YOUR_HOST:3306/test2?useSSL=false
spring.datasource2.username=YOUR_USERNAME
spring.datasource2.password=YOUR_PASSWORD
spring.datasource2.driver-class-name=com.mysql.cj.jdbc.Driver
```

- 分别定义两个实体对象，这个很简单

```java
import lombok.Data;

@Data
public class User {

    private String id;

    private String name;

    private String password;

}
```

```java
import lombok.Data;

@Data
public class Person {

    private String id;

    private String name;

    private String password;

}
```

- 编写2个配置类，读取不同的数据源，指定对应的mapper文件

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

@Slf4j
@Configuration
// basePackages指定对应mapper文件的url，定义sqlSessionFactory1的名字用于区分
@MapperScan(basePackages = {"com.example.demo.mapper.source1"}, sqlSessionFactoryRef = "sqlSessionFactory1")
public class DataSource1Config {

    // 指定数据源配置路径的前缀，用于区分数据源
    @ConfigurationProperties(prefix = "spring.datasource1")
    @Bean
    public DataSource dataBase1DataSource() {
        log.info("dataBase1DataSource初始化");
        return DataSourceBuilder.create().build();
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory1(DataSource dataBase1DataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataBase1DataSource);
        return sessionFactory.getObject();
    }

    @Bean
    public DataSourceTransactionManager dataSource1TransactionManager(DataSource dataBase1DataSource) {
        return new DataSourceTransactionManager(dataBase1DataSource);
    }

}
```

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.jdbc.DataSourceBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.datasource.DataSourceTransactionManager;

import javax.sql.DataSource;

@Slf4j
@Configuration
// basePackages指定对应mapper文件的url，定义sqlSessionFactory2的名字用于区分
@MapperScan(basePackages = {"com.example.demo.mapper.source2"}, sqlSessionFactoryRef = "sqlSessionFactory2")
public class DataSource2Config {

    // 指定数据源配置路径的前缀，用于区分数据源
    @ConfigurationProperties(prefix = "spring.datasource2")
    @Bean
    public DataSource dataBase2DataSource() {
        log.info("dataBase2DataSource初始化");
        return DataSourceBuilder.create().build();
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory2(DataSource dataBase2DataSource) throws Exception {
        final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataBase2DataSource);
        return sessionFactory.getObject();
    }

    @Bean
    public DataSourceTransactionManager dataSource2TransactionManager(DataSource dataBase2DataSource) {
        return new DataSourceTransactionManager(dataBase2DataSource);
    }

}
```

- 编写两个mapper接口，注意mapper接口的包路径要和前面配置类的包路径想对应，也不要搞反了也会报错

```java
import com.example.demo.entity.User;
import org.apache.ibatis.annotations.*;

import java.util.List;

@Mapper
public interface UserMapper {

    @Insert("insert into user(name,password) values(#{name},#{password})")
    @Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
    int insert(User user);

    @Select("select * from user")
    List<User> selectAll();

    @Delete("delete from user")
    void delete();

}
```

```java
import com.example.demo.entity.Person;
import org.apache.ibatis.annotations.*;

import java.util.List;

@Mapper
public interface PersonMapper {

    @Insert("insert into person(name,password) values(#{name},#{password})")
    @Options(useGeneratedKeys = true, keyProperty = "id", keyColumn = "id")
    int insert(Person person);

    @Select("select * from person")
    List<Person> selectAll();

    @Delete("delete from person")
    void delete();

}
```

- 编写主启动类，重写run方法，项目启动的时候进行测试

```java
import com.example.demo.entity.Person;
import com.example.demo.entity.User;
import com.example.demo.mapper.source1.UserMapper;
import com.example.demo.mapper.source2.PersonMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * <p>
 * 静态绑定数据源示例
 * </p>
 * @author: zhu.chen
 * @date: 2020/1/17
 * @version: v1.0.0
 */
@SpringBootApplication
public class DemoApplication implements CommandLineRunner {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private PersonMapper personMapper;

    @Override
    public void run(String... args) throws Exception {
        userMapper.delete();
        for (int i = 0; i < 5; i++) {
            User user = new User();
            user.setName("张三" + i);
            user.setPassword("abc" + i);
            userMapper.insert(user);
        }
        System.out.println("user : " + userMapper.selectAll());

        personMapper.delete();
        for (int i = 0; i < 5; i++) {
            Person person = new Person();
            person.setName("小明" + i);
            person.setPassword("efg" + i);
            personMapper.insert(person);
        }
        System.out.println("person : " + personMapper.selectAll());
    }

}
```

- 最终结果就是操作了不同的数据源

# 动态数据源切换demo

- 导入依赖
  - 所需要的依赖和`多数据源切换demo`差不多

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-configuration-processor</artifactId>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
		</dependency>

		<dependency>
			<groupId>com.baomidou</groupId>
			<artifactId>mybatis-plus-boot-starter</artifactId>
			<version>3.1.2</version>
		</dependency>

		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>8.0.12</version>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
			<exclusions>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
```

- 数据库也和`多数据源切换demo`一样

- 修改配置文件

```java
# master DB properties:
spring.datasource.default.url=jdbc:mysql://YOUR_HOST:3306/test1?useSSL=false
spring.datasource.default.username=YOUR_USERNAME
spring.datasource.default.password=YOUR_PASSWORD
spring.datasource.default.driver-class-name=com.mysql.cj.jdbc.Driver

# slave DB properties
spring.datasource.master.url=jdbc:mysql://YOUR_HOST:3306/test2?useSSL=false
spring.datasource.master.username=YOUR_USERNAME
spring.datasource.master.password=YOUR_PASSWORD
spring.datasource.master.driver-class-name=com.mysql.cj.jdbc.Driver
```

- 编写两个实体类，和`多数据源切换demo`一样
- 编写一个静态常量类，读取数据库配置

```java
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class DataSourceProperties implements InitializingBean {
    @Value("${spring.datasource.default.url}")
    private String defaultDBUrl;
    @Value("${spring.datasource.default.username}")
    private String defaultDBUser;
    @Value("${spring.datasource.default.password}")
    private String defaultDBPassword;
    @Value("${spring.datasource.default.driver-class-name}")
    private String defaultDBDriverName;

    @Value("${spring.datasource.master.url}")
    private String masterDBUrl;
    @Value("${spring.datasource.master.username}")
    private String masterDBUser;
    @Value("${spring.datasource.master.password}")
    private String masterDBPassword;
    @Value("${spring.datasource.default.driver-class-name}")
    private String masterDBDriverName;

    public static String DEFAULT_DB_URL;
    public static String DEFAULT_DB_USER;
    public static String DEFAULT_DB_PASSWORD;
    public static String DEFAULT_DB_DRIVER_NAME;

    public static String MASTER_DB_URL;
    public static String MASTER_DB_USER;
    public static String MASTER_DB_PASSWORD;
    public static String MASTER_DB_DRIVER_NAME;

    @Override
    public void afterPropertiesSet() {
        this.DEFAULT_DB_URL = defaultDBUrl;
        this.DEFAULT_DB_USER = defaultDBUser;
        this.DEFAULT_DB_PASSWORD = defaultDBPassword;
        this.DEFAULT_DB_DRIVER_NAME = defaultDBDriverName;

        this.MASTER_DB_URL = masterDBUrl;
        this.MASTER_DB_USER = masterDBUser;
        this.MASTER_DB_PASSWORD = masterDBPassword;
        this.MASTER_DB_DRIVER_NAME = masterDBDriverName;
    }
}
```

- 编写一个配置类，启动程序的时候默认先加载默认数据库

```java
import com.example.demo1.datasource.DynamicDataSource;
import com.zaxxer.hikari.HikariDataSource;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

@Configuration
@MapperScan(basePackages = {"com.example.demo1.mapper"}, sqlSessionFactoryRef = "sqlSessionFactory")
public class DataSourceConfig {
    /**
     * 因为配置类@Configuration的加载顺序比组件类@Component的顺序要早，所以不也能通过DataSourceProperties读取配置
     */
    @Value("${spring.datasource.default.url}")
    private String defaultDBUrl;
    @Value("${spring.datasource.default.username}")
    private String defaultDBUser;
    @Value("${spring.datasource.default.password}")
    private String defaultDBPassword;
    @Value("${spring.datasource.default.driver-class-name}")
    private String defaultDBDriverName;

    @Bean
    public DynamicDataSource dynamicDataSource() {
        DynamicDataSource dynamicDataSource = DynamicDataSource.getInstance();

        HikariDataSource defaultDataSource = new HikariDataSource();
        defaultDataSource.setJdbcUrl(defaultDBUrl);
        defaultDataSource.setUsername(defaultDBUser);
        defaultDataSource.setPassword(defaultDBPassword);
        defaultDataSource.setDriverClassName(defaultDBDriverName);

        Map<Object,Object> map = new HashMap<>();
        map.put("default", defaultDataSource);
        dynamicDataSource.setTargetDataSources(map);
        dynamicDataSource.setDefaultTargetDataSource(defaultDataSource);
        return dynamicDataSource;
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dynamicDataSource) throws Exception {
        SqlSessionFactoryBean bean = new SqlSessionFactoryBean();
        bean.setDataSource(dynamicDataSource);
        return bean.getObject();
    }

}
```

- 自定义动态数据源单例对象，切换不同的数据源操作都是操作这个单例对象

```java
import com.google.common.collect.Maps;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

import java.util.Map;

public class DynamicDataSource extends AbstractRoutingDataSource {

    private static DynamicDataSource instance;

    private static Object lock = new Object();

    private static Map<Object, Object> dataSourceMap = Maps.newConcurrentMap();

    @Override
    protected Object determineCurrentLookupKey() {
        return DynamicDataSourceContextHolder.getDbType();
    }

    @Override
    public void setTargetDataSources(Map<Object, Object> targetDataSources) {
        super.setTargetDataSources(targetDataSources);
        dataSourceMap.putAll(targetDataSources);
        super.afterPropertiesSet();
    }

    /**
     * 懒汉式获取动态数据源单例
     * @return
     */
    public static synchronized DynamicDataSource getInstance(){
        if(instance==null){
            synchronized (lock){
                if(instance==null){
                    instance=new DynamicDataSource();
                }
            }
        }
        return instance;
    }

    public Map<Object, Object> getDataSourceMap() {
        return dataSourceMap;
    }

}
```

- 自定义对象控制数据源的上下文切换，使用[ThreadLocal](https://www.liaoxuefeng.com/wiki/1252599548343744/1306581251653666)保证线程安全，对于`ThreadLocal`，可以简单这样理解：

  - 可以把`ThreadLocal`看成一个全局`Map<Thread, Object>`：每个线程获取`ThreadLocal`变量时，总是使用`Thread`自身作为key：

    ```java
    Object threadLocalValue = threadLocalMap.get(Thread.currentThread());
    ```

  - 因此，`ThreadLocal`相当于给每个线程都开辟了一个独立的存储空间，各个线程的`ThreadLocal`关联的实例互不干扰。

  - `ThreadLocal`要先`set`，然后再`get`，不然get的结果为`null`（一次也没有set）或者上一次`set`的结果

  - 并且`set`的同时，调用工厂更新数据源

```java
import com.example.demo1.factory.DataSourceFactory;

public class DynamicDataSourceContextHolder {

    private static final ThreadLocal<String> dbTypeHolder = new ThreadLocal<String>();

    public static void setDbType(String dbType){
        dbTypeHolder.set(dbType);
        DataSourceFactory.dynamicDataSource(dbType);
    }

    public static String getDbType(){
        return dbTypeHolder.get();
    }

    public static void clearDBType(){
        dbTypeHolder.remove();
    }

}
```

- 自定义简单工厂，根据参数设置不同的数据源

```java
import com.example.demo1.constant.DataSourceProperties;
import com.example.demo1.datasource.DynamicDataSource;
import com.zaxxer.hikari.HikariDataSource;

import java.util.HashMap;
import java.util.Map;

public class DataSourceFactory {
//    public final static String DEFAULT = "default";
    public final static String MASTER = "master";

    public static DynamicDataSource dynamicDataSource(String type) {
        DynamicDataSource dynamicDataSource = DynamicDataSource.getInstance();

        HikariDataSource dataSource = new HikariDataSource();
        if (MASTER.equals(type)) {
            dataSource.setJdbcUrl(DataSourceProperties.MASTER_DB_URL);
            dataSource.setUsername(DataSourceProperties.MASTER_DB_USER);
            dataSource.setPassword(DataSourceProperties.MASTER_DB_PASSWORD);
            dataSource.setDriverClassName(DataSourceProperties.MASTER_DB_DRIVER_NAME);
        } else {
            dataSource.setJdbcUrl(DataSourceProperties.DEFAULT_DB_URL);
            dataSource.setUsername(DataSourceProperties.DEFAULT_DB_USER);
            dataSource.setPassword(DataSourceProperties.DEFAULT_DB_PASSWORD);
            dataSource.setDriverClassName(DataSourceProperties.DEFAULT_DB_DRIVER_NAME);
        }

        Map<Object,Object> map = new HashMap<>();
        map.put(type, dataSource);
        dynamicDataSource.setTargetDataSources(map);
        dynamicDataSource.setDefaultTargetDataSource(dataSource);
        return dynamicDataSource;
    }
}
```

- 重写启动类的run方法，进行测试，就实现了数据源的动态切换

```java
import com.example.demo1.datasource.DynamicDataSourceContextHolder;
import com.example.demo1.entity.Person;
import com.example.demo1.entity.User;
import com.example.demo1.mapper.PersonMapper;
import com.example.demo1.mapper.UserMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * <p>
 * 动态绑定数据源示例
 * </p>
 * @author: zhu.chen
 * @date: 2020/1/17
 * @version: v1.0.0
 */
@SpringBootApplication
public class Demo1Application implements CommandLineRunner {

    public static void main(String[] args) {
        SpringApplication.run(Demo1Application.class, args);
    }

    @Autowired
    private UserMapper userMapper;

    @Autowired
    private PersonMapper personMapper;

    @Override
    public void run(String... args) throws Exception {
        DynamicDataSourceContextHolder.setDbType("default");
        userMapper.delete();
        for (int i = 0; i < 5; i++) {
            User user = new User();
            user.setName("张三" + i);
            user.setPassword("abc" + i);
            userMapper.insert(user);
        }
        System.out.println("user : " + userMapper.selectAll());

        DynamicDataSourceContextHolder.setDbType("master");
        personMapper.delete();
        for (int i = 0; i < 5; i++) {
            Person person = new Person();
            person.setName("小明" + i);
            person.setPassword("efg" + i);
            personMapper.insert(person);
        }
        System.out.println("person : " + personMapper.selectAll());
    }

}
```

# SQL比较两张表结构差异、两张结构相同表的数据差异

主要使用到union和minus这两个关键字，但是MySQL不支持minus

> 比较表结构

```sql
(select column_name,table_name
	from user_tab_columns
	where table_name = 'EMP'
minus
select column_name,table_name
	from user_tab_columns
	where table_name = 'DEPT')
union 
(select column_name,table_name
	from user_tab_columns
	where table_name = 'DEPT'
minus
select column_name,table_name
	from user_tab_columns
	where table_name = 'EMP');
```

***注意：表结构信息存储在user_tab_columns，规定表的名称全部为大写形式***

![在这里插入图片描述](mysql/70-164856462550422.png)

结果：得到了两张表不同结构的列名，以及表名

> 比较表数据：

```sql
(select * from EMP 
 minus
select *from EMP2)
union 
(select * from EMP2
minus
select * from EMP)
```

**注意：前提是表结构一样，可以进行数据差异查询**
得到下列结果：

![在这里插入图片描述](mysql/70-164856460151919.png)

结果：得到了两张结构相同表的差异数据

但是无法区分哪一行的数据，属于那张表，因此加上子查询，利用虚拟列名称，进行区分·，sql如下所示：

```sql
select a.*,'EMP' from 
	(select *from EMP
	minus
	select * FROM EMP2)  a
union 
select b.*,'EMP2' from
	(select * from EMP2
	minus
	select * FROM EMP) b
```

得到的查询结果，如下所示：

![在这里插入图片描述](mysql/70.png)

# MySQL三大日志(binlog、redo log和undo log)详解

## 前言

`MySQL` 日志 主要包括错误日志、查询日志、慢查询日志、事务日志、二进制日志几大类。其中，比较重要的还要属二进制日志 `binlog`（归档日志）和事务日志 `redo log`（重做日志）和 `undo log`（回滚日志）。

![img](mysql/01.png)

今天就来聊聊 `redo log`（重做日志）、`binlog`（归档日志）、两阶段提交、`undo log` （回滚日志）。

## redo log

`redo log`（重做日志）是`InnoDB`存储引擎独有的，它让`MySQL`拥有了崩溃恢复能力。

比如 `MySQL` 实例挂了或宕机了，重启时，`InnoDB`存储引擎会使用`redo log`恢复数据，保证数据的持久性与完整性。

![img](mysql/02.png)

`MySQL` 中数据是以页为单位，你查询一条记录，会从硬盘把一页的数据加载出来，加载出来的数据叫数据页，会放入到 `Buffer Pool` 中。

后续的查询都是先从 `Buffer Pool` 中找，没有命中再去硬盘加载，减少硬盘 `IO` 开销，提升性能。

更新表数据的时候，也是如此，发现 `Buffer Pool` 里存在要更新的数据，就直接在 `Buffer Pool` 里更新。

然后会把“在某个数据页上做了什么修改”记录到重做日志缓存（`redo log buffer`）里，接着刷盘到 `redo log` 文件里。

![img](mysql/03.png)

> 图片笔误提示：第 4 步 “清空 redo log buffe 刷盘到 redo 日志中”这句话中的 buffe 应该是 buffer。

理想情况，事务一提交就会进行刷盘操作，但实际上，刷盘的时机是根据策略来进行的。

> 小贴士：每条 redo 记录由“表空间号+数据页号+偏移量+修改数据长度+具体修改的数据”组成

### 刷盘时机

`InnoDB` 存储引擎为 `redo log` 的刷盘策略提供了 `innodb_flush_log_at_trx_commit` 参数，它支持三种策略：

- **0** ：设置为 0 的时候，表示每次事务提交时不进行刷盘操作
- **1** ：设置为 1 的时候，表示每次事务提交时都将进行刷盘操作（默认值）
- **2** ：设置为 2 的时候，表示每次事务提交时都只把 redo log buffer 内容写入 page cache

`innodb_flush_log_at_trx_commit` 参数默认为 1 ，也就是说当事务提交时会调用 `fsync` 对 redo log 进行刷盘

另外，`InnoDB` 存储引擎有一个后台线程，每隔`1` 秒，就会把 `redo log buffer` 中的内容写到文件系统缓存（`page cache`），然后调用 `fsync` 刷盘。

![img](mysql/04.png)

也就是说，一个没有提交事务的 `redo log` 记录，也可能会刷盘。

**为什么呢？**

因为在事务执行过程 `redo log` 记录是会写入`redo log buffer` 中，这些 `redo log` 记录会被后台线程刷盘。

![img](mysql/05.png)

除了后台线程每秒`1`次的轮询操作，还有一种情况，当 `redo log buffer` 占用的空间即将达到 `innodb_log_buffer_size` 一半的时候，后台线程会主动刷盘。

下面是不同刷盘策略的流程图。

#### innodb_flush_log_at_trx_commit=0

![img](mysql/06.png)

为`0`时，如果`MySQL`挂了或宕机可能会有`1`秒数据的丢失。

#### innodb_flush_log_at_trx_commit=1

![img](mysql/07.png)

为`1`时， 只要事务提交成功，`redo log`记录就一定在硬盘里，不会有任何数据丢失。

如果事务执行期间`MySQL`挂了或宕机，这部分日志丢了，但是事务并没有提交，所以日志丢了也不会有损失。

#### innodb_flush_log_at_trx_commit=2

![img](mysql/09.png)

为`2`时， 只要事务提交成功，`redo log buffer`中的内容只写入文件系统缓存（`page cache`）。

如果仅仅只是`MySQL`挂了不会有任何数据丢失，但是宕机可能会有`1`秒数据的丢失。

#### 日志文件组

硬盘上存储的 `redo log` 日志文件不只一个，而是以一个**日志文件组**的形式出现的，每个的`redo`日志文件大小都是一样的。

比如可以配置为一组`4`个文件，每个文件的大小是 `1GB`，整个 `redo log` 日志文件组可以记录`4G`的内容。

它采用的是环形数组形式，从头开始写，写到末尾又回到头循环写，如下图所示。

![img](mysql/10.png)

在个**日志文件组**中还有两个重要的属性，分别是 `write pos、checkpoint`

- **write pos** 是当前记录的位置，一边写一边后移
- **checkpoint** 是当前要擦除的位置，也是往后推移

每次刷盘 `redo log` 记录到**日志文件组**中，`write pos` 位置就会后移更新。

每次 `MySQL` 加载**日志文件组**恢复数据时，会清空加载过的 `redo log` 记录，并把 `checkpoint` 后移更新。

`write pos` 和 `checkpoint` 之间的还空着的部分可以用来写入新的 `redo log` 记录。

![img](mysql/11.png)

如果 `write pos` 追上 `checkpoint` ，表示**日志文件组**满了，这时候不能再写入新的 `redo log` 记录，`MySQL` 得停下来，清空一些记录，把 `checkpoint` 推进一下。

![img](mysql/12.png)

### redo log 小结

相信大家都知道 `redo log` 的作用和它的刷盘时机、存储形式。

现在我们来思考一个问题： **只要每次把修改后的数据页直接刷盘不就好了，还有 `redo log` 什么事？**

它们不都是刷盘么？差别在哪里？

```ini
1 Byte = 8bit
1 KB = 1024 Byte
1 MB = 1024 KB
1 GB = 1024 MB
1 TB = 1024 GB
```

实际上，数据页大小是`16KB`，刷盘比较耗时，可能就修改了数据页里的几 `Byte` 数据，有必要把完整的数据页刷盘吗？

而且数据页刷盘是随机写，因为一个数据页对应的位置可能在硬盘文件的随机位置，所以性能是很差。

如果是写 `redo log`，一行记录可能就占几十 `Byte`，只包含表空间号、数据页号、磁盘文件偏移 量、更新值，再加上是顺序写，所以刷盘速度很快。

所以用 `redo log` 形式记录修改内容，性能会远远超过刷数据页的方式，这也让数据库的并发能力更强。

> 其实内存的数据页在一定时机也会刷盘，我们把这称为页合并，讲 `Buffer Pool`的时候会对这块细说

## binlog

`redo log` 它是物理日志，记录内容是“在某个数据页上做了什么修改”，属于 `InnoDB` 存储引擎。

而 `binlog` 是逻辑日志，记录内容是语句的原始逻辑，类似于“给 ID=2 这一行的 c 字段加 1”，属于`MySQL Server` 层。

不管用什么存储引擎，只要发生了表数据更新，都会产生 `binlog` 日志。

那 `binlog` 到底是用来干嘛的？

可以说`MySQL`数据库的**数据备份、主备、主主、主从**都离不开`binlog`，需要依靠`binlog`来同步数据，保证数据一致性。

![img](mysql/01-20220305234724956.png)

`binlog`会记录所有涉及更新数据的逻辑操作，并且是顺序写。

### 记录格式

`binlog` 日志有三种格式，可以通过`binlog_format`参数指定。

- **statement**
- **row**
- **mixed**

指定`statement`，记录的内容是`SQL`语句原文，比如执行一条`update T set update_time=now() where id=1`，记录的内容如下。

![img](mysql/02-20220305234738688.png)

同步数据时，会执行记录的`SQL`语句，但是有个问题，`update_time=now()`这里会获取当前系统时间，直接执行会导致与原库的数据不一致。

为了解决这种问题，我们需要指定为`row`，记录的内容不再是简单的`SQL`语句了，还包含操作的具体数据，记录内容如下。

![img](mysql/03-20220305234742460.png)

`row`格式记录的内容看不到详细信息，要通过`mysqlbinlog`工具解析出来。

`update_time=now()`变成了具体的时间`update_time=1627112756247`，条件后面的@1、@2、@3 都是该行数据第 1 个~3 个字段的原始值（**假设这张表只有 3 个字段**）。

这样就能保证同步数据的一致性，通常情况下都是指定为`row`，这样可以为数据库的恢复与同步带来更好的可靠性。

但是这种格式，需要更大的容量来记录，比较占用空间，恢复与同步时会更消耗`IO`资源，影响执行速度。

所以就有了一种折中的方案，指定为`mixed`，记录的内容是前两者的混合。

`MySQL`会判断这条`SQL`语句是否可能引起数据不一致，如果是，就用`row`格式，否则就用`statement`格式。

### 写入机制

`binlog`的写入时机也非常简单，事务执行过程中，先把日志写到`binlog cache`，事务提交的时候，再把`binlog cache`写到`binlog`文件中。

因为一个事务的`binlog`不能被拆开，无论这个事务多大，也要确保一次性写入，所以系统会给每个线程分配一个块内存作为`binlog cache`。

我们可以通过`binlog_cache_size`参数控制单个线程 binlog cache 大小，如果存储内容超过了这个参数，就要暂存到磁盘（`Swap`）。

`binlog`日志刷盘流程如下

![img](mysql/04-20220305234747840.png)

- **上图的 write，是指把日志写入到文件系统的 page cache，并没有把数据持久化到磁盘，所以速度比较快**
- **上图的 fsync，才是将数据持久化到磁盘的操作**

`write`和`fsync`的时机，可以由参数`sync_binlog`控制，默认是`0`。

为`0`的时候，表示每次提交事务都只`write`，由系统自行判断什么时候执行`fsync`。

![img](mysql/05-20220305234754405.png)

虽然性能得到提升，但是机器宕机，`page cache`里面的 binlog 会丢失。

为了安全起见，可以设置为`1`，表示每次提交事务都会执行`fsync`，就如同 **redo log 日志刷盘流程** 一样。

最后还有一种折中方式，可以设置为`N(N>1)`，表示每次提交事务都`write`，但累积`N`个事务后才`fsync`。

![img](mysql/06-20220305234801592.png)

在出现`IO`瓶颈的场景里，将`sync_binlog`设置成一个比较大的值，可以提升性能。

同样的，如果机器宕机，会丢失最近`N`个事务的`binlog`日志。

### 两阶段提交

虽然它们都属于持久化的保证，但是侧重点不同。

在执行更新语句过程，会记录`redo log`与`binlog`两块日志，以基本的事务为单位，`redo log`在事务执行过程中可以不断写入，而`binlog`只有在提交事务时才写入，所以`redo log`与`binlog`的写入时机不一样。

![img](mysql/01-20220305234816065.png)

回到正题，`redo log`与`binlog`两份日志之间的逻辑不一致，会出现什么问题？

我们以`update`语句为例，假设`id=2`的记录，字段`c`值是`0`，把字段`c`值更新成`1`，`SQL`语句为`update T set c=1 where id=2`。

假设执行过程中写完`redo log`日志后，`binlog`日志写期间发生了异常，会出现什么情况呢？

![img](mysql/02-20220305234828662.png)

由于`binlog`没写完就异常，这时候`binlog`里面没有对应的修改记录。因此，之后用`binlog`日志恢复数据时，就会少这一次更新，恢复出来的这一行`c`值是`0`，而原库因为`redo log`日志恢复，这一行`c`值是`1`，最终数据不一致。

![img](mysql/03-20220305235104445.png)

为了解决两份日志之间的逻辑一致问题，`InnoDB`存储引擎使用**两阶段提交**方案。

原理很简单，将`redo log`的写入拆成了两个步骤`prepare`和`commit`，这就是**两阶段提交**。

![img](mysql/04-20220305234956774.png)

使用**两阶段提交**后，写入`binlog`时发生异常也不会有影响，因为`MySQL`根据`redo log`日志恢复数据时，发现`redo log`还处于`prepare`阶段，并且没有对应`binlog`日志，就会回滚该事务。

![img](mysql/05-20220305234937243.png)

再看一个场景，`redo log`设置`commit`阶段发生异常，那会不会回滚事务呢？

![img](mysql/06-20220305234907651.png)

并不会回滚事务，它会执行上图框住的逻辑，虽然`redo log`是处于`prepare`阶段，但是能通过事务`id`找到对应的`binlog`日志，所以`MySQL`认为是完整的，就会提交事务恢复数据。

### undo log

> 这部分内容为 JavaGuide 的补充：

我们知道如果想要保证事务的原子性，就需要在异常发生时，对已经执行的操作进行**回滚**，在 MySQL 中，恢复机制是通过 **回滚日志（undo log）** 实现的，所有事务进行的修改都会先记录到这个回滚日志中，然后再执行相关的操作。如果执行过程中遇到异常的话，我们直接利用 **回滚日志** 中的信息将数据回滚到修改之前的样子即可！并且，回滚日志会先于数据持久化到磁盘上。这样就保证了即使遇到数据库突然宕机等情况，当用户再次启动数据库的时候，数据库还能够通过查询回滚日志来回滚将之前未完成的事务。

另外，`MVCC` 的实现依赖于：**隐藏字段、Read View、undo log**。在内部实现中，`InnoDB` 通过数据行的 `DB_TRX_ID` 和 `Read View` 来判断数据的可见性，如不可见，则通过数据行的 `DB_ROLL_PTR` 找到 `undo log` 中的历史版本。每个事务读到的数据版本可能是不一样的，在同一个事务中，用户只能看到该事务创建 `Read View` 之前已经提交的修改和该事务本身做的修改

## 总结

> 这部分内容为 JavaGuide 的补充：

MySQL InnoDB 引擎使用 **redo log(重做日志)** 保证事务的**持久性**，使用 **undo log(回滚日志)** 来保证事务的**原子性**。

`MySQL`数据库的**数据备份、主备、主主、主从**都离不开`binlog`，需要依靠`binlog`来同步数据，保证数据一致性。

# InnoDB存储引擎对MVCC的实现

## 一致性非锁定读和锁定读

### 一致性非锁定读

对于 [一致性非锁定读（Consistent Nonlocking Reads）](https://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html)的实现，通常做法是加一个版本号或者时间戳字段，在更新数据的同时版本号 + 1 或者更新时间戳。查询时，将当前可见的版本号与对应记录的版本号进行比对，如果记录的版本小于可见版本，则表示该记录可见

在 `InnoDB` 存储引擎中，[多版本控制 (multi versioning)](https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html) 就是对非锁定读的实现。如果读取的行正在执行 `DELETE` 或 `UPDATE` 操作，这时读取操作不会去等待行上锁的释放。相反地，`InnoDB` 存储引擎会去读取行的一个快照数据，对于这种读取历史数据的方式，我们叫它快照读 (snapshot read)

在 `Repeatable Read` 和 `Read Committed` 两个隔离级别下，如果是执行普通的 `select` 语句（不包括 `select ... lock in share mode` ,`select ... for update`）则会使用 `一致性非锁定读（MVCC）`。并且在 `Repeatable Read` 下 `MVCC` 实现了可重复读和防止部分幻读

### 锁定读

如果执行的是下列语句，就是 [锁定读（Locking Reads）](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking-reads.html)

- `select ... lock in share mode`
- `select ... for update`
- `insert`、`update`、`delete` 操作

在锁定读下，读取的是数据的最新版本，这种读也被称为 `当前读（current read）`。锁定读会对读取到的记录加锁：

- `select ... lock in share mode`：对记录加 `S` 锁，其它事务也可以加`S`锁，如果加 `x` 锁则会被阻塞
- `select ... for update`、`insert`、`update`、`delete`：对记录加 `X` 锁，且其它事务不能加任何锁

在一致性非锁定读下，即使读取的记录已被其它事务加上 `X` 锁，这时记录也是可以被读取的，即读取的快照数据。上面说了，在 `Repeatable Read` 下 `MVCC` 防止了部分幻读，这边的 “部分” 是指在 `一致性非锁定读` 情况下，只能读取到第一次查询之前所插入的数据（根据 Read View 判断数据可见性，Read View 在第一次查询时生成）。但是！如果是 `当前读` ，每次读取的都是最新数据，这时如果两次查询中间有其它事务插入数据，就会产生幻读。所以， **`InnoDB` 在实现`Repeatable Read` 时，如果执行的是当前读，则会对读取的记录使用 `Next-key Lock` ，来防止其它事务在间隙间插入数据**

## InnoDB 对 MVCC 的实现

`MVCC` 的实现依赖于：**隐藏字段、Read View、undo log**。在内部实现中，`InnoDB` 通过数据行的 `DB_TRX_ID` 和 `Read View` 来判断数据的可见性，如不可见，则通过数据行的 `DB_ROLL_PTR` 找到 `undo log` 中的历史版本。每个事务读到的数据版本可能是不一样的，在同一个事务中，用户只能看到该事务创建 `Read View` 之前已经提交的修改和该事务本身做的修改

### 隐藏字段

在内部，`InnoDB` 存储引擎为每行数据添加了三个 [隐藏字段](https://dev.mysql.com/doc/refman/5.7/en/innodb-multi-versioning.html)：

- `DB_TRX_ID（6字节）`：表示最后一次插入或更新该行的事务 id。此外，`delete` 操作在内部被视为更新，只不过会在记录头 `Record header` 中的 `deleted_flag` 字段将其标记为已删除
- `DB_ROLL_PTR（7字节）` 回滚指针，指向该行的 `undo log` 。如果该行未被更新，则为空
- `DB_ROW_ID（6字节）`：如果没有设置主键且该表没有唯一非空索引时，`InnoDB` 会使用该 id 来生成聚簇索引

### ReadView

```c++
class ReadView {
  /* ... */
private:
  trx_id_t m_low_limit_id;      /* 大于等于这个 ID 的事务均不可见 */

  trx_id_t m_up_limit_id;       /* 小于这个 ID 的事务均可见 */

  trx_id_t m_creator_trx_id;    /* 创建该 Read View 的事务ID */

  trx_id_t m_low_limit_no;      /* 事务 Number, 小于该 Number 的 Undo Logs 均可以被 Purge */

  ids_t m_ids;                  /* 创建 Read View 时的活跃事务列表 */

  m_closed;                     /* 标记 Read View 是否 close */
}
```

[`Read View`](https://github.com/facebook/mysql-8.0/blob/8.0/storage/innobase/include/read0types.h#L298) 主要是用来做可见性判断，里面保存了 “当前对本事务不可见的其他活跃事务”

主要有以下字段：

- `m_low_limit_id`：目前出现过的最大的事务 ID+1，即下一个将被分配的事务 ID。大于等于这个 ID 的数据版本均不可见
- `m_up_limit_id`：活跃事务列表 `m_ids` 中最小的事务 ID，如果 `m_ids` 为空，则 `m_up_limit_id` 为 `m_low_limit_id`。小于这个 ID 的数据版本均可见
- `m_ids`：`Read View` 创建时其他未提交的活跃事务 ID 列表。创建 `Read View`时，将当前未提交事务 ID 记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。`m_ids` 不包括当前事务自己和已提交的事务（正在内存中）
- `m_creator_trx_id`：创建该 `Read View` 的事务 ID

**事务可见性示意图**（[图源](https://leviathan.vip/2019/03/20/InnoDB的事务分析-MVCC/#MVCC-1)）：

### undo-log

`undo log` 主要有两个作用：

- 当事务回滚时用于将数据恢复到修改前的样子
- 另一个作用是 `MVCC` ，当读取记录时，若该记录被其他事务占用或当前版本对该事务不可见，则可以通过 `undo log` 读取之前的版本数据，以此实现非锁定读

**在 `InnoDB` 存储引擎中 `undo log` 分为两种： `insert undo log` 和 `update undo log`：**

1. **`insert undo log`** ：指在 `insert` 操作中产生的 `undo log`。因为 `insert` 操作的记录只对事务本身可见，对其他事务不可见，故该 `undo log` 可以在事务提交后直接删除。不需要进行 `purge` 操作

**`insert` 时的数据初始状态：**

![img](mysql/317e91e1-1ee1-42ad-9412-9098d5c6a9ad.png)

1. **`update undo log`** ：`update` 或 `delete` 操作中产生的 `undo log`。该 `undo log`可能需要提供 `MVCC` 机制，因此不能在事务提交时就进行删除。提交时放入 `undo log` 链表，等待 `purge线程` 进行最后的删除

**数据第一次被修改时：**

![img](mysql/c52ff79f-10e6-46cb-b5d4-3c9cbcc1934a.png)

**数据第二次被修改时：**

![img](mysql/6a276e7a-b0da-4c7b-bdf7-c0c7b7b3b31c.png)

不同事务或者相同事务的对同一记录行的修改，会使该记录行的 `undo log` 成为一条链表，链首就是最新的记录，链尾就是最早的旧记录。

### 数据可见性算法

在 `InnoDB` 存储引擎中，创建一个新事务后，执行每个 `select` 语句前，都会创建一个快照（Read View），**快照中保存了当前数据库系统中正处于活跃（没有 commit）的事务的 ID 号**。其实简单的说保存的是系统中当前不应该被本事务看到的其他事务 ID 列表（即 m_ids）。当用户在这个事务中要读取某个记录行的时候，`InnoDB` 会将该记录行的 `DB_TRX_ID` 与 `Read View` 中的一些变量及当前事务 ID 进行比较，判断是否满足可见性条件

[具体的比较算法](https://github.com/facebook/mysql-8.0/blob/8.0/storage/innobase/include/read0types.h#L161)如下：[图源](https://leviathan.vip/2019/03/20/InnoDB的事务分析-MVCC/#MVCC-1)

![img](mysql/8778836b-34a8-480b-b8c7-654fe207a8c2.png)

1. 如果记录 DB_TRX_ID < m_up_limit_id，那么表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照之前就提交了，所以该记录行的值对当前事务是可见的
2. 如果 DB_TRX_ID >= m_low_limit_id，那么表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照之后才修改该行，所以该记录行的值对当前事务不可见。跳到步骤 5
3. m_ids 为空，则表明在当前事务创建快照之前，修改该行的事务就已经提交了，所以该记录行的值对当前事务是可见的
4. 如果 m_up_limit_id <= DB_TRX_ID < m_low_limit_id，表明最新修改该行的事务（DB_TRX_ID）在当前事务创建快照的时候可能处于“活动状态”或者“已提交状态”；所以就要对活跃事务列表 m_ids 进行查找（源码中是用的二分查找，因为是有序的）
   - 如果在活跃事务列表 m_ids 中能找到 DB_TRX_ID，表明：① 在当前事务创建快照前，该记录行的值被事务 ID 为 DB_TRX_ID 的事务修改了，但没有提交；或者 ② 在当前事务创建快照后，该记录行的值被事务 ID 为 DB_TRX_ID 的事务修改了。这些情况下，这个记录行的值对当前事务都是不可见的。跳到步骤 5
   - 在活跃事务列表中找不到，则表明“id 为 trx_id 的事务”在修改“该记录行的值”后，在“当前事务”创建快照前就已经提交了，所以记录行对当前事务可见
5. 在该记录行的 DB_ROLL_PTR 指针所指向的 `undo log` 取出快照记录，用快照记录的 DB_TRX_ID 跳到步骤 1 重新开始判断，直到找到满足的快照版本或返回空

## RC 和 RR 隔离级别下 MVCC 的差异

在事务隔离级别 `RC` 和 `RR` （InnoDB 存储引擎的默认事务隔离级别）下，`InnoDB` 存储引擎使用 `MVCC`（非锁定一致性读），但它们生成 `Read View` 的时机却不同

- 在 RC 隔离级别下的 **`每次select`** 查询前都生成一个`Read View` (m_ids 列表)
- 在 RR 隔离级别下只在事务开始后 **`第一次select`** 数据前生成一个`Read View`（m_ids 列表）

## MVCC 解决不可重复读问题

虽然 RC 和 RR 都通过 `MVCC` 来读取快照数据，但由于 **生成 Read View 时机不同**，从而在 RR 级别下实现可重复读

举个例子：

![img](mysql/6fb2b9a1-5f14-4dec-a797-e4cf388ed413.png)

### 在 RC 下 ReadView 生成情况

1. **`假设时间线来到 T4 ，那么此时数据行 id = 1 的版本链为`：**

![img](mysql/a3fd1ec6-8f37-42fa-b090-7446d488fd04.png)

由于 RC 级别下每次查询都会生成`Read View` ，并且事务 101、102 并未提交，此时 `103` 事务生成的 `Read View` 中活跃的事务 **`m_ids` 为：[101,102]** ，`m_low_limit_id`为：104，`m_up_limit_id`为：101，`m_creator_trx_id` 为：103

- 此时最新记录的 `DB_TRX_ID` 为 101，m_up_limit_id <= 101 < m_low_limit_id，所以要在 `m_ids` 列表中查找，发现 `DB_TRX_ID` 存在列表中，那么这个记录不可见
- 根据 `DB_ROLL_PTR` 找到 `undo log` 中的上一版本记录，上一条记录的 `DB_TRX_ID` 还是 101，不可见
- 继续找上一条 `DB_TRX_ID`为 1，满足 1 < m_up_limit_id，可见，所以事务 103 查询到数据为 `name = 菜花`

2. **`时间线来到 T6 ，数据的版本链为`：**

![markdown](mysql/528559e9-dae8-4d14-b78d-a5b657c88391.png)

因为在 RC 级别下，重新生成 `Read View`，这时事务 101 已经提交，102 并未提交，所以此时 `Read View` 中活跃的事务 **`m_ids`：[102]** ，`m_low_limit_id`为：104，`m_up_limit_id`为：102，`m_creator_trx_id`为：103

- 此时最新记录的 `DB_TRX_ID` 为 102，m_up_limit_id <= 102 < m_low_limit_id，所以要在 `m_ids` 列表中查找，发现 `DB_TRX_ID` 存在列表中，那么这个记录不可见
- 根据 `DB_ROLL_PTR` 找到 `undo log` 中的上一版本记录，上一条记录的 `DB_TRX_ID` 为 101，满足 101 < m_up_limit_id，记录可见，所以在 `T6` 时间点查询到数据为 `name = 李四`，与时间 T4 查询到的结果不一致，不可重复读！

3. **`时间线来到 T9 ，数据的版本链为`：**

![markdown](mysql/6f82703c-36a1-4458-90fe-d7f4edbac71a.png)

重新生成 `Read View`， 这时事务 101 和 102 都已经提交，所以 **m_ids** 为空，则 m_up_limit_id = m_low_limit_id = 104，最新版本事务 ID 为 102，满足 102 < m_low_limit_id，可见，查询结果为 `name = 赵六`

> **总结：** **在 RC 隔离级别下，事务在每次查询开始时都会生成并设置新的 Read View，所以导致不可重复读**

### 在 RR 下 ReadView 生成情况

**在可重复读级别下，只会在事务开始后第一次读取数据时生成一个 Read View（m_ids 列表）**

1. **`在 T4 情况下的版本链为`：**

![markdown](mysql/0e906b95-c916-4f30-beda-9cb3e49746bf.png)

在当前执行 `select` 语句时生成一个 `Read View`，此时 **`m_ids`：[101,102]** ，`m_low_limit_id`为：104，`m_up_limit_id`为：101，`m_creator_trx_id` 为：103

此时和 RC 级别下一样：

- 最新记录的 `DB_TRX_ID` 为 101，m_up_limit_id <= 101 < m_low_limit_id，所以要在 `m_ids` 列表中查找，发现 `DB_TRX_ID` 存在列表中，那么这个记录不可见
- 根据 `DB_ROLL_PTR` 找到 `undo log` 中的上一版本记录，上一条记录的 `DB_TRX_ID` 还是 101，不可见
- 继续找上一条 `DB_TRX_ID`为 1，满足 1 < m_up_limit_id，可见，所以事务 103 查询到数据为 `name = 菜花`

2. **`时间点 T6 情况下`：**

![markdown](mysql/79ed6142-7664-4e0b-9023-cf546586aa39.png)

在 RR 级别下只会生成一次`Read View`，所以此时依然沿用 **`m_ids` ：[101,102]** ，`m_low_limit_id`为：104，`m_up_limit_id`为：101，`m_creator_trx_id` 为：103

- 最新记录的 `DB_TRX_ID` 为 102，m_up_limit_id <= 102 < m_low_limit_id，所以要在 `m_ids` 列表中查找，发现 `DB_TRX_ID` 存在列表中，那么这个记录不可见
- 根据 `DB_ROLL_PTR` 找到 `undo log` 中的上一版本记录，上一条记录的 `DB_TRX_ID` 为 101，不可见
- 继续根据 `DB_ROLL_PTR` 找到 `undo log` 中的上一版本记录，上一条记录的 `DB_TRX_ID` 还是 101，不可见
- 继续找上一条 `DB_TRX_ID`为 1，满足 1 < m_up_limit_id，可见，所以事务 103 查询到数据为 `name = 菜花`

3.**时间点 T9 情况下：**

![markdown](mysql/cbbedbc5-0e3c-4711-aafd-7f3d68a4ed4e.png)

此时情况跟 T6 完全一样，由于已经生成了 `Read View`，此时依然沿用 **`m_ids` ：[101,102]** ，所以查询结果依然是 `name = 菜花`

## MVCC➕Next-key-Lock 防止幻读

`InnoDB`存储引擎在 RR 级别下通过 `MVCC`和 `Next-key Lock` 来解决幻读问题：

**1、执行普通 `select`，此时会以 `MVCC` 快照读的方式读取数据**

在快照读的情况下，RR 隔离级别只会在事务开启后的第一次查询生成 `Read View` ，并使用至事务提交。所以在生成 `Read View` 之后其它事务所做的更新、插入记录版本对当前事务并不可见，实现了可重复读和防止快照读下的 “幻读”

**2、执行 select...for update/lock in share mode、insert、update、delete 等当前读**

在当前读下，读取的都是最新的数据，如果其它事务有插入新的记录，并且刚好在当前事务查询范围内，就会产生幻读！`InnoDB` 使用 [Next-key Lockopen in new window](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-next-key-locks) 来防止这种情况。当执行当前读时，会锁定读取到的记录的同时，锁定它们的间隙，防止其它事务在查询范围内插入数据。只要我不让你插入，就不会发生幻读

## 参考

- **《MySQL 技术内幕 InnoDB 存储引擎第 2 版》**
- [Innodb 中的事务隔离级别和锁的关系open in new window](https://tech.meituan.com/2014/08/20/innodb-lock.html)
- [MySQL 事务与 MVCC 如何实现的隔离级别open in new window](https://blog.csdn.net/qq_35190492/article/details/109044141)
- [InnoDB 事务分析-MVCC](https://leviathan.vip/2019/03/20/InnoDB的事务分析-MVCC/)

# MySQL 数据表设计规范

数据命名规范

- 所有数据库对象名称必须使用小写字母并用下划线分割。
- 所有数据库对象名称禁止使用 MySQL 保留关键字（如果表名中包含关键字查询时，需要将其用单引号括起来）。
- 数据库对象的命名要能做到见名识意，并且最后不要超过 32 个字符。
- 临时库表必须以 tmp *为前缀并以日期为后缀，备份表必须以 bak* 为前缀并以日期 (时间戳) 为后缀。
- 所有存储相同数据的列名和列类型必须一致（一般作为关联列，如果查询时关联列类型不一致会自动进行数据类型隐式转换，会造成列上的索引失效，导致查询效率降低）。

## 数据库基本设计规范

### 1、所有表必须使用 InnoDB 存储引擎

> 没有特殊要求（即 InnoDB 无法满足的功能如：列存储，存储空间数据等）的情况下，所有表必须使用 InnoDB 存储引擎 `MySQL 5.5 之前默认使用 Myisam，5.6 以后默认的为 InnoDBInnoDB` 支持事务，支持行级锁，更好的恢复性，高并发下性能更好。

### 2、数据库和表的字符集统一使用 UTF8MB4

> 兼容性更好，统一字符集可以避免由于字符集转换产生的乱码，不同的字符集进行比较前需要进行转换会造成索引失效。

### 3、所有表和字段都需要添加注释

> 使用 comment 从句添加表和列的备注 从一开始就进行数据字典的维护。

### 4、尽量控制单表数据量的大小，建议控制在 500 万以内

> 500 万并不是 MySQL 数据库的限制，过大会造成修改表结构、备份、恢复都会有很大的问题，可以用历史数据归档`（应用于日志数据）`，分库分表`（应用于业务数据）`等手段来控制数据量大小。

### 5、谨慎使用 MySQL 分区表

> 分区表在物理上表现为多个文件，在逻辑上表现为一个表 谨慎选择分区键，跨分区查询效率可能更低 建议采用物理分表的方式管理大数据。

### 6、尽量做到冷热数据分离，减小表的宽度

> MySQL 限制每个表最多存储 4096 列，并且每一行数据的大小不能超过 65535 字节 减少磁盘 IO，保证热数据的内存缓存命中率`（表越宽，把表装载进内存缓冲池时所占用的内存也就越大,也会消耗更多的 IO）` 更有效的利用缓存，避免读入无用的冷数据 经常一起使用的列放到一个表中`（避免更多的关联操作）`

### 7、禁止在表中建立预留字段

> 预留字段的命名很难做到见名识义 预留字段无法确认存储的数据类型，所以无法选择合适的类型 对预留字段类型的修改，会对表进行锁定

### 8、禁止在数据库中存储图片，文件等大的二进制数据

> 通常文件很大，会短时间内造成数据量快速增长，数据库进行数据库读取时，通常会进行大量的随机 IO 操作，文件很大时，IO 操作很耗时 通常存储于文件服务器，数据库只存储文件地址信息。。

### 9、禁止在线上做数据库压力测试

### 10、禁止从开发环境，测试环境直接连接生成环境数据库

## 数据库字段设计规范

### 1. 优先选择符合存储需要的最小的数据类型

```
原因
```

> 列的字段越大，建立索引时所需要的空间也就越大，这样一页中所能存储的索引节点的数量也就越少也越少，在遍历时所需要的 IO 次数也就越多， 索引的性能也就越差

```
方法
```

> 1、将字符串转换成数字类型存储，如：将 IP 地址转换成整形数据

- MySQL 提供了两个方法来处理 IP 地址
- inet_aton 把 ip 转为无符号整型 (4-8 位)
- inet_ntoa 把整型的 ip 转为地址 插入数据前，先用 inet_aton 把 IP 地址转为整型，可以节省空间。显示数据时，使用 inet_ntoa 把整型的 IP 地址转为地址显示即可。
  2、对于非负型的数据（如自增 ID、整型 IP）来说，要优先使用无符号整型来存储，因为无符号相对于有符号可以多出一倍的存储空间。
- SIGNED INT -2147483648~2147483647
- UNSIGNED INT 0~4294967295

VARCHAR (N) 中的 N 代表的是字符数，而不是字节数。使用 UTF8 存储 255 个汉字 Varchar (255)=765 个字节。过大的长度会消耗更多的内存

### 2. 避免使用 TEXT、BLOB 数据类型，最常见的 TEXT 类型可以存储 64k 的数据

- 建议把 BLOB 或是 TEXT 列分离到单独的扩展表中
- MySQL 内存临时表不支持 TEXT、BLOB 这样的大数据类型，如果查询中包含这样的数据，在排序等操作时，就不能使用内存临时表，必须使用磁盘临时表进行。
- 而且对于这种数据，MySQL 还是要进行二次查询，会使 SQL 性能变得很差，但是不是说一定不能使用这样的数据类型。
- 如果一定要使用，建议把 BLOB 或是 TEXT 列分离到单独的扩展表中，查询时一定不要使用 select * 而只需要取出必要的列，不需要 TEXT 列的数据时不要对该列进行查询。
- TEXT 或 BLOB 类型只能使用前缀索引
- 因为 MySQL 对索引字段长度是有限制的，所以 TEXT 类型只能使用前缀索引，并且 TEXT 列上是不能有默认值的。

### 3. 避免使用 ENUM 类型

- 修改 ENUM 值需要使用 ALTER 语句
- ENUM 类型的 ORDER BY 操作效率低，需要额外操作
- 禁止使用数值作为 ENUM 的枚举值

### 4. 尽可能把所有列定义为 NOT NULL

```
原因
```

- 索引 NULL 列需要额外的空间来保存，所以要占用更多的空间。
- 进行比较和计算时要对 NULL 值做特别的处理。

### 5. 使用 TIMESTAMP（4 个字节）或 DATETIME 类型（8 个字节）存储时间

- TIMESTAMP 存储的时间范围 1970-01-01 00:00:01 ~ 2038-01-19-03:14:07。
- TIMESTAMP 占用 4 字节和 INT 相同，但比 INT 可读性高，超出 TIMESTAMP 取值范围的使用 DATETIME 类型存储。
- 经常会有人用字符串存储日期型的数据（不正确的做法）：
  缺点 1：无法用日期函数进行计算和比较。
  缺点 2：用字符串存储日期要占用更多的空间。

### 6. 同财务相关的金额类数据必须使用 decimal 类型

- 非精准浮点：float，double
- 精准浮点：decimal

Decimal 类型为精准浮点数，在计算时不会丢失精度。占用空间由定义的宽度决定，每 4 个字节可以存储 9 位数字，并且小数点要占用一个字节。可用于存储比 bigint 更大的整型数据。

## 索引设计规范

### 1. 限制每张表上的索引数量，建议单张表索引不超过 5 个

- 索引并不是越多越好！索引可以提高效率同样也可以降低效率；索引可以增加查询效率，但同样也会降低插入和更新的效率，甚至有些情况下会降低查询效率。
- 因为 MySQL 优化器在选择如何优化查询时，会根据统一信息，对每一个可以用到的索引来进行评估，以生成出一个最好的执行计划，如果同时有很多个索引都可以用于查询，就会增加 MySQL 优化器生成执行计划的时间，同样会降低查询性能

### 2. 禁止给表中的每一列都建立单独的索引

> 5.6 版本之前，一个 SQL 只能使用到一个表中的一个索引，5.6 以后，虽然有了合并索引的优化方式，但是还是远远没有使用一个联合索引的查询方式好

### 3. 每个 InnoDB 表必须有个主键

- InnoDB 是一种索引组织表：数据的存储的逻辑顺序和索引的顺序是相同的。每个表都可以有多个索引，但是表的存储顺序只能有一种 InnoDB 是按照主键索引的顺序来组织表的。
- 不要使用更新频繁的列作为主键，不适用多列主键（相当于联合索引） 不要使用 UUID、MD5、HASH、字符串列作为主键（无法保证数据的顺序增长）。主键建议使用自增 ID 值。

常见索引列建议

- 出现在 SELECT、UPDATE、DELETE 语句的 WHERE 从句中的列。
- 包含在 ORDER BY、GROUP BY、DISTINCT 中的字段。
- 并不要将符合 1 和 2 中的字段的列都建立一个索引，通常将 1、2 中的字段建立联合索引效果更好。
- 多表 JOIN 的关联列。

## 如何选择索引列的顺序

### 建立索引的目的是：

> 希望通过索引进行数据查找，减少随机 IO，增加查询性能 ，索引能过滤出越少的数据，则从磁盘中读入的数据也就越少。

- 区分度最高的放在联合索引的最左侧（区分度 = 列中不同值的数量 / 列的总行数）。
- 尽量把字段长度小的列放在联合索引的最左侧（因为字段长度越小，一页能存储的数据量越大，IO 性能也就越好）。
- 使用最频繁的列放到联合索引的左侧（这样可以比较少的建立一些索引）。

### 避免建立冗余索引和重复索引

- 因为这样会增加查询优化器生成执行计划的时间。
- 重复索引示例：primary key (id)、index (id)、unique index (id)
- 冗余索引示例：index (a,b,c)、index (a,b)、index (a）

### 优先考虑覆盖索引

> 对于频繁的查询优先考虑使用覆盖索引。

```
覆盖索引
```

> 就是包含了所有查询字段 (where,select,ordery by,group by 包含的字段) 的索引

```
覆盖索引的好处：
```

- 避免 InnoDB 表进行索引的二次查询
- InnoDB 是以聚集索引的顺序来存储的，对于 InnoDB 来说，二级索引在叶子节点中所保存的是行的主键信息，如果是用二级索引查询数据的话，在查找到相应的键值后，还要通过主键进行二次查询才能获取我们真实所需要的数据。而在覆盖索引中，二级索引的键值中可以获取所有的数据，避免了对主键的二次查询 ，减少了 IO 操作，提升了查询效率。
- 可以把随机 IO 变成顺序 IO 加快查询效率
- 由于覆盖索引是按键值的顺序存储的，对于 IO 密集型的范围查找来说，对比随机从磁盘读取每一行的数据 IO 要少的多，因此利用覆盖索引在访问时也可以把磁盘的随机读取的 IO 转变成索引查找的顺序 IO。

### 索引 SET 规范

- 尽量避免使用外键约束。
- 不建议使用外键约束（foreign key），但一定要在表与表之间的关联键上建立索引。
- 外键可用于保证数据的参照完整性，但建议在业务端实现。
- 外键会影响父表和子表的写操作从而降低性能。

## 数据库 SQL 开发规范

### 1. 建议使用预编译语句进行数据库操作

> 预编译语句可以重复使用这些计划，减少 SQL 编译所需要的时间，还可以解决动态 SQL 所带来的 SQL 注入的问题 只传参数，比传递 SQL 语句更高效 相同语句可以一次解析，多次使用，提高处理效率。

### 2. 避免数据类型的隐式转换

> 隐式转换会导致索引失效。如：
> select name,phone from customer where id = '111';

### 3. 充分利用表上已经存在的索引

- 避免使用双 % 号的查询条件。
- 如 a like '%123%'，（如果无前置 %，只有后置 %，是可以用到列上的索引的）
- 一个 SQL 只能利用到复合索引中的一列进行范围查询
- 如：有 a,b,c 列的联合索引，在查询条件中有 a 列的范围查询，则在 b,c 列上的索引将不会被用到，在定义联合索引时，如果 a 列要用到范围查找的话，就要把 a 列放到联合索引的右侧。
- 使用 left join 或 not exists 来优化 not in 操作， 因为 not in 也通常会使用索引失效。

### 4. 数据库设计时，应该要对以后扩展进行考虑

### 5. 程序连接不同的数据库使用不同的账号，禁止跨库查询

- 为数据库迁移和分库分表留出余地
- 降低业务耦合度
- 避免权限过大而产生的安全风险

### 6. 禁止使用 SELECT * 必须使用 SELECT <字段列表> 查询

- 消耗更多的 CPU 和 IO 以网络带宽资源
- 无法使用覆盖索引
- 可减少表结构变更带来的影响

### 7. 禁止使用不含字段列表的 INSERT 语句

```
如：
insert into values ('a','b','c');
应使用：
insert into t(c1,c2,c3) values ('a','b','c');
```

### 8. 避免使用子查询，可以把子查询优化为 JOIN 操作

- 通常子查询在 in 子句中，且子查询中为简单 SQL (不包含 union、group by、order by、limit 从句) 时，才可以把子查询转化为关联查询进行优化。

```
子查询性能差的原因：
```

1. 子查询的结果集无法使用索引，通常子查询的结果集会被存储到临时表中，不论是内存临时表还是磁盘临时表都不会存在索引，所以查询性能会受到一定的影响。
2. 特别是对于返回结果集比较大的子查询，其对查询性能的影响也就越大。
3. 由于子查询会产生大量的临时表也没有索引，所以会消耗过多的 CPU 和 IO 资源，产生大量的慢查询。

### 9. 避免使用 JOIN 关联太多的表

- 对于 MySQL 来说，是存在关联缓存的，缓存的大小可以由 join_buffer_size 参数进行设置。
- 在 MySQL 中，对于同一个 SQL 多关联（join）一个表，就会多分配一个关联缓存，如果在一个 SQL 中关联的表越多，所占用的内存也就越大。
- 如果程序中大量的使用了多表关联的操作，同时 join_buffer_size 设置的也不合理的情况下，就容易造成服务器内存溢出的情况，就会影响到服务器数据库性能的稳定性。
- 同时对于关联操作来说，会产生临时表操作，影响查询效率 MySQL 最多允许关联 61 个表，建议不超过 5 个。

### 10. 减少同数据库的交互次数

> 数据库更适合处理批量操作 合并多个相同的操作到一起，可以提高处理效率

### 11. 对应同一列进行 or 判断时，使用 in 代替 or

> In 的值不要超过 500 个， in 操作可以更有效的利用索引，or 大多数情况下很少能利用到索引。

### 12. 禁止使用 order by rand () 进行随机排序

- 会把表中所有符合条件的数据装载到内存中，然后在内存中对所有数据根据随机生成的值进行排序，并且可能会对每一行都生成一个随机值，如果满足条件的数据集非常大，就会消耗大量的 CPU 和 IO 及内存资源。
- 推荐在程序中获取一个随机值，然后从数据库中获取数据的方式。

### 13. WHERE 从句中禁止对列进行函数转换和计算

> 对列进行函数转换或计算时会导致无法使用索引。
> `不推荐`
> where date(create_time)='20190101'

```
推荐
where create_time >= '20190101' and create_time < '20190102'
```

### 14. 在明显不会有重复值时使用 UNION ALL 而不是 UNION

- UNION 会把两个结果集的所有数据放到临时表中后再进行去重操作。
- UNION ALL 不会再对结果集进行去重操作。

### 15. 拆分复杂的大 SQL 为多个小 SQL

- 大 SQL：逻辑上比较复杂，需要占用大量 CPU 进行计算的 SQL 。
- MySQL：一个 SQL 只能使用一个 CPU 进行计算。
- SQL 拆分后可以通过并行执行来提高处理效率。

## 数据库操作行为规范

### 1. 超 100 万行的批量写（UPDATE、DELETE、INSERT）操作，要分批多次进行操作

- 大批量操作可能会造成严重的主从延迟
- 主从环境中，大批量操作可能会造成严重的主从延迟，大批量的写操作一般都需要执行一定长的时间，而只有当主库上执行完成后，才会在其他从库上执行，所以会造成主库与从库长时间的延迟情况
- Binlog 日志为 row 格式时会产生大量的日志
- 大批量写操作会产生大量日志，特别是对于 row 格式二进制数据而言，由于在 row 格式中会记录每一行数据的修改，我们一次修改的数据越多，产生的日志量也就会越多，日志的传输和恢复所需要的时间也就越长，这也是造成主从延迟的一个原因。

```
避免产生大事务操作
```

- 大批量修改数据，一定是在一个事务中进行的，这就会造成表中大批量数据进行锁定，从而导致大量的阻塞，阻塞会对 MySQL 的性能产生非常大的影响。
- 特别是长时间的阻塞会占满所有数据库的可用连接，这会使生产环境中的其他应用无法连接到数据库，因此一定要注意大批量写操作要进行分批。

### 2. 对于大表使用 pt-online-schema-change 修改表结构

- 避免大表修改产生的主从延迟
- 避免在对表字段进行修改时进行锁表
- 对大表数据结构的修改一定要谨慎，会造成严重的锁表操作，尤其是生产环境，是不能容忍的。
- pt-online-schema-change 它会首先建立一个与原表结构相同的新表，并且在新表上进行表结构的修改，然后再把原表中的数据复制到新表中，并在原表中增加一些触发器。
- 把原表中新增的数据也复制到新表中，在行所有数据复制完成之后，把新表命名成原表，并把原来的表删除掉，把原来一个 DDL 操作，分解成多个小的批次进行。

### 3. 禁止为程序使用的账号赋予 super 权限

> 当达到最大连接数限制时，还运行 1 个 有 super 权限的用户连接 super 权限只能留给 DBA 处理问题的账号使用。

### 4. 对于程序连接数据库账号，遵循权限最小原则

> 程序使用数据库账号只能在一个 DB 下使用，不准跨库 程序使用的账号原则上不准有 drop 权限。
