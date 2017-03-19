+++
description = "GolangでJSONを読み込む方法"
date = "2017-03-19T22:53:42+09:00"
tags = ["Golang", "JSON"]
title = "GolangでJSON読込"

+++

### encoding/json

スキーマを定義する場合。実行結果は、 `{Id:12345 Message:Hi! User:{Id:23456 Name:Kent Beck ImageUrl:/images/profile.png}}` となる。

```go
package main

import (
	"encoding/json"
	"fmt"
)

var data = `
{
	"id": 12345,
	"message": "Hi!",
	"user": {
		"id": 23456,
		"name": "Kent Beck",
		"image_url": "/images/profile.png"
	}
}
`

type Greeting struct {
	Id      int32  `json:"id"`
	Message string `json:"message"`
	User    User   `json:"user"`
}

type User struct {
	Id       int32  `json:"id"`
	Name     string `json:"name"`
	ImageUrl string `json:"image_url"`
}

func main() {
	var g Greeting
	err := json.Unmarshal([]byte(data), &g)
	if err != nil {
		fmt.Println("error:", err)
	}
	fmt.Printf("%+v", g)
}
```

スキーマを定義しない場合は、`interface{}` を利用する。実行結果は、`map[user:map[id:23456 name:Kent Beck image_url:/images/profile.png] id:12345 message:Hi!]`となる。

```go
package main

import (
	"encoding/json"
	"fmt"
)

var data = `
{
	"id": 12345,
	"message": "Hi!",
	"user": {
		"id": 23456,
		"name": "Kent Beck",
		"image_url": "/images/profile.png"
	}
}
`

func main() {
	var g interface{}
	err := json.Unmarshal([]byte(data), &g)
	if err != nil {
		fmt.Println("error:", err)
	}
	fmt.Printf("%+v", g)
}
```

ファイルを読み込む場合。

```go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
)

func main() {
	buf, err := ioutil.ReadFile("data.json")
	if err != nil {
		fmt.Println(err)
	}

	var g interface{}
	err = json.Unmarshal(buf, &g)
	if err != nil {
		fmt.Println("error:", err)
	}

	fmt.Printf("%+v", g)
}
```

### [Jeffail/gabs](https://github.com/jeffail/gabs)

要素の取得もシンプルになる。事前に `go get github.com/Jeffail/gabs` が必要。結果は `{"id":12345,"message":"Hi!","user":{"id":23456,"image_url":"/images/profile.png","name":"Kent Beck"}}`となる。

```go
package main

import (
	"fmt"
	"github.com/Jeffail/gabs"
	"io/ioutil"
)

func main() {
	buf, _ := ioutil.ReadFile("data.json")
	jsonParsed, err := gabs.ParseJSON(buf)
	if err != nil {
		fmt.Println(err)
	}

	fmt.Printf("%+v", jsonParsed)
}
```

フワッと使いたい場合は、gabsが何かと便利な印象を受けた。要素も、`jsonParsed.Search("id").Data().(int32)` で取り出せる。
