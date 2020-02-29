---
title: 初学 Redis 作缓存层
tags:
  - Redis
  - Go
categories:
  - notes
date: 2018-05-15 23:32:45
---

> 项目需求：数据库用的是MySQL，考虑用Redis/memcached做数据库的缓存层。在读DB前，先读缓存层，如果有直接返回，如果没有再读DB，然后写入缓存层并返回。


## 思路：
###缓存读取流程
1. 先到缓存中查数据                         
2. 缓存中不存在则到实际数据源中取，取出来后放入缓存                     
3. 下次再来取同样信息时则可直接从缓存中获取

###缓存更新流程
1. 更新数据库                         
2. 使缓存过期或失效，这样会促使下次查询数据时在缓存中查不到而重新从数据库去一次。

###通用缓存机制
1. 用查询的方法名 + 参数作为查询时的 key value 对中的 key 值
2. 向 memcache 或 redis 之类的 nosql 数据库（或者内存 hashmap）插入数据3、取数据时也用方法名 + 参数作为 key 向缓存数据源获取信息。

## 应用场景
### 判断缓存是否存在

```go
dataList, err := dataRedis.Get(CacheKey).Result()
if err == redis.Nil {
	// 不存在redis key
	return "", false
} else if err != nil {
	log.Errorf("Get cache err, cache_key: %s, error: %s", CacheKey, err)
	return "", false
} else {
	// 缓存值为空
	if dataList == "" {
		return "", false
	}
}

return dataList, true
```

### 对数据做处理

```go
CacheKey := "key"
Data, isTrue := isGetDataCache(CacheKey)

if isTrue {
	// 从`redis`中获取数据直接返回
	err := json.Unmarshal([]byte(Data), &dataList)
	if err != nil {
		log.Errorf("Json unmarshal datalist err:%s", err)
		return dataList
	}
} else {
	// 从db中获取数据
	dataList = GetDataWhere("`deleted`=0 order by sort limit ? offset ?", limit, offset)

	// 更新redis缓存
	err := dataRedis.Set(CacheKey, xutil.ToJSONString(dataList), 1*time.Minute).Err()
	if err != nil {
		log.Errorf("Set rediscache error:%s", err)
		return dataList
	}

}
return dataList
```

## 思考

### 写入数据是否立刻更新缓存

如果需要立刻更新缓存的话，使用 findAndModify 获取最新结果并设置进缓存中，但这带来了一个弊端，有些并不是热数据，这时候放进了 redis 会导致占用内存，如果设置过快，redis 内存的使用量增长得非常快；好处需要与另一种方式对比才明显，另一种方式是写入数据后立刻删掉该缓存，等待有需要才再加载缓存，这样的好处是 redis 占用缓存不会过高，总是保留着热数据，弊端就是加载缓存读取从库的数据还是未同步的，这样导致脏数据，所以前者的直接更新保证了不会是脏数据。至于采取那种方式，要看使用场景，如果不需要最新的数据，最方便是删除掉该缓存就可以了，否则就需要立刻更新缓存数据，如果键数量特别多的情况下，可以设置过期时间短一些，尽快释放 redis 的内存占用。

### 读取 Set 和 SortedSet 数据缓存失效问题

这种大数据集合，无法单个操作就可以判断其 key 存在还是该 key 是否有值。通常需要两个操作：加载和获取需要的数据。问题就出现在加载和获取的间隙中，这个间隙缓存刚好过期，就会出现数据错误的后果，我觉得最佳实践是这样: 保证在服务运行过程中，这些数据集合不会缓存过期。像我们使用排行榜的时候用到大量了 SortedSet，当时设置了 1 天就过期，这样导致出现排行榜数据出现不全的问题。又如 Set 保存着比赛名单的数据，发现 key 是存在，不再加载数据，准备判断某个项是否存在该 set 的时候，这个 set 就过期了，用sismember就会返回False，就无法判断这个 False 的意思是该数据项不存在该 set 里面还是该 set 的 key 已经过期了。

### 在项目中 Redis 做缓存
- 首先，对于数据缓存不是所有东西都缓存到 redis 就是好的，而是要针对一些改动不大或者访问率大的数据进行缓存来减少关系型数据库的压力。
- redis 作为缓存使用，不作数据库用途，遵循以下规则：如果缓存没有数据，即加载数据到缓存，并会设置过期时间。
- 凡是可以用 String 保存的，尽量将数据用json.dumps之后再放进 String，如果需要 Set 和 SortedSet 就需要警惕缓存 key 刚好过期时候，会有一定读取错误的问题，这个无法避免。
- 对于一些不分页，不需要实时（需要多表查询）的列表，我们可以将列表结果缓存到 redis 中，设定一定缓存时间作为该数据的存活时间。用获取该列表的方法名作为 key，列表结果为 value；这种情况只试用于不经常更新且不需要实时的情况下。
- 不需要实时的，需要分页的列表：可以把分页的结果列表放到一个 map（key 为分页标识，value 为分页结果）中，然后将该 map 存到 redis 的 list 中（用该方法名为 key）。然后给该 list 设置一个缓存存活时间（用 expire）。这样通过方法名 lrange 出来就能获取存有分页列表的数据，遍历该 list，通过遍历 list 中 map 的 key 判断该分页数据是否在缓存内，是则返回，不存在则 rpush 进去。这种做法能解决比如 1-5 页的数据已经重新加载，而 6-10 页的数据依然是缓存的数据而导致脏数据的情况。



