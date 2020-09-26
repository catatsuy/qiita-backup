---
title: LaTeX でブロックコメントを使いたい
tags: LaTeX
author: catatsuy
slide: false
---
LaTeX には特にブロックコメントは用意されていないが，

```latex:
\if0
ここはコメント
\fi
```

で代用できる

Git などでバージョン管理していると差分は増やしたくないので活用しましょう
