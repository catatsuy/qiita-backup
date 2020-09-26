---
title: bashのスクリプトを書くときはshebangで-xeを指定するよりset -xeした方がよさそう
tags: Bash
author: catatsuy
slide: false
---
例えばこういうスクリプトを書いたとします。

```bash:tmp.sh
#!/bin/bash -xe

ls | grep 'ababababababa'

echo 'abcd'
```

grepは存在しなければexit codeとして1を返します。なので下のechoは実行されないはずです。

しかし以下のように実行するとechoが実行されてしまいます。

```sh
bash tmp.sh
```

bashをつけて実行するとshebangの指定は無視されるみたいです。以下のようにsetで指定する方がよさそうです。

```bash
#!/bin/bash

set -xe

ls | grep 'ababababababa'

echo 'abcd'
```

今までどっちも同じやろ位にしか思っていなかったのですが、set派に改宗することにします。

