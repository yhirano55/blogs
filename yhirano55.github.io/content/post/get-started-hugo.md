+++
tags = ["Hugo", "CircleCI"]
title = "HugoとCircleCIは最高の組み合わせ"
date = "2017-03-18T12:46:09+09:00"
description = "HugoとCircleCIを使ってGithub Pageを運用する"

+++

### tl;dr

Github Pagesの運用をするならば、HugoとCircleCIを使うと快適

---

[Hugo](https://gohugo.io/)は、Go製の静的サイトジェネレーターだ。筆者は過去にJekyll, Octopress, Middlemanを利用したことがあるが、Hugoのビルドの速さには驚いた。実際に計測したわけではないが、1ファイルを1msで生成するらしい。

### 使い方

最低限にその手軽さだけ伝えると、Homebrewかgo getでインストールする。その後、次のコマンドを実行すれば、すぐに静的サイトが構築される。

```sh
$ hugo new site your_site
$ huge new post/hello-world.md
$ hugo server
$ open http://localhost:1313
```

マークダウンで記述でき、[テーマ](http://themes.gohugo.io/)も充実している。

### ビルドとデプロイ

筆者はビルドとデプロイのフロー（運用）について、Githubのリポジトリにマージされたらデプロイされる仕組みを採用した。具体的な構築については、次の記事で紹介されていたので、詳細な説明は割愛したい。

[HugoとCircleCIでGitHub PagesにBlogを公開してみた · Hori Blog](https://hori-ryota.com/blog/create-blog-with-hugo-and-circleci/)

