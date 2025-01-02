---
title: 'Redis进阶与实战(未完结)'
date: 2024-12-19T14:36:29+08:00
draft: false
layout: "post"
images: 
  - "/avatar.jpg"   # 设置封面图
description: ""     
categories: ["笔记"]
tags: ["后端","中间件"]
---
# Redis

## Session共享问题

Tomcat中自动维护着所有的session，保证每一次的请求，拿出来的session根据sessionId，都是该请求**客户端浏览器**对应的session。但面对，多个tomcat，也就是Tomcat集群的时候，session中存取数据，就会存在着，多个集群中无法共享session数据的情况。

session的有效期：30min。

### 使用Redis实现共享session登录

#### Hash数据结构与String结构

Hash：就像是一个map，可以单独的对每一个字段进行修改删除等操作，并且**内存占用更少**。

String：使用JSON字符串进行存储，信息更加直观。

**存储方式：**将一个随机的**Token**作为登录凭证（存储的**KEY**），存储用户数据。使用HASH存储。

**有效期设置:**需要模仿session30min的有效期，也就是热更新，会随时更新token（KEY）的有效期。

#### 存储时的序列化问题

因为最终使用的是`Hash`存储方式将一个对象转换为map存储。但这里使用的是`StringRedisTemplate`，并且并没有配置额外的字符串序列化的配置，那么当对象中存在属性不为`String`的时候，就会出现强转类型错误。

- 使用`RedisTemplate`，不只是支持`String`类型的`value`，和`Key`，但注意要配置一下序列化器，使得在数据库中存的数据展示，对用户友好。

  ```java
  /**
   * 对redis进行配置，序列化
   */
  @Configuration
  @Slf4j
  public class RedisConfiguration {
  
      @Bean
      public RedisTemplate redisTemplate(RedisConnectionFactory redisConnectionFactory) {
          log.info("开始创建redis模板对象");
          RedisTemplate redisTemplate = new RedisTemplate();
          redisTemplate.setConnectionFactory(redisConnectionFactory);
          // 为了在数据库中显示时正常，将其用string字符串进行转化
          redisTemplate.setKeySerializer(new StringRedisSerializer());
          return redisTemplate;
      }
  }
  
  ```

- 对要存储的map做手脚，当使用的是Hutool包中的`Beanutil.beanToMap`的时候，可以对字段进行自定义的转换，这样就可以将对象中不是String类型的属性，给在map中转换成String类型了。

```java
UserDTO userDTO = BeanUtil.copyProperties(user, UserDTO.class);
 Map<String, Object> userMap = BeanUtil.beanToMap(userDTO, new HashMap<>(),
                    CopyOptions.create()
                            .ignoreNullValue()
                            .setFieldValueEditor((key, value) -> value.toString()) // 转换字段的类型
                    );
```

### 登录刷新问题

因为当现在只有一个拦截器时，这一个拦截器的作用是**作登录校验的同时刷新token的TTL**，但是该拦截器（校验拦截器），只会对需要做登录校验的网页进行拦截，所以当访问不需要做登录校验的网页的时候，`token`就不会正确的刷新。

**再加一层刷新token拦截器**，校验拦截器，只需要判断从前面刷新token的拦截器中，`Threadloal`中是否存有用户信息，即可做到登录校验的目的。

## Web应用中的缓存

- 降低后端负载
- 提高读写效率，降低请求响应时间

- 数据一致性
- 代码维护成本
- 缓存（集群）部署以及维护运维成本

### 缓存更新策略

-  内存淘汰策略 （根据cache内部的内存淘汰机制）
-  超时
-  主动更新

#### 主动更新

- Cahe Aside Pattern （更新数据的同时，更新缓存）
- Read/Write Through Pattern（Write through 将缓存和数据库作为一个独立的服务进行维护）
- Write Behind Caching (Write Back 直接在缓存上进行操作)

#### 删除缓存 or 更新缓存

更新缓存存在着多个无效的读写操作，只有当要访问数据库的时候，缓存未命中，在整个过程中只需要更新一次缓存（**设置超时时间**作为兜底方案）。

#### 数据库和缓存的同时成功和同时失败	

- 单体项目，通过事务来保持数据的强一致性
- 分布式架构：只能通过分布式事务达到数据的强一致性

#### 先删除缓存 or 后删除缓存

出现数据不一致的核心：操作数据库的时间远大于操作缓存的时间。

### 缓存穿透

客户端请求的数据再缓存中和数据库中都不存在，这样的缓存永远都不会生效，这些请求都会返回到数据库。数据库压力 up

- 缓存空对象
- 布隆过滤器
- 增加id的复杂度，避免id太多规律
- 对id数据的基础格式的校验
- 加强参数的限流
- 对数据格式做强的基础校验

#### 缓存空对象

- 内存占用高（设置TTL解决空对象KEY越来越多的问题）
- 存入的是空字符串

#### 布隆过滤器

使用二进制位数存储对象，Redis中提供BitMap，达到布隆过滤器的功能。

- 实现复杂、存在误判的可能性

### 缓存雪崩

在同一时间段**大量的缓存key都同时的失效**，或者**Redis直接服务宕机**，服务器数据库压力剧增

- 给不同的Key的TTL添加随机值
- 利用Redis的**集群**提高服务的可用性（Redis哨兵机制）
- 给缓存业务，添加降级限流，策略。（提前做好容错处理）
- 给业务添加多级的缓存（在nginx方添加缓存）

### 缓存击穿（热点Key问题）

**一个**被**高并发**访问的，并且缓存重建业务复杂（获取该key的value的业务逻辑复杂，耗时久）的key突然间失效了，导致数据库收到了高并发的请求，压力陡增。

#### 互斥锁（一致性）

循环中不断休眠等待获取锁，获取所锁之后，就会得到更新后的数据。

- 没有额外的内存消耗
- 保证了数据的一致性
- 性能降低，大多时间都花在了等待获取锁的时间上
- 存在死锁的隐患

#### 逻辑过期（可用性）

在value中添加一个新的字段，`expire`字段，若当前时间大于该字段，则判定该数据就已经过期了。

当发现了过期的数据之后：

- 若成功获取到了锁，那么就新开一个线程，该线程负责更新缓存数据的一系列步骤，**并且更新数据的过期字段**。之后**主线程直接返回获取到的过期的数据**。
- 若没有获取到锁，那么就说明已经有线程正在重建数据，**直接返回获取到的过期的数据**。
- 不保证一致性
- 有额外的内存消耗

#### 基于互斥锁来解决缓存击穿

利用`Redis-CLI`中的`setnx`命令

> SETNX： Only set the key when the key is not exist which means when the key exists SETNX will failed

利用该命令的特性来模拟互斥锁的效果。

- `finally`代码块的使用]
- 获取到锁之后的**DoubleCheck**

#### 基于逻辑过期的方式解决缓存击穿

**热点数据**，都会在实际使用之前进行**缓存预热**（先把数据放入到redis中）并且设置逻辑过期时间。

```java
    /**
     * @param id
     * @param expireSeconds
     * 预热缓存，将热点key添加到缓存中
     */
    public void saveShop2RedisData(Long id, Long expireSeconds) {
        Shop shop = getById(id);
        RedisData<Shop> shopRedisData = new RedisData<>();
        shopRedisData.setData(shop);
        // 设置预过期时间
        shopRedisData.setExpire(LocalDateTime.now().plusSeconds(expireSeconds));
        // 这里就不设置过期时间TTL了，依靠手动逻辑过期方式  进行管理
        stringRedisTemplate.opsForValue().set(CACHE_SHOP_KEY + id, JSONUtil.toJsonStr(shopRedisData));
    }
```

新建一个DTO redis data 对象用于保存过期时间，以及data对象

```java
    public Shop getShopByIdUseTTL(Long id) {
        String key = CACHE_SHOP_KEY + id;
        String strJson = stringRedisTemplate.opsForValue().get(key);

        // 不存在那么就直接返回,因为已经进行了缓存的预热了
        if (strJson == null || strJson.isEmpty()) {
            return null;
        }

        // JsonUtil 底部使用的是字节byte进行强转
        // 并不知道data的数据类型是什么，所以最后返回的还是JSONObject
        RedisData redisData = JSONUtil.toBean(strJson, RedisData.class);
        Shop shop = JSONUtil.toBean((JSONObject) redisData.getData(), Shop.class); 
        if (LocalDateTime.now().isBefore(expire)) {
            // 代表没有过期呢
            return shop;
        }

        // 过期，重建缓存
        String lockKey = CACHE_SHOP_KEY + id;
        if (!getTheLock(lockKey)) {
            // 没有获取到锁
            // 直接返回旧数据
            return shop;
        }
        // DOUBLE CHECK
        if (LocalDateTime.now().isBefore(expire)) {
            // 代表没有过期呢
            return shop;
        } 
         // 开启一个独立的线程，实现缓存的重建
        CACHE_REBUILD_EXECUTOR.execute(() -> {
            try {
                saveShop2RedisData(id, 20L);
            } catch (Exception e) {
                throw new RuntimeException(e);
            } finally {
                // 最后要做的就是 释放锁
                releaseTheLock(lockKey);
            }
        }
```

## 电商秒杀问题

### 全局ID生成器（利用Redis的自增长实现）

在分布式系统下，生成全局唯一的ID的工具。

为保证ID的安全性，不使用Redis自增的数值，而是拼接一些其他的信息（64位）：

- 符号位（1）
- 时间戳（31），秒为单位
- 序列号（32)，在每一秒内的计数器，支持每秒产生2^32个不同的ID

- Redis increment 自增范围最大为 2 ^ 64
- 使用`KEY`的保存：每天使用一个Key方便统计订单量

利用`INCR`命令自增ID策略，任意的分布式系统都使用Reis生成ID

其他策略：

- UUID
- Redis自增
- snowflake算法
- 数据库的自增

### 乐观锁与悲观锁

- **悲观锁**：认为线程安全一定会发生，在操作数据之前先获取锁，确保线程串行执行`Synchornized` `Lock`(数据库中的锁)
  - 性能一般
  - 简单粗暴 

- **乐观锁**：认为线程安全不一定会发生，所以不加锁，是在**更新数据之前**去判断有没有其他线程做了修改。
  - 如果出现了被其他的线程修改了，这是已经出现了安全问题，可以重试或者等待
  - 性能好
  - 成功率低

#### 乐观锁

如何判断得到的数据，之前是否被修改过。

- 版本号法，添加新的字段`version`
  - 在查询之后，当要更新数据的时候，判断当前的`version`是否等于查询得到的`version`
- CAS(Compare and set), 使用`stock`库存值作为`version`的代替字段。
  - `set stock = stock - 1 where id = 10 and stock = 1`

问题: 成功率太低

- 不用强制性等于之前的值，也可以只是判断`set stock = stock - 1 where id = 10 and stock > 0`

> 对数据库的操作，都要考虑并发多线程问题

#### `sychronized` `toString()`加锁问题

当需要，每个用户限定操作数据的数量的时候，需要进入数据库判断当前用户已经访问（购买）次数，再决定是否对让该用户继续访问数据库。

而这里，使用悲观锁`sychronized`，将当前用户`UserID`转换为字符串作为锁对象，就引出了`tostring`的内部实现问题。

```java
sychronized(userId.toString().intern()) {
....
}
```

**因为tostring内部其实还是new了一个新的字符串对象，所以如果当前用户不同请求的时候，加锁的对象，也不会保持不变，此时就需要使用`intern()`方法**。

#####  intern方法

从**字符串常量池**中，寻找到和该字符串值一样的字符串常量，进行返回。

#### `sychronized` `Transactinal()`加锁问题

`Transactional`事务提交的时间，是在整个函数结束之后再进行提交的，所以当加锁的位置不同时，会还在事务没有提交的时候，进入新的进程，也会出现问题。

保证顺序 **获取锁 -> 开启事务 -> 事务提交 ->  释放锁**

#### 事务失效问题

当事务方法调用非事务方法时，此时事务就会失效。

> 因为spring中的事务管理是基于代理继续管理的，在底层使用该类的动态代理对象，对事务进行管理，它只能拦截public方法的调用。如果一个事务方法内部直接调用了同一个类中的非事务方法，这个调用不会经过Spring的代理，因此事务管理不会生效

同时，简单的在非事务方法上添加Transactional注解也是不行的，因为最终的方法调用还是不是代理对象进行调用的，总之以下的解决方案，都是围绕**使用代理对象调用目标方法，使得两个方法处在同一个事务上下文**

- 通过依赖注入自身的被spring管理的代理对象
- 使用`AppContext.getCurrentContext()`获得当前的代理对象
- 分离事务方法和非事务方法到不同的类中 

## Redis实现分布式锁

**分布式锁：**满足分布式系统集群模式下**多进程可见**的并且**互斥**的锁。

`sychronized`，每一个JVM（每一个进程）内部使用的是独立的单个的锁监视器，所以就不能在集群中使用该锁。

解决思路：所有的JVM（进程）都使用同一个锁监视器。

#### 实现方式

```bash
# 获取锁
SET lock thread1 NX EX 10
# 释放锁
DEL key 
# 超时过期时间释放
```

  **非阻塞方式**分布式锁，当没有拿到锁的时候，直接返回false。

存在的误删问题

- 前一线程业务阻塞的时间过长，锁超时释放
- 超时释放之后，后一线程获取到锁
- 前一线程业务完成，删除了后一线程的锁\

```java
  @Override
    public boolean tryLock(long timeoutSec) {
       	// 将当前的线程id存入到锁的value中，防止线程误删操作
        // 但是这里获取到的线程的id只是一个在JVM中递增的数字，也会进行重复
        long thread_id = Thread.currentThread().getId();
        Boolean result = stringRedisTemplate.opsForValue()
                .setIfAbsent(KEY_PREFIX + name, id + "", timeoutSec, TimeUnit.SECONDS);
        return result != null && result;
    }
```

改进解决线程误删问题，在锁中存储value。分为两部分：

- 使用UUID（区分不同的JVM）加上当前的线程ID（区分同进程中的多线程）作为vlaue存储为锁的value.

> UUID: (Universal Unique identify)
>
> - Uniques rely on random or pseudorandom generation, combined with the hardware and the software information
> - reprensted as a 32 hexadecimal characters, but HAVE TOTAL 36 characters because it's divied into 5 groups by the hyphens

- 在删除的时候进行判断当前的value是否匹配是自己的，如果不是，什么都不做，不删除锁

```java
  @Override
    public boolean unlock() {
        // 获取当前的value 进行判断是不是自己本身产生的ID
        String value = ID_PREFIX + Thread.currentThread().getId();
        String redis_id = stringRedisTemplate.opsForValue().get(KEY_PREFIX + name);
        if (redis_id == null) {return false;}
        if (redis_id.equals(value)) {
            // 判断成功
            Boolean delete = stringRedisTemplate.delete(KEY_PREFIX + name);
            return Boolean.TRUE.equals(delete);
        }
        return false;
    }
```

两个步骤的原子性问题

> 只要在两个操作中有间隔，就会出现线程安全并发问题，要保证一个操作的原子性。

### Lua脚本

Redis官方提供的Lua脚本功能，能够确保多条命令执行的原子性。使用Lua语言编写。

```bash
EVAL "return redis.call("set","name", "jack")" 0 
# 等同于执行`set name jack` 
# 0 代表的是脚本所需要的key类型的参数的个数
```

脚本中也可以添加参数，作为参数传递

```bash
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 name Rose
# 其中1说明，KEYS类型的参数有1个，也就是从参数中的第二个开始后面的都属于其他的参数
```

```lua
-- 锁的key
local key = "locl:orde:5"

-- 获取当前的线程的吧
```



















