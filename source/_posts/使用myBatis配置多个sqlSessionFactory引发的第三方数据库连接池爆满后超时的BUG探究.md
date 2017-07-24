---
title: 使用myBatis配置多个sqlSessionFactory引发的第三方数据库连接池爆满后超时的BUG探究
date: 2017-03-16 21:48:21
tags:
  - java
  - myBatis
  - 日常问题
---

##### 背景:
今天项目开始进行日常测试，由于有一个功能需要在运行的时候实时加载sqlmap.xml进入sqlSessionFactory的configuration里面，因此，配置了两个sqlSessionFactory,一个用来做普通的数据库操作，一个在运行时动态的加载sqlmap文件后做特殊的业务处理。

普通的sqlSessionFacotry只是简单使用了org.mybatis.spring.SqlSessionFactoryBean，然后配置了数据源以及静态的mybatis-config，如下：
<!-- more -->

```xml
<bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="myDataSource"/>
    <property name="configLocation" value="classpath:META-INF/mybatis-config.xml"/>
</bean>
```

而在系统运行时要动态加载sqlmap文件的sqlSessionFactory，则不能直接使用Mybatis自带的SqlSessionFactoryBean，于是自己实现了一个DynamicSqlSessionFactoryBean，除了保留SqlSessionFactoryBean里面的所有变量和方法实现，额外定制了动态加载sqlmap文件的一些其他定制实现，具体在这里不阐述。配置文件如下：

```xml
<bean id="dynamicSessionFactory" class="com.xiaojin.mybatis.support.DynamicSqlSessionFactoryBean">
    <property name="dynamicSqlSessionFactoryParser" ref="dynamicSessionFactoryParser"/>
</bean>

<!-- dynamicSessionFactoryParser 主要就是用来加载动态的sqlmap文件的 -->
<bean id="dynamicSessionFactoryParser" class="com.tmall.udcp.support.DynamicSqlSessionFactoryParser">
    <property name="configLocation" value="classpath:META-INF/mybatis-dynamic-config.xml"/>
    <property name="dataSource" ref="myDataSource"/>
    <property name="dynamicSqlMapLoadService" ref="dynamicSqlMapLoadService"/>
</bean>
```

##### 问题现象：
在某个业务步骤，需要使用到动态加载到的sqlmap文件做数据库访问，加载数据，这个时候出现了一个诡异的现象，在应用启动后，开始的几次加载数据成功了，后面就一直会抛一个超时的异常。

由于是跨系统应用，在业务应用A调用核心的应用B的接口后会出现第一个异常，就是接口调用超时异常。


##### 问题追溯：

一开始以为是数据访问过慢，导致应用A调用应用B接口超时了，因为超时时间设的是3秒，而业务是有一定的复杂度的。于是对应用B的接口实现做了一整套优化，这里不细讲。

然后重新部署之后，发现问题还是存在，这个时候详细检查了系统日志，发现是报了一个获取不到数据库连接的超时异常，而出异常的代码则是开源的数据库连接池中间件Druid，部分异常日志如下：
 ```
 org.springframework.jdbc.support.SQLErrorCodesFactory:227 - Error while extracting database product name - falling back to empty error codes                      
org.springframework.jdbc.support.MetaDataAccessException: Error while extracting DatabaseMetaData; nested exception is com.alibaba.druid.pool.GetConnectionTimeoutException: wait millis 5000, act
ive 10                                                                                                                                                                                            
        at org.springframework.jdbc.support.JdbcUtils.extractDatabaseMetaData(JdbcUtils.java:305) ~[spring-jdbc-4.3.0.RC2.jar:4.3.0.RC2]                                                          
        at org.springframework.jdbc.support.JdbcUtils.extractDatabaseMetaData(JdbcUtils.java:329) ~[spring-jdbc-4.3.0.RC2.jar:4.3.0.RC2]                                                          
        at org.springframework.jdbc.support.SQLErrorCodesFactory.getErrorCodes(SQLErrorCodesFactory.java:214) ~[spring-jdbc-4.3.0.RC2.jar:4.3.0.RC2]                                              
        at org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator.setDataSource(SQLErrorCodeSQLExceptionTranslator.java:134) [spring-jdbc-4.3.0.RC2.jar:4.3.0.RC2]                   
        at org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator.<init>(SQLErrorCodeSQLExceptionTranslator.java:97) [spring-jdbc-4.3.0.RC2.jar:4.3.0.RC2]            
        ....
 Caused by: com.alibaba.druid.pool.GetConnectionTimeoutException: wait millis 5000, active 10                                                                                                      
        at com.alibaba.druid.pool.DruidDataSource.getConnectionInternal(DruidDataSource.java:1071) ~[druid-1.0.2.jar:1.0.2]                                                                       
        at com.alibaba.druid.pool.DruidDataSource.getConnectionDirect(DruidDataSource.java:898) ~[druid-1.0.2.jar:1.0.2]                                                                          
        at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:4544) ~[druid-1.0.2.jar:1.0.2]                                                                        
        at com.alibaba.druid.filter.stat.StatFilter.dataSource_getConnection(StatFilter.java:661) ~[druid-1.0.2.jar:1.0.2]                                                                        
        at com.alibaba.druid.filter.FilterChainImpl.dataSource_connect(FilterChainImpl.java:4540) ~[druid-1.0.2.jar:1.0.2]                                                                        
        at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:880) ~[druid-1.0.2.jar:1.0.2]                                                                                
        at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:872) ~[druid-1.0.2.jar:1.0.2]                                                                                
        at com.alibaba.druid.pool.DruidDataSource.getConnection(DruidDataSource.java:97) ~[druid-1.0.2.jar:1.0.2]                
        ....      
 ```
于是怀疑是不是这个中间件与我们应用的某些二方包有冲突或有BUG，开始进入Druid的源码进行检查，首先进入到DruidDataSource的出错位置，部分代码如下：
```java
public DruidPooledConnection getConnectionDirect(long maxWaitMillis) throws SQLException {
    for (;;) {
      //出错代码在这一行
        DruidPooledConnection poolableConnection = getConnectionInternal(maxWaitMillis);

        if (isTestOnBorrow()) {
            boolean validate = testConnectionInternal(poolableConnection.getConnection());
            if (!validate) {
                if (LOG.isDebugEnabled()) {
                    LOG.debug("skip not validate connection.");
                }
                .....

```

这里貌似没看出什么问题，于是深入这个方法的具体实现，最后发现，所有的链接都需要从DruidDataSource的一个成员变量：    private volatile DruidConnectionHolder[] connections; 里面获取的，这就是一个数据库连接池，每次都在条件允许的情况下，都会取链接池的最后一个可用链接，而当我调试到这里面的时候，发现connections已经满了（10个），而且所有的链接都是不可用的（active,即活跃状态），因此就会陷入等待，超过了5000ms之后就会抛出超时异常，因此才会出现这个异常。然后调试的窗口发现所有的connections虽然都是活跃状态，但都是空的。那么结果就显而易见了，即是我们的数据库访问代码中，存在一种情况：创建连接之后，进行相关的数据库操作，之后没有将相关的链接关闭，而随着方法执行完毕，链接对象已经被回收，因此而链接池中就存在了10个null的活跃链接。

这个时候回去检查了一下相关的数据库操作代码，如下：

```java
public class DynamicDAOImpl implements DynamicDAO {

    //这个sessionFacotry就是需要执行动态文件加载的sessionFacotry
    private SqlSessionFactory sqlSession;

    @Override
    public List<Map<String, Object>> getList(String statement, Map<String, Object> param) {
            return this.sqlSession.openSession().selectList(statement, param);
        }
    }

    @Override
    public Map<String, Object> getObject(String statement, Map<String, Object> param) {
            return this.sqlSession.openSession().selectOne(statement, param);
    }

    public void setSqlSession(SqlSessionFactory sqlSession) {
        this.sqlSession = sqlSession;
    }
}

```

看了一下这段代码，觉得应该是openSession()方法有问题，因此看了一下openSession的DefaultSqlSessionFactory的实现代码：
```java
@Override
public SqlSession openSession() {
  return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
}

private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
  Transaction tx = null;
  try {
    final Environment environment = configuration.getEnvironment();
    final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
    tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
    final Executor executor = configuration.newExecutor(tx, execType);
    return new DefaultSqlSession(configuration, executor, autoCommit);
  } catch (Exception e) {
    closeTransaction(tx); // may have fetched a connection so lets call close()
    throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
  } finally {
    ErrorContext.instance().reset();
  }
}

```

这个方法直接创建了一个DefaultSqlSession，即数据库连接，一开始以为是ExecutorType这个参数有问题，然后看了一下官方的文档，发现这个枚举只是在做sql的PrepareStatement的一个参数而已，分别是SIMPLE（每次都使用心得PrepareSatement）,REUSE(重复使用同一种sql的PrePareStatement)，BATCH（支持批量PrePareStateMent），然后看了一下每个枚举对应的Executor的实现（这里就不详细讲了，有机会新开文章），发现确实如此，和具体的问题没有关系，于是看了一下DefaultSqlSession的具体代码：

```java
public class DefaultSqlSession implements SqlSession {

private Configuration configuration;
private Executor executor;

private boolean autoCommit;
private boolean dirty;
private List<Cursor<?>> cursorList;
....

}
```

看到这里，答案已经付出水面了，因为这个类实现了SqlSession，而Mybatis的SqlSession又实现了Closeable，因此这个SqlSession是需要被手动关闭的。于是我把代码改了一下：

```java
public class DynamicDAOImpl implements DynamicDAO {

    private SqlSessionFactory sqlSession;

    @Override
    public List<Map<String, Object>> getList(String statement, Map<String, Object> param) {
        try (SqlSession sqlSession = this.sqlSession.openSession(ExecutorType.REUSE)) {
            return sqlSession.selectList(statement, param);
        }
    }

    @Override
    public Map<String, Object> getObject(String statement, Map<String, Object> param) {
        try (SqlSession sqlSession = this.sqlSession.openSession(ExecutorType.REUSE)) {
            return sqlSession.selectOne(statement, param);
        }
    }

    public void setSqlSession(SqlSessionFactory sqlSession) {
        this.sqlSession = sqlSession;
    }
}

```

然后启动应用，发现这个问题果然修复了！

##### 复盘:
这个问题其实是一个很简单的问题，因为一时的粗心，导致找答案的时候，找错了方向，如果一开始就确定是数据连接的问题，其实这个问题很容易就找出来。

最重要的是，这个代码写的真的很不好，没有手动关闭一个链接，这真的是一个初级错误（虽然这个代码不是我写的...），很明显，会出现这个错误是因为coder对Mybatis不够熟悉了解导致的。

因此个人觉得，在使用一个开源框架的时候，在不确定是否出错的情况下，第一是先看注释，然后看看官方文档，实现不行直接看源码。
