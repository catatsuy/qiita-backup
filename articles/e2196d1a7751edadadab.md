---
title: Homebrew で gcc4.7 を入れる
tags: homebrew MacOSX C++ C++11 C++0x
author: catatsuy
slide: false
---
Mac って `gcc` のバージョンが古くて C++ の C++11 の機能が全く使えなくて悲しくなります

ということで変えましょう

前提として

* Homebrew を使っている
* `brew doctor` してもエラーが出ない

が必要です

    brew tap homebrew/versions
    brew install gcc47

ひたすら待ちましょう

終わってもエイリアスは `llvm` に貼られたままなので上書きます

    sudo ln -sf /usr/local/bin/gcc-4.7 /usr/bin/gcc
    sudo ln -sf /usr/local/bin/g++-4.7 /usr/bin/g++

`gcc --version` とかして望みのバージョンが出れば完了です

ただ `-static` オプションつけたらエラーが出たりするのでガッツリ使いたいなら Linux ディストリビューション使いましょう…

