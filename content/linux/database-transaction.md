# 数据库事务

数据库事务总结，主要包括数据库事务 ACID 属性介绍、数据库并发问题总结、事务传播行为和隔离级别。

## 概念

事务（Transaction）是并发控制的基本单位。它是一个操作序列，这些操作要么都执行，要么都不执行，它是一个不可分割的工作单位。标准定义：指作为单个逻辑工作单元执行的一系列操作，而这些逻辑工作单元需要具有原子性， 一致性，隔离性和持久性四个属性，统称为 ACID 特性。

### Atomic（原子性）

事务中包含的操作被看做一个不可分割的逻辑单元，这个逻辑单元中的操作要么全部成功，要么全部失败。

### Consistency（一致性）

事务执行的结果必须是使数据库从一个一致性状态变到另一个一致性状态。只有合法的数据可以被写入数据库，否则事务应该将其回滚到最初状态。关于数据库一致性，更专业的解释请参考专业的书籍。

### Isolation（隔离性）

事务允许多个用户对同一个数据进行并发访问，而不破坏数据的正确性和完整性。同时，并行事务的修改必须与其他并行事务的修改相互独立。

### Durability（持久性）

事务提交后，对系统的影响是永久的。简单理解就是写进去了，不会因为时间、系统环境，关机重启等变化而变化。

## 数据库并发问题

数据库是共享资源，通常有许多个事务同时在运行。当多个事务并发地存取数据库时就会产生同时读取和（或）修改同一数据的情况。若对并发操作不加控制就可能会存取和存储不正确的数据，破坏数据库的一致性。所以数据库管理系统必须提供并发控制机制。

并发操作一般可能带来以下几种问题。为说明问题，先来准备一个例子，假设现在有一个账户表 TBL_BANK_ACCOUNT。

```sql
CREATE TABLE TBL_BANK_ACCOUNT（
AccountId CHAR(4) NOT NULL, -- 银行账号
Username NVARCHAR(63) NOT NULL, -- 用户名
Balance BIGINT NOT NULL -- 余额
)
INSERT INTO TBL_BANK_ACCOUNT
VALUES ('9555', '小明', 1000) -- 北京分行账号
INSERT INTO dbo.BankAccount
VALUES ('9556', '小明', 2000) -- 上海分行账号
```

### 脏读（Dirty reads）

一个事务读到另一个事务**未提交**的更新数据。

| 事务 1 取款事务                     | 事务 2 工资转账事务                 |
| ----------------------------------- | ----------------------------------- |
| 开始事务                            |                                     |
|                                     | 开始事务                            |
| 查询余额 1000 元                    |                                     |
| 取出 100 变为 900                   |                                     |
|                                     | 查询余额为 900                      |
| 异常发生，事务回滚，余额恢复为 1000 |                                     |
|                                     | 汇入工资 2000 元，余额为 2900       |
|                                     | 提交事务，最终余额 2900，损失了 100 |

### 不可重复读（Non-Repeatable Reads）

在同一个事务内，读取表中的某一行记录，多次读取的结果不同。与幻读区别的重点在于修改，同样的条件，已经读取过的数据，再次读取出来和上一次的值不一样。

| 事务 1 工资计算    | 事务 2 汇款和通知                                      |
| ------------------ | ------------------------------------------------------ |
|                    | 开始事务                                               |
| 开始事务           | 查询工资 2000 元，通知银行汇款 2000                    |
| 增加加班费 6000 元 |                                                        |
| 提交事务           |                                                        |
|                    | 再次查询工资应发 8000 元，邮件通知员工本月发了 8000 元 |
|                    | 提交事务                                               |

### 幻读(Phantom Reads)

一个事务读到另一个事务已提交的新插入的数据,导致前后不一致。与不可重复读有点类似，都是两次读取。区别的重点在于增加或者删除。

| 事务 1 加班录入  | 事务 2 加班天数统计，计算加班费                      |
| ---------------- | ---------------------------------------------------- |
|                  | 开始事务                                             |
| 开始事务         | 统计员工小明加班 3 天                                |
|                  | 通知银行发 3 天的加班费                              |
| 增加一天加班数据 |                                                      |
| 提交事务         |                                                      |
|                  | 再次统计加班天数，是 4 天，通知小明发了 4 天的加班费 |
|                  | 提交事务                                             |

## 事务隔离级别

事务隔离级别(Transaction Isolation Level)就是对事务并发控制的等级。标准组织 ANSI 定义了四个隔离级别，读未提交 Read uncommitted、读已提交 Read committed、可重复读 Repeatable read、串行化(序列化)Serializable，这四个级别严格程度越来越高，同时并发性能越来越低。

### 读未提交 Read uncommitted

一个事务在执行过程中可以看到其他事务没有提交的记录。

### 读已提交 Read committed

只能读已提交的数据。但是读取的数据可以被其他事务修改，这样也就会导致不可重复读。

### 可重复读 Repeatable read

所有被 Select 获取的数据都不能被修改。

### 序列化 Serializable

所有事务一个接一个的执行。

数据库并发问题还有常说的第一类更新丢失、第二类更新丢失，为减少概念复杂度，在这里没有列出来。

各个隔离级别和问题对应，√: 可能出现，×: 不会出现

|                  | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| Read uncommitted | √    | √          | √    |
| Read committed   | ×    | √          | √    |
| Repeatable read  | ×    | ×          | √    |
| Serializable     | ×    | ×          | ×    |

Oracle 和 SqlServer 默认隔离级别是**读已提交**，MySql 默认隔离级别是**可重复读**。

Oracle 只支持 READ COMMITTED 和 SERIALIZABLE 这两种标准隔离级别，另外增加了一个非标准的“只读(read-only)”隔离级别。顺便提一句，他的 Serializable 隔离级别，并不真正阻塞事务的执行（更深层次的理解另外单说）。

**为避免幻读和不可重复读问题，一般是在一个事务里确保只读取数据一次，而不是提高事务的隔离级别。** 况且 Oracle 也没法设置 Repeatable read。

## 事务传播行为

- PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。Spring 默认的事务传播行为。
- PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
- PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
- PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。
- PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与 PROPAGATION_REQUIRED 类似的操作。

### 嵌套事务

嵌套是子事务套在父事务中执行，子事务是父事务的一部分，在进入子事务之前，父事务建立一个回滚点，叫 save point，然后执行子事务，这个子事务的执行也算是父事务的一部分，然后子事务执行结束，父事务继续执行。重点就在于那个 save point。看几个问题就明了了：

- 如果子事务回滚，会发生什么？
  > 父事务会回滚到进入子事务前建立的 save point，然后尝试其他的事务或者其他的业务逻辑，父事务之前的操作不会受到影响，更不会自动回滚。
- 如果父事务回滚，会发生什么？
  > 父事务回滚，子事务也会跟着回滚！为什么呢，因为父事务结束之前，子事务是不会提交的，我们说子事务是父事务的一部分，正是这个道理。那么：
- 事务的提交，是什么情况？
  > 是父事务先提交，然后子事务提交，还是子事务先提交，父事务再提交？答案是第二种情况，还是那句话，子事务是父事务的一部分，由父事务统一提交。

### 只读事务 readOnly

概念：从这一点设置的时间点开始到这个事务结束的过程中，其他事务所提交的数据，该事务将看不见。

应用场合：

1. 如果你一次执行单条查询语句，则没有必要启用事务支持，数据库默认支持 SQL 执行期间的读一致性；
2. 如果你一次执行多条查询语句，例如统计查询，报表查询，在这种场景下，多条查询 SQL 必须保证整体的读一致性，否则，在前条 SQL 查询之后，后条 SQL 查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持。

总结：

1. readonly 并不是所有数据库都支持的，不同的数据库下会有不同的结果。
2. 设置了 readonly 后，connection 都会被赋予 readonly，效果取决于数据库的实现。
3. 在 ORM 中，设置了 readonly 会赋予一些额外的优化，例如在 Hibernate 中，会被禁止 flush 等。
4. 由于只读事务不存在数据的修改，因此数据库将会为只读事务提供一些优化手段，例如 Oracle 对于只读事务，不启动回滚段，不记录回滚 log。

## Spring 声明式事务

Spring 提供了编程式事务和声明式事务两种机制。为便于理解，简单回顾下 JDBC 和 Hibernate 的事务管理方式。

- JDBC 方式：

```java
Connection conn = DataSourceUtils.getConnection();
//开启事务
conn.setAutoCommit(false);
try {
  Object retVal =
  callback.doInConnection(conn);
  conn.commit(); //提交事务
  return retVal;
}catch (Exception e) {
  conn.rollback();//回滚事务
  throw e;
}finally {
  conn.close();
}
```

- Hibernate 方式：

```java
Session session = null;
Transaction transaction = null;
try {
  session = factory.openSession();
  //开启事务
  transaction = session.beginTransaction();
  transation.begin();
  session.save(user);
  transaction.commit();//提交事务
} catch (Exception e) {
  transaction.rollback();//回滚事务
  return false;
}finally{
  session.close();
}
```

看下 Spring 编程式方式：

```java
//1.获取事务管理器
PlatformTransactionManager txManager =ctx.getBean("txManager");
//2.定义事务属性
DefaultTransactionDefinition td = new DefaultTransactionDefinition();
td.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
//3开启事务,得到事务状态
TransactionStatus status = txManager.getTransaction(td);
try {
  //4.执行数据库操作
  jdbcTempate.queryForInt("select count(*) from tbl_doc");
  //5、提交事务
  txManager.commit(status);

}catch (Exception e) {
  //6、回滚事务
  txManager.rollback(status);
}
```

可以看到，以上几种方式都比较复杂，需要我们自己处理事务，要做的事情比较多。而 Spring 的声明式事务使用简单，它支持注解和 xml 配置，这里以注解为例。

### @Transactional

Spring 声明式事务的使用，一切都落在注解**@Transactional**上。

先看一个简单的例子，在实现类的加注解，实现事务控制。

```java
<!-- the service class that we want to make transactional -->
@Transactional
public class DefaultFooService implements FooService {
  Foo getFoo(String fooName);
  Foo getFoo(String fooName, String barName);
  void insertFoo(Foo foo);
  void updateFoo(Foo foo);
}
```

#### 使用方法

- @Transactional 可用于接口、接口方法、实现类以及类方法上。放在接口或类上，相当于为此接口或类下所有的 public 方法都加了这样一个注解。
- Spring 团队的建议是你在具体的类（或类的方法）上使用 @Transactional 注解，而不要使用在类所要实现的任何接口上。你当然可以在接口上使用 @Transactional 注解，但是这将只能当你设置了基于接口的代理时它才生效。因为注解是不能继承的，这就意味着如果你正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装（将被确认为严重的）。因此，请接受 Spring 团队的建议并且在具体的类上使用@Transactional 注解。
- **@Transactional 注解应该只被应用到 public 的方法上。** 如果你在 protected、private 或者 package-visible 的方法上使用，它也不会报错，也不会生效。
- 方法的@Transactional 会覆盖类上面声明的事务，也就是方法上的优先级高。

#### 传播行为（Propagation）

所谓事务传播行为就是多个事务方法相互调用时，事务如何在这些方法间传播。Spring 支持 7 种事务传播行为：

- **PROPAGATION_REQUIRED** 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。
- **PROPAGATION_SUPPORTS** 支持当前事务，如果当前没有事务，就以非事务方式执行。
- **PROPAGATION_MANDATORY** 使用当前的事务，如果当前没有事务，就抛出异常。
- **PROPAGATION_REQUIRES_NEW** 新建事务，如果当前存在事务，把当前事务挂起。
- **PROPAGATION_NOT_SUPPORTED** 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- **PROPAGATION_NEVER** 以非事务方式执行，如果当前存在事务，则抛出异常。
- **PROPAGATION_NESTED** 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则开启一个新的事务。PROPAGATION_NESTED 开始一个 "嵌套的" 事务, 它是已经存在事务的一个真正的子事务. 潜套事务开始执行时, 它将取得一个 savepoint. 如果这个嵌套事务失败, 我们将回滚到此 savepoint. 潜套事务是外部事务的一部分, 只有外部事务结束后它才会被提交. 嵌套事务回滚不影响外部事务，但外部事务回滚将导致嵌套事务回滚。 使用嵌套事务需要 JDBC3.0 并且事务管理器开启嵌套事务（常用的 JpaTransactionManager 和 HibernateTransactionManager 默认是不开启的），如果没有开启，运行时将抛出异常。

举例，现有用户和地址管理，假如每增加一个新用户就需要自动增加一个与此用户相关的地址（这例子真挫，至今没有见到过这样的需求）。那么代码大致是这个样子：

```java
//用户管理类
public class UserService {

  @Resource
  private UserDao userDao;

  /** 地址管理类 */
  @Resource
  private AddressService addressService;

  @Transactional
  public void save(User user){
    //执行sql保存用户信息
    userDao.add(user);

    Address address=new Address();//设置地址信息

    //执行sql保存地址信息
    addressService.save(address);
  }
}

//测试类
public class ApplicationTest {
  @Resource
  private UserService userService;

  @Test
  public void transactionalTest() {
    User user = new User();
    user.setUsername("Test-001");
    userService.save(user);
  }
}
```

@Transactional 注解加在 UserService.save 和 AddressService.save 两个方法上，看下不同的传播形式。
[![spring-transaction](./spring-transaction.png)](./spring-transaction.png)

具体的事务开启和关闭流程，设置 spring 的日志级别为 debug 后，运行，可看到类似于这面这样的日志。这里使用了 Spring data jpa，打印的是 JpaTransactionManager 的日志。UserServices 使用的传播行为是 REQUIRED，AddressService 使用 REQUIRES_NEW。

```log
- Creating new transaction with name [UserService.save]:PROPAGATION_REQUIRED,ISOLATION_DEFAULT; ''
- Opened new EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@16602333] for JPA transaction
- Exposing JPA transaction as JDBC transaction [org.springframework.orm.jpa.vendor.HibernateJpaDialect$HibernateConnectionHandle@2dd4a7a9]
- Found thread-bound EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@16602333] for JPA transaction
- Participating in existing transaction
- Found thread-bound EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@16602333] for JPA transaction
- Suspending current transaction, creating new transaction with name [AddressService.save]
- Opened new EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@28b7646] for JPA transaction
- Exposing JPA transaction as JDBC transaction [org.springframework.orm.jpa.vendor.HibernateJpaDialect$HibernateConnectionHandle@40239b34]
- Found thread-bound EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@28b7646] for JPA transaction
- Participating in existing transaction
- Initiating transaction commit
- Committing JPA transaction on EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@28b7646]
- Closing JPA EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@28b7646] after transaction
- Closing JPA EntityManager
- Resuming suspended transaction after completion of inner transaction
- Initiating transaction commit
- Committing JPA transaction on EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@16602333]
- Closing JPA EntityManager [org.hibernate.jpa.internal.EntityManagerImpl@16602333] after transaction
```
