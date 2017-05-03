+++
date = "2017-05-04T00:47:31+09:00"
tags = ["Rubocop", "OSS"]
title = "RubocopでCopを追加する（実装編）"
description = "Copを実装してプルリクエストを送るまで"

+++

今回、実装したかったCopは、 `bin/**/*` または `exe/**/*` のファイルが適切に実行権限が与えられていることを保証する、という要件だった。そのため、本来、Copを追加する際は、ASTのnodeを比較する必要があるのだが、今回の要件だと全くそれは必要なかった。

まず、コードの生成から始まるのだが、[準備編](/2017/05/03/how-to-create-new-cop-01/)に記載した通り、 `rake new_cop[Category/Name]` で生成する。名前空間のどれを選択するか、または、Copの命名は非常に迷うところだが、Layout/Rails/Performance/Style/Lint/Bundlerのどれかなので、名前空間はおおよそ検討がつく。名前については、素直に、そのCopの本質が伝わればよさそうだ。

長々と説明するよりも、実際のPRを見てもらった方が話は早い（まだマージされたわけではないが）。

[Add new Style/ScriptPermission cop by yhirano55 · Pull Request #4342 · bbatsov/rubocop](https://github.com/bbatsov/rubocop/pull/4342)

Copの実装は以下の通り。 `File::stat` でファイルの実行権限をチェックして、問題があれば、 `add_offense` に該当範囲とメッセージを渡すのみ。AST利用する場合も、比較して、違反しているコードがあれば、add_offenseするのは同じだ。

```ruby
# frozen_string_literal: true

module RuboCop
  module Cop
    module Style
      # This cop checks whether the file permission is executable.
      class ScriptPermission < Cop
        MSG = 'Set file permission with `chmod +x`.'.freeze

        def investigate(processed_source)
          return if executable?(processed_source)
          range = source_range(processed_source.buffer, 1, 0)
          add_offense(nil, range, MSG)
        end

        private

        def executable?(processed_source)
          # Returns true if stat is executable or if the operating system
          # doesn't distinguish executable files from nonexecutable files.
          # See at: https://github.com/ruby/ruby/blob/ruby_2_4/file.c#L5362
          File.stat(processed_source.buffer.name).executable?
        end
      end
    end
  end
end
```

テストを実行してもよいが、最初は任意のRailsプロジェクトで、ローカルのGemを `path` オプションを利用してインストールして、試すのがよい気がした。テストする前に、初期設定を定義した。

```
Style/ScriptPermission:
  Description: 'Set file permission with `chmod +x`.'
  Include:
    - bin/**/*
    - exe/**/*
  Enabled: true
```

ここまで準備できたら、何度か実装して、意図した通りに動くことを確認する。それが完了したら、テストを書いて、コード生成時にTODOの記載があった、`rake generate_cops_documentation` を実行すると、ドキュメント用のmarkdownが自動生成される。

これらを1つのコミットにまとめて、PRを作成する。Copの命名やコメントに迷いがなければ、PRのSubject/Descriptionは容易に書くことができる。Rubocopは、PRを作成するときのテンプレートに、チェックリストが含まれているので、それを残しつつ、PRを作成する。

変更履歴は、PRを一旦作成しないとIDが分からないので、作成後に他の体裁に合わせて、更新し、 `--amend` する（変更履歴が適切かどうかも、ちゃんと自動テストしている）。なお、PR作成後、Twitterのタイムラインを眺めていると、問題点の指摘などがListenできるので、imoな指摘は修正しておくと気が利いているかもしれない。具体的な以下のようなTweetだ。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Windows だとどうなるんだろう <a href="https://t.co/4UhCWUBV9u">https://t.co/4UhCWUBV9u</a></p>&mdash; Pocke (@p_ck_) <a href="https://twitter.com/p_ck_/status/859644291102199809">2017年5月3日</a></blockquote>

たしかに、ファイルシステムはOSに依存しているので、動作しない環境もありそうだ。**とりあえず、自分が欲しい機能を作成し、mergeするかどうかは作者に委ねよう**くらいの気持ちではあったが、何らかのエクスキューズは、PRに含めておいた方が良さそうな気がした。そんなところで、次の貴重な情報が流れてきたので、実装を確認し、コメントを追加した（感謝）。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Windows だと FIle::Stat#executable? は常に true が返りそう <a href="https://t.co/7HCC3wWaHY">https://t.co/7HCC3wWaHY</a></p>&mdash; Pocke (@p_ck_) <a href="https://twitter.com/p_ck_/status/859646820221321216">2017年5月3日</a></blockquote>

今回作成したCopが採用されるかどうかは、現状では分からないが、大体の流れは掴めたので、また生産性を上げるアイディアが湧いたら、Rubocopに貢献したいと思う。なお、ASTについては、次のエントリがたいへん参考になりそうだ。

- [RuboCopのASTの書き方の例 - koicの日記](http://koic.hatenablog.com/entry/2017/01/05/000000)
- [RuboCop の Cop の実装について - Qiita](http://qiita.com/pocke/items/1ac1fbabb861f08c96e2)
- [Vim script の Lint 作者による誰得 Lint デザインパターン - Qiita](http://qiita.com/Kuniwak/items/d6a2d22711e4d7856edd)

<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
