---
title: 仮想IPを付けたいそんな時
tags: Linux
author: catatsuy
slide: false
---
仮想IPを付ける

    ip addr add <付けたいIP>/<プレフィックス長> brd <ブロードキャストアドレス> dev eth0

send_arp する

    send_arp <付けたいIP> <そのIPのMACアドレス> <伝えたい相手IP> ff:ff:ff:ff:ff:ff <デバイス名:eth0>

ff:ff:ff:ff:ff:ff は 255.255.255.255 みたいな感じで MAC アドレスのブロードキャストアドレス．
伝えたい相手 IP を持つ機器の MAC アドレスが何であっても send_arp を受けてくれるらしい．
send_arp は一定時間空けて複数回打つとよいらしい．

