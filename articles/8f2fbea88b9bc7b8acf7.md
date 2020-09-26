---
title: レプリケーションを受けているMySQLを安全に停止したい
tags: MySQL
author: catatsuy
slide: false
---
```sql
SET GLOBAL innodb_fast_shutdown=0;
stop slave IO_THREAD;
```

してから

```
show slave status\G
```

して`Read_Master_Log_Pos`と`Exec_Master_Log_Pos`が一致していることを確認する

一致していたら

```
stop slave all;
```

してから止める

