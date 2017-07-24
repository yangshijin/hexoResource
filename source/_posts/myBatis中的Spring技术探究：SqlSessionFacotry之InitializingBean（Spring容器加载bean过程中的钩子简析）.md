---
title: myBatis中的Spring技术探究：SqlSessionFacotry之InitializingBean（Spring容器加载bean过程中的钩子简析）
date: 2017-03-30 21:12:36
tags:
 - java
 - spring
 - myBatis
---

这个系列的上一篇文章讲了SqlSessionFacotry的FacotryBean工厂类，并留下了已给伏笔，在SqlSessionFacotryBean的getObject()方法中调用了afterPropertiesSet()方法，这个方法其实也是Spring技术的一个钩子方法，也就是本篇文章的主体类：InitializingBean接口定义的钩子（回调）方法，先来看看类定义代码及注释：

<!-- more -->
```java
/**
 * Interface to be implemented by beans that need to react once all their
 * properties have been set by a BeanFactory: for example, to perform custom
 * initialization, or merely to check that all mandatory properties have been set.
 *
 * <p>An alternative to implementing InitializingBean is specifying a custom
 * init-method, for example in an XML bean definition.
 * For a list of all bean lifecycle methods, see the BeanFactory javadocs.
 *
 * @author Rod Johnson
 * @see BeanNameAware
 * @see BeanFactoryAware
 * @see BeanFactory
 * @see org.springframework.beans.factory.support.RootBeanDefinition#getInitMethodName
 * @see org.springframework.context.ApplicationContextAware
 */
public interface InitializingBean {

 /**
 * Invoked by a BeanFactory after it has set all bean properties supplied
 * (and satisfied BeanFactoryAware and ApplicationContextAware).
 * <p>This method allows the bean instance to perform initialization only
 * possible when all bean properties have been set and to throw an
 * exception in the event of misconfiguration.
 * @throws Exception in the event of misconfiguration (such
 * as failure to set an essential property) or if initialization fails.
 */
 void afterPropertiesSet() throws Exception;

}

```
　　从注释文档可以得知，实现这个接口的类一般目的是为了在Spring容器加载bean并将所有属性(其实就是bean的成员变量)都设置好之后，回调这个方法来做一个check，当然，方法的注释也说了，当检查发现错误配置的时候，这个方法也可以用来初始化正确的配置属性。

　　另外，从注释中野了解到了相关的一些类，包括BeanFactoryAware、ApplicationContextAware、BeanNameAware系列接口，以及上一次讲获取bean的时候用来处理FacotryBean这个工厂类的主角：BeanFactory，这一次同样的，从InitializingBean的注释文档，很容易猜出，这是在加载Bean的时候会做的回调操作，那么直接看BeanFactory的默认实现有没有类似加载、创建Bean的方法，首先查看了一下AbstractBeanFactory这个抽象类，发现有一个抽象方法是CreateBean()方法，没有具体实现，由于我使用的是强大的IntelliJ Idea,所以直接搜索这个方法的具体实现有哪些，发现AbstractAutowireCapableBeanFactory这个类继承并实现了它，虽然这个类也是一个抽象类，但是这里已经实现了createBean()方法，因此我们直接看它的代码就好：

```java
/**
 * Central method of this class: creates a bean instance,
 * populates the bean instance, applies post-processors, etc.
 * @see #doCreateBean
 */
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
 if (logger.isDebugEnabled()) {
 logger.debug("Creating instance of bean '" + beanName + "'");
 }
 RootBeanDefinition mbdToUse = mbd;

 // 根据bean名称处理一下bean的Class,此处不细讲
 Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
 if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
 mbdToUse = new RootBeanDefinition(mbd);
 mbdToUse.setBeanClass(resolvedClass);
 }

 // 校验和准备bean重载的方法，不细讲，有兴趣可以看看，虽然没什么好看的。。
 try {
 mbdToUse.prepareMethodOverrides();
 }
 catch (BeanDefinitionValidationException ex) {
 throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
 beanName, "Validation of method overrides failed", ex);
 }

 try {
 // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
 Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
 if (bean != null) {
 return bean;
 }
 }
 catch (Throwable ex) {
 throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
 "BeanPostProcessor before instantiation of bean failed", ex);
 }
 //真正创建bean的方法
 Object beanInstance = doCreateBean(beanName, mbdToUse, args);
 if (logger.isDebugEnabled()) {
 logger.debug("Finished creating instance of bean '" + beanName + "'");
 }
 return beanInstance;
}

protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args) {
 BeanWrapper instanceWrapper = null;
 //如果是单例，先从实例缓存获取bean，若不存在，实例化一个bean
 if (mbd.isSingleton()) {
 instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
 }
 if (instanceWrapper == null) {
 instanceWrapper = createBeanInstance(beanName, mbd, args);
 }
 final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
 Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);

 //允许实现了MergedBeanDefinitionPostProcessor接口的bean去指定bean的定义，这里也就会调用接口的postProcessMergedBeanDefinition()钩子方法
 synchronized (mbd.postProcessingLock) {
 if (!mbd.postProcessed) {
 applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
 mbd.postProcessed = true;
 }
 }

 //如果是单例，为单例bean增加一个单例工厂来构建指定的bean
 boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
 isSingletonCurrentlyInCreation(beanName));
 if (earlySingletonExposure) {
 if (logger.isDebugEnabled()) {
 logger.debug("Eagerly caching bean '" + beanName +
  "' to allow for resolving potential circular references");
 }
 addSingletonFactory(beanName, new ObjectFactory<Object>() {
 @Override
 public Object getObject() throws BeansException {
 return getEarlyBeanReference(beanName, mbd, bean);
 }
 });
 }

 // 实例化bean
 Object exposedObject = bean;
 try {
 //填充bean
 populateBean(beanName, mbd, instanceWrapper);
 if (exposedObject != null) {
 //真正实例化bean，我们要关注的重点方法！
 exposedObject = initializeBean(beanName, exposedObject, mbd);
 }
 }
 catch (Throwable ex) {
 if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
 throw (BeanCreationException) ex;
 }
 else {
 throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
 }
 }

 //如果是earlySingletonExposure为真，做相关的bean处理及依赖校验处理
 if (earlySingletonExposure) {
 Object earlySingletonReference = getSingleton(beanName, false);
 if (earlySingletonReference != null) {
 if (exposedObject == bean) {
 exposedObject = earlySingletonReference;
 }
 else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
 String[] dependentBeans = getDependentBeans(beanName);
 Set<String> actualDependentBeans = new LinkedHashSet<String>(dependentBeans.length);
 for (String dependentBean : dependentBeans) {
  if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
  actualDependentBeans.add(dependentBean);
  }
 }
 if (!actualDependentBeans.isEmpty()) {
  throw new BeanCurrentlyInCreationException(beanName,
  "Bean with name '" + beanName + "' has been injected into other beans [" +
  StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
  "] in its raw version as part of a circular reference, but has eventually been " +
  "wrapped. This means that said other beans do not use the final version of the " +
  "bean. This is often the result of over-eager type matching - consider using " +
  "'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
 }
 }
 }
 }

 // Register bean as disposable.
 try {
 registerDisposableBeanIfNecessary(beanName, bean, mbd);
 }
 catch (BeanDefinitionValidationException ex) {
 throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
 }

 return exposedObject;
 }

 /**
 * Initialize the given bean instance, applying factory callbacks
 * as well as init methods and bean post processors.
 * <p>Called from {@link #createBean} for traditionally defined beans,
 * and from {@link #initializeBean} for existing bean instances.
 * @param beanName the bean name in the factory (for debugging purposes)
 * @param bean the new bean instance we may need to initialize
 * @param mbd the bean definition that the bean was created with
 * (can also be {@code null}, if given an existing bean instance)
 * @return the initialized bean instance (potentially wrapped)
 * @see BeanNameAware
 * @see BeanClassLoaderAware
 * @see BeanFactoryAware
 * @see #applyBeanPostProcessorsBeforeInitialization
 * @see #invokeInitMethods
 * @see #applyBeanPostProcessorsAfterInitialization
 */
 protected Object initializeBean(final String beanName, final Object bean, RootBeanDefinition mbd) {

 //1.调用XXXAware接口系列钩子方法，后面会讲
 if (System.getSecurityManager() != null) {
 AccessController.doPrivileged(new PrivilegedAction<Object>() {
 @Override
 public Object run() {
 invokeAwareMethods(beanName, bean);
 return null;
 }
 }, getAccessControlContext());
 }
 else {
 invokeAwareMethods(beanName, bean);
 }

 //2、调用PostProcessors系列接口的beforePost方法，后面会讲
 Object wrappedBean = bean;
 if (mbd == null || !mbd.isSynthetic()) {
 wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 }

 try {
 //3.调用初始化方法，InitializingBean的钩子方法就在此处调用
 invokeInitMethods(beanName, wrappedBean, mbd);
 }
 catch (Throwable ex) {
 throw new BeanCreationException(
 (mbd != null ? mbd.getResourceDescription() : null),
 beanName, "Invocation of init method failed", ex);
 }

 //4.调用PostProcessors系列接口的afterPost方法，
 if (mbd == null || !mbd.isSynthetic()) {
 wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
 }
 return wrappedBean;
 }

```
　　通过阅读代码，忽略一些无关紧要的校验、日志、扩展配置处理的代码之后，发现真正在bean的生命周期中调用的钩子方法都在initializeBean()方法中，有4处调用钩子的方法，接下来我们一个一个来讲。

第一个是invokeAwareMethods()方法，这个方法会调用bean所实现的所有实现了Aware接口的接口，先看看代码：

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
 if (bean instanceof Aware) {
  if (bean instanceof BeanNameAware) {
   (BeanNameAware) bean).setBeanName(beanName);
 }
  if (bean instanceof BeanClassLoaderAware) {
   ((BeanClassLoaderAware) bean).setBeanClassLoader(getBeanClassLoader());
  }
  if (bean instanceof BeanFactoryAware) {
   ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
  }
 }
}
```
在抽象类型默认实现了3中Aware系列接口的钩子回调，分别是BeanNameAware、BeanClassLoaderAware、BeanFacotryAware，直接查看Aware接口的注释文档，可以知道，所有实现了Aware的接口需要定义一个返回void，且只有一个入参的方法，这个入参可以在容器创建bean定义的时候以钩子调用形式传入，在方法内使用。比如上述默认实现的代码中，三个接口分别要求容器传入bean名称、类的加载器、bean工厂，一般bean实现之后，直接引用给类成员变量，供类的其他方法使用。

第二个方法是applyBeanPostProcessorsBeforeInitialization()方法，直接看实现：

```java
@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
    throws BeansException {

  Object result = existingBean;
  for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
    result = beanProcessor.postProcessBeforeInitialization(result, beanName);
    if (result == null) {
      return result;
    }
  }
  return result;
}

```
　　这里直接获取了所有的BeanPostProcessors，然后调用postProcessBeforeInitialization()方法，那么BeanPostProcessor类是做什么的呢？来看看定义：

```java
/**
 * Factory hook that allows for custom modification of new bean instances,
 * e.g. checking for marker interfaces or wrapping them with proxies.
 *
 * <p>ApplicationContexts can autodetect BeanPostProcessor beans in their
 * bean definitions and apply them to any beans subsequently created.
 * Plain bean factories allow for programmatic registration of post-processors,
 * applying to all beans created through this factory.
 *
 * <p>Typically, post-processors that populate beans via marker interfaces
 * or the like will implement {@link #postProcessBeforeInitialization},
 * while post-processors that wrap beans with proxies will normally
 * implement {@link #postProcessAfterInitialization}.
 *
 * @author Juergen Hoeller
 * @since 10.10.2003
 * @see InstantiationAwareBeanPostProcessor
 * @see DestructionAwareBeanPostProcessor
 * @see ConfigurableBeanFactory#addBeanPostProcessor
 * @see BeanFactoryPostProcessor
 */
public interface BeanPostProcessor {

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 */
	Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;

	/**
	 * Apply this BeanPostProcessor to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other BeanPostProcessor callbacks.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 */
	Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;

}

```

注释文档很长，也很详细，简而言之呢，这是spring留给使用者的钩子接口，这个接口有两个钩子，一个是在bean初始化之前调用，也就是第三步invokeInitMethods()方法调用之前调用，另一个钩子则是在第三步骤调用之后调用，也就是后面要讲的第四步骤applyBeanPostProcessorsAfterInitialization()方法。这个钩子的用途很多很多，各种各种的定制化的东西都可以利用这个钩子实现，比如AOP(面向切面编程)中的切面就可以用这个钩子来实现，又或者你对某个bean使用了注解，那么这个钩子也可以当作注解解释器来使用呢，总之很有用就对了！

接下来看看invokeInitMethods()方法的实现，也就是今天InitializingBean的afterPropertiesSet()方法的调用处，当然，肯定不是这么简单：

```java
/**
	 * Give a bean a chance to react now all its properties are set,
	 * and a chance to know about its owning bean factory (this object).
	 * This means checking whether the bean implements InitializingBean or defines
	 * a custom init method, and invoking the necessary callback(s) if it does.
	 * @param beanName the bean name in the factory (for debugging purposes)
	 * @param bean the new bean instance we may need to initialize
	 * @param mbd the merged bean definition that the bean was created with
	 * (can also be {@code null}, if given an existing bean instance)
	 * @throws Throwable if thrown by init methods or by the invocation process
	 * @see #invokeCustomInitMethod
	 */
	protected void invokeInitMethods(String beanName, final Object bean, RootBeanDefinition mbd)
  	throws Throwable {

  //首先判断是否是实现了InitializingBean接口
  boolean isInitializingBean = (bean instanceof InitializingBean);
  //如果是，日志记录并调用InitializingBean接口的钩子方法afterPropertiesSet()
  if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
  	if (logger.isDebugEnabled()) {
    logger.debug("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
  	}
  	if (System.getSecurityManager() != null) {
    try {
    	AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
      @Override
      public Object run() throws Exception {
      	((InitializingBean) bean).afterPropertiesSet();
      	return null;
      }
    	}, getAccessControlContext());
    }
    catch (PrivilegedActionException pae) {
    	throw pae.getException();
    }
  	}
  	else {
    ((InitializingBean) bean).afterPropertiesSet();
  	}
  }
  //如果bean定义不为空
  if (mbd != null) {
    //获取定义的初始化方法，并确保初始化方法名不是afterPropertiesSet之后，调用初始化方法
  	String initMethodName = mbd.getInitMethodName();
  	if (initMethodName != null && !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
    	!mbd.isExternallyManagedInitMethod(initMethodName)) {
        //这里具体实现不细讲，大概就是根据bean定义的构造方法和初始化方法是否是public，然后通过不同处理进行反射调用
    invokeCustomInitMethod(beanName, bean, mbd);
  	}
  }
	}

```

　　上面代码我已经加了相关注释，大致就是如果bean实现了INitializingBean接口，会先调用其钩子方法，然后判断bean定义中是否还有init方法，如果有即调用，这里多说一句，init方法是通过bean配置的时候，有一个配置项叫init-method 来配置的。

　　最后来看看第四步骤applyBeanPostProcessorsAfterInitialization()方法：

```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

  Object result = existingBean;
  for (BeanPostProcessor beanProcessor : getBeanPostProcessors()) {
    result = beanProcessor.postProcessAfterInitialization(result, beanName);
    if (result == null) {
      return result;
    }
  }
  return result;
}

```

和第二步骤的applyBeanPostProcessorsBeforeInitialization方法如出一辙，都是获取了所有的BeanPostProcessor，然后调用其postProcessAfterInitialization()钩子方法。没什么好讲的。
