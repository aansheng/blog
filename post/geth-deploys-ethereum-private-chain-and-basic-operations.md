# Geth部署以太坊私链及基本操作

[以太坊](https://ethereum.org/zh/)是一个为去中心化应用程序而生的全球开源平台。

在以太坊上，你可以通过编写代码管理数字资产、运行程序，更重要的是，这一切都不受地域限制。

[Go Ethereum（Geth）](https://geth.ethereum.org/)是以太坊官方以Go语言编写，完全[开源](https://github.com/ethereum/go-ethereum)。

为了方便学习，我们将先在本地通过Geth部署一套以太坊，便于我们以后的学习使用。

## 环境

我在GCP上面开了一台2核4G内存100G硬盘的VPS用来做测试，再创建VPS的时候记得开启允许HTTP流量、允许HTTPS流量，并允许全部端口访问

- 查看系统信息

```bash
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal
DISTRIB_DESCRIPTION="Ubuntu 20.04.1 LTS"
```

- 当前操作用户

```bash
$ whoami
root
```

- 更新系统到最新版并安装所需软件包

```bash
$ apt update && apt upgrade -y
```

- 关闭Firewalld（可选）

```bash
$ systemctl disable firewalld
```

- 重启

```bash
$ reboot
```

## 安装

我将在`Ubuntu 20.04`下面安装geth，如果你是其他平台，可以参考官方的[安装指南](https://geth.ethereum.org/docs/install-and-build/installing-geth)，官方文档支持多种方式多种平台的安装

```bash
$ apt install software-properties-common vim -y
$ add-apt-repository -y ppa:ethereum/ethereum
$ apt update
$ apt install ethereum -y
```

查看帮助文档

```bash
$ geth --help
```

查看geth版本

```bash
$ geth version
Geth
Version: 1.9.22-stable
Git Commit: c71a7e26a8b1e332bbf3262d88ba3ff32071456c
Architecture: amd64
Protocol Versions: [65 64 63]
Go Version: go1.15
Operating System: linux
GOPATH=
GOROOT=go
```

## 部署

- 创建geth数据存放目录

```bash
$ mkdir /usr/local/etc/geth
```

- 创建账户

```bash
$ geth --datadir /usr/local/etc/geth/ account new
......
# 输入密码
Password:
Repeat password:
# 记住下面的key
Your new key was generated
Public address of the key:   0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa
......
```

- 准备创世区块配置文件

```bash
$ vim /usr/local/etc/geth/genesis.json
{
   "config":{
      "chainId":9,
      "homesteadBlock":0,
      "eip150Block":0,
      "eip155Block":0,
      "eip158Block":0,
      "byzantiumBlock":0,
      "constantinopleBlock":0,
      "petersburgBlock":0,
      "istanbulBlock":0
   },
   "alloc":{
      "0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa":{
         "balance":"1000000000000000000000000"
      }
   },
   "coinbase":"0x0000000000000000000000000000000000000000",
   "difficulty":"0x20000",
   "extraData":"",
   "gasLimit":"0x2fefd8",
   "nonce":"0x0000000000000042",
   "mixhash":"0x0000000000000000000000000000000000000000000000000000000000000000",
   "parentHash":"0x0000000000000000000000000000000000000000000000000000000000000000",
   "timestamp":"0x00"
}
```

参数说明

|参数|说明|
|:--|:--|
|config.chainId|网络ID，由于是私链，可以随便设置|
|alloc|	预置账号的余额|
|coinbase	|矿工的默认账户|
|difficulty|	当前区块的难度，如果难度过大，CPU挖矿就很难，这里设置较小难度|
|extraData	|附加信息，随便填|
|gasLimit	|该值设置对GAS的消耗总量限制，用来限制区块能包含的交易信息总和，因｜｜为我们是私有链，所以|最大|
|nonce	|64位随机数，用于挖矿|
|mixhash	|与nonce配合用于挖矿，由上一个区块的一部分生成的hash|
|parentHash	|上一个区块的hash值，因为是创世块，所以这个值是0|
|timestamp	|设置创世块的时间戳|


- 初始化

```bash
$ geth --nousb --datadir /usr/local/etc/geth init /usr/local/etc/geth/genesis.json
```

## 启动

```bash

$ geth --nousb \\
       --identity "mychain1" \\
       --networkid 9 \\
       --datadir /usr/local/etc/geth \\
       --cache 2048 \\
       --allow-insecure-unlock \\
       --http \\
       --http.addr 0.0.0.0 \\
       --http.port 23456 \\
       --http.api admin,debug,web3,eth,txpool,personal,ethash,miner,net \\
       --http.corsdomain "*"

```

参数说明
|参数|说明|
|:--|:--|
|--nousb	|关闭USB硬件钱包|
|--identity "mychain1"	|自定义节点名称|
|--networkid 9	|设置networkid|
|--datadir /usr/local/etc/geth	|数据和密钥的数据目录|
|--cache 2048	|缓存大小，单位M|
|--allow-insecure-unlock	|允许通过HTTP-RPC的解锁account|
|--http	|开启HTTP-RPC服务|
|--http.addr 0.0.0.0	|HTTP-RPC服务监听的地址|
|--http.port 23456	|HTTP-RPC服务监听的端口|
|--http.api admin,debug,web3,eth,txpool,personal,ethash,miner,net	|开启那些HTTP-RPC服务API|
|`--http.corsdomain "*"`	|允许那些域名跨域链接，可以填写域名列表，我这里写*则表示允许全部|

通过`systemd`管理geth进程

```bash
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
ExecStart=/bin/sh -c '/usr/bin/geth --nousb --identity "mychain1" --networkid 9 --datadir /usr/local/etc/geth --cache 2048 --allow-insecure-unlock --http --http.addr 0.0.0.0 --http.port 23456 --http.api admin,debug,web3,eth,txpool,personal,ethash,miner,net --http.corsdomain "*" >> /usr/local/etc/geth/geth.log 2>&1'

Restart=on-failure
RestartPreventExitStatus=23

[Install]
WantedBy=multi-user.target

```

- 重载配置文件

```bash
$ systemctl daemon-reload

```

- 启动并开机启动

```bash
$ systemctl enable --now geth
Created symlink /etc/systemd/system/multi-user.target.wants/geth.service → /etc/systemd/system/geth.service.

```

- 查看状态

```bash
$ systemctl status geth
● geth.service - Geth
     Loaded: loaded (/etc/systemd/system/geth.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2020-10-14 09:19:23 UTC; 3s ago
   Main PID: 3231 (sh)
      Tasks: 10 (limit: 4713)
     Memory: 39.6M
     CGroup: /system.slice/geth.service
             ├─3231 /bin/sh -c /usr/bin/geth --nousb --identity "mychain1" --networkid 9 --datadir /usr/local/etc/geth --cache 2048 --allow-insecure-unlock --http --http.addr 0.0.0.0 --http.port 23456 --http.api admin,debug,web3,eth,txpool,person
al,ethash,miner,net --http.corsdomain "*" >> /usr/local/etc/geth/geth.log 2>&1
             └─3232 /usr/bin/geth --nousb --identity mychain1 --networkid 9 --datadir /usr/local/etc/geth --cache 2048 --allow-insecure-unlock --http --http.addr 0.0.0.0 --http.port 23456 --http.api admin,debug,web3,eth,txpool,personal,ethash,min
er,net --http.corsdomain *
Oct 14 09:19:23 geth systemd[1]: Started Geth.

```

- 查看日志

```bash
$ tail -f /usr/local/etc/geth/geth.log

```

- 开放端口

如果你开启了`Firewalld`服务，还需要开放`23456端口`以供外部访问

```bash
$ firewall-cmd --add-port=23456/tcp --permanent
success
$ firewall-cmd --reload
success

```

## 基本操作

在接下来的操作中，我们将在客户端通过[Web3.py](https://web3py.readthedocs.io/en/stable/)于与以太坊进行交互，Web3.py是用于与以太坊进行交互的Python库

- 安装pip包

```bash
$ pip3 install web3

```

- 进入python交互式程序

```bash
$ python3

```

- 基本操作

创建连接

```bash
>>> from web3 import Web3
>>> web3 = Web3(Web3.HTTPProvider("<http://34.92.29.146:23456>", request_kwargs={'timeout': 60}))

```

是否连接

```bash
>>> web3.isConnected()
True

```

查看客户端版本

```bash
>>> web3.clientVersion
'Geth/mychain1/v1.9.22-stable-c71a7e26/linux-amd64/go1.15'

```

列出所有账户

```bash
>>> web3.eth.accounts
['0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa']

```

创建账户，并返回账户地址

```bash
>>> web3.parity.personal.new_account('ansheng.me')
'0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F'
>>> web3.eth.accounts
['0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa', '0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F']

```

查看账户余额

```bash
>>> for address in web3.eth.accounts: print(address, web3.eth.getBalance(address))
...
0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa 1000000000000000000000000
0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F 0

```

开始挖矿，1表示启用的线程数

```bash
>>> web3.geth.miner.start(1)

```

开始挖矿之后我们需要查看日志，直到出现以下内容才表示矿机已经开始工作，如果矿机不工作，下面的转账等操作都是无法进行的，我这里用了6分35秒，时间还是挺久的

```bash
$ tail -f /usr/local/etc/geth/geth.log
······
INFO [10-14|08:34:01.829] Generating DAG in progress               epoch=0 percentage=99 elapsed=6m35.928s
INFO [10-14|08:34:01.832] Generated ethash verification cache      epoch=0 elapsed=6m35.931s
INFO [10-14|08:34:03.645] Successfully sealed new block            number=1 sealhash="286785…cb4fda" hash="383e18…dfadb5" elapsed=6m38.496s
INFO [10-14|08:34:03.645] 🔨 mined potential block                  number=1 hash="383e18…dfadb5"
INFO [10-14|08:34:03.646] Commit new mining work                   number=2 sealhash="be3fbb…d5a1f4" uncles=0 txs=0 gas=0 fees=0 elapsed="166.792µs"
······

```

解锁account，如果要进行交易操作，我们必须解锁account，否则无法进行交易，`web3.eth.coinbase`是预设余额的账户

```bash
>>> web3.eth.coinbase
'0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa'
>>> web3.geth.personal.unlock_account(web3.eth.coinbase, 'ansheng.me', 600) # （地址, 密码, 解锁时长）
True

```

估算一笔交易需要多少gas

```bash
>>> web3.eth.estimateGas({'to': '0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F','from': web3.eth.coinbase,'value': 100})
21000

```

进行转账，返回一个交易ID

```bash
>>> web3.eth.sendTransaction({'to': '0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F','from': web3.eth.coinbase,'value': 100})
HexBytes('0x87967cc25217c145e40a11818544d576365fbac944f16e42b2ee75280163f2d7')

```

查看交易信息，如果发现blockHash为None，有可能是矿机还没有启动或者矿机还没有处理，等待一下在查询即可

```bash
>>> web3.eth.getTransaction('0x87967cc25217c145e40a11818544d576365fbac944f16e42b2ee75280163f2d7')
AttributeDict({'blockHash': HexBytes('0x7e40f68be09961e6ea2767c291b5fccfbdc6de5dae5ee32593b61cdfe6d43613'), 'blockNumber': 30, 'from': '0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa', 'gas': 121000, 'gasPrice': 1000000000, 'hash': HexBytes('0x87967cc25217c145e40a11818544d576365fbac944f16e42b2ee75280163f2d7'), 'input': '0x', 'nonce': 1, 'to': '0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F', 'transactionIndex': 0, 'value': 100, 'v': 53, 'r': HexBytes('0x78b85a707c1174e8bd6bb92f8f8a08aaa3d05df62d83fef4c424548fb841a160'), 's': HexBytes('0x1db825aae194940277335c113336cd8f27ef8b9a83cd705cbb27460e681b6757')})

```

查看收据

```bash
>>> web3.eth.getTransactionReceipt('0x87967cc25217c145e40a11818544d576365fbac944f16e42b2ee75280163f2d7')
AttributeDict({'blockHash': HexBytes('0x7e40f68be09961e6ea2767c291b5fccfbdc6de5dae5ee32593b61cdfe6d43613'), 'blockNumber': 30, 'contractAddress': None, 'cumulativeGasUsed': 21000, 'from': '0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa', 'gasUsed': 21000, 'logs': [], 'logsBloom': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000'), 'status': 1, 'to': '0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F', 'transactionHash': HexBytes('0x87967cc25217c145e40a11818544d576365fbac944f16e42b2ee75280163f2d7'), 'transactionIndex': 0})

```

查询账户下面的交易笔数

```bash
>>> web3.eth.getTransactionCount('0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F')
0
>>> web3.eth.getTransactionCount(web3.eth.coinbase)
1
```

查看最新块编号

```bash
>>> web3.eth.blockNumber
76
```

查看某个区块信息

```bash
>>> web3.eth.getBlock(70)
AttributeDict({'difficulty': 133575, 'extraData': HexBytes('0xd683010916846765746886676f312e3135856c696e7578'), 'gasLimit': 3363639, 'gasUsed': 0, 'hash': HexBytes('0x89ca2885f01d1d60114784cbb7e60c98992bfd53a4fb2994554523f6507408bd'), 'logsBloom': HexBytes('0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000'), 'miner': '0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa', 'mixHash': HexBytes('0xda2bba8feeda0d8ecb77c003efd0064c43712bb8d2b8aa962a4cb3823c6e8bf0'), 'nonce': HexBytes('0x3c6269471b595232'), 'number': 70, 'parentHash': HexBytes('0xf20363ee879612c43cf7730e0ba8d98360401288ea000f056037b21593e2c881'), 'receiptsRoot': HexBytes('0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421'), 'sha3Uncles': HexBytes('0x1dcc4de8dec75d7aab85b567b6ccd41ad312451b948a7413f0a142fd40d49347'), 'size': 534, 'stateRoot': HexBytes('0x66a24b0a9a45905371a4ae5dd9f13531a8a45c02fdef848a0e756204abc816e5'), 'timestamp': 1602668396, 'totalDifficulty': 9360476, 'transactions': [], 'transactionsRoot': HexBytes('0x56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421'), 'uncles': []})

```

若要查询最近的一个区块信息，可以通过以下方式

```bash
>>> web3.eth.getBlock('latest')
```

检查账户余额，因为矿机一直在工作，所以`0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa`账户余额会增加

```bash
>>> for address in web3.eth.accounts: print(address, web3.eth.getBalance(address))
...
0xD31F14a7C839d352FE3EBD8745a0868Be042fBEa 1000195999999999999999800
0xAD606Fa02F3D66D749085B7f0D18Da3f85aC1f7F 100
```

停止挖矿

```bash
>>> web3.geth.miner.stop()
```