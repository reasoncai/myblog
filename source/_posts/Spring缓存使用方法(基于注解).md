---
title: Spring缓存使用方法(基于注解)
date: 2017-05-10 23:24:19
tags: ['spring','缓存','redis']
categories: spring
---

### 1.启用Spring对缓存的支持。
Spring对缓存的支持有两种方式：
- 注解驱动的缓存
- XML声明的缓存
#### 1.1.通过使用@EnableCaching启用注解驱动的缓存

```Java
package com.cai.springcache.config;

import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.concurrent.ConcurrentMapCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created by reason on 17/5/11.
 */
@Configuration
@EnableCaching  //启用缓存
public class CacheConfig {
    /**
     * 声明缓存管理器
     * @return
     */
    @Bean
    public CacheManager cacheManager(){
        return new ConcurrentMapCacheManager();
    }
}
```
#### 1.2.通过使用`<cache:annotation-driven/>`启用注解式缓存

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:cache="http://www.springframework.org/schema/cache"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/cache http://www.springframework.org/schema/cache/spring-cache.xsd">
    <!--启用注解式缓存-->
    <cache:annotation-driven/> 
    <!--声明缓存管理器-->
    <bean id = "cacheManager" class=
            "org.springframework.cache.concurrent.ConcurrentMapCacheManager"/>
</beans>
```
以上两种方式的工作原理是相同的，都是创建一个切面(aspect)并触发Spring缓存注解的切点(pointcut)。根据所使用的注解以及缓存的状态，切面会从缓存中获取数据，将数据添加到缓存中或者从缓存中移除某个值。例子中出了启用了注解驱动的缓存，还声明了一个缓存管理器(cache manager)的bean。缓存管理器是Spring缓存的抽象核心，它能够与多个流行的缓存实现(Ehcache,Redis等等)进行集成。例子中用了ConcurrentMapCacheManager这个简单的缓存管理器，底层其实就是用了ConcurrentHashMap作为存储，其存储是基于内存额，适用于开发，测试或简单的应用。
#### 1.3.配置缓存管理器
可用的缓存管理器：
- SimpleCacheManager
- NoOpCacheManager
- ConcurrentMapCacheManager
- CompositeCacheManager
- EhCacheCacheManager
- JCacheCacheManager
- RedisCacheManager
- GemfireCacheManager

**使用Ehcache缓存**

需要依赖ehcache
##### 1.3.1.以Java配置的方式设置EhCacheCacheManager

```Java
package com.cai.springcache.config;

import net.sf.ehcache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.ehcache.EhCacheCacheManager;
import org.springframework.cache.ehcache.EhCacheManagerFactoryBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.ClassPathResource;

/**
 * Created by reason on 17/5/11.
 */
@Configuration
@EnableCaching
public class EhCacheConfig {
    @Bean
    public EhCacheCacheManager cacheCacheManager(CacheManager cm){
        return new EhCacheCacheManager(cm);
    }
    @Bean
    public EhCacheManagerFactoryBean ehcache(){
        EhCacheManagerFactoryBean ehCacheManagerFactoryBean = new EhCacheManagerFactoryBean();
        ehCacheManagerFactoryBean.setConfigLocation(new ClassPathResource("ehcache.xml"));
        return ehCacheManagerFactoryBean;
    }
}

```
说明：EhCache的CacheManager要被注入到Spring的EhCacheCacheManager之中。Spring提供了EhCacheManagerFactoryBean来生成EhCache的CacheManager。方法ehcache()会创建并返回一个EhCacheManagerFactoryBean实例。因为它是一个工厂bean（实现了Spring的FactoryBean接口），所以注册在Spring应用上下文中的并不是EhCacheManagerFactoryBean的实例，而是CacheManager的一个实例。
```xml
<ehcache>
    <cache name="test" maxBytesLocalHeap="50m" timeToLiveSeconds="100">
    </cache>
</ehcache>
```
ehcache.xml的配置示例

**使用Redis缓存**

需要依赖spring-data-redis

spring-data-redis提供了RedisCacheManager,这是CacheManager的一个实现。RedisCacheManager通过RedisTemplate将缓存存到Redis中。为了使用
RedisCacheManager,需要RedisTemplate和RedisConnectionFactory实现类(如JedisConnectionFactory)的一个bean.

```Java
package com.cai.springcache.config;

import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;

/**
 * Created by reason on 17/5/11.
 */
@Configuration
@EnableCaching
public class RedisCacheConfig {
    /**
     * Redis缓存管理器
     * @param redisTemplate
     * @return
     */
    @Bean
    public CacheManager cacheManager(RedisTemplate redisTemplate){
        return new RedisCacheManager(redisTemplate);
    }

    /**
     * RedisTemplate
     * @param redisConnectionFactory
     * @return
     */
    @Bean
    public RedisTemplate<String,String> redisTemplate(RedisConnectionFactory redisConnectionFactory){
        RedisTemplate<String, String> redisTemplate = new RedisTemplate<String, String>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.afterPropertiesSet();
        return redisTemplate;
    }

    /**
     * Redis连接工厂
     * @return
     */
    @Bean
    public JedisConnectionFactory jedisConnectionFactory(){
        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory();
        jedisConnectionFactory.afterPropertiesSet();
        return jedisConnectionFactory;
    }
}

```
**使用多个缓存管理器**

```java
@Configuration
@EnableCaching
public class CompositeCacheConfig {
    @Bean
    public CacheManager cacheManager(){
        CompositeCacheManager cacheManager = new CompositeCacheManager();
        ArrayList<CacheManager> managers = new ArrayList<CacheManager>();
        managers.add(new JCacheCacheManager());
        managers.add(new EhCacheCacheManager());
        managers.add(new RedisCacheManager(new RedisTemplate()));
        cacheManager.setCacheManagers(managers);
        return cacheManager;
    }
```
当查找缓存时，CompositeCacheManager会先检查jcache,接着ehcache,最后redis来查找缓存。
### 2.为方法添加注解以支持缓存

| 注解          | 描述                                       |
| ----------- | ---------------------------------------- |
| @Cacheable  | 适用于查询。 表明在调用方法之前，首先在缓存中查找方法的返回值，如果找到，则返回缓存的值。否则这个方法就会被调用，返回值放到缓存中。 |
| @CachePut   | 适用于新增。表明将方法的返回值放到缓存中。调用前并不会检查缓存，方法始终会被调用。 |
| @CacheEvict | 适用于删除。表明在缓存中删除一个或多个条目。                   |
| @Caching    | 这是一个分组注解，能够同时应用多个其他的缓存注解。                |
以上注解都能运用在方法或类上。当将其放在单个方法上时，注解所描述的行为只会作用到这个方法上。如果放在类级别的话，那么缓存行为就会应用到这个类的所有方法上。
#### 2.1填充缓存
@Cacheable和CachePut的一些共有的属性。

| 属性        | 类型       | 描述                         |
| --------- | -------- | -------------------------- |
| value     | String[] | 要使用的缓存名称                   |
| condition | String   | SpEl表达式，false不会将缓存应用到方法调用上 |
| key       | String   | SpEl表达式，用来计算自定义的缓存key      |
| unless    | String   | SpEl表达式,true，返回值不会放到缓存中    |
**Cacheable**
```java
@Cacheable("userCache")
public User findOne(long id){
    return jdbc.queryById(id);
}
```
当findOne()被调用时，缓存切面会拦截调用并在缓存中查找，缓存的key是传递到findOne方法中的id参数。
```java
@Cacheable("userCache")
public User findOne(long id);
```
当注解放在实现类时，作用只限于这个实现类，但如果放在接口的方法上，那么所有实现类都会应用相同的缓存规则

**CachePut**
```java
@CachePut("userCache")
User save(User user);
```
**++当save()方法被调用时，首先会保存user,然后返回的User会放到缓存中。那么问题来了，缓存的key默认是方法的参数即user,缓存的value是方法的返回值User。但是我们的查询方法是根据id作为key的。所以这里我们需要缓存的key也应该是User的id。因此我们需要自定义缓存的key.++**

**自定义缓存key**

@Cacheable和@CachePut的key属性能替换掉默认的key，它是根据SpEL表达式计算得到的。常见的用法是定义的表达式与存储在缓存中的值有关。具体到我们上面的例子，需要将key设置为所保存的user的id,但参数中user还没保存，因此没有id，只能通过save()返回的保存后的user得到id。

Spring提供了多个用来定义缓存规则的SpEL扩展

| 表达式               | 描述                                 |
| ----------------- | ---------------------------------- |
| #root.args        | 传递给缓存方法的参数，形式为数组                   |
| #root.caches      | 该方法执行时所对应的缓存，形式为数组                 |
| #root.target      | 目标对象                               |
| #root.targetClass | 目标对象的类，是#root.target.class的缩写      |
| #root.method      | 缓存方法                               |
| #root.methodName  | 缓存方法的名字，是#root.method.name的缩写      |
| #result           | 方法调用的返回值(不能用在@Cacheable注解上)        |
| #Argument         | 任意的方法参数名（如#argName）或参数索引（如#a0或#p0） |

解决上面遇到的问题

```java
@CachePut(value="userCache",key="#result.id")
User save(User user);
```
返回的User会保存在缓存中，key会返回的User的id属性。

**条件化缓存**

通过为方法添加Spring的缓存注解，Spring就会围绕这个方法创建一个缓存切面。但是有些场景下我们希望将缓存功能关闭。@Cacheable和@CachePut的属性unless和condition就可以做到。如果unless表达式计算为true，那么返回的数据就不会放到缓存中，只是禁止放缓存，依然会找缓存。如果condition表达式计算为false，那么对这个方法的缓存就会被禁掉，即既不会找也不会放缓存。

```java
@Cacheable(value="userCache",unless="#result.message.contains('NoCache')")
User findOne(long id);
```
示例中为unless设置的SpEL表达式会表示检查返回的User对象的message属性，如果包含'NoCache'文本neirong,则这个表达式为true,所以这个返回值不会放到缓存中。否则如果为false,则这个返回值会被缓存。

```java
@Cacheable(value="userCache",unless="#result.message.contains('NoCache')",condition="#id>=10")
User findOne(long id);
```
如果参数值id小于10，那么缓存就会被禁用，像没有加这个注解一样。
#### 2.2移除缓存条目（@CacheEvict）

```java
@CacheEvict("userCache")
void remove(long id);
```
当remove()方法调用时，会从缓存中删除一个条目。被删除条目的key与传递进来的id参数的值相等。
与@Cacheable和@CachePut不同，@CacheEvict能够应用于返回值为void的方法上。

| 属性               | 类型       | 描述                                       |
| ---------------- | -------- | ---------------------------------------- |
| value            | String[] | 要使用的缓存名称                                 |
| condition        | String   | SpEl表达式，false不会将缓存应用到方法调用上               |
| key              | String   | SpEl表达式，用来计算自定义的缓存key                    |
| allEntries       | boolean  | 如果为true，特定缓存的所有条目都会被移除                   |
| beforeInvocation | boolean  | 如果为true,在方法调用之前移除条目，如果为fasle(默认)的话，在方法成功调用之后再移除条目 |

