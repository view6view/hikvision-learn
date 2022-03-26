## SpringBoot快速集成SpringData JPA

- 导入依赖，主要有以下：

```xml
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.28</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
```

- 配置文件

```yml
spring:
  datasource:
    url: jdbc:mysql://YOUR_URL:3306/jpa?useSSL=false
    username: YOUR_USERNAME
    password: YOUT_PASSWORD
  jpa:
    database: MySQL
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    show-sql: true
    hibernate:
      ddl-auto: update
```

- JavaConfig配置

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.JpaVendorAdapter;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.orm.jpa.vendor.Database;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

@Configuration
@EnableTransactionManagement
public class SpringDataJpaConfig {
    @Bean
    public JpaVendorAdapter jpaVendorAdapter() {
        HibernateJpaVendorAdapter jpaVendorAdapter = new HibernateJpaVendorAdapter();
        // 设置数据库类型（可使用org.springframework.orm.jpa.vendor包下的Database枚举类）
        jpaVendorAdapter.setDatabase(Database.MYSQL);
        // 设置打印sql语句
        jpaVendorAdapter.setShowSql(true);
        // 设置不生成ddl语句
        jpaVendorAdapter.setGenerateDdl(false);
        // 设置hibernate方言
        jpaVendorAdapter.setDatabasePlatform("org.hibernate.dialect.MySQL5Dialect");
        return jpaVendorAdapter;
    }

    /**
     * 配置实体管理器工厂
     * @param dataSource
     * @param jpaVendorAdapter
     * @return
     */
    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(
            DataSource dataSource, JpaVendorAdapter jpaVendorAdapter) {
        LocalContainerEntityManagerFactoryBean emfb = new LocalContainerEntityManagerFactoryBean();
        // 注入数据源
        emfb.setDataSource(dataSource);
        // 注入jpa厂商适配器
        emfb.setJpaVendorAdapter(jpaVendorAdapter);
        // 设置扫描基本包
        emfb.setPackagesToScan("com.wu.jpatest.entity");
        return emfb;
    }

    /**
     * 配置jpa事务管理器
     * @param entityManagerFactory
     * @return
     */
    @Bean
    public PlatformTransactionManager transactionManager(EntityManagerFactory entityManagerFactory) {
        JpaTransactionManager txManager = new JpaTransactionManager();
        txManager.setEntityManagerFactory(entityManagerFactory);
        return txManager;
    }
}
```

- 实体对象的配置

```java
import lombok.Data;

import javax.persistence.*;
import java.io.Serializable;

/**
 * (User)实体类
 *
 * @author makejava
 * @since 2022-02-11 10:38:17
 */
@Entity
@Table(name = "user")
@Data
public class User implements Serializable {
    private static final long serialVersionUID = -23733744927088877L;

    /**
     * 用户id
     */
    @Id
    @GeneratedValue(generator = "idGenerator", strategy = GenerationType.IDENTITY)
    private Long id;
    /**
     * 用户姓名
     */
    @Column(name = "name", length = 20)
    private String name;
    /**
     * 用户年龄
     */
    @Column(name = "age")
    private Integer age;
    /**
     * 用户性别
     */
    @Column(name = "sex")
    private Integer sex;
    /**
     * 用户电话
     */
    @Column(name = "phone", length = 11)
    private String phone;
    /**
     * 用户地址
     */
    @Column(name = "address", length = 70)
    private String address;
    /**
     * 用户邮箱
     */
    @Column(name = "email", length = 30)
    private String email;

}
```

- repository接口实现操作数据库
  - 分别继承JpaRepository和JpaSpecificationExecutor可以直接使用父类已经写好的一些简单查询和实现复杂的查询条件
  - nativeQuery = true，表示使用原生的sql进行查询
  - 设置参数有两种方式分别`:参数名`和`?参数序号`，第一种方式不用指定顺序，第二种方式对参数的顺序有严格要求

```sql
import com.wu.jpatest.entity.User;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import org.springframework.data.jpa.repository.Query;
import org.springframework.stereotype.Repository;

@Repository
public interface UserRepository extends JpaRepository<User, Long>, JpaSpecificationExecutor<User> {
    /**
     * 通过ID查询单条数据
     *
     * @param id 主键
     * @return 实例对象
     */
    @Query(value = "select id, name, age, sex, phone, address, email from user where id = :id", nativeQuery = true)
    User queryById(Long id);

    /**
     * 统计总行数
     *
     * @return 总行数
     */
    @Query(value = "select count(1) from user",
            nativeQuery = true)
    long count();

    /**
     * 通过年龄分页查询
     * @param age
     * @param pageable
     * @return
     */
    @Query(nativeQuery = true,
            value = "select id, name, age, sex, phone, address, email FROM user WHERE age = ?1",
            countQuery = "select count(*) FROM user WHERE age = ?1")
    Page<User> findAll(Integer age, Pageable pageable);
}
```

- 使用断言实现复杂的条件查询

```java
import com.wu.jpatest.entity.User;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import java.util.List;

public interface UserService {
    /**
     * 通过姓名模糊分页查询
     * @param name
     * @param pageable
     * @return
     */
    Page<User> findAllByName(String name, Pageable pageable);

    /**
     * 通过邮箱模糊查询
     * @param email
     * @return
     */
    List<User> queryByEmail(String email);

    /**
     * 通过年龄范围分页查询
     * @param minAge
     * @param maxAge
     * @param pageable
     * @return
     */
    Page<User> findAllByAge(Integer minAge, Integer maxAge, Pageable pageable);
}
```

```java
import com.wu.jpatest.entity.User;
import com.wu.jpatest.repository.UserRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.domain.Specification;
import org.springframework.stereotype.Service;

import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Predicate;
import javax.persistence.criteria.Root;
import java.util.List;

@Service("userService1")
public class UserServiceImpl implements UserService{
    public static final String LIKE_WILDCARD = "%";
    @Autowired
    private UserRepository userRepository;

    @Override
    public Page<User> findAllByName(String name, Pageable pageable) {
        Specification<User> specification = new Specification<User>() {
            @Override
            public Predicate toPredicate(Root<User> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) {
                Predicate[] predicates = new Predicate[1];
                predicates[0] = criteriaBuilder.like(root.get("name").as(String.class), LIKE_WILDCARD + name + LIKE_WILDCARD);
                return query.where(predicates).getRestriction();
            }
        };
        return userRepository.findAll(specification, pageable);
    }

    @Override
    public List<User> queryByEmail(String email) {
        Specification<User> specification = new Specification<User>() {
            @Override
            public Predicate toPredicate(Root<User> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) {
                Predicate[] predicates = new Predicate[1];
                predicates[0] = criteriaBuilder.like(root.get("email").as(String.class), "%" + email + "%");
                return query.where(predicates).getRestriction();
            }
        };
        return userRepository.findAll(specification);
    }

    @Override
    public Page<User> findAllByAge(Integer minAge, Integer maxAge, Pageable pageable) {
        Specification<User> specification = new Specification<User>() {
            @Override
            public Predicate toPredicate(Root<User> root, CriteriaQuery<?> query, CriteriaBuilder criteriaBuilder) {
                Predicate[] predicates = new Predicate[2];
                predicates[0] = criteriaBuilder.ge(root.get("age").as(Integer.class), minAge);
                predicates[1] = criteriaBuilder.lt(root.get("age").as(Integer.class), maxAge);
                return query.where(predicates).getRestriction();
            }
        };
        return userRepository.findAll(specification, pageable);
    }
}
```

