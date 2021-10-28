---
title: Redis分布式锁实现方案
date: 2021-10-28 10:54:30
tags: [Redis, 分布式锁, 八股文]
---

# Redis分布式锁的几种实现方案

## 1. SETNX [key] [value]

抢占锁，`SETNX`是`set if not exists`的简写，如果返回`1`表示成功，返回`0`表示失败，这样然后再 `expire [key] seconds`，但是由于这两种指令不是原子操作，如果执行到`if`之后立马崩溃了，就永远无法过期释放该锁了。一种方案是用`lua`脚本原子实现，一种方案是用`set`指令的扩展参数，`SET key value[EX seconds][PX milliseconds][NX|XX]`，但是存在`锁`过期释放的问题，可能会出现两种错误：1. 由于过期时锁释放了，但是业务没有执行完，这样就导致锁失效，临界区并发就不安全了；2. 可能在最后`finally`释放锁的时候，正好锁过期，其他线程进来，但是刚加完锁，就被当前进程释放了。

```lua
if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then
   redis.call('expire',KEYS[1],ARGV[2])
else
   return 0
end;
```

```java
if（jedis.set(key_resource_id, uni_request_id, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch(){
  }
  finally {
       //判断是不是当前线程加的锁,是才释放
       if (uni_request_id.equals(jedis.get(key_resource_id))) {
        jedis.del(lockKey); //释放锁
        }
    }
}
```

在这种情况下，首先可以设置`uni_request_id`，在释放的时候保证释放的是自己的线程持有的锁，当然依然不是原子操作，需要用`lua`脚本实现。

```lua
if redis.call('get',KEYS[1]) == ARGV[1] then 
   return redis.call('del',KEYS[1]) 
else
   return 0
end;
```

## 2. 最后还是会存在“锁过期，但是业务没有执行完”的情况

可以开启一个守护线程，看门狗定时去查看锁是否过期，如果即将过期就延长锁时间。

## 3. Redis 集群分布式锁 RedLock+Redisson，以确保高可用：使用多个完全独立 Master 节点，这些节点与 client 用相同方法加锁、释放锁。(大于 N/2 则加锁成功，否则失败，需要释放剩余的锁)