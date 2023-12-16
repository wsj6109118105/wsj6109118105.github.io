---
title: Redis缓存穿透/击穿/雪崩
date: 2021-08-13 19:15:35
tags: Redis
---
## 介绍
- 缓存穿透：查询的数据并不存在，从缓存中取不到，则去数据库中查找，如果数量过多，可能会压垮数据库。
- 缓存击穿：查询的数据在缓存中存在，但是数据过期，此时有大量请求的话，就会直接去数据库中查找并加载回缓存，导致数据库压力过大。
- 缓存雪崩：当缓存服务器重启或者有大量数据集中过期，那么也会有大量请求去数据库中查找数据，给数据库造成很大压力。
## 缓存穿透两种解决方法
- 采用布隆过滤器，将所有可能存在的数据哈希到一个足够大的bitmap中，一个一定不存在的数据会被 这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。
- 如果查询数据返回为空(不管是不存在，还是系统故障)，仍然把这个空值进行缓存，但过期时间很短。
```java
//伪代码
public object GetProductListNew() {
    int cacheTime = 30;
    String cacheKey = "product_list";

    String cacheValue = CacheHelper.Get(cacheKey);
    if (cacheValue != null) {
        return cacheValue;
    }

    cacheValue = CacheHelper.Get(cacheKey);
    if (cacheValue != null) {
        return cacheValue;
    } else {
        //数据库查询不到，为空
        cacheValue = GetProductListFromDB();
        if (cacheValue == null) {
            //如果发现为空，设置个默认值，也缓存起来
            cacheValue = string.Empty;
        }
        CacheHelper.Add(cacheKey, cacheValue, cacheTime);
        return cacheValue;
    }
}
```
## 缓存击穿解决方法
使用互斥锁(mutex key)
在缓存失效的时候，不直接去数据库中查找，而是使用缓存中带有成功操作的返回值的操作，当操作成功时，才去数据库中查找，比如setnx,当第一个请求来时setnx操作成功，去数据库中查找，并设置缓存，其他请求执行setnx操作不成功，无法去数据库查找，等待第一个请求之后，再去缓存中查找。
```java
public String get(key) {
    String value = redis.get(key);
    if (value == null) { //代表缓存值过期
          //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
        if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
            value = db.get(key);
            redis.set(key, value, expire_secs);
            redis.del(key_mutex);
        } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
            sleep(50);
            get(key);  //重试
        }
    } else {
              return value;      
    }
 }
```
## 缓存雪崩解决方法
- 通过加锁或者队列的方法来防止大量的线程对数据库一次性进行读写操作，从而避免失效时大量的并发请求落到底层存储系统上。
- 可以再原本的失效时间上加上一个随机值，这样缓存的过期时间的重复率就会降低，很难引发集体失效的事件。

加锁排队方式伪代码
```java
//伪代码
public object GetProductListNew() {
    int cacheTime = 30;
    String cacheKey = "product_list";
    String lockKey = cacheKey;

    String cacheValue = CacheHelper.get(cacheKey);
    if (cacheValue != null) {
        return cacheValue;
    } else {
        synchronized(lockKey) {
            cacheValue = CacheHelper.get(cacheKey);
            if (cacheValue != null) {
                return cacheValue;
            } else {
              //这里一般是sql查询数据
                cacheValue = GetProductListFromDB(); 
                CacheHelper.Add(cacheKey, cacheValue, cacheTime);
            }
        }
        return cacheValue;
    }
}
```
随机值伪代码
```java
//伪代码
public object GetProductListNew() {
    int cacheTime = 30;
    String cacheKey = "product_list";
    //缓存标记
    String cacheSign = cacheKey + "_sign";

    String sign = CacheHelper.Get(cacheSign);
    //获取缓存值
    String cacheValue = CacheHelper.Get(cacheKey);
    if (sign != null) {
        return cacheValue; //未过期，直接返回
    } else {
        CacheHelper.Add(cacheSign, "1", cacheTime);
        ThreadPool.QueueUserWorkItem((arg) -> {
      //这里一般是 sql查询数据
            cacheValue = GetProductListFromDB(); 
          //日期设缓存时间的2倍，用于脏读
          CacheHelper.Add(cacheKey, cacheValue, cacheTime * 2);                 
        });
        return cacheValue;
    }
}
```
解释说明：

- 缓存标记：记录缓存数据是否过期，如果过期会触发通知另外的线程在后台去更新实际key的缓存；
- 缓存数据：它的过期时间比缓存标记的时间延长1倍，例：标记缓存时间30分钟，数据缓存设置为60分钟。这样，当缓存标记key过期后，实际缓存还能把旧数据返回给调用端，直到另外的线程在后台更新完成后，才会返回新缓存。

参考相关链接：https://www.cnblogs.com/midoujava/p/11277096.html