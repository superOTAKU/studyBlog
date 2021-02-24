# Spring如何解决循环引用

## 场景解析
A -> B
B -> A

## getBean流程
1. 首先调用构造器new A
2. 发现A依赖B，去缓存查找B，B不存在，于是new B
3. 发现B依赖A，此时从缓存查找A，找到半成品的A，注入A，完成B的初始化
4. A注入B
### 方法调用链

<pre>
getBean(A.class) //获取A
  getBean(B.class) //获取A依赖的B
    getBean(A.class) //获取B依赖的A
</pre>

## 总结
通过多级的缓存，解决该问题，主要思想是缓存“正在构造中的对象”。
网上分析较多，不再赘述，等完成源码的分析后再补充。
