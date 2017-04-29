+++
date = "2017-04-30T04:41:17+09:00"
title = "OSS Contribution of Apr 2017"
description = "2017年4月のOSSへの貢献について"
tags = ["OSS", "mastodon"]

+++

以下のプルリクエストがマージされた。

[Add rake task to report actual code statistics by yhirano55 · Pull Request #2489 · tootsuite/mastodon](https://github.com/tootsuite/mastodon/pull/2489)

今月の話題といえば、脱中央集権型のミニブログサービス [mastodon](https://mastodon.social/about)だ。マストドンのサーバーは誰でも運用できるため、そのコードは、GitHubで公開されている。WAFは、Rails 5なので、コードリーディングの教材としても最適だ。先のPRは、コードリーディングをする際、 `rake stats` を実行してみて、独自に追加されたServicesやValidatorsのレイヤーが、解析結果に反映されていなかったので修正した次第。

### Taskの実装

大元のRailsでの実装は次のようになっている。

```ruby
# While global constants are bad, many 3rd party tools depend on this one (e.g
# rspec-rails & cucumber-rails). So a deprecation warning is needed if we want
# to remove it.
STATS_DIRECTORIES = [
  %w(Controllers        app/controllers),
  %w(Helpers            app/helpers),
  %w(Jobs               app/jobs),
  %w(Models             app/models),
  %w(Mailers            app/mailers),
  %w(Channels           app/channels),
  %w(JavaScripts        app/assets/javascripts),
  %w(Libraries          lib/),
  %w(APIs               app/apis),
  %w(Controller\ tests  test/controllers),
  %w(Helper\ tests      test/helpers),
  %w(Model\ tests       test/models),
  %w(Mailer\ tests      test/mailers),
  %w(Job\ tests         test/jobs),
  %w(Integration\ tests test/integration),
  %w(System\ tests      test/system),
].collect do |name, dir|
  [ name, "#{File.dirname(Rake.application.rakefile_location)}/#{dir}" ]
end.select { |name, dir| File.directory?(dir) }

desc "Report code statistics (KLOCs, etc) from the application or engine"
task :stats do
  require "rails/code_statistics"
  CodeStatistics.new(*STATS_DIRECTORIES).to_s
end
```

定数 `STAS_DIRECTORIES` に対象ディレクトリが定義されている。なお、コメントを意訳すると、次のような内容となる。

> グローバルの定数は微妙だけど、多くのサードパーティ（rspec-railsやcucumber-rails）が依存している。
> まあ、必要だったら、警告出すようにして、これを削除しましょう。

良し悪しはさておき、rspec-railsで同じ定義があるようなので、そちらの実装を読むと、定数に対して、追加定義をpushしている。

```ruby
task :statsetup do
  require 'rails/code_statistics'
  types.each do |type, dir|
    name = type.singularize.capitalize

    ::STATS_DIRECTORIES << ["#{name} specs", dir]
    ::CodeStatistics::TEST_TYPES << "#{name} specs"
  end
end
```

これに倣って、PRは作られ、数時間後にマージされた。説明文は、何が問題で何を解決したのか、そして、スタックオーバーフローなど、調査結果であるリンクを提示した。

あとは、コードリーディングをしていて、本来必要でない実行権限が与えられているファイルを見つけたので、0755から0644に変更するだけのPRを送った。

[Change permission from 0755 to 0644 by yhirano55 · Pull Request #2536 · tootsuite/mastodon](https://github.com/tootsuite/mastodon/pull/2536)

どういうわけか、今月はこういうPRが多い。**パーミッション警察**よろしく、床に落ちているゴミを拾う感覚で、各所にパッチを送った（まだマージはされていない）。

[Change permission to 0755 from 0644 by yhirano55 · Pull Request #1279 · puma/puma](https://github.com/puma/puma/pull/1279)

[change permission 0755 => 0644 by yhirano55 · Pull Request #456 · ctran/annotate_models](https://github.com/ctran/annotate_models/pull/456)

他には、[ctran/annotate_models](https://github.com/ctran/annotate_models)で定義されたモジュールで、無効な定義を発見したので、PRを送った。

[Modify private class methods by yhirano55 · Pull Request #455 · ctran/annotate_models](https://github.com/ctran/annotate_models/pull/455)

過去には、下記はprivateになりませんよ、という事案だ。

```ruby
module FooBar
  private
  def self.hello
    "hello"
  end
end
```

解決としては、`private_class_method` で定義するか、特異クラス内で `private` キーワードを使うかだが、メソッド数が多いのもあって、冗長になるのを避けるため後者を選んだ。

引き続き、オープンソースソフトウェアに対して、機能改善やパッチなどの貢献を行っていきたい。
