+++
description = "ginとsqliteでAPIサーバーを試作した"
tags = ["Golang", "gin", "sqlite"]
date = "2017-03-26T14:06:09+09:00"
title = "Go製の軽量WAF gin"

+++

「[Go言語製WAF GinでWebアプリを作ってみる【準備編】 | eureka tech blog](https://developers.eure.jp/tech/go_web_application_1/)」を参考に、軽量WAFである[gin-gonic/gin](https://github.com/gin-gonic/gin)を試用する。まずは雛形の作成から。

```sh
$ mkdir gin-api-server && cd $_
$ go get github.com/gin-gonic/gin
```

続けて `main.go` を実装。`localhost:8080` にアクセスすると、固定値のJSONが返る。

```go
package main

import (
	"github.com/gin-gonic/gin"
)

func main() {
	router := gin.Default()

	router.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{"message": "hello, gin!"})
	})
	router.Run(":8080")
}
```

`{"message":"hello, gin!"}` が返り、レスポンスヘッダーを見ると、`Content-Type:application/json; charset=utf-8` かつ `200 OK` であることが分かる。

続いて、モデルの実装。

```sh
$ mkdir models
$ go get github.com/go-xorm/xorm
$ go get github.com/mattn/go-sqlite3
```

モデルは、ID・姓・名で構成されたUserで、データベースの操作はUserRepositoryを介して行う。

```go
package models

import (
	"github.com/go-xorm/xorm"

	_ "github.com/mattn/go-sqlite3"
)

var engine *xorm.Engine

func init() {
  var err error
	engine, err = xorm.NewEngine("sqlite3", "./development.db")
	if err != nil {
		panic(err)
	}
}

type User struct {
	ID        int    `json:"id" xorm:"'id'"`
	FirstName string `json: "first_name" xorm: "'first_name'"`
	LastName  string `json: "last_name" xorm: "'last_name'"`
}

func NewUser(id int, first_name string, last_name string) User {
	return User{
		ID:        id,
		FirstName: first_name,
		LastName:  last_name,
	}
}

type UserRepository struct{}

func NewUserRepository() UserRepository {
	return UserRepository{}
}

func (m UserRepository) GetByID(id int) *User {
	var user = User{ID: id}
	has, _ := engine.Get(&user)
	if has {
		return &user
	}

	return nil
}
```

データベースとテーブルの作成。実践ならmigrationを使いたいところだが、試用なので手作業で準備した。

```
$ sqlite3 development.db
sqlite> CREATE TABLE user(id INTEGER PRIMARY KEY AUTOINCREMENT, first_name VARCHAR(255) NOT NULL, last_name VARCHAR(255) NOT NULL);
sqlite> .tables
sqlite> INSERT INTO user(first_name, last_name) VALUES("Yamada", "Taro");
sqlite> INSERT INTO user(first_name, last_name) VALUES("Sato", "Hajime");
sqlite> INSERT INTO user(first_name, last_name) VALUES("Koike", "Tatsuo");
sqlite> SELECT * FROM user;
1|Yamada|Taro
2|Sato|Hajime
3|Koike|Tatsuo
sqlite> .quit
```

コントローラを実装する。コントローラは、`id` パラメータの値でリポジトリに照会し、その結果を返す。

```sh
$ mkdir controllers
```

```go
package controllers

import (
	"github.com/yhirano55/gin-api-server/models"
)

type User struct{}

func NewUser() User {
	return User{}
}

func (c User) Get(id int) interface{} {
	repo := models.NewUserRepository()
	user := repo.GetByID(id)
	return user
}
```

最後に `main.go` の修正。

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/yhirano55/gin-api-server/controllers"
	"reflect"
	"strconv"
)

func main() {
	router := gin.Default()

	router.GET("/", func(c *gin.Context) {
		c.JSON(200, gin.H{"message": "hello, gin!"})
	})

	router.GET("/user/:id", func(c *gin.Context) {
		v := c.Param("id")
		id, err := strconv.Atoi(v)
		if err != nil {
			c.JSON(400, err)
			return
		}
		if id <= 0 {
			c.JSON(400, gin.H{"message": "id should be greater than 0"})
			return
		}
		ctrl := controllers.NewUser()
		result := ctrl.Get(id)
		if result == nil || reflect.ValueOf(result).IsNil() {
			c.JSON(404, gin.H{"message": "not found"})
			return
		}

		c.JSON(200, result)
	})

	router.Run(":8080")
}
```

`go run main.go` でサーバーを起動し、 `localhost:8080/user/1` にアクセスすると、 `{"id":1,"FirstName":"Yamada","LastName":"Taro"}` が返る。

本記事のサンプルは[yhirano55/gin-api-server](https://github.com/yhirano55/gin-api-server)にアップロードした。
