+++
tags = ["OSS", "Ruby"]
date = "2017-03-19T11:00:31+09:00"
title = "OSS Contribution of Mar 2017"
description = "2017年3月のOSSへの貢献について"

+++

以下のプルリクエストがマージされた。

[Add Npm with node-check-updates by yhirano55 · Pull Request #286 · sanemat/tachikoma](https://github.com/sanemat/tachikoma/pull/286)

[sanemat/tachikoma](https://github.com/sanemat/tachikoma)は『__指定プロジェクトのパッケージ管理ツールの更新プルリクエストを送る、Rakeタスク__』だ。今回作成したプルリクエストは、bundlerやcomposerなど、対応しているパッケージ管理ストラテジーに、[npm-check-updates](https://github.com/tjunnone/npm-check-updates)を追加する内容だ。

tachikomaのRakeタスクは非常にシンプルだ。コードを追うと、下記のようにtaskが定義されていて

```ruby
# tachikoma/lib/tasks/app.rake
namespace :tachikoma do
  desc 'run tachikoma with bundler'
  task :run_bundler do
    Tachikoma::Application.run 'bundler'
  end
end
```

`Tachikoma::Application.run` の処理も分かりやすい。流れとしては、次の通り。

- 指定リポジトリをClone
- 新規ブランチを作成
- 指定したストラテジーでパッケージを更新
- コミットを作成
- プルリクエストを作成（Octokit）

利用者が定義したい値（Githubアカウントやコミッタの情報）は、[YAML](https://github.com/sanemat/tachikoma/blob/master/data/default.yaml)で定義できる。

```ruby
tachikoma/lib/tachikoma/application.rb
module Tachikoma
  class Application
    include FileUtils

    def self.run(strategy)
      new.run(strategy)
    end

    # load/fetch/pull_requestは共通
    # load: configure
    # featch: git clone
    # strategy: git checkout ... git commit
    # pull_request: create pull request
    def run(strategy)
      load
      fetch
      send(strategy) if respond_to?(strategy)
      pull_request
    end

    def bundler
      Dir.chdir("#{Tachikoma.repos_path}/#{@build_for}") do
        Bundler.with_clean_env do
          sh(*['ruby', '-i', '-pe', '$_.gsub! /^ruby/, "#ruby"', 'Gemfile'])
          sh(*['git', 'config', 'user.name', @commiter_name])
          sh(*['git', 'config', 'user.email', @commiter_email])
          sh(*['git', 'checkout', '-b', "tachikoma/update-#{@readable_time}", @base_remote_branch])
          if File.exist?('Gemfile')
            @bundler_key_file = 'Gemfile'
            @bundler_lock_file = 'Gemfile.lock'
          elsif File.exist?('gems.rb')
            @bundler_key_file = 'gems.rb'
            @bundler_lock_file = 'gems.locked'
          else
            @bundler_key_file = 'Gemfile'
            @bundler_lock_file = 'Gemfile.lock'
          end
          sh(*([
            'bundle',
            '--gemfile', @bundler_key_file,
            '--no-deployment',
            '--without', 'nothing',
            '--path', 'vendor/bundle',
            @parallel_option
          ].compact))
          sh(*%w(bundle update))

          if @bundler_restore_bundled_with
            # restore_bundled_with
            lock_file_contents = File.read(@bundler_lock_file)
            lock_file = RestoreBundledWith::Lock.restore(
              lock_file_contents, @bundler_lock_file)
            File.write(@bundler_lock_file, lock_file.body)
          end

          sh(*['git', 'add', @bundler_lock_file])
          sh(*['git', 'commit', '-m', "Bundle update #{@readable_time}"]) do
            # ignore exitstatus
          end
          sh(*['git', 'push', @authorized_compare_url, "tachikoma/update-#{@readable_time}"])
        end
      end
    end
  end
end
```

今回マージされたのは、npm-check-updatesだったが、`package.json` を管理する方法としては、[Yarn](https://yarnpkg.com/en/)を利用する方がスタンダードかもしれない。そのため、筆者は上記プルリクエストと同じタイミングで、Yarnの更新を行うプルリクエストも送っている。

引き続き、オープンソースソフトウェアに対して、機能改善やパッチなどの貢献を行っていきたい。
