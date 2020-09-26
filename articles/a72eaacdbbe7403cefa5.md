---
title: Rack アプリケーションのプロファイリングを手軽に取りたい
tags: Ruby
author: catatsuy
slide: false
---
Sinatra とか Rails とかは Rack の上で動きます

ということで Rack ミドルウェアを使って手軽にプロファイリングを取りましょう

Gemfile の適当な場所に以下を足します

```rb:Gemfile
gem 'ruby-prof'
gem 'rack-contrib'
```

gem の最新は使えないので github から取るようにしましょう（2015/08/02 今年に入ってからgemも更新されるようになったようなので普通にgemでいいみたいです）

そして `Rack::Profiler` を読み込みます

```rb:config.ru
require 'ruby-prof'
require 'rack/contrib/profiler'

use Rack::Profiler
```

これで準備完了です

あとはプロファイリングを取りたい URL の GET クエリに `profile=process_time` を付与します

こうするといい感じの HTML が表示されるのでそれを見ましょう！

これでお手軽にプロファイリングを取れます

