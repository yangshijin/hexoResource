---
title: myBatis中的Spring技术探究：SqlSessionFactoryBean之FactoryBean
date: 2017-03-18 19:44:54
tags:
  - java
  - spring
  - myBatis
---
　　今天有人问了我一个问题：为什么下面这样的spring代码代码能够正常运行呢,sessionFactory这个bean是SqlSessionFactoryBean类型的，但是genericDAO的成员变量sqlSession是SqlSessionFactory类型的，为什么可以这么配置呢？我们先来看看代码：

<!-- more -->

首先是配置代码
```xml
<!-- 这是myBatis的sessionFactory配置-->
<bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <property name="dataSource" ref="myDataSource"/>
    <property name="configLocation" value="classpath:META-INF/mybatis-config.xml"/>
</bean>

<bean id="genericDAO" class="com.xiaojin.dal.common.impl.GenericDAOImpl">
    <property name="sqlSession"  ref="sessionFactory"/>
</bean>
```

接着是java代码

```java
public interface GenericDAO {
    /**
     * 获取数据列表
     */
    List<Map<String, Object>> getList(String statement, Map<String, Object> param);
    /**
     * 获取单个数据对象
     */
    Map<String, Object> getObject(String statement, Map<String, Object> param);
}

public class GenericDAOImpl implements GenericDAO {

    private SqlSessionFactory sqlSession;

    @Override
    public List<Map<String, Object>> getList(String statement, Map<String, Object> param) {
        try (SqlSession sqlSession = this.sqlSession.openSession()) {
            return sqlSession.selectList(statement, param);
        }
    }

    @Override
    public Map<String, Object> getObject(String statement, Map<String, Object> param) {
        try (SqlSession sqlSession = this.sqlSession.openSession()) {
            return sqlSession.selectOne(statement, param);
        }
    }

    public void setSqlSession(SqlSessionFactory sqlSession) {
        this.sqlSession = sqlSession;
    }
}
```

看到这里熟悉spring技术的同学可能已经大概知道这是为什么了，不熟悉的同学可能以为SqlSessionFactoryBean是SqlSessionFactory的子类，这里我们先不讲，我们再来看看SqlSessionFactoryBean的类定义（只贴上相关的代码，其他先忽略不看）：

```java
public class SqlSessionFactoryBean implements
FactoryBean<SqlSessionFactory>, InitializingBean, ApplicationListener<ApplicationEvent> {

private SqlSessionFactoryBuilder sqlSessionFactoryBuilder = new SqlSessionFactoryBuilder();

private SqlSessionFactory sqlSessionFactory;
    ...
}
```
看到这个类型定义，发现sqlSessionFacotryBean这个类并不是sqlSessionFactory的子类，只是实现了FactoryBean这个泛型接口的时候，用了SqlSessionFactory做泛型参数，另外有一个SqlSessionFacotry的成员变量，似乎还有一个专门构建SqlSessionFacotry的构建器类。

类成员变量没什么好说的，我们直接来看看FacotryBean的定义：

```java
/**
 * Interface to be implemented by objects used within a {@link BeanFactory}
 * which are themselves factories. If a bean implements this interface,
 * it is used as a factory for an object to expose, not directly as a bean
 * instance that will be exposed itself.
 *
 * <p><b>NB: A bean that implements this interface cannot be used as a
 * normal bean.</b> A FactoryBean is defined in a bean style, but the
 * object exposed for bean references ({@link #getObject()} is always
 * the object that it creates.
 *
 * <p>FactoryBeans can support singletons and prototypes, and can
 * either create objects lazily on demand or eagerly on startup.
 * The {@link SmartFactoryBean} interface allows for exposing
 * more fine-grained behavioral metadata.
 *
 * <p>This interface is heavily used within the framework itself, for
 * example for the AOP {@link org.springframework.aop.framework.ProxyFactoryBean}
 * or the {@link org.springframework.jndi.JndiObjectFactoryBean}.
 * It can be used for application components as well; however,
 * this is not common outside of infrastructure code.
 *
 * <p><b>NOTE:</b> FactoryBean objects participate in the containing
 * BeanFactory's synchronization of bean creation. There is usually no
 * need for internal synchronization other than for purposes of lazy
 * initialization within the FactoryBean itself (or the like).
 *
 * @author Rod Johnson
 * @author Juergen Hoeller
 * @since 08.03.2003
 * @see org.springframework.beans.factory.BeanFactory
 * @see org.springframework.aop.framework.ProxyFactoryBean
 * @see org.springframework.jndi.JndiObjectFactoryBean
 */

public interface FactoryBean<T> {

  T getObject() throws Exception;

  Class<?> getObjectType();

  boolean isSingleton();

}
```

看了注释，大致明白了FactoryBean的用处，实现这个接口的类主要是BeanFacotry内创建实例化的bean，这个接口的作用是让BeanFacotry内的Bean能够实现自己的工厂，来创建不是自身类型的对象。简单的说，就是在Spring容器里面的bean实现这个接口，就是创建一个和自身类型不同的类对象，这么用的理由是因为spring内部的一些bean实例化都非常复杂，很多配置信息，需要先处理一下，得到一个可用的实例，因此实现了FacotryBean接口后，可用通过实现getObject()方法，来做一些你想要的定制化处理。

那么，getObject()方法是在什么时候调用的呢？尽然是BeanFacotry容器内的Bean，那自然和BeanFacotry有关，看了一下BeanFacotry的默认实现AbstractBeanFactory中的getBean方法实现（无关的日志和其他代码已经去掉），真相已经付出水面了：

```java
public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {
    @Override
  public Object getBean(String name) throws BeansException {
      return doGetBean(name, null, null, false);
  }

  protected <T> T doGetBean(final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly) throws BeansException {

    //将bean名称做一个相关的转换，比如别名、以及FacotryBean的'&'解引用，意思是不拿getObject方法返回的bean，而是拿实现了 FacotryBean接口的那个类对象
    final String beanName = transformedBeanName(name);
    Object bean;

    // 首先从单例缓存中尝试获取bean对象
    Object sharedInstance = getSingleton(beanName);

    if (sharedInstance != null && args == null) {
      //获取bean
      bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    }else {
    // 如果再最近的一个周期内，我们已经创建了bean，那么抛出异常
    if (isPrototypeCurrentlyInCreation(beanName)) {
      throw new BeanCurrentlyInCreationException(beanName);
    }

    // 如果类已经在这个容器里面定义（未实例化）
    BeanFactory parentBeanFactory = getParentBeanFactory();
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
      // 没找到，查父工厂
      String nameToLookup = originalBeanName(name);
      //由父工厂代理来获取bean
      if (args != null) {
        // Delegation to parent with explicit args.
        return (T) parentBeanFactory.getBean(nameToLookup, args);
      }
      else {
        // No args -> delegate to standard getBean method.
        return parentBeanFactory.getBean(nameToLookup, requiredType);
      }
    }
    try {
      //获取bean的定义，进行bean的合并性检查，可能抛出异常，不细讲
      final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
      checkMergedBeanDefinition(mbd, beanName, args);

      // 获取bean依赖到的bean的名称列表，并进行相关处理检查
      String[] dependsOn = mbd.getDependsOn();
      if (dependsOn != null) {
        for (String dependsOnBean : dependsOn) {
          //检查被依赖的bean是否有循环依赖（即A->B，B->A)
          if (isDependent(beanName, dependsOnBean)) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "Circular depends-on relationship between '" + beanName + "' and '" + dependsOnBean + "'");
          }
          //在容器内注册依赖关系
          registerDependentBean(dependsOnBean, beanName);
          //获取bean，这里其实就是先把依赖的bean初始化，防止被依赖到的bean还没实例化
          getBean(dependsOnBean);
        }
      }
      // 这里是创建bean的实例
      if (mbd.isSingleton()) {
        //如果是单例，先创建已给自身的实例，并注册
        sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
          @Override
          public Object getObject() throws BeansException {
            try {
              return createBean(beanName, mbd, args);
            }
            catch (BeansException ex) {
              destroySingleton(beanName);
              throw ex;
            }
          }
        });
        //这里就是对Bean做相关的处理，返回处理后新的bean，FacotryBean的getObject方法就是在这里处理调用
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
      }
      //如果不是单例
      else if (mbd.isPrototype()) {
        // It's a prototype -> create a new instance.
        Object prototypeInstance = null;
        try {
          //先做预处理，然后创建bean
          beforePrototypeCreation(beanName);
          prototypeInstance = createBean(beanName, mbd, args);
        }
        finally {
          //后处理
          afterPrototypeCreation(beanName);
        }
        //这里就是对Bean做相关的处理，返回处理后新的bean，FacotryBean的getObject方法就是在这里处理调用
        bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
      }
      //Scope类型的bean的相关创建，类似Proptype，这里不细讲
      else {
        String scopeName = mbd.getScope();
        final Scope scope = this.scopes.get(scopeName);
        if (scope == null) {
          throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
        }
        try {
          Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
            @Override
            public Object getObject() throws BeansException {
              beforePrototypeCreation(beanName);
              try {
                return createBean(beanName, mbd, args);
              }
              finally {
                afterPrototypeCreation(beanName);
              }
            }
          });
          bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
        }
      }
    }   
    catch (BeansException ex) {
      cleanupAfterBeanCreationFailure(beanName);
      throw ex;
    }
  }
    return (T) bean;
  }
}
```
看完真个方法，大概知道了Bean的创建获取过程，接下来看看本期主题FacotryBean最重要的方法，getObjectForBeanInstance()的代码实现：

```java
protected Object getObjectForBeanInstance(
  Object beanInstance, String name, String beanName, RootBeanDefinition mbd) {

// 如果bean是一个解引用（上面有讲，&+beanName）但又不是FacotryBean类型，抛异常
if (BeanFactoryUtils.isFactoryDereference(name) && !(beanInstance instanceof FactoryBean)) {
  throw new BeanIsNotAFactoryException(transformedBeanName(name), beanInstance.getClass());
}

//如果不属于FacotryBean或者是FacotryBean的解引用（解引用即返回自身，不做处理），直接返回
if (!(beanInstance instanceof FactoryBean) || BeanFactoryUtils.isFactoryDereference(name)) {
  return beanInstance;
}

Object object = null;
//如果类定义为空，试图从容器缓存中读取
if (mbd == null) {
  object = getCachedObjectForFactoryBean(beanName);
}
if (object == null) {
  //将前面实例化的bean强转为Facotry类型，因为不是这个类型不能走到这行代码
  FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
  // 如果bean定义为空，且容器具有这个定义，直接获取
  if (mbd == null && containsBeanDefinition(beanName)) {
    mbd = getMergedLocalBeanDefinition(beanName);
  }
  //这个bean是否是'合成的'，即实现了postprocessor的，具体可看org.springframework.beans.factory.config.BeanPostProcessor，或者关注后面的相关文章
  boolean synthetic = (mbd != null && mbd.isSynthetic());
  //获取真正的Bean对象
  object = getObjectFromFactoryBean(factory, beanName, !synthetic);
}
return object;
}


protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
  //是单例且容器是否已经注册了这个个单例
  if (factory.isSingleton() && containsSingleton(beanName)) {
    //单例的同步锁
    synchronized (getSingletonMutex()) {
      Object object = this.factoryBeanObjectCache.get(beanName);
    if (object == null) {
        //单例的实例还不存在，调用方法获取
      object = doGetObjectFromFactoryBean(factory, beanName);
        // 这里考虑到了bean有可能做postProcess，所以再次从容器获取bean
        Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
        if (alreadyThere != null) {
          object = alreadyThere;
        }
        else {
          if (object != null && shouldPostProcess) {
            try {
              //做postProcess处理
              object = postProcessObjectFromFactoryBean(object, beanName);
            }
            catch (Throwable ex) {
              throw new BeanCreationException(beanName,
                  "Post-processing of FactoryBean's singleton object failed", ex);
            }
          }
          this.factoryBeanObjectCache.put(beanName, (object != null ? object : NULL_OBJECT));
        }
    }
    //空对象
    return (object != NULL_OBJECT ? object : null);
  }
  }
    else {
      //获取真正的bean
      Object object = doGetObjectFromFactoryBean(factory, beanName);
      if (object != null && shouldPostProcess) {
        try {
          //postPrecess处理
          object = postProcessObjectFromFactoryBean(object, beanName);
        }
        catch (Throwable ex) {
          throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);
        }
      }
      return object;
    }
  }


private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
      throws BeanCreationException {

    Object object;
    try {
      //校验当前系统是否建立了安全管理器
      if (System.getSecurityManager() != null) {
        //基于封装的上下文做出系统资源访问
        AccessControlContext acc = getAccessControlContext();
        try {
          object = AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
            @Override
            public Object run() throws Exception {
                return factory.getObject();
              }
            }, acc);
        }
        catch (PrivilegedActionException pae) {
          throw pae.getException();
        }
      }
      else {
        //没有建立系统安全管理器，则直接调用getObject方法
        object = factory.getObject();
      }
    }
    catch (FactoryBeanNotInitializedException ex) {
      throw new BeanCurrentlyInCreationException(beanName, ex.toString());
    }
    catch (Throwable ex) {
      throw new BeanCreationException(beanName, "FactoryBean threw exception on object creation", ex);
    }

    // Do not accept a null value for a FactoryBean that's not fully
    // initialized yet: Many FactoryBeans just return null then.
    if (object == null && isSingletonCurrentlyInCreation(beanName)) {
      throw new BeanCurrentlyInCreationException(
          beanName, "FactoryBean which is currently in creation returned null from getObject");
    }
    return object;
  }
```

上面的代码所有相关的重要的部分，我均做了注释，这下流程已经很清晰了，那么回到问题上来，在使用genericDAO这个bean的时候，因为依赖到了SqlSessionFactory这个类，所以基于成员变量的名字，又调用了getBean()方法，在这里因为这个bean已经通过配置注入了bean容器，因此直接获取到相关实例，然后进行getObjectForBeanInstance（）方法的处理，后面自然就获取到了实现了FacotryBean时的泛型参数SqlSessionFactory的真正类型实例。

讲到这了，那么我们来看看SqlSessionFactoryBean的getObject()方法是怎么实现的：

```java

@Override
public SqlSessionFactory getObject() throws Exception {
  if (this.sqlSessionFactory == null) {
    afterPropertiesSet();
  }
  return this.sqlSessionFactory;
}

@Override
public void afterPropertiesSet() throws Exception {
  notNull(dataSource, "Property 'dataSource' is required");
  notNull(sqlSessionFactoryBuilder, "Property 'sqlSessionFactoryBuilder' is required");
  state((configuration == null && configLocation == null) || !(configuration != null && configLocation != null),
            "Property 'configuration' and 'configLocation' can not specified with together");

  this.sqlSessionFactory = buildSqlSessionFactory();
}

```
　　很明显，SqlSessionFactoryBean 定义了一个sqlSessionFacotry的类成员变量，调用了afterPropertiedSet()(熟悉spring的同学一定会觉得这个方法很熟，后面的文章会讲相关技术)方法来构建SQLSessionFacotry实例，构建的具体实现代码这里我就不贴了，有兴趣的可以去看看，大概就是先构建configuration这个类成员变量，做了各种判断，最后用这个变量作为SqlSessionFacotry的构造函数入餐构建一个DefaultSqlSessionFactory类实例。

***说一句题外话，在遇到问题的时候，你最先想到的不应该是寻找别人帮助回答问题，而应该尝试自己的探索一下问题的根源，这样探索的过程的可能会遇到新的问题，继续尝试去解决这个问题，这样一来，在解决一个问题的过程中，你其实已经解决了很多相关依赖性的问题或知识。***
