---
title: zshユーザーでEmacsのtramp-modeで不具合を起こしている方へ
tags: Emacs Zsh
author: catatsuy
slide: false
---
Emacs の tramp-mode は ssh 先のファイルを手元で編集できるようにする elisp です

ssh 先のファイルをローカルのファイルと同じように編集できるのでとてもおすすめです

しかし，私の環境では ssh 接続が切れた後に Emacs が操作不能になるなどの不具合が頻発していました

ずっと原因が分からず悩んでいたのですが，普通に Emacs Wiki に解決策が書いてありました

[EmacsWiki: Tramp Mode](http://www.emacswiki.org/emacs/TrampMode#toc7)

.zshrc で `zle/prompt_cr/prompt_subst/precmd/preexec` などを使用していると不具合を起こすようです

私は既に複数箇所で使用していたために以下の設定を最後に呼ぶようにしました

```zsh:.zshrc_tramp
if [[ "$TERM" == "dumb" ]]; then
  unsetopt zle
  unsetopt prompt_cr
  unsetopt prompt_subst
  unfunction precmd
  unfunction preexec
  PS1='$ '
fi
```

`dumb` は tramp-mode の時の環境変数 `$TERM` の値です
tramp-mode の時は設定の一部を無効にすることで不具合が発生しなくなります

もしくはこれらを元々使わない，もしくは自分の環境の `$TERM` でしか有効にしないなどの対策が考えられますが，私は複数箇所に使用していたためにそれらの解決策は取りませんでした

これで快適な Emacs + tramp-mode + zsh ライフを送れそうです

Emacs 最高！！

