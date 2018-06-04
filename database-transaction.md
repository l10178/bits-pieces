# 数据库事务
数据库事务总结，主要包括数据库事务ACID属性介绍、数据库并发问题总结、事务传播行为和隔离级别。

# 概念
事务（Transaction）是并发控制的基本单位。它是一个操作序列，这些操作要么都执行，要么都不执行，它是一个不可分割的工作单位。标准定义：指作为单个逻辑工作单元执行的一系列操作，而这些逻辑工作单元需要具有原子性，  一致性，隔离性和持久性四个属性，统称为ACID特性。
#### Atomic（原子性）
事务中包含的操作被看做一个不可分割的逻辑单元，这个逻辑单元中的操作要么全部成功，要么全部失败。
#### Consistency（一致性）
事务执行的结果必须是使数据库从一个一致性状态变到另一个一致性状态。只有合法的数据可以被写入数据库，否则事务应该将其回滚到最初状态。关于数据库一致性，更专业的解释请参考专业的书籍。
#### Isolation（隔离性）
事务允许多个用户对同一个数据进行并发访问，而不破坏数据的正确性和完整性。同时，并行事务的修改必须与其他并行事务的修改相互独立。
#### Durability（持久性）
事务提交后，对系统的影响是永久的。简单理解就是写进去了，不会因为时间、系统环境，关机重启等变化而变化。

# 数据库并发问题
数据库是共享资源，通常有许多个事务同时在运行。当多个事务并发地存取数据库时就会产生同时读取和（或）修改同一数据的情况。若对并发操作不加控制就可能会存取和存储不正确的数据，破坏数据库的一致性。所以数据库管理系统必须提供并发控制机制。

并发操作一般可能带来以下几种问题。为说明问题，先来准备一个例子，假设现在有一个账户表TBL_BANK_ACCOUNT。
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

#### 脏读（Dirty reads）
一个事务读到另一个事务**未提交**的更新数据。

| 事务1 取款事务 | 事务2 工资转账事务 |
| -- | -- |
| 开始事务      |  |
|             | 开始事务 |
|查询余额1000元 | |
|取出100变为900 |  |
|             |  查询余额为900|
|异常发生，事务回滚，余额恢复为1000 |  |
|             |汇入工资2000元，余额为2900|
|             | 提交事务，最终余额2900，损失了100|

#### 不可重复读（Non-Repeatable Reads）
在同一个事务内，读取表中的某一行记录，多次读取的结果不同。与幻读区别的重点在于修改，同样的条件，已经读取过的数据，再次读取出来和上一次的值不一样。

| 事务1 工资计算 | 事务2 汇款和通知 |
| -- | -- |
|    |  开始事务 |
| 开始事务 | 查询工资2000元，通知银行汇款2000 |
|增加加班费6000元 |  |
| 提交事务       | |
|              |再次查询工资应发8000元，邮件通知员工本月发了8000元 |
|              | 提交事务|

#### 幻读(Phantom Reads)
一个事务读到另一个事务已提交的新插入的数据,导致前后不一致。与不可重复读有点类似，都是两次读取。区别的重点在于增加或者删除。

| 事务1 加班录入 | 事务2 加班天数统计，计算加班费 |
| -- | -- |
|      | 开始事务 |
| 开始事务 | 统计员工小明加班3天 |
|             | 通知银行发3天的加班费|
|增加一天加班数据|  |
|提交事务 |  |
|        | 再次统计加班天数，是4天，通知小明发了4天的加班费 |
|              |提交事务 |

# 事务隔离级别
事务隔离级别(Transaction Isolation Level)就是对事务并发控制的等级。标准组织ANSI定义了四个隔离级别，读未提交Read uncommitted、读已提交Read committed、可重复读Repeatable read、串行化(序列化)Serializable，这四个级别严格程度越来越高，同时并发性能越来越低。

#### 读未提交Read uncommitted
一个事务在执行过程中可以看到其他事务没有提交的记录。
#### 读已提交Read committed
只能读已提交的数据。但是读取的数据可以被其他事务修改，这样也就会导致不可重复读。
#### 可重复读Repeatable read
所有被Select获取的数据都不能被修改。
#### 序列化Serializable
所有事务一个接一个的执行。

数据库并发问题还有常说的第一类更新丢失、第二类更新丢失，为减少概念复杂度，在这里没有列出来。

各个隔离级别和问题对应，√: 可能出现，×: 不会出现

|  | 脏读 | 不可重复读|幻读|
| -- | -- | -- | -- |
| Read uncommitted | √ |√ |√ |
| Read committed | × |√ |√ |
|  Repeatable read  | × | × |√ |
|Serializable| × | × |× |

Oracle和SqlServer默认隔离级别是**读已提交**，MySql默认隔离级别是**可重复读**。

Oracle只支持READ COMMITTED和SERIALIZABLE这两种标准隔离级别，另外增加了一个非标准的“只读(read-only)”隔离级别。顺便提一句，他的Serializable隔离级别，并不真正阻塞事务的执行（更深层次的理解另外单说）。

**为避免幻读和不可重复读问题，一般是在一个事务里确保只读取数据一次，而不是提高事务的隔离级别。** 况且Oracle也没法设置Repeatable read。

# 事务传播行为
* PROPAGATION_REQUIRED：如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，该设置是最常用的设置。Spring默认的事务传播行为。
* PROPAGATION_SUPPORTS：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。
* PROPAGATION_MANDATORY：支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。
* PROPAGATION_REQUIRES_NEW：创建新事务，无论当前存不存在事务，都创建新事务。
* PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
* PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。
* PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。

### 嵌套事务
嵌套是子事务套在父事务中执行，子事务是父事务的一部分，在进入子事务之前，父事务建立一个回滚点，叫save point，然后执行子事务，这个子事务的执行也算是父事务的一部分，然后子事务执行结束，父事务继续执行。重点就在于那个save point。看几个问题就明了了：
* 如果子事务回滚，会发生什么？
> 父事务会回滚到进入子事务前建立的save point，然后尝试其他的事务或者其他的业务逻辑，父事务之前的操作不会受到影响，更不会自动回滚。
* 如果父事务回滚，会发生什么？
> 父事务回滚，子事务也会跟着回滚！为什么呢，因为父事务结束之前，子事务是不会提交的，我们说子事务是父事务的一部分，正是这个道理。那么：
* 事务的提交，是什么情况？
> 是父事务先提交，然后子事务提交，还是子事务先提交，父事务再提交？答案是第二种情况，还是那句话，子事务是父事务的一部分，由父事务统一提交。

Thanks: http://www.xuebuyuan.com/3247866.html

#### 只读事务 readOnly
概念：从这一点设置的时间点开始到这个事务结束的过程中，其他事务所提交的数据，该事务将看不见。

应用场合：
1. 如果你一次执行单条查询语句，则没有必要启用事务支持，数据库默认支持SQL执行期间的读一致性；
2. 如果你一次执行多条查询语句，例如统计查询，报表查询，在这种场景下，多条查询SQL必须保证整体的读一致性，否则，在前条SQL查询之后，后条SQL查询之前，数据被其他用户改变，则该次整体的统计查询将会出现读数据不一致的状态，此时，应该启用事务支持。

总结：
1. readonly并不是所有数据库都支持的，不同的数据库下会有不同的结果。
2. 设置了readonly后，connection都会被赋予readonly，效果取决于数据库的实现。
3. 在ORM中，设置了readonly会赋予一些额外的优化，例如在Hibernate中，会被禁止flush等。
4. 由于只读事务不存在数据的修改，因此数据库将会为只读事务提供一些优化手段，例如Oracle对于只读事务，不启动回滚段，不记录回滚log。

# Spring声明式事务

Spring提供了编程式事务和声明式事务两种机制。为便于理解，简单回顾下JDBC和Hibernate的事务管理方式。

* JDBC方式：

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

* Hibernate方式：

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


看下Spring编程式方式：

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

可以看到，以上几种方式都比较复杂，需要我们自己处理事务，要做的事情比较多。而Spring的声明式事务使用简单，它支持注解和xml配置，这里以注解为例。

## @Transactional

Spring声明式事务的使用，一切都落在注解**@Transactional**上。

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

* @Transactional可用于接口、接口方法、实现类以及类方法上。放在接口或类上，相当于为此接口或类下所有的public方法都加了这样一个注解。
* Spring团队的建议是你在具体的类（或类的方法）上使用 @Transactional 注解，而不要使用在类所要实现的任何接口上。你当然可以在接口上使用 @Transactional注解，但是这将只能当你设置了基于接口的代理时它才生效。因为注解是不能继承的，这就意味着如果你正在使用基于类的代理时，那么事务的设置将不能被基于类的代理所识别，而且对象也将不会被事务代理所包装（将被确认为严重的）。因此，请接受Spring团队的建议并且在具体的类上使用@Transactional注解。
* **@Transactional注解应该只被应用到public的方法上。** 如果你在 protected、private 或者 package-visible 的方法上使用，它也不会报错，也不会生效。
* 方法的@Transactional会覆盖类上面声明的事务，也就是方法上的优先级高。

### @Transactional属性

@Transactional有以下几个属性可以设置。

| 属性 | 类型 | 描述 |
| :--- | :--- | :--- |
| value | String | Optional qualifier specifying the transaction manager to be used. |
| propagation | enum: Propagation | Optional propagation setting. |
| isolation | enum: Isolation | Optional isolation level. |
| readOnly | boolean | Read\/write vs. read-only transaction |
| timeout | int \(in seconds granularity\) | Transaction timeout. |
| rollbackFor | Array of Class objects, which must be derived from Throwable. | RuntimeException | Optional array of exception classes that must cause rollback. |
| rollbackForClassName | Array of class names. Classes must be derived from Throwable. | Optional array of names of exception classes that must cause rollback. |
| noRollbackFor | Array of Class objects, which must be derived from Throwable. | Optional array of exception classes that must not cause rollback. |
| noRollbackForClassName | Array of String class names, which must be derived from Throwable. | Optional array of names of exception classes that must not cause rollback. |

注意@Transactional的默认值，传播行为是Required，隔离级别是Default（Oracle和SqlServer是读已提交，MySql是可重复读），**尤其注意rollbackFor，在抛出RuntimeException时才回滚**，意思是，不抛出异常不回滚，抛出异常不是RuntimeException（及其子类）也不回滚，再进一步说，有异常就让它抛出来，不要主动catch消化掉。


#### 传播行为（Propagation）

所谓事务传播行为就是多个事务方法相互调用时，事务如何在这些方法间传播。Spring 支持 7 种事务传播行为：

* **PROPAGATION\_REQUIRED** 如果当前没有事务，就新建一个事务，如果已经存在一个事务中，加入到这个事务中。这是最常见的选择。
* **PROPAGATION\_SUPPORTS** 支持当前事务，如果当前没有事务，就以非事务方式执行。
* **PROPAGATION\_MANDATORY** 使用当前的事务，如果当前没有事务，就抛出异常。
* **PROPAGATION\_REQUIRES\_NEW** 新建事务，如果当前存在事务，把当前事务挂起。
* **PROPAGATION\_NOT\_SUPPORTED** 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
* **PROPAGATION\_NEVER** 以非事务方式执行，如果当前存在事务，则抛出异常。
* **PROPAGATION\_NESTED** 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则开启一个新的事务。PROPAGATION\_NESTED 开始一个 "嵌套的" 事务,  它是已经存在事务的一个真正的子事务. 潜套事务开始执行时,  它将取得一个 savepoint. 如果这个嵌套事务失败, 我们将回滚到此 savepoint. 潜套事务是外部事务的一部分, 只有外部事务结束后它才会被提交. 嵌套事务回滚不影响外部事务，但外部事务回滚将导致嵌套事务回滚。 使用嵌套事务需要JDBC3.0并且事务管理器开启嵌套事务（常用的JpaTransactionManager和HibernateTransactionManager默认是不开启的），如果没有开启，运行时将抛出异常。

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

@Transactional注解加在UserService.save和AddressService.save两个方法上，看下不同的传播形式。
![](spring-transaction.png)

具体的事务开启和关闭流程，设置spring的日志级别为debug后，运行，可看到类似于这面这样的日志。这里使用了Spring data jpa，打印的是JpaTransactionManager的日志。UserServices使用的传播行为是REQUIRED，AddressService使用REQUIRES\_NEW。

```
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
