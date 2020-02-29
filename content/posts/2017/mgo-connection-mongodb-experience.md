---
title: mgo 连接 MongoDB 数据库的使用实例
date: 2017-12-10 23:32:45
tags:
  - MongoDB
  - Go
categories:
  - notes
---

```Go
// mgotest project main.go
package main

import (
	"fmt"
	"time"

	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
)

type User struct {
	Id        bson.ObjectId `bson:"_id"`
	Username  string        `bson:"name"`
	Pass      string        `bson:"pass"`
	Regtime   int64         `bson:"regtime"`
	Interests []string      `bson:"interests"`
}

const URL = "127.0.0.1:27017"

var c *mgo.Collection
var session *mgo.Session

var GlobalMgoSession, err = mgo.Dial(URL)

func (user User) ToString() string {
	return fmt.Sprintf("%#v", user)
}

// func init() {
// 	session = GlobalMgoSession.Clone()
// 	// session, _ = mgo.Dial(URL)
// 	//切换到数据库
// 	db := session.DB("blog")
// 	//切换到collection
// 	c = db.C("mgotest")
// }

//新增数据
func add() {
	session = GlobalMgoSession.Clone()
	// session, _ = mgo.Dial(URL)
	//切换到数据库
	db := session.DB("blog")
	//切换到collection
	c = db.C("mgotest")
	defer session.Close()
	stu1 := new(User)
	stu1.Id = bson.NewObjectId()
	stu1.Username = "stu1_name"
	stu1.Pass = "stu1_pass"
	stu1.Regtime = time.Now().Unix()
	stu1.Interests = []string{"象棋", "游泳", "跑步"}
	err := c.Insert(stu1)
	if err == nil {
		fmt.Println("插入成功")
	} else {
		fmt.Println(err.Error())
		defer panic(err)
	}
}

//查询
func find() {
	session = GlobalMgoSession.Clone()
	// session, _ = mgo.Dial(URL)
	//切换到数据库
	db := session.DB("blog")
	//切换到collection
	c = db.C("mgotest")
	defer session.Close()
	var users []User
	//    c.Find(nil).All(&users)
	c.Find(bson.M{"name": "stu1_name"}).All(&users)
	for _, value := range users {
		fmt.Println(value.ToString())
	}
	//根据ObjectId进行查询
	idStr := "599a46ca1d8ec1068da6c620"
	objectId := bson.ObjectIdHex(idStr)
	user := new(User)
	c.Find(bson.M{"_id": objectId}).One(user)
	fmt.Println(user)
}

//根据id进行修改
func update() {
	session = GlobalMgoSession.Clone()
	// session, _ = mgo.Dial(URL)
	//切换到数据库
	db := session.DB("blog")
	//切换到collection
	c = db.C("mgotest")
	interests := []string{"象棋", "游泳", "跑步"}
	err := c.Update(bson.M{"_id": bson.ObjectIdHex("599a46f71d8ec106b29c6681")}, bson.M{"$set": bson.M{
		"name":      "修改后的name",
		"pass":      "修改后的pass",
		"regtime":   time.Now().Unix(),
		"interests": interests,
	}})
	if err != nil {
		fmt.Println("修改失败")
	} else {
		fmt.Println("修改成功")
	}
}

//删除
func del() {
	session = GlobalMgoSession.Clone()
	// session, _ = mgo.Dial(URL)
	//切换到数据库
	db := session.DB("blog")
	//切换到collection
	c = db.C("mgotest")
	err := c.Remove(bson.M{"_id": bson.ObjectIdHex("599a4ea31d8ec10749c5f119")})
	if err != nil {
		fmt.Println("删除失败" + err.Error())
	} else {
		fmt.Println("删除成功")
	}
}
func main() {
	add()
	find()
	update()
	del()
}
```

