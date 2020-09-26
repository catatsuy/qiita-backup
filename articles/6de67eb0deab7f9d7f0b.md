---
title: CentOS5 での tmux のインストール
tags: tmux:1.8
author: catatsuy
slide: false
---
# libevent のインストール

http://libevent.org/

2.x 系の安定版を

# tmux のインストール

http://tmux.sourceforge.net/

最新の 1.8 系を入れました

# ldconfig

実行すると

    tmux: error while loading shared libraries: libevent-2.0.so.5: cannot open shared object file: No such file or directory

と怒られるので

`/etc/ld.so.conf.d/libevent.conf` に

    /usr/local/lib

を追加します

そして

    sudo /sbin/ldconfig

を実行します

これで普通に動くと思います

tmux の使い方はこちらも参考になります

[tmuxのすすめ | catatsuyのBlog](http://blog.catatsuy.org/a/243)

