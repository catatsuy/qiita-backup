---
title: ISUCONのベンチマーカーでGoのhttp.Clientをhttp2で使おうとしてハマった話
tags: Go http2
author: catatsuy
slide: false
---
Goの`net/http.Client`はGo1.6からhttp2に対応しています。そのことは皆さん知っていることだと思いますが、ちょっと特殊な使い方をしようとして、ハマった話を書きます。

ISUCON6本選の問題作成で以下のような必要が発生しました。

  * Goのhttp.Clientを使って大量のコネクションを張りたい
  * サーバーはhttpsにしたい
    * ISUCONという大会の性質上、オレオレ証明書
    * http2にも対応する

以下の記事を見て分かるように、基本的には`http.Get()`など標準の関数を使ってグローバルで確保されている`http.DefaultClient`を使うべきです。

[Goでnet/httpを使う時のこまごまとした注意 - Qiita](http://qiita.com/ono_matope/items/60e96c01b43c64ed1d18)

しかし、今回は以下のような特殊事情がありました。

  * 複数のクライアントから接続される実際のWebサービスのアクセスを模した動きを1台のベンチマーカーが行わなければならない
  * ベンチマーカーは都度起動して、1分ほど実行した後に死ぬので、多少のメモリリークは許容できる
    * それよりも複数のクライアントから接続されている事を模せるように、コネクションをしっかり増やせる事が重要

ということで検証していきます。

## 検証方法

とりあえずhttp2に対応したサーバーを立てます。これはもちろんGoでできますね。opensslとかでいい感じの自己証明書を作っておきます。

```go:server.go
package main

import (
  "fmt"
  "log"
  "net/http"
  "path/filepath"
  "time"
)

func main() {
  certFile, _ := filepath.Abs("ssl/oreore.crt")
  keyFile, _ := filepath.Abs("ssl/oreore.key")

  http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
    time.Sleep(5 * time.Second)
    fmt.Fprintf(w, "Protocol: %s\n", r.Proto)
  })

  err := http.ListenAndServeTLS(":3000", certFile, keyFile, nil)

  if err != nil {
    log.Printf("[ERROR] %s", err)
  }
}
```

自己証明書を正当な証明書にしたい場合は、Ubuntuだと以下のような手順を踏みます。

1. `/usr/local/share/ca-certificates` に証明書を置く
2. `sudo update-ca-certificates` を打つ

とりあえず正当な証明書になっている、という前提でしばらく話を進めます。

コネクション数は `sudo netstat -ta` を使ってコネクションをいくつ作っているのか見るという方法で検証します。

Goのhttp.Clientはhttp2が使えるサーバーだと、常にhttp2を使うようになるはずです。HTTP/1.1で接続する場合は`GODEBUG=http2client=0 ./client`という感じで実行すればよいです。

```go:http.Clientを使い回す
package main

import (
  "fmt"
  "net/http"
  "time"
)

func main() {
  c := http.Client{}

  for i := 0; i < 100; i++ {
    req, err := http.NewRequest("GET", "https://test.isucon.net:3000", nil)
    if err != nil {
      fmt.Println(err.Error())
      return
    }

    go func() {
      res, err := c.Do(req)
      if err != nil {
        fmt.Println(err)
        return
      }

      defer res.Body.Close()
      fmt.Println(res.Proto)
    }()
  }

  time.Sleep(100 * time.Second)
}
```

```go:http.Clientを2つ作って使い回す
package main

import (
  "fmt"
  "net/http"
  "time"
)

func main() {
  c1 := http.Client{}
  c2 := http.Client{}

  for i := 0; i < 50; i++ {
    req, err := http.NewRequest("GET", "https://test.isucon.net:3000", nil)
    if err != nil {
      fmt.Println(err.Error())
      return
    }

    go func() {
      res, err := c1.Do(req)
      if err != nil {
        fmt.Println(err)
        return
      }

      defer res.Body.Close()
      fmt.Println(res.Proto)
    }()
  }

  for i := 0; i < 50; i++ {
    req, err := http.NewRequest("GET", "https://test.isucon.net:3000", nil)
    if err != nil {
      fmt.Println(err.Error())
      return
    }

    go func() {
      res, err := c2.Do(req)
      if err != nil {
        fmt.Println(err)
        return
      }

      defer res.Body.Close()
      fmt.Println(res.Proto)
    }()
  }

  time.Sleep(100 * time.Second)
}
```

```go:http.Clientを都度作る
package main

import (
  "fmt"
  "net/http"
  "time"
)

func main() {
  for i := 0; i < 100; i++ {
    c := http.Client{}

    req, err := http.NewRequest("GET", "https://test.isucon.net:3000", nil)
    if err != nil {
      fmt.Println(err.Error())
      return
    }

    go func() {
      res, err := c.Do(req)
      if err != nil {
        fmt.Println(err)
        return
      }

      defer res.Body.Close()
      fmt.Println(res.Proto)
    }()
  }

  time.Sleep(100 * time.Second)
}
```

これらのプログラムをそれぞれ実行することで、以下の結論が得られます。

  * http.Clientの作り方に関係なく、http1.1はコネクションを都度張る
  * http.Clientの作り方に関係なく、http2はコネクションを1本だけ張る
    * `http.Client`を都度生成してもコネクションは1本だけで使い回される
    * これについては後述

これは困ります。実際のアクセスを模した動きにしたいため、例えhttp2を使っていたとしても、コネクションは都度接続したいです。そうしなければISUCONの場合、http2にした瞬間にコネクション数が激減してスコアが爆上がりします。これはゲームバランスを崩しうる秘孔になりえます。

これを解消する方法は後回しにして、次は証明書を無視する件について話します。

## 証明書を無視する

今までは正当な証明書を使う前提になっていました。しかし、テスト環境で正当な証明書を用意するのはドメインを設定したりしないといけないので面倒です。特にISUCONの問題とする場合はかなり面倒です。

自己証明書でワイルドカード証明書を発行して、チーム毎にドメインを決めるという手もありましたが、ベンチマーカーを動かすサーバーに自己証明書をインストールしなければならないので、開発も煩雑になります。またISUCONが終わった後に試してみたいという場合にも試しにくくなります。

そこでGoのhttp.Clientで証明書を無視するようにしてみます。それ自体は以下のように簡単にできます。

```go
c := &http.Client{
  Transport: &http.Transport{
    TLSClientConfig: &tls.Config{
      InsecureSkipVerify: true,
    },
  },
}
```

ただGo1.6のhttp.Clientの実装では、これだとhttp2が使えなくなります。理由は以下のissueに書かれています。

[net/http: Transport's automatic http2 too aggressive? · Issue #14275 · golang/go](https://github.com/golang/go/issues/14275)

この辺りの挙動はまた近い内に変わりそうなので、使う際には必ず調べて欲しいのですが、とりあえず現在の実装では証明書を無視するとhttp2が使えなくなってしまいます。そしてISUCON6本選では問題の根幹に関わる問題でした。

そこで今回は強引ですが、Goの`net/http`のコードをリポジトリ内にコピーしつつ、該当のコードをコメントアウトしました。

[TLSClientConfigを上書きしつつ、http2も使えるようにするパッチ by catatsuy · Pull Request #80 · isucon/isucon6-final](https://github.com/isucon/isucon6-final/pull/80/files)

Goの標準パッケージのソースコードをリポジトリ内にコピーするのには是非があると思いますが、Go言語の場合、標準パッケージの動きを少し変えたい、みたいな場合に外から動きを変えることが容易にできません。そのため標準パッケージのソースコードをリポジトリ内部にコピーして、少しだけ挙動を変えているオープンソースプロダクトはいくつか見ます。http周りをいじっているプロダクトは[puma/puma-dev](https://github.com/puma/puma-dev)などがあります。ISUCONは本番1日ちゃんと動けばいいという性質もあるので、今回は標準パッケージにパッチを当てることにしました。

そしてTransportを外から渡すようにすると、先程問題になったhttp2でコネクションが1本だけになってしまう問題が解消できます。`http.Client`を都度生成するのと同時にTransportも都度生成することで、http2になったときにも都度コネクションを生成するようにできます。そのためISUCON6本選はパッチを当てた上で、証明書を無視する`http.Client`を都度生成するベンチマーカーを作成しました。

## 結論

Go言語の`http.Client`では

  * http1.1はコネクションを都度張る
  * http2はコネクションを1本だけ張る
    * `http.Client`を都度生成してもコネクションは1本だけで使い回される
    * Transportを外から渡せば都度コネクションを張るようになる
  * 証明書を無視するようにすると、http2は使わなくなる
    * http2を使いたい場合は現状パッチを当てるしかない

