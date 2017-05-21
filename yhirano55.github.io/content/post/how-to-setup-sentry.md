+++
title = "Sentryを手元で動かす"
description = "イベントログ収集ツール Sentryを手元で動かすまで"
tags = ["Sentry", "Docker"]
date = "2017-05-21T22:35:12+09:00"

+++

イベントログを収集するアラートツールといえば、これまで[Airbrake](https://airbrake.io/)（[Errbit](https://github.com/errbit/errbit)）を使うことが多かったが、[Sentry](https://sentry.io/welcome/)が良いという話を聞いて、既に導入はしていたが、細かく動作チェックしたくなった。

Sentryは、SaaSで提供されているが、[GitHub](https://github.com/getsentry/sentry)でコードも公開されている（Python製）。それと同時にDockerコンテナで動作させる方法も手厚く書かれているので、それに則って、セットアップしていく。なお、本記事の環境は、Docker for Macを利用している。

### Sentry環境の構築

Redisコンテナを起動する。

    $ docker run -d --name sentry-redis redis

Postgresコンテナを起動する。

    $ docker run -d --name sentry-postgres -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=sentry postgres

すべてのSentryコンテナで共有する秘密鍵を生成する（秘密鍵は以降のステップで利用するのでメモしておく）

    $ docker run --rm sentry config generate-secret-key

もしも新規のDBならば、アップグレードする。途中で、新規アカウントを作るか聞かれるので、Emailとパスワードを入力し、SuperUserとして作成しておくとよい。

    $ docker run -it --rm -e SENTRY_SECRET_KEY='<秘密鍵>' --link sentry-postgres:postgres --link sentry-redis:redis sentry upgrade

Sentryサーバーコンテナを起動する。

    $ docker run -d --name my-sentry -e SENTRY_SECRET_KEY='<秘密鍵>' --link sentry-redis:redis --link sentry-postgres:postgres sentry

CronとWorkerのコンテナを起動する。

    $ docker run -d --name sentry-cron -e SENTRY_SECRET_KEY='<秘密鍵>' --link sentry-postgres:postgres --link sentry-redis:redis sentry run cron

    $ docker run -d --name sentry-worker-1 -e SENTRY_SECRET_KEY='<秘密鍵>' --link sentry-postgres:postgres --link sentry-redis:redis sentry run worker

Dockerコンテナ内部では9000番ポートで動作しているので、Macの任意のポート（今回は8080番）にマッピングする。こちらは、Kitematicを使うとGUIで簡単に変更できる。`my-sentry` コンテナを選択し、SettingsのPortsを表示。`DOCKER PORT`が9000と表示されているので、その隣の`PUBLISHED IP:PORT`を、`localhost:8080` と変更して、保存する。

![](https://cloud.githubusercontent.com/assets/15371677/26284625/c22f9e6e-3e7a-11e7-9ffb-dacc391febef.png)

`http://localhost:8080` にアクセスすると、ログイン画面が表示される。

![](https://cloud.githubusercontent.com/assets/15371677/26284599/56f70c4a-3e7a-11e7-9158-8b15170ab8e4.png)

### イベントログを送る

せっかく起動させたので、Railsアプリケーションからイベントログを送ってみる。

`New Project` からプロジェクトを作成し、Railsを選択すると必要なステップが表示される。

まずは `gem "sentry-raven"` を追加してbundle。その後、`config/initializers/sentry.rb` を追加。

```ruby
Raven.configure do |config|
  config.dsn = 'http://xxxxx:xxxxx@0.0.0.0:8080/2'
  config.sanitize_fields = Rails.application.config.filter_parameters.map(&:to_s)
end
```

`app/controllers/application_controller.rb` を修正する。今回は認証を提供していないアプリケーションのため、ユーザーコンテキストは送っていない。

```ruby
class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception

  before_action :set_raven_context

  private

  def set_raven_context
    # Raven.user_context(id: session[:current_user_id]) # or anything else in session
    Raven.extra_context(params: params.to_unsafe_h, url: request.url)
  end
end
```

今回は、development環境からイベントログを送るので、 `config.consider_all_requests_local = false` にした後、適当な箇所で例外を発生させる。Sentry側を見ると、通知が送られてきているのが確認できる。

![](https://cloud.githubusercontent.com/assets/15371677/26284830/32edf4b2-3e7f-11e7-8468-ed83325cb356.png)

エラーが発生した部分も非常に分かりやすい。

![](https://cloud.githubusercontent.com/assets/15371677/26284844/5d1ee430-3e7f-11e7-9bd7-8498e73c4458.png)

### JavaScriptのエラーログを収集する

`http://0.0.0.0:8080/sentry/<project>/settings/install/javascript/` にアクセスすると、インストール方法が記載されている。

まずはapplication.jsの前に以下スクリプトをインクルードして、

```html
<script src="https://cdn.ravenjs.com/3.14.2/raven.min.js" crossorigin="anonymous"></script>
```

下記を設定する。

```javascript
Raven.config('http://xxxxx@0.0.0.0:8080/2').install()
```

適当なエラーを発生させると、JSのエラーが送られていることが確認できる。

![](https://cloud.githubusercontent.com/assets/15371677/26285001/d2138e42-3e81-11e7-9b38-e3296af934fc.png)

なお、CDNを使わずにnpmパッケージからインストール場合も[ドキュメント](https://docs.sentry.io/clients/javascript/install/)に記載があるので、そちらを参考にするとよい。
