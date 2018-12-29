# TDDLTree
TDDL淘宝分布式数据库技术研究


###模型定位
![](https://i.imgur.com/uNL6Xal.png)

<pre>
分布式数据库的演化：

      TDDL是一个分布式数据库中间件，主要是为了解决分布式数据库产生的相关问题。

      1）单一数据库无法满足性能需求
         随着业务的发展，数据量急剧增大，使用单库单表时，数据库压力过大，因此通过分库分表，
      读写分离的方式，减轻数据库的压力。
      2）系统容灾
         使用单机数据库时，如果数据库宕机，则无法对外提供服务，因此产生了主备数据库的方案，
      当主库宕机后，会自动切换到备库，不影响对外的服务。
      3）运维管理
         直接连接单机数据库时，无法动态的切换数据源，使用TDDL中间件，可以动态的切换数据源。
</pre>

![](https://i.imgur.com/H8N5ZHm.png)

<pre>
     除了最底层的物理DB，上两层的逻辑层是分布式数据库中间件着重要解决的问题，tddl的设计亦
     是以图中的三层逻辑模型为基础进行的。为了解决每一层所要应对的问题，tddl在结构上至上而
     下也分了三层，分别是 Matrix 层、Group层以及 Atom 层。具体来说，从逻辑上来看，最顶层
     是要进行分库分表才过渡到中间层的，分库分表所带来的问题是在 Matrix 层解决 ，包括SQL
     解释、优化和执行等；中间层是经过读写分离和主备切换才会出现最底层，因此读写分离与主备
     切换的工作由 Group 层解决；至于 Atom 层，它面对的是实实在在的每一个数据库，更多的工
     作在与对数据库的连接管理，比如说当数据库的 IP 地址发生改变时，Atom 层要动态感知，以
     免连接找不到地址。
</pre>

![](https://i.imgur.com/wZS84sa.png)

![](https://i.imgur.com/eV6JWoX.png)

<pre>
Senquence全局唯一id标识生成原理

     方案一
          oracle sequence：基于第三方oracle的SEQ.NEXTVAL来获取一个ID
          优势：简单可用。
          缺点：需要依赖第三方oracle数据库。

     方案二
          mysql id区间隔离：不同分库设置不同的起始值和步长，比如2台mysql，就可以设置一
          台只生成奇数，另一台生成偶数. 或者1台用0~10亿，另一台用10~20亿.。

          优势：利用mysql自增id 。

          缺点：运维成本比较高，数据扩容时需要重新设置步长。

      方案三
          基于数据库更新＋内存分配：在数据库中维护一个ID，获取下一个ID时，会对数据库进
          行ID=ID+100 WHERE ID=XX，拿到100个ID后，在内存中进行分配 。

          优势：简单高效。

          缺点：无法保证自增顺序。
</pre>


