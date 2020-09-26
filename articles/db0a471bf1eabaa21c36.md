---
title: tmuxのおすすめ設定
tags: tmux Mac Linux
author: catatsuy
slide: false
---
[tmuxのすすめ | catatsuyのBlog](http://blog.catatsuy.org/a/243)

以前にこの記事を書きましたが，現在の私の設定とは一部乖離しているので追記します

# ESC キーの効きを良くする

tmux では ESC キーの反応が遅いです

私はたまに vim も使うので ESC の反応が悪いのは少し困ります

```.tmux.conf
set -s escape-time 0
```

# マウスを使えるようにする

マウスのスクロールを使えるようにします

```.tmux.conf
set-window-option -g mode-mouse on
```

# Solarized を使って色をいい感じにする

[Solarized - Ethan Schoonover](http://ethanschoonover.com/solarized)

これを使うと色がいい感じになります

dircolors なども用意されていて大変良いです

`source-file` などを使って外部ファイルとして読み込むのもよいですが，`.tmux.conf` に関しては大して長くもないのでコピペしてしまっても良いと思います

[solarized/tmux at master · altercation/solarized](https://github.com/altercation/solarized/tree/master/tmux)

これの好きな色を選択しましょう

ちなみに dircolors も設定したい場合は zsh ならば [.zshrcを色んな環境で共有する方法を考えてみた - Qiita [キータ]](http://qiita.com/catatsuy/items/00ebf78f56960b6d43c2#2-5) が参考になるかと思います


# tmux のコピーについて

[Ubuntu - tmux上のコピペをうまく設定する方法 - Qiita [キータ]](http://qiita.com/catatsuy/items/71b0f4932f00c6711ef5)

以前こんな記事も書いたのですが，これではリモートのサーバー上で tmux を立ち上げた時に無力です

実は Mac の場合，iTerm2 の開発版が OSC52/PASTE64 に対応しているので tmux のコピーとクリップボードの同期が取れます

[iTerm2のクリップボードインテグレーション(OSC 52/PASTE64)のつかいかた - Qiita [キータ]](http://qiita.com/kefir_/items/1f635fe66b778932e278)

この通りやれば，ローカルだろうとリモートだろうとクリップボードに同期が取れますので `tmux-copy` なるコマンドを用意する必要はありません
`reattach-to-user-namespace` 自体は `pbcopy/pbpaste` を使う際に必要ですので設定しておくと良いでしょう

残念ながら Ubuntu に関しては gnome-terminal が対応していないためこの機能を現在使うことはできません


# tmux の設定

現在の私の設定です


```.tmux.conf
# Prefix
set-option -g prefix C-z

# 日本語環境なら必須？？
setw -g utf8 on
set -g status-utf8 on

# status
set -g status-interval 10

# KeyBindings
# pane
unbind 1
bind 1 break-pane
bind 2 split-window -v
bind 3 split-window -h

bind C-r source-file ~/.tmux.conf
bind C-k kill-pane
bind k kill-window
unbind &
bind -r ^[ copy-mode
bind -r ^] paste-buffer

set -s escape-time 0

# shell
set-option -g default-shell /bin/zsh
set-option -g default-command /bin/zsh

set-window-option -g mode-mouse on

#### COLOUR (Solarized dark)
#### cf: https://github.com/altercation/solarized/blob/master/tmux/tmuxcolors-dark.conf

# default statusbar colors
set-option -g status-bg black #base02
set-option -g status-fg yellow #yellow
set-option -g status-attr default

# default window title colors
set-window-option -g window-status-fg brightblue #base0
set-window-option -g window-status-bg default
#set-window-option -g window-status-attr dim

# active window title colors
set-window-option -g window-status-current-fg brightred #orange
set-window-option -g window-status-current-bg default
#set-window-option -g window-status-current-attr bright

# pane border
set-option -g pane-border-fg black #base02
set-option -g pane-active-border-fg brightgreen #base01

# message text
set-option -g message-bg black #base02
set-option -g message-fg brightred #orange

# pane number display
set-option -g display-panes-active-colour blue #blue
set-option -g display-panes-colour brightred #orange
```

参考になれば幸いです

