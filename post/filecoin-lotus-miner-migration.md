# FileCoin Lotus Miner迁移

FileCoin官方文档有写[Lotus Miner: Backup and restore](https://docs.filecoin.io/mine/lotus/backup-and-restore/)，感兴趣的可以去看看，坑比较多，建议读本篇文章，这里会将一台有扇区的`miner`迁移到另外一台机器上。

## 备份

首先我们需要创建备份目录

```
mkdir -p ~/lotus-backups/`date +%F`

```

查看钱包列表

```
$ lotus wallet list -i
Address                                                                                 ID      Balance                          Nonce  Default
t3wrtndyosqzswvkodhxhsv6kdf6ikbrxtjunyarjyespio4rrduee6pkfcedl6seqflvyhcbae4p4uiimeuca  t01003  10950571.297152337120942244 FIL  1316   X

```

备份矿工私钥

```
lotus wallet export t3wrtndyosqzswvkodhxhsv6kdf6ikbrxtjunyarjyespio4rrduee6pkfcedl6seqflvyhcbae4p4uiimeuca > ~/lotus-backups/`date +%F`/t01003_wallet.key

```

使用`lotus-miner backup`备份元数据

```
$ lotus-miner backup --offline ~/lotus-backups/`date +%F`/backup.cbor
2021-07-16T11:47:37.911Z        INFO    backupds        backupds/datastore.go:75        Starting datastore backup
2021-07-16T11:47:39.309Z        INFO    backupds        backupds/datastore.go:130       Datastore backup done

```

> 如果你的miner正在运行，只需要把--offline参数去掉即可
> 

备份miner需要的文件文件

```
cp -r ~/.lotusminer/cache ~/.lotusminer/config.toml ~/.lotusminer/storage.json ~/.lotusminer/sectorstore.json ~/lotus-backups/`date +%F`

```

因为我这里搭建的是2K本地测试网，生成的扇区文件并不大，我会直接把`sealed`也备份过去，如果你使用的NFS或者对象存储，按照最开始的配置在配置一下就可以

```
cp -r ~/.lotusminer/sealed ~/lotus-backups/`date +%F`

```

然后将文件打包，传输到目标机器上

```
tar zcf lotus-backups.tar.gz lotus-backups/
scp lotus-backups.tar.gz ansheng@home:~/dist/

```

## 恢复

在恢复之前请确保你已经初始化了`lotus`环境并且节点已运行，最好是等待链同步完在执行恢复的操作

```
$ lotus sync wait
Worker: 72; Base: 154690; Target: 154692 (diff: 2)
State: complete; Current Epoch: 154692; Todo: 0
Validated 2 messages (1 per second)

Done!

```

解压备份的文件

```
tar xf lotus-backups.tar.gz

```

倒入矿工私钥

```
$ lotus wallet import ~/lotus-backups/`date +%F`/t01003_wallet.key
imported key t3wrtndyosqzswvkodhxhsv6kdf6ikbrxtjunyarjyespio4rrduee6pkfcedl6seqflvyhcbae4p4uiimeuca successfully!

```

查看钱包信息

```
$ lotus wallet list -i
Address                                                                                 ID      Balance                          Nonce
t3wrtndyosqzswvkodhxhsv6kdf6ikbrxtjunyarjyespio4rrduee6pkfcedl6seqflvyhcbae4p4uiimeuca  t01003  10950571.297152337120942244 FIL  1316

```

使用`lotus-miner`恢复元数据

```
$ lotus-miner init restore ~/lotus-backups/`date +%F`/backup.cbor
......
2021-07-16T12:42:33.301Z        INFO    paramfetch      go-paramfetch@v0.0.2-0.20210614165157-25a6c7769498/paramfetch.go:207    parameter and key-fetching complete
2021-07-16T12:42:33.301Z        INFO    main    lotus-storage-miner/init_restore.go:262 Initializing libp2p identity
2021-07-16T12:42:33.302Z        INFO    main    lotus-storage-miner/init_restore.go:274 Configuring miner actor
2021-07-16T12:42:33.343Z        INFO    main    lotus-storage-miner/init.go:596 Waiting for message: bafy2bzacec6lycxteqihbaetfmwu7eqa4cgzj4hthts247t5hayeixknpyigw

```

复制miner备份的配置文件

```
cp -r ~/lotus-backups/`date +%F`/cache ~/lotus-backups/`date +%F`/config.toml ~/lotus-backups/`date +%F`/storage.json ~/lotus-backups/`date +%F`/sectorstore.json ~/.lotusminer

```

将扇区文件目录复制过去

```
cp -r ~/lotus-backups/`date +%F`/sealed ~/.lotusminer/

```

启动矿工

```
lotus-miner run

```

## 检查

查看miner信息

```
$ lotus-miner info
Chain: [sync ok] [basefee 100 aFIL]
Miner: t01005 (2 KiB sectors)
Power: 128 Ki / 168 Ki (76.1904%)
        Raw: 128 KiB / 132 KiB (96.9696%)
        Committed: 192 KiB
        Proving: 192 KiB (64 KiB Faulty, 33.33%)
Expected block win rate: 21600.0000/day (every 4s)

Deals: 0, 0 B
        Active: 0, 0 B (Verified: 0, 0 B)

Miner Balance:    10845181.845 FIL
      PreCommit:  0
      Pledge:     5.722 _FIL
      Vesting:    1292521.568 FIL
      Available:  9552660.277 FIL
Market Balance:   0
       Locked:    0
       Available: 0
Worker Balance:   10950571.297 FIL
Total Spendable:  20503231.574 FIL

Sectors:
        Total: 96
        Proving: 96

```

```
$ lotus-miner proving deadlines
Miner: t01005
deadline  partitions  sectors (faults)  proven partitions
0         1           2 (0)             0
1         1           2 (0)             0
2         1           2 (0)             0
3         1           2 (0)             0
4         1           2 (0)             0
5         1           2 (0)             0
6         1           2 (0)             0
7         1           2 (0)             0
8         1           2 (0)             0
9         1           2 (0)             0
10        1           2 (0)             0
11        1           2 (0)             0
12        1           2 (0)             0
13        1           2 (0)             0
14        1           2 (0)             0
15        1           2 (0)             0
16        1           2 (0)             0
17        1           2 (0)             0
18        1           2 (0)             0
19        1           2 (0)             0
20        1           2 (0)             0
21        1           2 (0)             0
22        1           2 (0)             0
23        1           2 (0)             0
24        1           2 (0)             0
25        1           2 (0)             0
26        1           2 (0)             0
27        1           2 (2)             0
28        1           2 (2)             0
29        1           2 (2)             0
30        1           2 (2)             0
31        1           2 (2)             0
32        1           2 (2)             0
33        1           2 (2)             0
34        1           2 (2)             0
35        1           2 (2)             0
36        1           2 (2)             0
37        1           2 (2)             0
38        1           2 (2)             0
39        1           2 (2)             0
40        1           2 (0)             0
41        1           2 (0)             0
42        1           2 (2)             0
43        1           2 (2)             0
44        1           2 (2)             0
45        1           2 (0)             0  (current)
46        1           2 (0)             0
47        1           2 (0)             0

```

检查扇区文件的有效性，检查基本上都是ok的，我们等一轮wdpost就会变成有效算力了

```
$ for i in {0..47};do lotus-miner proving check $i ;done
deadline  partition  sector  status
0         0          0       good
0         0          1       good
deadline  partition  sector  status
1         0          5       good
1         0          9       good
deadline  partition  sector  status
2         0          2       good
2         0          3       good
deadline  partition  sector  status
3         0          4       good
3         0          6       good
deadline  partition  sector  status
4         0          7       good
4         0          8       good
deadline  partition  sector  status
5         0          10      good
5         0          11      good
deadline  partition  sector  status
6         0          14      good
6         0          15      good
deadline  partition  sector  status
7         0          17      good
7         0          16      good
deadline  partition  sector  status
8         0          12      good
8         0          19      good
deadline  partition  sector  status
9         0          13      good
9         0          18      good
deadline  partition  sector  status
10        0          20      good
10        0          30      good
deadline  partition  sector  status
11        0          23      good
11        0          27      good
deadline  partition  sector  status
12        0          24      good
12        0          21      good
deadline  partition  sector  status
13        0          25      good
13        0          26      good
deadline  partition  sector  status
14        0          22      good
14        0          28      good
deadline  partition  sector  status
15        0          29      good
15        0          37      good
deadline  partition  sector  status
16        0          31      good
16        0          35      good
deadline  partition  sector  status
17        0          33      good
17        0          34      good
deadline  partition  sector  status
18        0          32      good
18        0          39      good
deadline  partition  sector  status
19        0          40      good
19        0          48      good
deadline  partition  sector  status
20        0          51      good
20        0          63      good
deadline  partition  sector  status
21        0          56      good
21        0          57      good
deadline  partition  sector  status
22        0          36      good
22        0          44      good
deadline  partition  sector  status
23        0          38      good
23        0          49      good
deadline  partition  sector  status
24        0          41      good
24        0          42      good
deadline  partition  sector  status
25        0          43      good
25        0          45      good
deadline  partition  sector  status
26        0          46      good
26        0          47      good
deadline  partition  sector  status
27        0          50      good
27        0          71      good
deadline  partition  sector  status
28        0          53      good
28        0          60      good
deadline  partition  sector  status
29        0          58      good
29        0          59      good
deadline  partition  sector  status
30        0          52      good
30        0          54      good
deadline  partition  sector  status
31        0          55      good
31        0          64      good
deadline  partition  sector  status
32        0          61      good
32        0          68      good
deadline  partition  sector  status
33        0          62      good
33        0          66      good
deadline  partition  sector  status
34        0          67      good
34        0          69      good
deadline  partition  sector  status
35        0          65      good
35        0          70      good
deadline  partition  sector  status
36        0          72      good
36        0          73      good
deadline  partition  sector  status
37        0          74      good
37        0          75      good
deadline  partition  sector  status
38        0          77      good
38        0          84      good
deadline  partition  sector  status
39        0          78      good
39        0          79      good
deadline  partition  sector  status
40        0          76      good
40        0          81      good
deadline  partition  sector  status
41        0          82      good
41        0          85      good
deadline  partition  sector  status
42        0          86      good
42        0          80      good
deadline  partition  sector  status
43        0          83      good
43        0          94      good
deadline  partition  sector  status
44        0          87      good
44        0          95      good
deadline  partition  sector  status
45        0          88      good
45        0          89      good
deadline  partition  sector  status
46        0          90      good
46        0          92      good
deadline  partition  sector  status
47        0          93      good
47        0          91      good

```

> 恢复现已完成，记得多备份
>