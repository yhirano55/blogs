+++
date = "2017-05-22T22:16:02+09:00"
title = "Inherited Resources 再入門"
description = "不明点が多かったのでコードを読みながら"
tags = ["Rubygems", "ActiveAdmin"]

+++

[Inherited Resources](https://github.com/activeadmin/inherited_resources)は、[activeadmin](https://github.com/activeadmin/activeadmin)で使われていて、CRUDの実装を楽にしてくれるGemだ。しかし、以前より作者は保守に熱心ではなく、つい最近、管理がactiveadminに移った。READMEの冒頭にもこんな風に書かれている。

> Inherited Resources is no longer actively maintained by the original author and has been transferred to the ActiveAdmin organization for maintenance. New feature requests are not encouraged.（継承されたリソースは元の作者によってもはや積極的に維持されず、メンテナンスのためにActiveAdminのオーガニゼーションに移管されました。そういうわけで新機能のリクエストを推奨していません）

activeadminが依存しているのでメンテナンスされている、そんな雰囲気が漂っている。とはいえ、プロジェクトで利用されていることも稀にあるため、使い方に癖があると以前から感じていたので、コードを読んでみたいと思った。きっと大多数の方からしたら、この記事は何の役にも立たないだろう。

### コンセプト

> Inherited Resources speeds up development by making your controllers inherit all restful actions so you just have to focus on what is important. It makes your controllers more powerful and cleaner at the same time.（Inherited Resourcesは、すべてのコントローラが安定したアクションを継承するように開発をスピードアップするので、重要なことに集中するだけで済みます。同時に、コントローラをより強力でクリーンにすることができます）

### 依存するGem

respondersとhas_scopeというGemに依存している。respondersはコントローラのレスポンス処理をDRYに書けるGemで、has_scopeはコントローラにリソースのscopeを生やして、名前付きスコープにマッピングするGemだ。いずれもコントローラを拡張するGemで、Railsのコミッターである[Rafael França](https://github.com/rafaelfranca)氏らがメンテナンスしているようだ。

### 基本的な使い方

基本的な使い方は、`InheritedResources::Base` を継承すると、全アクションが継承される。古い動画となるが、Rails Castsの説明が最もわかりやすかったので、まずはザッと確認してみるとよい。

<iframe width="560" height="315" src="https://www.youtube.com/embed/YZs7cyPBtSc" frameborder="0" allowfullscreen></iframe>

動画にあるように、scaffoldを生成して、その後、`InheritedResources::Base` を継承させ、内部コードを削除してみる。ただ、StrongParameters対応もしないといけないので、実際は次のようになる。

```ruby
class ArticlesController < InheritedResources::Base
  private

  def permitted_params
    params.require(:article).permit(:title, :content)
  end
end
```

ただ、この基本的な使い方だけで使用されるケースは殆どない。

### アクションを上書きする

そのまま利用していると、N+1が発生したりと、結局はアクションに手を入れることとなる。
方法はいくつかあるのだが、最も単純な方法は、そのアクションを上書きすることだ。

```ruby
class ProjectsController < InheritedResources::Base
  def index
    @projects = Project.includes(:leader).page(params[:page])
    index!
  end
end
```

`index!` メソッドは、 [/lib/inherited_resources/actions.rb](https://github.com/activeadmin/inherited_resources/blob/master/lib/inherited_resources/actions.rb)で定義されている。

```ruby
module InheritedResources
  # Holds all default actions for InheritedResouces.
  module Actions
    # GET /resources
    def index(options={}, &block)
      respond_with(*with_chain(collection), options, &block)
    end
    alias :index! :index
  end
end
```

InheritedResourcesには、リソースを表現するヘルパーメソッドが定義されていて、それらを理解しておくと調整しやすい。

ここで、Gemのコードレイアウトをざっと確認しておく。

```
app/
└── controllers
    └── inherited_resources
        └── base.rb # 各コントローラで継承することになる親クラス
lib/
├── generators # scaffoldのコントローラをすげ替える
│   └── rails
│       ├── USAGE
│       ├── inherited_resources_controller_generator.rb
│       └── templates
│           └── controller.rb
├── inherited_resources
│   ├── actions.rb             # アクションを定義
│   ├── base_helpers.rb        # collection/resourceなどを定義
│   ├── belongs_to_helpers.rb  # belongs_toの実装
│   ├── blank_slate.rb         # blank slate
│   ├── class_methods.rb       # DSLが定義されている(belongs_to, actionsなど)
│   ├── dsl.rb                 # アクションを記述するためのDSLを生やす
│   ├── engine.rb              # Railtie
│   ├── polymorphic_helpers.rb # ポリモーフィック関連に対応するためのヘルパー
│   ├── responder.rb           # responderの定義
│   ├── shallow_helpers.rb     # shallowに対応するためのヘルパー
│   ├── singleton_helpers.rb   # singletonなリソースを定義するためのヘルパー
│   ├── url_helpers.rb         # urlヘルパー
│   └── version.rb             # バージョン
└── inherited_resources.rb      # Gemのエントリーポイント
```

### 覚えるべきヘルパーメソッド

`InheritedResources::Base` のコードを見ると、次のようにhelperメソッドが定義されている。

```ruby
module InheritedResources
  # = Base
  #
  # This is the base class that holds all actions. If you see the code for each
  # action, they are quite similar to Rails default scaffold.
  #
  # To change your base behavior, you can overwrite your actions and call super,
  # call <tt>default</tt> class method, call <<tt>actions</tt> class method
  # or overwrite some helpers in the base_helpers.rb file.
  #
  class Base < ::ApplicationController
    # Overwrite inherit_resources to add specific InheritedResources behavior.
    def self.inherit_resources(base)
      base.class_eval do
        include InheritedResources::Actions
        include InheritedResources::BaseHelpers
        extend  InheritedResources::ClassMethods
        extend  InheritedResources::UrlHelpers

        # Add at least :html mime type
        respond_to :html if self.mimes_for_respond_to.empty?
        self.responder = InheritedResources::Responder

        helper_method :resource, :collection, :resource_class, :association_chain,
                      :resource_instance_name, :resource_collection_name,
                      :resource_url, :resource_path,
                      :collection_url, :collection_path,
                      :new_resource_url, :new_resource_path,
                      :edit_resource_url, :edit_resource_path,
                      :parent_url, :parent_path,
                      :smart_resource_url, :smart_collection_url

        self.class_attribute :resource_class, :instance_writer => false unless self.respond_to? :resource_class
        self.class_attribute :parents_symbols,  :resources_configuration, :instance_writer => false

        protected :resource_class, :parents_symbols, :resources_configuration,
          :resource_class?, :parents_symbols?, :resources_configuration?
      end
    end

    inherit_resources(self)
  end
end
```

重要なメソッドは、resource, collection でそれさえ覚えれば、他はある程度使いこなせるはずだ。

それぞれ下記のように実装されている。

```ruby
module InheritedResources
  module BaseHelpers
    protected
    # This is how the collection is loaded.
    #
    # You might want to overwrite this method if you want to add pagination
    # for example. When you do that, don't forget to cache the result in an
    # instance_variable:
    #
    #   def collection
    #     @projects ||= end_of_association_chain.paginate(params[:page]).all
    #   end
    #
    def collection
      get_collection_ivar || begin
        c = end_of_association_chain
        if defined?(ActiveRecord::DeprecatedFinders)
          # ActiveRecord::Base#scoped and ActiveRecord::Relation#all
          # are deprecated in Rails 4.  If it's a relation just use
          # it, otherwise use .all to get a relation.
          set_collection_ivar(c.is_a?(ActiveRecord::Relation) ? c : c.all)
        else
          set_collection_ivar(c.respond_to?(:scoped) ? c.scoped : c.all)
        end
      end
    end

    # This is how the resource is loaded.
    #
    # You might want to overwrite this method when you are using permalink.
    # When you do that, don't forget to cache the result in an
    # instance_variable:
    #
    #   def resource
    #     @project ||= end_of_association_chain.find_by_permalink!(params[:id])
    #   end
    #
    # You also might want to add the exclamation mark at the end of the method
    # because it will raise a 404 if nothing can be found. Otherwise it will
    # probably render a 500 error message.
    #
    def resource
      get_resource_ivar || set_resource_ivar(end_of_association_chain.send(method_for_find, params[:id]))
    end
  end
end
```

コメントで上書きするときの例が書いてあるが、どちらもリソースやコレクションを表現するインスタンス変数がセットされていれば、セットされることはないので、InheritedResourcesの挙動を抑えたいのであれば、Projectモデルで、コレクションなら `@projects` を、リソースなら `@project` を定義すればよい。

それでは両者で呼ばれている、`end_of_association_chain` とは何か? 見るからに `ActiveRecord::Relation` であることは察しがつくが、実装は次のようになっている（同じくBaseHelpersのコードより）

```ruby
# This methods gets your begin_of_association_chain, join it with your
# parents chain and returns the scoped association.
def end_of_association_chain #:nodoc:
  if chain = association_chain.last
    if method_for_association_chain
      apply_scopes_if_available(chain.send(method_for_association_chain))
    else
      # This only happens when we specify begin_of_association_chain in
      # a singleton controller without parents. In this case, the chain
      # is exactly the begin_of_association_chain which is already an
      # instance and then not scopable.
      chain
    end
  else
    apply_scopes_if_available(resource_class)
  end
end
```

条件のブランチの上で `association_chain.last` を評価しているが、`begin_of_association_chain` などを定義していない場合は大抵nilなので、`apply_scopes_if_available(resource_class)` が評価される。この実装も覗いてみる。

```ruby
# Hook to apply scopes. By default returns only the target_object given.
# It's extend by HasScopeHelpers.
#
def apply_scopes_if_available(target_object) #:nodoc:
  respond_to?(:apply_scopes, true) ? apply_scopes(target_object) : target_object
end
```

これは has_scope が提供するscopeを定義していなければ、その対象となるオブジェクト（つまりresouce_class）を返す。よって、次のように上書きした場合も結局のところは、`Project.paginate(params[:page]).all` を評価しているのと変わらない。

```ruby
def collection
  @projects ||= end_of_association_chain.paginate(params[:page]).all
end
```

### まとめ

InheritedResourcesは、複雑なことをやっているようだが、コードを見れば、そこまで癖がないことがわかる。belongs_toやポリモーフィック関連などでも利用できるが、今のところ、使われるケースと遭遇していないため、今回の入門記事では割愛する。とりあえずは、collection, resourceのメソッドを上書きする方法と、アクションを上書きする方法さえ分かれば、ストレスを感じずに使いこなせるはず。

著者は、punditやbankenを使うのに、authorize!をどこで実行するかでコードを読み始めたが、activeadminでも使われているので、activeadminを利用する場合でも読んでおいて損はないように感じた。
