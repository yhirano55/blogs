+++
date = "2017-03-24T16:38:32+09:00"
title = "Gogland EAP版を試す"
description = "Jetbrains社のIDE Gogland EAP版の試用"
tags = ["Golang", "Editor", "Jetbrains"]

+++

普段はiTerm2とVimを使っているが、Jetbrains社のIDEが気になっていたので、[Gogland](https://www.jetbrains.com/go/)を試した。

インストールは、`$ brew cask install gogland-eap` で。とりあえず以下を設定した。GOPATHの設定は、環境変数を見てくれるらしく、何も設定する必要が無かった。キーバインドは、IdeaVimというプラグインをインストールした。

| Option | Value |
|:------:|:-----:|
| Theme | Darcula |
| Fonts | Ricty Diminished Discord |
| Antialiasing | Greyscale |

試しにプロジェクトを作成して、次のコードを書いてみる。

```go
package main

import "fmt"

func main() {
	var input string
	fmt.Println("Enter Your Name: ")
	fmt.Scanln(&input)
	fmt.Printf("Hello, %s", input)
}
```

IDEは初めてだが、補完が非常に良く、`fmt` と入力するだけでスニペットが表示されるだけではなく、引数の定義元の名前が表示されたり、importを自動で追加するなど、とても便利な感じがした。あとはフォーマットを整えてくれるなど、至れり尽くせりである。もう少し使ってから評価したいところではあるが、Vimのキーバインドも使えるので、しばらくはGoglandを利用してみたいと思った。
