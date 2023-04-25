# Geth部署以太坊测试链(testnet)

关于以太坊的基本概念和安装可以参考[Geth部署以太坊私链及基本操作](https://ansheng.me/geth-deploys-ethereum-private-chain-and-basic-operations/)这篇文章，在本篇文章中我们将直接进入部署部分。

在有些时候我们需要搭建测试网来发布智能合约等操作，毕竟如果是主网是需要用ETH进行支付的，在开发期间这些操作都可使用测试网络进行即可，可以免费获取测试网络的ETH，而且测试网络上面的数据也比较齐全，还提供了很多Web UI来进行操作和查看，所以这篇文章我们将来搭建测试网络，测试网络的数据截至目前为止大概在40~50GB左右，所以请保证你的硬盘>=50GB。

## 部署

- 创建geth数据存放目录

```
$ mkdir /usr/local/etc/geth

```

- 启动命令

```
$ geth --nousb \\
     --identity "ansheng" \\
     --rinkeby \\
     --networkid 4 \\
     --syncmode "fast" \\
     --datadir /usr/local/etc/geth \\
     --cache 4096 \\
     --allow-insecure-unlock \\
     --http \\
     --http.addr 0.0.0.0 \\
     --http.port 23456 \\
     --http.api admin,debug,web3,eth,txpool,personal,clique,miner,net \\
     --http.corsdomain "*"

```
|参数|说明|
|:--|:--|
|--nousb	|关闭USB硬件钱包|
|--identity "ansheng"	|节点名称|
|--rinkeby	|使用rinkeby testnet|
|--networkid 4	|设置networkid，rinkeby testnet的网络ID是4|
|--syncmode "fast"	|数据同步模式，默认就是fast|
|--datadir /usr/local/etc/geth	|数据和密钥的数据目录|
|--cache 4096	|缓存大小，单位M|
|--allow-insecure-unlock	|允许通过HTTP-RPC的解锁account|
|--http	|开启HTTP-RPC服务|
|--http.addr 0.0.0.0	|HTTP-RPC服务监听的地址|
|--http.port 23456	|HTTP-RPC服务监听的端口|
|--http.api admin,debug,web3,eth,txpool,personal,ethash,miner,net	|开启那些HTTP-RPC服务API|
|`--http.corsdomain "*"`	|允许那些域名跨域链接，可以填写域名列表，我这里写*则表示允许全部|

通过`systemd`管理geth进程

```
$ vim /etc/systemd/system/geth.service
[Unit]
Description=Geth
After=network.target nss-lookup.target
Wants=network-online.target

[Service]
Type=simple
User=root
CapabilityBoundingSet=CAP_NET_BIND_SERVICE CAP_NET_RAW
NoNewPrivileges=yes

WorkingDirectory=/usr/local/etc/geth
ExecStart=/bin/sh -c '/usr/bin/geth --nousb --identity "ansheng" --rinkeby --networkid 4 --syncmode "fast" --datadir /usr/local/etc/geth --cache 4096 --allow-insecure-unlock --http --http.addr 0.0.0.0 --http.port 23456 --http.api admin,debug,web3,eth,txpool,personal,clique,miner,net --http.corsdomain "*" >> /usr/local/etc/geth/geth.log 2>&1'

Restart=on-failure
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target

```

- 重载配置文件

```
$ systemctl daemon-reload

```

- 启动并开机启动

```
$ systemctl enable --now geth
Created symlink /etc/systemd/system/multi-user.target.wants/geth.service → /etc/systemd/system/geth.service.

```

- 查看状态

```
$ systemctl status geth
● geth.service - Geth
     Loaded: loaded (/etc/systemd/system/geth.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2020-10-16 07:34:36 UTC; 3s ago
   Main PID: 1782 (sh)
      Tasks: 16 (limit: 19203)
     Memory: 526.0M
     CGroup: /system.slice/geth.service
             ├─1782 /bin/sh -c /usr/bin/geth --nousb --identity "ansheng" --rinkeby --networkid 4 --syncmode "fast" --datadir /usr/local/etc/geth --cache 4096 --allow-insecure-unlock --http --http.addr 0.0.0.0 --http.port 23456 --http.api admin,d
ebug,web3,eth,txpool,personal,clique,miner,net --http.corsdomain "*" >> /usr/local/etc/geth/geth.log 2>&1
             └─1783 /usr/bin/geth --nousb --identity ansheng --rinkeby --networkid 4 --syncmode fast --datadir /usr/local/etc/geth --cache 4096 --allow-insecure-unlock --http --http.addr 0.0.0.0 --http.port 23456 --http.api admin,debug,web3,eth,t
xpool,personal,clique,miner,net --http.corsdomain *
Oct 16 07:34:36 geth systemd[1]: Started Geth.

```

整个数据同步的时间根据VPS的配置和网络来决定，配置自然是越高越好了，我这里同步30多G大概用了三四个小时。

- 查看日志

```
$ tail -f /usr/local/etc/geth/geth.log
......
# 出现下面的日志就表示已经在同步了
INFO [10-16|07:31:31.612] Imported new state entries               count=1152 elapsed=6.616ms     processed=2921684 pending=12755 trieretry=0   coderetry=0 duplicate=0 unexpected=0
INFO [10-16|07:31:31.690] Imported new state entries               count=768  elapsed=6.218ms     processed=2922452 pending=13310 trieretry=0   coderetry=0 duplicate=0 unexpected=0
INFO [10-16|07:31:31.930] Imported new state entries               count=1152 elapsed=36.726ms    processed=2923604 pending=13065 trieretry=0   coderetry=0 duplicate=0 unexpected=0
INFO [10-16|07:31:32.058] Imported new block receipts              count=2048 elapsed=259.768ms   number=1038069 hash="01dd37…37b992" age=3y3w1d   size=3.04MiB
INFO [10-16|07:31:32.131] Imported new state entries               count=1152 elapsed=28.794ms    processed=2924756 pending=16582 trieretry=0   coderetry=0 duplicate=0 unexpected=0
INFO [10-16|07:31:32.150] Imported new state entries               count=384  elapsed=11.855ms    processed=2925140 pending=16545 trieretry=0   coderetry=0 duplicate=0 unexpected=0
INFO [10-16|07:31:32.255] Imported new state entries               count=768  elapsed=17.394ms    processed=2925908 pending=19236 trieretry=0   coderetry=0 duplicate=0 unexpected=0
INFO [10-16|07:31:32.289] Imported new block receipts              count=2048 elapsed=219.134ms   number=1040117 hash="1a65e3…8ca820" age=3y3w23h  size=2.67MiB
INFO [10-16|07:31:32.331] Imported new block receipts              count=204  elapsed=40.376ms    number=1040321 hash="d29156…bc9444" age=3y3w22h  size=380.17KiB
......

```

- 开放端口

如果你开启了Firewalld服务，还需要开放23456端口以供外部访问

```
$ firewall-cmd --add-port=23456/tcp --permanent
success
$ firewall-cmd --reload
success

```

## 操作

下面的操作依旧使用[Web3.py](https://web3py.readthedocs.io/en/stable/)

进入[python](https://ansheng.me/python-full-stack-way/)并创建连接

```
>>> from web3 import Web3
>>> web3 = Web3(Web3.HTTPProvider("<http://34.92.29.146:23456>", request_kwargs={'timeout': 60}))

```

查看同步状态

```
>>> web3.eth.syncing
AttributeDict({'currentBlock': 1034212, 'highestBlock': 7378440, 'knownStates': 3468664, 'pulledStates': 3448376, 'startingBlock': 0})

```

|值|描述|
|:--|:--|
|currentBlock|当前正在导入的区块编号|
|highestBlock|需要同步的最后块号，这是一个估计值|
|knownStates|当前已知的待拉取的总状态条目数|
|pulledStates|当前已经拉取的状态条目数|
|startingBlock|开始同步的起始区块编号|

查看有多少个peer在工作

```
>>> web3.net.peerCount
11

```

`peer`类似p2p的东西，连接数越多速度会更快

创建账户

```
>>> web3.parity.personal.new_account('ansheng.me')
'0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98'
>>> web3.parity.personal.new_account('ansheng')
'0xA218f9D21df8E31119fba91f0DeC1798168a8C14'

```

检查余额

```
>>> for address in web3.eth.accounts: print(address, web3.eth.getBalance(address))
...
0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98 0
0xA218f9D21df8E31119fba91f0DeC1798168a8C14 0

```

解锁，方便以后进行交易，0表示一直解锁

```
>>> web3.geth.personal.unlock_account('0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98', 'ansheng.me', 0)
True

```

启动挖矿

```
>>> web3.geth.miner.start(1)

```

为了方便交易，我们需要获取[Rinkeby](https://www.rinkeby.io/)测试网络的ETH，打开[Rinkeby](https://www.rinkeby.io/#faucet)页面，通过分享链接到推特或者Facebook都可以获取一定数量的ETH，再分享的时候只需要填入直接账号的address即可，我这里填写的是`0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98`

点击链接登陆推特账号分享

![Untitled](/images/2020/10/16/1.png)

填入刚才发表微博的地址，并点击`Give me Ether`，有三种选项，前面是获得的以太币数量，后面是冷却时间，在冷却时间过后才能进行下一次以太币申请。例如第一项是生成3个以太币，8小时后才能再次申请。

![Untitled](/images/2020/10/16/2.png)

最后会把ETH转入你指定的账号，你可以打开下面的链接检查你的账户余额，请把address替换为自己的

```
<https://rinkeby.etherscan.io/address/0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98>

```

![Untitled](/images/2020/10/16/3.png)

再次检查余额

```
>>> for address in web3.eth.accounts: print(address, web3.eth.getBalance(address))
...
0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98 0
0xA218f9D21df8E31119fba91f0DeC1798168a8C14 0

```

然后你会发现余额还是为0，这是为什么呢？这个时候我们查看一下同步的状态

```
>>> web3.eth.syncing
AttributeDict({'currentBlock': 7378695, 'highestBlock': 7378773, 'knownStates': 57359204, 'pulledStates': 57190786, 'startingBlock': 0})

```

你会发现数据还没有同步玩，等我们同步完了在检查余额，如果数据同步完了，查看同步状态的时候会返回`False`

```
>>> web3.eth.syncing
False
>>> for address in web3.eth.accounts: print(address, web3.eth.getBalance(address))
...
0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98 3000000000000000000
0xA218f9D21df8E31119fba91f0DeC1798168a8C14 0

```

建议同步完成之后在进行下面的操作，同步完要等三四个小时。

`0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98`账户往`0xA218f9D21df8E31119fba91f0DeC1798168a8C14`账户转1个ETH

```
>>> from web3.middleware import geth_poa_middleware
>>> web3.middleware_onion.inject(geth_poa_middleware, layer=0)
>>> web3.eth.sendTransaction({'to': '0xA218f9D21df8E31119fba91f0DeC1798168a8C14','from': "0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98",'value': web3.toWei('1','ether')})
HexBytes('0x69b5d5610ca824b7733a4229d3391b7cefb6ebb0680393c18c999463be95e153')

```

查看交易状态

```
>>> web3.eth.getTransaction('0x69b5d5610ca824b7733a4229d3391b7cefb6ebb0680393c18c999463be95e153')
AttributeDict({'blockHash': HexBytes('0x759104649e97ca072182753a12ee39494384f7ddfbf5a490aaa4a7b1b4cc103a'), 'blockNumber': 7379378, 'from': '0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98', 'gas': 121000, 'gasPrice': 1000000000, 'hash': HexBytes('0x69b5d5610ca824b7733a4229d3391b7cefb6ebb0680393c18c999463be95e153'), 'input': '0x', 'nonce': 0, 'to': '0xA218f9D21df8E31119fba91f0DeC1798168a8C14', 'transactionIndex': 16, 'value': 1000000000000000000, 'v': 43, 'r': HexBytes('0xe711221e02405e011542ff54485c12f67b7d1b7d2fa99cc35d31bf39367fe703'), 's': HexBytes('0x2dca9e6b09d0b93b1ef7b9257d73d2c2ec74f4d079ced3bd4fa74f9017be1c90')})

```

查看交易的块信息

```
>>> web3.eth.getBlock(7379378)
AttributeDict({'difficulty': 1, 'proofOfAuthorityData': HexBytes('0xd883010917846765746888676f312e31352e32856c696e7578000000000000003c39c304d6a4703d76dcca7e829f40fac9002f4e6ec53b312946d896e6b6237517a3136e0c6efcdcb59ce24ba916dd6e3e38f9d4bf272c1ed13d04cd4a70fcfe00'), 'gasLimit': 10000000, 'gasUsed': 2198318, 'hash': HexBytes('0x759104649e97ca072182753a12ee39494384f7ddfbf5a490aaa4a7b1b4cc103a'), 'logsBloom': HexBytes('0x000000000000030002000a002402000000000100010001000000000000004041044800040220000000000000008004001000100008201000200000400028004100001280000001800002006811800000000001000000000000000000000000400000200006000002000000080000082000000010000000000000001000840000002810000800000000000000000000100000080000040000000080840000002406000400002000000050000001000000000000000000020020008004030000000844100200000040008a00a000008010000800000080000006000080080020040010000100040600088001080000000020000080020000400400100000200020'), 'miner': '0x0000000000000000000000000000000000000000', 'mixHash': HexBytes('0x0000000000000000000000000000000000000000000000000000000000000000'), 'nonce': HexBytes('0x0000000000000000'), 'number': 7379378, 'parentHash': HexBytes('0xb4993ee0519090bdcab60dfc8e3b532d212e85bafd2e6a106789bee482f0d4a6'), 'receiptsRoot': HexBytes('0x4a2dbea1105e53dd02b45c474e1e324bba03a82fa126c69c711bb3262f8d2267'), 'sha3Uncles': HexBytes('0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347'), 'size': 5639, 'stateRoot': HexBytes('0x945bd68764cf6ecfbe622da588f9cdd34854b1f1b378ab84c4bb7f21b6bd9e47'), 'timestamp': 1602847803, 'totalDifficulty': 13109048, 'transactions': [HexBytes('0x1efe8466218503bad125a7b2dd9195de7ac032624e0425370ff3d55e95bc0b3d'), HexBytes('0xa39116e38eceb2e8ae1fd4ed23006c7ecb58465aba7220f3b16758466e5c3392'), HexBytes('0x8722c93f879af7e3cc2ef6472f5106ca7f5d3346a2eed328499b92f8ceb4929b'), HexBytes('0x9bddaaf6095867b18ecb6030ea521e45018e68c9ac26d3a475f3df74ccc728fc'), HexBytes('0x90257dc2be83012ad4637763954b724dd2c44c19b9f41c52e1fab4b51babc88e'), HexBytes('0xb2f8484c5d1e63d4b53ef5b046dfefd629542f4e36a2c6b4c6a159dded8041dc'), HexBytes('0x6f15e5d85aca3dc51346742add06732befdc5368e83cc7ba9da07a0844ee187f'), HexBytes('0xedadcfd68f467b17b66672e56fc51d446b1f8c841021bff78ca002c3d9ab96f3'), HexBytes('0x444f89004eff05f0d5c2eeab0d76c43a709ef406291eec65a4e5f8d0c8d237d0'), HexBytes('0xbd775e0c41884ffac3e10d0162c9d85c5bba5c7507093986c8bc970dadc6843f'), HexBytes('0x2e85e3761e36175b6be70b2b6b00c8ef7d24615c6f984f664b1a217b0b7f912d'), HexBytes('0xcca0e5a67b22eb16f223cd360c5d87860255073abc7c59e6505cba86b1714003'), HexBytes('0x58f1cf50e8b97225bd4b8d19b49eeed840f557407f19fb2ff3635986900364aa'), HexBytes('0xc78136ecf4c8082c720f7d3518623c143305fdd22aa642742b49e5fc1e2912d1'), HexBytes('0xfec80da3214582ada06e41876f14591b38a271b26be94c2d5c331224e657b969'), HexBytes('0x0373c62c2125e1d35e19387c9cf59fa46a97b1860bfc404f751da8cd4bc5c1be'), HexBytes('0x69b5d5610ca824b7733a4229d3391b7cefb6ebb0680393c18c999463be95e153'), HexBytes('0x284bd4f80b32e484d28cd821f8aac7a5f92694b4f397aebbc2914e25cbef8b17'), HexBytes('0xe192c655bcaacb1df16147edd7850ae9811d048c7817bb28e6cc019b20591c84')], 'transactionsRoot': HexBytes('0x00dacdc210057335c714643643f8ca89be90f51e95929a66ddf3851db317d978'), 'uncles': []})

```

可以通过下面的地址查询交易状态

```
<https://rinkeby.etherscan.io/tx/0x69b5d5610ca824b7733a4229d3391b7cefb6ebb0680393c18c999463be95e153>

```

![Untitled](/images/2020/10/16/4.png)

查询余额，因为转账收了一些手续费，所以`0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98`账户看起来不是一个整数

```
>>> for address in web3.eth.accounts: print(address, web3.eth.getBalance(address))
...
0xBeBD6250b2A18a20c645989Ce81F5c869Ed01d98 1999979000000000000
0xA218f9D21df8E31119fba91f0DeC1798168a8C14 1000000000000000000

```

## 一些坑

如果你的日志中一直出现下面的log，则表示找不到peer，这种情况通常在VPS上面是基本不会发生的，如果在本地会经常遇到这种情况，这种情况可能是你的网络问题，所以换一个网络就好了

```
......
INFO [10-16|15:27:22.194] Looking for peers                        peercount=0 tried=94 static=0
INFO [10-16|15:27:32.887] Looking for peers                        peercount=0 tried=75 static=0
INFO [10-16|15:27:42.911] Looking for peers                        peercount=0 tried=90 static=0
INFO [10-16|15:27:52.954] Looking for peers                        peercount=1 tried=79 static=0
INFO [10-16|15:28:03.032] Looking for peers                        peercount=0 tried=45 static=0
INFO [10-16|15:28:13.089] Looking for peers                        peercount=0 tried=61 static=0
......

```

或者出现下面这种log，同步着同步着就不同步了，然后等一段时间又开始同步了，所以，还是网络问题，换个网络

```
......
WARN [10-06|18:47:18.619] Rewinding blockchain                     target=162171
WARN [10-06|18:47:20.839] Rolled back chain segment                header=164220->162171 fast=163954->162171 block=0->0 reason="syncing canceled (requested)"
WARN [10-06|18:47:20.839] Synchronisation failed, dropping peer    peer=1de6fc6c000adb83 err="retrieved hash chain is invalid: timeout"
......

```