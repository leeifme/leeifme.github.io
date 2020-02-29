---
title: MongoDB 入门操作
date: 2017-11-20 23:32:45
tags:
  - MongoDB
categories:
  - notes
---

> 前言：因为最近业务转型，公司后台服务器也需要调整之前的业务架构，现在也想从之前低版本，低可用的数据库迁移，最近在了解MongoDB,所以就记录一下📝 

## 什么是 MongoDB

MongoDB 是由 C++ 语言编写的，是一个基于分布式文件存储的开源数据库系统。在高负载的情况下，添加更多的节点，可以保证服务器性能。
MongoDB 旨在为 WEB 应用提供可扩展的高性能数据存储解决方案。
MongoDB 将数据存储为一个文档，数据结构由键值 (key=>value) 对组成。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档，数组及文档数组。

```sql
{
  name: "leeifme"               //field:value
  age: "21"                     //field:value
  status: "A"                   //field:value
  groups: ["news" , "sports"]   //field:value
}
```

## 主要特点

- `MongoDB` 的提供了一个面向文档存储，操作起来比较简单和容易
- 你可以在 `MongoDB` 记录中设置任何属性的索引 (如：FirstName="Sameer",Address="8 Gandhi Road") 来实现更快的排序
- 你可以通过本地或者网络创建数据镜像，这使得 `MongoDB` 有更强的扩展性
- 如果负载的增加（需要更多的存储空间和更强的处理能力） ，它可以分布在计算机网络中的其他节点上这就是所谓的分片
- `Mongo` 支持丰富的查询表达式。查询指令使用 JSON 形式的标记，可轻易查询文档中内嵌的对象及数组
- `MongoDB` 使用 update() 命令可以实现替换完成的文档（数据）或者一些指定的数据字段
- `MongoDB` 中的 `Map/reduce` 主要是用来对数据进行批量处理和聚合操作
- `Map` 和 `Reduce`。Map 函数调用 emit(key,value) 遍历集合中所有的记录，将 key 与 value 传给 Reduce 函数进行处理
- `Map` 函数和 `Reduce` 函数是使用 Javascript 编写的，并可以通过 db.runCommand 或 mapreduce 命令来执行 MapReduce 操作
- `GridFS` 是 `MongoDB` 中的一个内置功能，可以用于存放大量小文件
- `MongoDB` 允许在服务端执行脚本，可以用 Javascript 编写某个函数，直接在服务端执行，也可以把函数的定义存储在服务端，下次直接调用即可
- `MongoDB` 支持各种编程语言: RUBY，PYTHON，JAVA，C++，PHP，C# 等多种语言
- `MongoDB` 安装简单

## 安装下载

> 官网下载 ——> [下载地址](https://www.mongodb.com/download-center#community)

接下来我们使用 curl 命令来下载安装：

```sql
# 进入 /usr/local
cd /usr/local

# 下载
sudo curl -O https://fastdl.mongodb.org/osx/mongodb-osx-x86_64-3.4.7.tgz  //目前最新版本号3.4.7

# 解压
sudo tar -zxvf mongodb-osx-x86_64-3.4.7.tgz

# 重命名为 mongodb 目录

sudo mv mongodb-osx-x86_64-3.4.7 mongodb
```

> 使用 brew 安装

使用 OSX 的 brew 来安装 mongodb：

```sql
sudo brew install mongodb
```

如果要安装支持 TLS/SSL 命令如下：z'z

```sql
sudo brew install mongodb --with-openssl
```

## 运行
1. 先创建一个数据库存储目录 `/data/db`（_注意：如果你的数据库目录不是 `/data/db`，可以通过 `--dbpath` 来指定_）

```sql
sudo mkdir -p /data/db
```

2. 启动 mongodb，默认数据库目录即为 /data/db

```sql
sudo mongod

# 如果没有创建全局路径 PATH，需要进入以下目录
cd /usr/local/mongodb/bin
sudo ./mongod
```

3. 再打开一个终端进入执行以下命令

```sql
$ cd /usr/local/mongodb/bin 
$ ./mongo
```

## 基本操作

1.1 列出所有数据库

> show dbs 
> local  0.000GB
> myblog 0.000GB

1.2 使用某个数据库

> use myblog
> switched to db myblog

1.3 列出所有集合

> show collections
> articles
> replicationColletion
> sessions
> users
> wangduanduan

2 插入数据 insert(value)

// 在已经存在的集合中插入数据
> db.users.insert({name:'hh',age:23})
> Inserted 1 record(s) in 43ms
> // 在不存在的集合中插入数据,集合不存在则自动创建集合并插入
> db.students.insert({name:'hh',age:23})
> Inserted 1 record(s) in 72ms

3 查询 find(option) 
3.1 查询集合里所有的文档

> db.users.find()
> /* 1 */
> {
>   "_id" : ObjectId("583e908453be942d0c5419dc"),
>   "login_name" : "wangduanduan",
>   "password" : "wrong age"
> }
> /* 2 */
> {
>   "_id" : ObjectId("583ed2a5cc9a937db049616d"),
>   "login_name" : "hh",
>   "password" : "sdfsdf"
> }
> /* 3 */
> {
>   "_id" : ObjectId("583fb2e9b12f8b7a7aa37572"),
>   "name" : "wangduanduan",
>   "age" : 34.0
> }
> /* 4 */
> {
>   "_id" : ObjectId("583fb707b12f8b7a7aa37573"),
>   "name" : "hh",
>   "age" : 23.0
> }

3.2 按条件查询文档

> db.users.find({name:'wangduanduan'})
> /* 1 */
> {
>   "_id" : ObjectId("583fb2e9b12f8b7a7aa37572"),
>   "name" : "wangduanduan",
>   "age" : 34.0
> }
> 注意
> // 这是错的，查不到结果
> db.users.find({_id:'583fb2e9b12f8b7a7aa37572'})
> Fetched 0 record(s) in 1ms
> // 这是正确的
> db.users.find({_id:ObjectId('583fb2e9b12f8b7a7aa37572')})
> /* 1 */
> {
>   "_id" : ObjectId("583fb2e9b12f8b7a7aa37572"),
>   "name" : "wangduanduan",
>   "age" : 34.0
> }

3.3 查询集合内文档的个数
> db.users.count()

3.4 比较运算符 
gt: 大于 gte: 大于等于 
lt: 小于 lte: 小于等于 
$ne: 不等于

// 查询用户里年龄大于30岁的人， 其他条件以此类推
> db.user.find({age:{$gt:30}})

/* 1 */
{
  "_id" : ObjectId("583fb2e9b12f8b7a7aa37572"),
  "name" : "wangduanduan",
  "age" : 34.0
}

3.5 逻辑运算符 
3.5.1 与

// 查询名字是wangduanduan,age=34的用户
> db.users.find({name:'wangduanduan',age:34})
> /* 1 */
> {
>   "_id" : ObjectId("583fb2e9b12f8b7a7aa37572"),
>   "name" : "wangduanduan",
>   "age" : 34.0
> }

3.5.2 $in 或

// 查询名字是wangduanduan,或hh的用户
> db.users.find({name:{$in:['wangduanduan','hh']}})
> /* 1 */
> {
>   "_id" : ObjectId("583fb2e9b12f8b7a7aa37572"),
>   "name" : "wangduanduan",
>   "age" : 34.0
> }

3.5.3 $nin 非

// 查询名字不是wangduanduan或者hh的用户
> db.users.find({name:{$nin:['wangduanduan','hh']}})
> /* 1 */
> {
>   "_id" : ObjectId("583e908453be942d0c5419dc"),
>   "login_name" : "wangduanduan",
>   "password" : "wrong age"
> }
> /* 2 */
> {
>   "_id" : ObjectId("583ed2a5cc9a937db049616d"),
>   "login_name" : "hh",
>   "password" : "sdfsdf"
> }

3.6 正则匹配

// 查询名字是中含有duan的用户
> db.users.find({name:/duan/})
> /* 1 */
> {
>   "_id" : ObjectId("583fb2e9b12f8b7a7aa37572"),
>   "name" : "wangduanduan",
>   "age" : 34.0
> }
> /* 2 */
> {
>   "_id" : ObjectId("583fc919b12f8b7a7aa37575"),
>   "name" : "wangduanduan",
>   "age" : 45.0
> }

4 更新 update(); 
4.1 整体更新

> db.users.update({login_name:'wangduanduan'},{name:'heihei',age:34})
> Updated 1 existing record(s) in 116ms

4.2 $set 局部更新

// 只是将用户年龄设置成101
> db.users.update({name:'wangduanduan'},{$set:{age:101}})

4.3 $inc

// 将用户年龄增加1岁，如果文档没有age这个字段，则会增加这个字段
> db.users.update({name:'wangduanduan'},{$inc:{age:1}})

4.3 upsert 操作

// 如果查不到文档，则增加文档
> db.users.update({name:'nobody'},{$inc:{age:1}},true)
> Updated 1 new record(s) in 3ms
> /* 6 */
> {
    "_id" : ObjectId("583fd20f2cfa6a4817c4171c"),
    "name" : "nobody",
    "age" : 1.0
}

4.4 批量更新

// upadate 的第四个参数设置成true的时候，就会批量更新
> db.users.update({name:'wangduanduan'},{$set:{age:1891}},false,true)

5 删除

// 删除某些文档
db.person.remove({"name":"joe"})
// 删除整个集合
db.person.remove()

## 后记
这只是初次的记录，之后应该会出几篇相关的学习记录或是实战分享，这个flag算是立下了，但是我的超强拖延症，不知道会不会折腾死🐶✌️

