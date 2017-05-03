+++
date = "2017-05-03T07:35:55+09:00"
title = "RubocopでCopを追加する（準備編）"
description = "Copを追加するのに前提となる情報を調べた"
tags = ["Rubocop", "OSS"]

+++

RubocopにCopを追加すると、それを利用するすべてのプロジェクトの品質向上に貢献できて、とても良い。とはいえ、追加するにあたって、何をすればいいのか分からないことも多いと思うので、緒を提示できればと思い、本記事を書くに至った。

Rubocopに限らず、OSSで新規実装をする場合のアプローチとして、試行錯誤する前にやることは、ドキュメントとマージされたPRに目を通すことから始めたい。ということで、まずは情報収集から。

### 情報収集

まず最初に目を通すべきは、[CONTRIBUTING.md](https://github.com/bbatsov/rubocop/blob/master/CONTRIBUTING.md)だ。ここには、Issueに報告するとき、または、PRを送るときのガイドラインが制定されている。今回は実装なので、[Pull Request](https://github.com/bbatsov/rubocop/blob/master/CONTRIBUTING.md#pull-requests)に目を通すと良い。簡単に和訳すると、次のような内容が記載してある。

> - 『[GitHubでオープンソースプロジェクトに正しく貢献する方法](http://gun.io/blog/how-to-github-fork-branch-and-pull-request)』を読んでください。
> - プロジェクトをフォークしてください。
> - あとでPRを修正しやすくなりますので、必要に応じてブランチを切ってください。
> - [適切なコミットメッセージ](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)を書いてください。
> - プロジェクトのコーディング規約に沿ってください。
> - 必要なコミットを積んでください。
> - Issueに関連する場合は、`[Fix #github-issue-number]` をコミットメッセージの接頭辞に追加してください。
> - テストを書いてください。これは重要なことで、意図せず将来のバージョンで壊すことがなくなります。
> - [変更履歴のフォーマット](https://github.com/bbatsov/rubocop/blob/master/CONTRIBUTING.md#changelog-entry-format)を見て、履歴を追加してください。
> - Rakefile, バージョン, 履歴を乱雑にしないようにしてください。もし独自のバージョンなどを定義する必要があれば、コミットを分けてください。
> - rbxやjrubyを含めて、すべてのテストスイートをパスさせてください。あとRubocop違反も引き起こさないように。
> - 関連コミットはスカッシュしてください。
> - 正しい主題と説明文のPRを作成してください。

他のプロジェクトと大きく異なる手順はないので、次に[マージされたPR](https://github.com/bbatsov/rubocop/pulls?utf8=%E2%9C%93&q=is%3Apr%20is%3Aclosed%20is%3Amerged)を眺める。以下あたりが、関連しているようだ。

[[Fix #4236] Add new Rails/ApplicationJob and Rails/ApplicationRecord cops by tjwp · Pull Request #4279 · bbatsov/rubocop](https://github.com/bbatsov/rubocop/pull/4279)

接頭辞にIssueのリンクがあるので、それも眺めてみる。

> ActiveRecord :: Baseではなく、ApplicationRecordからすべてのモデルを継承するようにすることは可能でしょうか?

という提案に対して、[Salsify社](https://www.salsify.com/)の[@tjwp](https://github.com/tjwp)氏が、自社で利用している[カスタムCop](https://github.com/salsify/salsify_rubocop/blob/master/lib/rubocop/cop/salsify/rails_application_record.rb)を提示すると、作者から上流（Rubocop本体）に流してくれと要請され、PRに至る、ということが分かる。

このPRでは、親クラスを強制する `enforce_superclass` というミックスインが追加され、`Rails/ApplicationJob` と `Rails/ApplicationRecord` が規約として追加されている。specを眺めると、テストの仕方のお手本が分かる。コミットが1つにまとまっているので、もう少し分割されていると作業のプロセスが分かるのでありがたいが、このPRを真似る方向でよさそうだ（他PRを見る限り、スカッシュするのが作法っぽい）。

あと、実際のCopを眺めて、その履歴を探ると、Cop追加時のPRまで辿れるので、他にも情報が必要であれば、そのようにして収集すると良い。いろいろ探っているなかで、`Rails/ReversibleMigration` が作成されたPRを見つけたので、作者とのやり取りを含めて、とても参考になりそうだ。

[Add Rails/ReversibleMigration cop by sue445 · Pull Request #3854 · bbatsov/rubocop](https://github.com/bbatsov/rubocop/pull/3854)

### 環境構築

[bbatsov/rubocop](https://github.com/bbatsov/rubocop)からフォークして、手元にcloneし、`bundle install` しよう。その後、何も変更しない状態で `bundle exec rake` を実行して、すべてのテストが通ることを確認することを確かめよう（仮に失敗するテストがある場合、大元のブランチでも落ちているか、フォーク元から確認できる）。

### 実行できるタスクを確認

コードを追加する前に、Rakeタスクを確認すると、 `rake new_cop` というジェネレータが存在するのが分かる。

```
% bundle exec rake -T
rake ascii_spec                           # Run RSpec code examples
rake bench_cop[cop,srcpath,times]         # Benchmark a cop on given source file/dir
rake build                                # Build rubocop-0.48.1.gem into the pkg directory
rake clean                                # Remove any temporary products
rake clobber                              # Remove any generated files
rake coverage                             # Run RSpec with code coverage
rake generate_cops_documentation          # Generate docs of all cops departments
rake install                              # Build and install rubocop-0.48.1.gem into system gems
rake install:local                        # Build and install rubocop-0.48.1.gem into system gems without network access
rake internal_investigation               # Run RuboCop over itself
rake internal_investigation:auto_correct  # Auto-correct RuboCop offenses
rake new_cop[cop]                         # Generate a new cop template
rake release[remote]                      # Create tag v0.48.1 and build and push rubocop-0.48.1.gem to Rubygems
rake repl                                 # Open a REPL for experimentation
rake spec                                 # Run RSpec code examples
rake yard                                 # Generate YARD Documentation
```

### Copを生成する

[ドキュメント](https://rubocop.readthedocs.io/en/latest/development/)を見ると、タスク名の後に `[Category/Name]` と記載する必要がある（zshを利用しているため、`[]` の前にバックスラッシュを足す必要があった）。

```sh
% bundle exec rake new_cop\[Rails/Foo\]
created
- lib/rubocop/cop/rails/foo.rb
- spec/rubocop/cop/rails/foo_spec.rb

Do 4 steps
- Add an entry to `New feature` section in CHANGELOG.md
  - e.g. Add new `Foo` cop. ([@your_id][])
- Add `require 'rubocop/cop/rails/foo'` into lib/rubocop.rb
- Add an entry into config/enabled.yml or config/disabled.yml
- Implement a new cop to the generated file!
```

### 生成コードを確認

生成された `lib/rubocop/cop/rails/foo.rb` は次の通り。コメントの書き方から、実装方法まで示唆してあり、手厚さを感じる。

```ruby
# frozen_string_literal: true

# TODO: when finished, run `rake generate_cops_documentation` to update the docs
module RuboCop
  module Cop
    module Rails
      # TODO: Write cop description and example of bad / good code.
      #
      # @example
      #   # bad
      #   bad_method()
      #
      #   # bad
      #   bad_method(args)
      #
      #   # good
      #   good_method()
      #
      #   # good
      #   good_method(args)
      class Foo < Cop
        # TODO: Implement the cop into here.
        #
        # In many cases, you can use a node matcher for matching node pattern.
        # See. https://github.com/bbatsov/rubocop/blob/master/lib/rubocop/node_pattern.rb
        #
        # For example
        MSG = 'Message of Foo'.freeze

        def_node_matcher :bad_method?, <<-PATTERN
          (send nil :bad_method ...)
        PATTERN

        def on_send(node)
          return unless bad_method?(node)
          add_offense(node, :expression)
        end
      end
    end
  end
end
```

specは次の通り。

```ruby
# frozen_string_literal: true

describe RuboCop::Cop::Rails::Foo do
  let(:config) { RuboCop::Config.new }
  subject(:cop) { described_class.new(config) }

  # TODO: Write test code
  #
  # For example
  it 'registers an offense for offending code' do
    inspect_source(cop, 'bad_method')
    expect(cop.offenses.size).to eq(1)
    expect(cop.messages)
      .to eq(['Message of Foo'])
  end

  it 'accepts' do
    inspect_source(cop, 'good_method')
    expect(cop.offenses).to be_empty
  end
end
```

これで一通りの準備が整ったので、あとは実装するだけとなった。
