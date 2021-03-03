# ConcurrentHashMap

源码方法论：
1. 看注释中的简介
2. 看关心的方法实现

## put
1. hashCode处理，spread方法将hashCode高位扩散到低位，以减少hash冲突（如果只有高位不同，很多都冲突，这种方案避免这种情况）
2. lazyInit hash table，使用sizeCtl这个字段做CAS，控制只有一个线程可以初始化，其他线程使用Thread.yield()进行自旋等待（学习）
3. 如果slot为null，CAS竞争设置值到slot，否则锁头节点head，再设置slot，如果冲突过多，tree化链表
4. 如果有必要，进行resize，resize时会将ForwardingNode设置到头节点，他的hash是-1，表明当前正在执行resize

## get
get比较简单，就是根据hash计算index，然后根据index，找到key相等的node，返回对应node的value
这里的可见性都是通过volatile来保证的...

## ConcurrentHashMap提高性能的手段
1. hashCode处理，将高位扩散到低位，减少hash冲突
2. lazy init table，CAS延迟初始化table
3. 如果头节点为null，CAS设置链表表头
4. 如果头节点不为null，synchronized锁住头节点，然后更新
5. 如果hash冲突过多，将链表改为tree，加速查找
