---
layout:     post
title:      RocksDB&& LevelDB
subtitle:   springboot+RocksDB与LevelDB
date:       2018-12-11
author:     Cyber
header-img: img/post-bg-os-metro.jpg
catalog: true
tags:
    - java
    - DB
    - 数据库
    - springBoot
---

>  偶尔做一些小项目，根本没必要用什么mysql之类的，然后，看了看这两种轻型的DB..........



### RockDB

######  1.简介

 RocksDB是FaceBook起初作为实验性质开发的一个高效数据库软件，旨在充分实现快存上存储数据的服务能力。**RocksDB是一个c++库，可以用来存储keys和values，且keys和values可以是任意的字节流，支持原子的读和写**。除此外，RocksDB深度支持各种配置，可以在不同的生产环境（纯内存、Flash、hard disks or HDFS）中调优，支持不同的数据压缩算法、和生产环境debug的完善工具。 RocksDB的主要设计点是在快存和高服务压力下性能表现优越，所以该db需要充分挖掘Flash和RAM的读写速率。RocksDB需要支持高效的point lookup和range scan操作，需要支持配置各种参数在高压力的随机读、随机写或者二者流量都很大时性能调优。

###### 2.High Level Architecture

***RocksDB是一个嵌入式的K-V（任意字节流）存储***。所有的数据在引擎中是有序存储，可以支持Get(key)、Put（Key）、Delete（Key）和NewIterator()。RocksDB的基本组成是memtable、sstfile和logfile。memtable是一种内存数据结构，写请求会先将数据写到memtable中，然后可选地写入logfile。logfile是一个顺序写的文件。当内存表溢出的时候，数据会flush到sstfile中，然后这个memtable对应的logfile也会安全地被删除。sstfile中的数据也是有序存储以方便查找。

###### 3.Features

***Column Families***。RocksDB支持将一个数据库实例分片为多个列族。每个DB新建时默认带一个名为"default"的列族，如果一个操作没有携带列族信息，则默认使用这个列族。如果WAL开启，当实例crash再恢复时，RocksDB可以保证用户一个一致性的视图。通过WriteBatch API，可以实现跨列族操作的原子性。

***Updates***。Put 接口可以把一对k-v数据写入DB，如果k已经存在的话，则已有的v会被新的v覆盖。Write接口可以实现将多个k-v对写入DB，RockdDB可以保证要么所有的k-v对都写入DB，要么一个都不写入。同理，不管哪个k在DB中已经存在，旧值都会被覆盖。

***Gets、Iterators、Snapshots***。RocksDB中的key和value完全是byte stream，key和value的大小没有任何限制。Get接口提供用户一种从DB中查询key对应value的方法，MultiGet提供批量查询功能。DB中的所有数据都是按照key有序存储，其中key的compare方法可以用户自定义。Iterator方法提供用户RangeScan功能，首先seek到一个特定的key，然后从这个点开始遍历。Iterator也可以实现RangeScan的逆序遍历，当执行Iterator时，用户看到的是一个时间点的一致性视图。Snapshot接口可以创建数据库在某一个时间点的快照。Get和Iterator接口也可以执行在某一个Snapshot上。某种意义上，Iterator和Snapshot提供了DB在某个时间点的一个一致性视图，但是其实现原理却不一样。快速短期/前台的scan操作比较适合用Iterator，长期/后台操作适合用Snapshot。当使用Iterator时，会对数据库相应时间点的所有底层文件增加引用计数，直到Iterator结束或者释放了引用计数后，这些文件才允许被删除。Snapshot不关注数据文件是否被删除的问题，Compation进程会感知Snapshot的存在，会保证对应视图的数据不会被删除。当实例重启时，Snapshot会丢失，这是因为RocksDB不会持久化Snapshot相关数据。

***Transations***。RocksDB提供了多个操作的事务性，支持悲观和乐观模式。

来源：[CSDN](https://blog.csdn.net/xuelovexiao/article/details/81046595)

### LevelDB

###### 1.简介

Leveldb是一个google实现的非常高效的***kv数据库***，目前的版本1.2能够支持billion级别的数据量了。 在这个数量级别下还有着非常高的性能，主要归功于它的良好的设计。特别是LSM算法。LevelDB 是单进程的服务，性能非常之高，在一台4核Q6600的CPU机器上，每秒钟写数据超过40w，而随机读的性能每秒钟超过10w。

###### 2.Features

- key和value都是任意长度的字节数组；
- entry（即一条K-V记录）默认是按照key的字典顺序存储的，当然开发者也可以重载这个排序函数；
- 提供的基本操作接口：Put()、Delete()、Get()、Batch()；
- 支持批量操作以原子操作进行；
- 可以创建数据全景的snapshot(快照)，并允许在快照中查找数据；
- 可以通过前向（或后向）迭代器遍历数据（迭代器会隐含的创建一个snapshot）；
- 自动使用Snappy压缩数据；
- 可移植性；

###### 3.局限

- 一次只允许一个进程访问一个特定的数据库；
- 没有内置的C/S架构，但开发者可以使用LevelDB库自己封装一个server；



### springBoot+RL

>  依赖
>
> ```
> <!--rocksdb start-->
> <dependency>
>     <groupId>org.rocksdb</groupId>
>     <artifactId>rocksdbjni</artifactId>
>     <version>5.10.3</version>
> </dependency>
> <!--rocksdb end-->
> <!--leveldb start-->
> <dependency>
>     <groupId>org.iq80.leveldb</groupId>
>     <artifactId>leveldb</artifactId>
>     <version>0.10</version>
> </dependency>
> <dependency>
>     <groupId>org.iq80.leveldb</groupId>
>     <artifactId>leveldb-api</artifactId>
>     <version>0.10</version>
> </dependency>
> <!--leveldb end-->
> ```



>bean
>
>```
>@Configuration
>public class DbInitConfig {
>
>    @Bean
>    @ConditionalOnProperty("db.rocksDB")
>    public RocksDB rocksDB() {
>        RocksDB.loadLibrary();
>
>        Options options = new Options().setCreateIfMissing(true);
>        try {
>            return RocksDB.open(options, "./rocksDB");
>        } catch (RocksDBException e) {
>            e.printStackTrace();
>            return null;
>        }
>    }
>
>    @Bean
>    @ConditionalOnProperty("db.levelDB")
>    public DB levelDB() throws IOException {
>        org.iq80.leveldb.Options options = new org.iq80.leveldb.Options();
>        options.createIfMissing(true);
>        return Iq80DBFactory.factory.open(new File("./levelDB"), options);
>    }
>}
>```



>接口
>
>```
>public interface DbStore {
>    /**
>     * 数据库key value
>     *
>     * @param key
>     *         key
>     * @param value
>     *         value
>     */
>    void put(String key, String value);
>
>    /**
>     * get By Key
>     *
>     * @param key
>     *         key
>     * @return value
>     */
>    String get(String key);
>
>    /**
>     * remove by key
>     *
>     * @param key
>     *         key
>     */
>    void remove(String key);
>}
>```



>levelDB实现类
>
>```
>@Component
>@ConditionalOnProperty("db.levelDB")
>public class LevelDbStoreImpl implements DbStore {
>    @Resource
>    private DB db;
>
>    @Override
>    public void put(String key, String value) {
>        db.put(bytes(key), bytes(value));
>    }
>
>    @Override
>    public String get(String key) {
>        return asString(db.get(bytes(key)));
>    }
>
>    @Override
>    public void remove(String key) {
>        db.delete(bytes(key));
>    }
>}
>```



>rocksDB实现
>
>```
>@Component
>@ConditionalOnProperty("db.rocksDB")
>public class RocksDbStoreImpl implements DbStore {
>    @Resource
>    private RocksDB rocksDB;
>
>    @Override
>    public void put(String key, String value) {
>        try {
>            rocksDB.put(key.getBytes("utf-8"), value.getBytes("utf-8"));
>        } catch (RocksDBException | UnsupportedEncodingException e) {
>            e.printStackTrace();
>        }
>    }
>
>
>    @Override
>    public String get(String key) {
>        try {
>            byte[] bytes = rocksDB.get(key.getBytes("utf-8"));
>            if (bytes != null) {
>                return new String(bytes, "utf-8");
>            }
>            return null;
>        } catch (Exception e) {
>            e.printStackTrace();
>            return null;
>        }
>    }
>
>    @Override
>    public void remove(String key) {
>        try {
>            rocksDB.delete(rocksDB.get(key.getBytes("utf-8")));
>        } catch (RocksDBException | UnsupportedEncodingException e) {
>            e.printStackTrace();
>        }
>    }
>
>}
>```

代码其实挺简单，项目demo参见我的***[码云](https://gitee.com/junruPan/common-tools/tree/master/Lrocks-DbspringBoot)***





