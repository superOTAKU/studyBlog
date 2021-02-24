# Spring如何解决循环引用

## 场景解析
A -> B <br/>
B -> A

```java
public class A {
  @AutoWired
  private B b;
}

public class B {
  @AutoWired
  private A a;
}
```

## getBean流程
1. 首先调用构造器new A
2. 发现A依赖B，去缓存查找B，B不存在，于是new B
3. 发现B依赖A，此时从缓存查找A，找到半成品的A，注入A，完成B的初始化
4. A注入属性B，完成A的创建

### 方法调用链

<pre>
getBean(A.class) //获取A
  getBean(B.class) //获取A依赖的B
    getBean(A.class) //获取B依赖的A
</pre>

## 源码
1. 调用*BeanFactory.getBean*，触发Bean创建流程
<pre>
  BeanFactory.getBean
    -- AbstractBeanFactory.doCreateBean
      -- DefaultSingletonBeanRegistry.getSingleton
</pre>
2. 多级缓存解决循环依赖问题
*DefaultSingletonBeanRegistry.getSingleton*
```java
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		// Quick check for existing instance without full singleton lock
    //查找1级缓存
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) { //是否正在创建？一个ConcurrentHashMap做标记
      //查找2级缓存，earlySingletonObject指未完成初始化的对象，如依赖的属性未完成注入
			singletonObject = this.earlySingletonObjects.get(beanName);
			if (singletonObject == null && allowEarlyReference) {
        //这把锁同步缓存的更新 singletonObjects -> earlySingletonObjects -> singletonFactories
        //加锁后再次查询，因为其他线程可能获取到锁，并完成了更新
				synchronized (this.singletonObjects) {
					// Consistent creation of early reference within full singleton lock
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						singletonObject = this.earlySingletonObjects.get(beanName);
						if (singletonObject == null) {
              //关键在这里，最终从3级缓存拿到ObjectFactory，获取到early reference,解决了循环依赖问题
              //ObjectFactory从哪里来？下面会讲到
							ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
							if (singletonFactory != null) {
								singletonObject = singletonFactory.getObject();
								this.earlySingletonObjects.put(beanName, singletonObject);
								this.singletonFactories.remove(beanName);
							}
						}
					}
				}
			}
		}
		return singletonObject;
	}
```
3. ObjectFactory设置
<pre>
AbstractAutowireCapableBeanFactory.doCreateBean
  -- DefaultSingletonBeanRegistry.addSingletonFactory
</pre>

*AbstractAutowireCapableBeanFactory.doCreateBean*代码片段
```java
		// Eagerly cache singletons to be able to resolve circular references
		// even when triggered by lifecycle interfaces like BeanFactoryAware.
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
      //创建当前Bean实例，先设置ObjectFactory，后续可能用来解决循环引用问题
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}
```
*DefaultSingletonBeanRegistry.addSingletonFactory*代码片段
```java
	protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
			if (!this.singletonObjects.containsKey(beanName)) {
				this.singletonFactories.put(beanName, singletonFactory);
				this.earlySingletonObjects.remove(beanName);
				this.registeredSingletons.add(beanName);
			}
		}
	}
```

## 总结
通过多级的缓存，解决该问题，主要思想是缓存“正在构造中的对象”。
