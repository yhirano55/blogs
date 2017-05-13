+++
date = "2017-05-13T22:38:08+09:00"
title = "はてなブログからのインポート"
description = "はてなブログのAPIを用いて記事を移管した話"
tags = ["api", "Ruby"]

+++

2015年くらいから1年程度、はてなブログで技術的な備忘録を書いていて、なかでもGoogle Apps Scriptの記事が割とアクセスされているようなので、全削除してもいいのだが、どうせならこちらのGitHub Pageに移行できればと思い、いろいろ調べてみた。

当然ながら、Hugoのフォーマットに変換されることはないので、REST APIはあるはずだと思い、漁ってみたところ、[kymmt90/hatenablog](https://github.com/kymmt90/hatenablog) というGemが見つかったので、それを使って、15分程度で移行できた。

書いたスクリプトは、 [yhirano55/hatena-blog-exporter](https://github.com/yhirano55/hatena-blog-exporter) に置いてあるが、内容としては素朴なRakeタスクで、OAuthを使うのでそれらの情報を取得した後、`bundle exec get_access_token <client_key> <client_secret>` （前述のhatenablogのgemで定義されたスクリプト）でアクセストークンを取得。それらを `config.yml` に書いて、 `bundle exec rake hatena-blog-exporter:hugo` を実行すると、 `post/*.md` が大量に作成される。

書いたRakefileは以下。loopで回してAPIにアクセスし、エントリをmd形式にして保存していくだけ。

```ruby
require "rubygems"
require "bundler/setup"
Bundler.require

require "fileutils"
require "erb"

namespace :"hatena-blog-exporter" do
  desc "export entries for hugo markdown format"
  task :hugo do
    client = Hatenablog::Client.create
    erb    = File.read("templates/hugo.md.erb")
    feed   = nil

    FileUtils.mkdir('post')

    loop do
      feed = client.next_feed(feed)

      feed.each_entry do |entry|
        File.open("post/#{entry.id}.md", "w+") do |f|
          f.puts ERB.new(erb).result(binding)
          puts "created post/#{entry.id}.md"
        end
      end

      break unless feed.has_next?
    end
  end
end
```

途中、 `/oauth/autholize` のドメインが、はてなブログの場合、トークンリクエスト時のドメインと異なることに気づき、アクセストークン発行するのに、少しだけ手間取った。それについては、kymmt90/hatenablog宛に[PR](https://github.com/kymmt90/hatenablog/pull/12)を送っておいた。
