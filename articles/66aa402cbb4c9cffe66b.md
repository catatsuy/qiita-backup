---
title: rsync したいときの秘伝のタレ
tags: Linux
author: catatsuy
slide: false
---
秘伝のタレ

    /usr/bin/rsync -vau -e 'ssh -c arcfour256' /hoge/fuga/ catatsuy@catatsuy.org:/hoge/fuga/

ディレクトリの最後に `/` があるかないかで挙動が違うので必ずつけましょう

オプション | 意味
----|----
`-v`, `--verbose` | ファイルの情報と、最後にサマリーが表示される
`-a`, `--archive` | よくあるオプションをひとまとめにしたものでいい感じにしてくれる
`-u`, `--update` | 新しいファイルだけコピーする
`-e`, `--rsh` | リモートシェルに接続するプログラムを指定する

