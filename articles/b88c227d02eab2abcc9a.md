---
title: どこのJavaScriptを読み込んでいるか知りたい
tags: JavaScript
author: catatsuy
slide: false
---
配信広告などを読み込んでいると、どんなJavaScriptを読み込んでいるかサービス提供者でも分かりません。しかし少しでも分かる手がかりが欲しいところです。探したところこんなものを見つけました。

[パフォーマンスまわりのAPIについて \- Qiita](http://qiita.com/makotot/items/70bd392a62afd43d3189#resource-timing-api)

`performance.getEntriesByType('resource')` を使えばダウンロードしたURLについての情報くらいは取ることが出来ます。残念なことにiframeのURLはこれで取れるのですが、iframeの中でリクエストしたURLは取れません。配信広告の場合、iframeの中でiframeが呼ばれてその中でさらに…（iframe地獄）、というようにして様々な配信広告に繋ぎにいきますが、これについては取得することはできません。

[Can I use\.\.\. Support tables for HTML5, CSS3, etc](http://caniuse.com/#feat=resource-timing)

Can I useを見るとSafariは対応してないのでSafariについては取れませんし、APIも存在していない可能性を考慮して書く必要があります。なので

```js
(function(){
  window.setTimeout(function(){
    if (typeof performance !== 'undefined' && typeof performance.getEntriesByType !== 'undefined') {
      var p = performance.getEntriesByType('resource');
      var i = 0;
      for (i = 0; i < p.length; i++) {
        console.log(p[i].name);
      }
    }
  }, 2000);
})();
```

このようなコードを書けば、とりあえずざっくりとどこにリクエストを送っているかは取得できます。

本来はパフォーマンス用途ですが、こういう使い方もできますよーということで。

