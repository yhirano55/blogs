+++
description = "勉強会に参加してきた話"
tags = ["golang", "study", "event"]
date = "2017-04-27T19:26:05+09:00"
title = "golang.tokyo #5"

+++

2017年4月27日（木）に行われた、golang.tokyo #5に参加してきました。今回は**ブログ参加枠**でエントリーしたので、参加してきた記事を書こうと思います。イベントの詳細は、[connpass](https://golangtokyo.connpass.com/event/53209/)に記載がある通り、さかなのオブジェで話題の株式会社メルカリ様で行われました。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">メルカリさんのオフィスにお邪魔してmercari×サイボウズ主催 オフィスdeナイト<br>テーマは「当たり前を打ち破る、総務・コーポレート部門の企画力」 <a href="https://t.co/wcMzz4qs7n">pic.twitter.com/wcMzz4qs7n</a></p>&mdash; kondo　yukiko (@yukko_chiru) <a href="https://twitter.com/yukko_chiru/status/836193039399858176">2017年2月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

※上記引用は本勉強会とは関係なし。バランスシートの借方が充実してそうな雰囲気！

### 会場・雰囲気

- メルカリさんのイベントスペース広い。
- 140名くらい収容できそう。
- 18Fの受付でドリンク類をもらい、軽食が並んでいる。
- この配布方法なら、人手が少なくても大丈夫なので真似したい。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">メルカリさんありがとうございます！ <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a> @ 株式会社メルカリ <a href="https://t.co/2AoTJ8OES9">https://t.co/2AoTJ8OES9</a></p>&mdash; Kiyoshi Nomo 半泣き黒猫団員 (@kysnm) <a href="https://twitter.com/kysnm/status/857544749464436736">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">サンドイッチタスクを必死に消化してるけど、僕のスケーラビリティでは勝てなさそう <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; 真 (@sinmetal) <a href="https://twitter.com/sinmetal/status/857546696821137408">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### 勉強会のコンセプト

- GCPUG Tokyoさんとの共同開催。
- とはいえ、通常通り、登壇x2とLTの構成。
- いつもは倍率高いが、今日は**定員割れ**。
- 連休前だからかも。
- 初心者割合はぼちぼち高い。

### 登壇1. GO*GAE

※スライド公開待ち※

株式会社カブクの吉海将太さんによる発表。インフラ構築からサーバサイドまで携わっているそう。Pepperアプリの開発もしていたとか。以下登壇内容。

- GAEとは?
- Goだと高速に開発できる
- App Engineを使いたくなる発表
- GAEの種類
- GoのVersionは1.6
- スピンアップが早いのが特徴
- 好きなdocker imageを使える
- GCEのVMが起ち上がる
- SE(Standard Environment)とFE(Flexible Environment)どっちがいいの?（SEがよい）
- バイナリ使いたいならFEしかない。
- オートスケール/ロギング/モニタリング/バージョニングがSEの機能
- ベータ/好きなライブラリをインストールできる/何でも動く/CGOが動くがFEの機能
- `google.golang.org/appengine` パッケージが提供されている
- Google Cloud Datastore APIを使うためのパッケージが、datastore package。使ってみると、面倒なので、ラッパーであるgoonを使うのが良い（メリットはコード量が少なくなること）。
- 非同期処理を行うためのパッケージが、taskqueue。
- Golangは、ログレベルがない！（SEのlogパッケージを使えば、ログレベルが提供される）
- aetestというテストするためのモジュールがある。HTTP通信のテストをする際に利用するらしい。
- FEは標準で1.8が使える。自前でDockerImageを用意すれば、好きなバージョンが使える。
- Linuxの64bitのアプリケーションなら何でも動く。へぇぇ。
- SE用のGAEのパッケージは使えない。ロギングはログレベルなし。標準出力・標準エラーでログを出そう、という点がFEのデメリット。
- 自前のDocker Imageならば、localとproductionの差異がない。
- デプロイするときに、imageを事前にpushする必要がある（gcr.ioなど?）
- 構成例の話。SEとFEを使っている。
- SEは最大60secでタイムアウトする。
- TaskQueueのワーカーは、結局SEで実行するので、タイムアウト制限の対象となる。
- TaskQueueはFEで利用できない。そのため、SEでリクエストを受けて、FEで解析する構成となった。
- ディスパッチで割り振れる。簡単なYAMLの設定で振り分けられる感じ。
- つらかったことは、Go分からない（普段はPython）。モックする方法がわかりにくい（Interface定義する必要あるの?）。SEはコンテキストを渡す必要があるので面倒。
- よかったことは、gofmtいいね！型定義いいね！goroutineいいね！
- 結論としては、GoでWebアプリ作りたいならGAEは良い選択肢。GO * GAEは他の言語に比べて最速。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">TaskQueueは最高に便利なのでGAE使うなら絶対使ったほうが幸せになります <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; わかめ@TypeScript味 (@vvakame) <a href="https://twitter.com/vvakame/status/857548080140308481">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">appengineのドキュメントに、オートスケールの場合、requestで実行された場合には60秒、TQ経由だと最大10分、マニュアルスケールの場合には24時間とあるけど、試したこと無いなぁ。 <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; kiyo (@chidakiyo) <a href="https://twitter.com/chidakiyo/status/857550295655632898">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### 登壇2. GAE/GOの勘どころ

<script async class="speakerdeck-embed" data-id="3eed20636b084a98adc1d918d2d2c2dc" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

株式会社ソウゾウの主森理さんによる発表。登壇1と同じ話になるという事故が発生した。

- GAEはソウゾウに入ってから始めた。
- awesome goにライブラリが2つ載っている（PR出せば載せてくれるらしい）
- GAEは独自のルールが多い。
- FEを使う場面は、ネットワーク起因の要求がある場合と一時的なディスク書き込みが必要な場合（SEだとtmpfileが作れない）。
- SEはGo1.6.4。unsafe packageは基本的に使えない。build appengineの認知が低い。
- contextがないと何もできないのがSE。
- もうすぐ1.8がSEに来るかも。
- Google社内は、SEで1.8が使えている。
- 4月末に使えるようになるかも。
- DBどうするの?（Cloud Datastoreを使う。BigTableベースのNoSQLだけどmongoぽく使うと死ぬ。Cloud SQLも使える）
- 公式のdatastoreパッケージが使わない方がよい。必ずwrapして使った方がよい。NoSQLなので、自由にスキーマが組めるので危険。goon使うのがいい（中身を確認したうえで）。
- Google Cloud Datastoreデザインパターンはなかなか難しく、設計力が要求されるとのこと。
- SEだとdep, glide, gvt使えないだとー!!!!（vendorディレクトリも使えない）
- Configurationは、Twelve-Factor Appにあるように、環境変数使うべし。
- Testはdirenv使おう

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">環境変数に関してはconfiguration用のrepository作ってやってる。 <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; timakin (@__timakin__) <a href="https://twitter.com/__timakin__/status/857554984312815617">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">vendering普通にdepつかってやってる <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; timakin (@__timakin__) <a href="https://twitter.com/__timakin__/status/857557684035264514">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### LT1. Datastore/Go のデータ設計と struct の振る舞いについて

<iframe src="//www.slideshare.net/slideshow/embed_code/key/g6li165csBd7yn" width="595" height="485" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="//www.slideshare.net/pospome/datastorego-struct" title="Datastore/Go のデータ設計と struct の振る舞いについて" target="_blank">Datastore/Go のデータ設計と struct の振る舞いについて</a> </strong> from <strong><a target="_blank" href="https://www.slideshare.net/pospome">pospome</a></strong> </div>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">DB設計の話、大好物なやつだ…！<a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; Koki Ide (@niconegoto) <a href="https://twitter.com/niconegoto/status/857568213617451008">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">Datastoreを使う上でネストしたstructは自動生成系ツールとの相性が悪い可能性があるのでバッドプラクティスだよ論者です <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; わかめ@TypeScript味 (@vvakame) <a href="https://twitter.com/vvakame/status/857568390210179073">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### LT2. Context アンチパターン

<script async class="speakerdeck-embed" data-id="c7ecf0661f6d4eea8c193a7f0fcd7c32" data-ratio="1.33333333333333" src="//speakerdeck.com/assets/embed.js"></script>

@timakinさん（グノシー）による発表。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">何故Contextを構造体に含めてはいけないかはGoogleの <a href="https://twitter.com/Sajma">@Sajma</a> 氏が答えてます <a href="https://t.co/lb1mTLCQUa">https://t.co/lb1mTLCQUa</a> <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; そな太 (@sonatard) <a href="https://twitter.com/sonatard/status/857571567114440705">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### LT3. ASTのライブコーディング

登壇者が病欠とのことで、急遽tenntennさんによるAST（抽象構文木）のライブコーディングが始まりました。Shinjuku.goでも発表されていましたけれど、正直、ASTよくわからない勢です。すみません。

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">ASTの話しかしない。尊さがある。 <a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; わかめ@TypeScript味 (@vvakame) <a href="https://twitter.com/vvakame/status/857572117222637569">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

<blockquote class="twitter-tweet" data-lang="ja"><p lang="ja" dir="ltr">抽象構造文、ね。あーあれね（わかってない）<a href="https://twitter.com/hashtag/golangtokyo?src=hash">#golangtokyo</a></p>&mdash; サクラ使い (@namaimoe) <a href="https://twitter.com/namaimoe/status/857573669547069441">2017年4月27日</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

### 感想

- GAE使ってみようと思った。
- でも特殊な知見が豊富ぽくてゲッソリしそうな気がした。
- 1.8がcomingしたら始めようと思いマス
- 業務で使えるの羨ましさある
