+++
date = "2017-05-15T21:48:37+09:00"
tags = ["Rubygems", "OSS"]
title = "OSSのメンテナンスとコスト"
description = "自作Gemの保守に関する所感"

+++

半年以上前に機会があって、[yhirano55/active_admin_versioning](https://github.com/yhirano55/active_admin_versioning)というGemを実装した。このGemは、[airblade/paper_trail](https://github.com/airblade/paper_trail)という監査ログやバージョン管理を行うGemを、管理ツール系Gemの[activeadmin/activeadmin](https://github.com/activeadmin/activeadmin)に簡単に組み込める代物だ。

リリース後にactiveadminのwikiのプラグインに記載したところ、海外の数名の方からStarを戴いたりして、嬉しい気持ちになった。とはいえ、自分の初期衝動を満たすことと勉強のため、というのが、実装のモチベーションの大部分だったので、ユーザーはそこまでいないだろうと思い、リポジトリにはIssueすら追加していなかった。

リリースしてから数週間後、とある方からpaper_trailのwhodunnitを設定できるようにしたいので、Issueを書けるようにしてくれないかとメールをいただいた。ウクライナからである。そして、それとともに[PR](https://github.com/yhirano55/active_admin_versioning/pull/2)が送られてきた。見知らぬ誰かから、PRが送られてきて、それを自分がレビューして、取り込むべきかどうか判断する。こういった一連の作業は、初めての体験だった。

記憶がまだ鮮明だったので、主張は理解できたし、問題なくマージし、公開Gemのバージョンもすぐに上げた。作者やメンテナはこういったコストを払っているものかと、ほんの僅かではあるが垣間見えた気がした。

### ActiveAdminを触らなくなった

著者自身がactiveadminを触っている時期であれば良いのだが、案件・プロジェクトによっては使用を嫌うケースも少なくはない。特にDSLを覚えなくてはならないのが最悪、とまで言われることもある。devise, doorkeeper, cancancan, ransackなどと同じく、activeadminも好き嫌いの分かれるGemではある。ちなみに私はGemを実装するくらいなので、activeadminはそれなりにコードも読んだし、DSLの追加も行ったりしているくらいなので、特に嫌ってもいない（むしろビジネスの課題である、迅速に管理ツールを起ちあげることが適っているので、良いツールだと思っているくらいだ）。

とはいえ触らなくなると、勘所が鈍くなり、情熱も失せていくのが性である。そんな折に、つい先日だが、PRが送られてきた。

[Add a configuration option to override the user used for whodunnit](https://github.com/yhirano55/active_admin_versioning/pull/10)

ざっくり言うと設定できる項目を追加したいというものだった。activeadminを殆ど触らなくなった状態だったので、自分が書いたコード含め、1時間ほどかけて、イチから見直してみた。コードを読む限りでは、その設定自体はpaper_trailで定義されているメソッドを、オーバーライドすれば良さそうなので、approveはしなかったが、どうにも自身が持てなかった。

### Collaboratorを追加する

OSSは作者が誰であれ、ライセンスに基づいた公共物でもある。なので、activeadminに常に詳しい状態であることは難しいし、メンテナンスの負荷も膨大に掛かってしまう。よって、過去に2度PRを送ってくれた、先のウクライナの方にメンションを飛ばして、Collaboratorになってくれないか、さりげなく打診をした。

> Hi. How do you think about it? Please take a 👀 , if you get a chance. (I wanna add you to this repo's collaborators...)

その後、彼は気前よくコードレビューをしてくれ、結局、著者と同意見だった。お礼と共に、今後のこのGemの行く末も見据え、彼をCollaboratorに追加した。そして、我々のレビューに対してPRのRequesterは、実際に困っているケースを記載してくれた。なるほど、そういうことで困っていたのか。

### モチベーションと責任感

実際に困っているケースと直せそうにない雰囲気もあった（実際にactiveadminのコードを読まないと分からなそうな部分があった）ので、私が休日の数時間を投入して修正し、Requesterの困りごとは解決したようだ。不思議なことだが、Collaboratorを追加すると、「**緊張感が生まれ、作者に責任感が芽生える**」「**作者とRequesterの1対1の構図ではないため、モチベーションがあがる**」という副作用があるように感じた。

今後、このGemがそこまで使われることはないだろうが、Gemを自作して公開して、メンテナンスまでして、尚且つ、こうした知見まで得られたので、作ってみてよかったと改めて感じた。それとともに、利用者の多いGemの作者はさぞかし多くの時間を割いていて、有難い存在だなとなんとなく空に向かって手を合わせたくなった。
