+++
date = "2017-04-08T23:25:40+09:00"
title = "form modelの変遷"
description = "Railsにおけるform modelの変遷"
tags = ["rails", "active_type"]

+++

ARモデルに紐づかないフォーム用のモデルを実装するとき、form model（form objectとも言う）を利用することは少なくない。

### form modelの基本

基本はActiveModelを使った実装だ（Structで実装するパターンもあるが、こちらの方が楽だ）。

```ruby
class Signup
  include ActiveModel::Model

  attr_accessor :email, :password, :terms

  with_options presence: true do
    validates :email, email: true
    validates :password, confirmation: true
  end

  validates :terms, acceptance: true

  def persisted?
    false
  end

  def signup
    if valid?
      persist!
      true
    else
      false
    end
  end

  private

  def persist!
    ActiveRecord::Base.transaction do
      # do something..
    end
  end
end
```

### 初期値を定義したい場合

[solnic/virtus](https://github.com/solnic/virtus)を使うと初期値が宣言的に実装できる。

```ruby
class Signup
  include Virtus.model
  include ActiveModel::Model

  attribute :email, String, default: "admin@example.com"
  attribute :password, String
end
```

gemに依存せず、attributeをセットするタイミングで初期値を渡すこともできるが、やや煩雑か。

```ruby
def email=(value)
  @email = value.presence || "admin@example.com"
end
```

### ActiveTypeを使った実装

[makandra/active_type](https://github.com/makandra/active_type)を使うと、 `ActiveModel::Model` をインクルードせず、form objectが定義でき、また、Virtusのように、attributeを列挙する際に初期値を定義できる

```ruby
class Signup < ActiveType::Object
  # ..中略..
end
```

親クラスである `ActiveType::Object` のコードを見ると、 `ActiveRecord::Base` を継承している。コネクションについては、 `ActiveType::NoTable` でDummpyを割り当てているので、DBとの接続を行わないARモデルとなる。

このgemの優れている点は、ARモデルのコンテキストに依拠したクラスが、ARで実装するときと同様に実装可能な点ではあるが、form modelとしても先の実装よりも容易で、優れているのではないか。

いずれ試したら、また記事にしたい。
