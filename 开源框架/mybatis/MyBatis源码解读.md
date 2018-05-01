# MyBatis源码解读
## 1.dao层接口为什么不需要写实现类？
MyBatis通过JDK的动态代理方式，在启动加载配置文件时，根据配置Mapper的xml去生成Dao的实现。

MapperRegistry是Mapper接口及其对应的代理对象工厂的注册中心。Configuration是MyBatis全局性的配置对象，在MyBatis初始化的过程中，所有配置信息会被解析成相应的对象并记录到Configuration对象中，而Configuration.mapperRegistry字段，记录当前使用的MapperRegistry对象。

第一步：在Mybatis初始化过程中会读取映射配置文件以及Mapper接口中的注解信息，并调用MapperRegistry.addMapper()方法填充MapperRegistry.knowMappers集合，该集合的key是Mapper接口对应的Class对象，value为MapperProxyFactory工厂对象，可以为Mapper接口创建代理对象。

第二步：在需要执行某SQL语句时，会先调用MapperRegistry.getMapper()方法获取实现了Mapper接口的代理对象，session.getMapper(BlogMapper.class)方法得到的实际上是MyBatis通过JDK动态代理为BlogMapper接口生成的代理对象。

MapperMethod中封装了Mapper接口中对应方法的信息，以及对应SQL语句的信息。可以将MapperMethod看作连接Mapper接口以及映射配置文件中定义的SQL语句的桥梁。

## 2.Mapper映射配置文件有什么用？
每个映射配置文件的命名空间可以绑定一个Mapper接口，并注册到MapperRegistry中。

## 3.MyBatis的缓存机制
一般提到MyBatis缓存的时候，都是指二级缓存。一级缓存（也叫本地缓存）默认会启用，并且不能控制，因此很少会提到。

#### 一级缓存
MyBatis的一级缓存存在于SqlSession的生命周期中，在同一个SqlSession中查询时，MyBatis会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。如果同一个SqlSession中执行的方法和参数完全一致，那么通过算法会生成相同的键值，当Map缓存对象中已经存在该键值时，则会返回缓存中的对象。

该修改在原来方法的基础上增加了flushCache=“true”，这个属性配置为true后，会在查询数据前清空当前的一级缓存，因此该方法每次都会重新从数据库中查询数据，此时的user2和user1就会成为两个不同的实例，可以避免上面的问题。但是由于这个方法清空了一级缓存，会影响当前SqlSession中所有缓存的查询，因此在需要反复查询获取只读数据的情况下，会增加数据库的查询次数，所以要避免这么使用。

任何的INSERT、UPDATE、DELETE操作都会清空一级缓存。

#### 二级缓存
MyBatis的二级缓存非常强大，它不同于一级缓存只存在于SqlSession的生命周期中，而是可以理解为存在于SqlSessionFactory的生命周期中。

在MyBatis的全局配置settings中有一个参数cacheEnabled，这个参数是二级缓存的全局开关，默认值是true，初始状态为启用状态。如果把这个参数设置为false，即使有后面的二级缓存配置，也不会生效。由于这个参数值默认为true，所以不必配置。

MyBatis的二级缓存是和命名空间绑定的，即二级缓存需要配置在Mapper.xml映射文件中，或者配置在Mapper.java接口中。在映射文件中，命名空间就是XML根节点mapper的namespace属性。在Mapper接口中，命名空间就是接口的全限定名称。

在保证二级缓存的全局配置开启的情况下，开启二级缓存只需要再xml中添加\<cache/\>元素即可。

默认的二级缓存会有以下效果：
- 映射语句文件中的所有SELECT语句将会被缓存。
- 映射语句文件中的所有INSERT、UPDATE、DELETE语句会刷新缓存。
- 缓存会使用LRU算法来收回。
- 根据时间表（如no Flush Interval，没有刷新间隔），缓存不会以任何时间顺序来刷新。
- 缓存会存储集合或对象（无论查询方法返回什么类型的值）的1024个引用。
- 缓存会被视为read/write（可读/可写）的，意味着对象检索不是共享的，而且可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

所有的这些属性都可以通过缓存元素的属性来修改，示例如下：\<cache eviction=“FIFO” flushInterval=“60000” size=“512” readOnly=“true”/\>

#### 深入二级缓存
Cache可以配置的属性中，readOnly属性可以被设置为true或false。只读的缓存会给所有调用者返回缓存对象的相同实例，因此这些对象不能被修改，这提供了很重要的性能优势。可读写的缓存会通过序列化返回缓存对象的拷贝，这些方式会慢一些，但是安全，因此默认为false。

需要注意的是，由于配置的是可读写的缓存，而MyBatis使用SerializedCache序列化缓存来实现可读写缓存类，并通过序列化和反序列化来保证通过缓存获取数据时，得到的是一个新的实例。因此，如果配置为只读缓存，MyBatis就会使用Map来存储缓存值，这种情况下，从缓存中获取的对象就是同一个实例。

MyBatis默认提供的缓存实现是基于Map实现的内存缓存，已经可以满足基本的应用。但是当需要缓存大量的数据时，不能仅仅通过提高内存来使用mybatis的二级缓存，还可以选择一些类似EhCache的缓存框架或redis缓存数据库等工具来保存mybatis的二级缓存数据。

RedisCache在保存缓存数据和获取缓存数据时，使用了Java的序列化和反序列化，因此还需要保证被缓存的对象必须实现Serializable接口。

二级缓存虽然好处很多，但并不是什么时候都可以使用。以下场景适用：
- 以查询为主的应用中，只有尽可能少的增删改操作。
- 绝大多数以单表操作存在时，由于很少存在互相关联的情况，因此不会出现脏数据。
- 可以按业务划分对表进行分组时，如关联的表比较少，可以通过参照缓存进行配置。



