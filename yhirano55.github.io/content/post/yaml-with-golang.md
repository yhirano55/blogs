+++
description = "GolangでYAMLを読み込む方法"
date = "2017-03-21T06:27:45+09:00"
tags = ["Golang", "YAML"]
title = "GolangでYAML読込"

+++

### [go-yaml/yaml](https://github.com/go-yaml/yaml)

まずは、プロジェクトの作成からパッケージのインストールまで。

```sh
$ brew install glide
$ mkdir go-yaml-sample && cd $_
$ git init
$ glide create
$ git add . && git commit -m "glide create"
$ echo "vendor/" >> .gitignore
$ git add . && git commit -m "add .gitignore"
$ glide get gopkg.in/yaml.v2
$ git add . && git commit -m "glide get gopkg.in/yaml.v2"
```

`main.go` は以下。JSONと同じく、Typeを定義していても、`interface{}`でも読取可能。

```go
package main

import (
	"fmt"
	"gopkg.in/yaml.v2"
)

var data = `
date: 2017-03-21
fruits:
- id: 1
  name: apple
  count: 5
- id: 2
  name: banana
  count: 3
- id: 3
  name: orange
  count: 1
`

type Fruit struct {
	Id    int
	Name  string
	Count int
}

type Config struct {
	Date   string
	Fruits []Fruit
}

func main() {
	t := Config{}

	err := yaml.Unmarshal([]byte(data), &t)
	if err != nil {
		fmt.Printf("error: %v", err)
	}
	fmt.Printf("--- t:\n%v\n\n", t)

	d, err := yaml.Marshal(&t)
	if err != nil {
		fmt.Printf("error: %v", err)
	}
	fmt.Printf("--- t dump:\n%s\n\n", string(d))
}
```

実行結果。

```
$ go run main.go
--- t:
{2017-03-21 [{1 apple 5} {2 banana 3} {3 orange 1}]}

--- t dump:
date: 2017-03-21
fruits:
- id: 1
  name: apple
  count: 5
- id: 2
  name: banana
  count: 3
- id: 3
  name: orange
  count: 1
```

go-yaml/yamlの内部は、[libyaml](http://pyyaml.org/wiki/LibYAML)を使って実装されている。ライセンスは、Apache License 2.0。
