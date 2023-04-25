# 部署BTC(Bitcoin)私有网络(Regtest链)

Bitcoin 有三个独立的网络

- `Mainnet`: 主网
- `Testnet`: 测试网
- `Regtest`: 私网，即自己部署创世节点，可以将其他节点添加到此网络中，或者仅仅运行单个节点来进行测试

下面的文章中，我们将会运行一个创世节点，一个普通节点，并实现创建钱包以及转账。

## 环境准备

请在两台机器上面运行以下同样的操作

- 创建数据存放目录

```bash
mkdir -p ~/deploy/btc/data
```

- 创建 `docker-compose.yml` 文件

```bash
$ vim ~/deploy/btc/docker-compose.yml
version: "3"

services:

  btc:
    image: debian:11
    container_name: btc
    hostname: btc01 # 这里创世节点用btc01，第二个节点用btc02
    restart: always
    tty: true
    network_mode: host
    volumes: ["./data:/data"]
```

- 下载 bitcoin

```bash
cd ~/deploy/btc/data
# 可以在这里获得最新的下载地址https://bitcoin.org/en/download
wget <https://bitcoin.org/bin/bitcoin-core-22.0/bitcoin-22.0-x86_64-linux-gnu.tar.gz>
tar xf bitcoin-22.0-x86_64-linux-gnu.tar.gz
rm -f bitcoin-22.0-x86_64-linux-gnu.tar.gz
cd ../
```

- 运行并进入容器

```bash
docker compose up -d
docker exec -it btc bash
```

- 安装软件包

```bash
apt update && apt upgrade -y
apt install tmux vim -y
```

- 创建工作用的 `tmux session`

```bash
tmux new -s btc
```

- 创建数据存放目录

```bash
mkdir /data/bitcoin
```

## 运行创世节点

- 创建配置文件

```bash
$ vim /data/bitcoin.conf
# 以 regtest 模式运行
regtest=1

# 数据存放目录
datadir=/data/bitcoin

# 交易详细数据
txindex=1

# JSON-RPC
rpcuser=rpcuser
rpcpassword=rpcpassword
# 接收 JSON-RPC 命令
server=1

fallbackfee=0.01
[regtest]
bind=0.0.0.0
port=18444
```

- 运行节点程序

```
/data/bitcoin-22.0/bin/bitcoind -conf=/data/bitcoin.conf
```

## 添加其他节点

其他节点可以有多个，配置都是一样的

- 创建配置文件

```bash
$ vim /data/bitcoin.conf
regtest=1
datadir=/data/bitcoin
txindex=1
rpcuser=rpcuser
rpcpassword=rpcpassword
server=1

[regtest]
connect=35.76.52.60 # 这里填写创世节点的IP地址
```

- 运行节点

```bash
/data/bitcoin-22.0/bin/bitcoind -conf=/data/bitcoin.conf
```

## 钱包操作

- 创建默认钱包

当前版下是不会创建默认钱包的，需要手动创建

在 `btc01` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword createwallet "btc01"
{
  "name": "btc01",
  "warning": ""
}
```

在 `btc02` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword createwallet "btc02"
```

如果重启节点，需要重新 loadwallet，指令如下

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword loadwallet "btc01"
or
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword loadwallet "btc02"
```

- 生成一个新的钱包地址

在 `btc01` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getnewaddress
bcrt1qffhdrl0eec9tvfnzjq2ktyvghagzc5pj2ppqpn
```

在 `btc02` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getnewaddress
bcrt1qhg4grn0nh258l3wp6a7qf807pquk4pd7zqwz5l
```

- 获取钱包所有地址的余额

在 `btc01` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getbalance
0.00000000
```

在 `btc02` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getbalance
0.00000000
```

- 为创世节点地址生产 101 个块

在 `btc01` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword generatetoaddress 101 bcrt1qffhdrl0eec9tvfnzjq2ktyvghagzc5pj2ppqpn
[
  "03867b0f06cf29ee591799a6d0f56aa5098438b46ee95738645ed144b9b33196",
  ......
]
```

检查创始节点钱包余额

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getbalance
50.00000000
```

- 获取钱包信息

在 `btc01` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getwalletinfo
{
  "walletname": "btc01",
  "walletversion": 169900,
  "format": "bdb",
  "balance": 50.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 5000.00000000,
  "txcount": 101,
  "keypoololdest": 1658502912,
  "keypoolsize": 999,
  "hdseedid": "5b11137eaa37218348f0ce2efdce32e43d9b4b76",
  "keypoolsize_hd_internal": 1000,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": false
}
```

在 `btc02` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getwalletinfo
{
  "walletname": "btc02",
  "walletversion": 169900,
  "format": "bdb",
  "balance": 0.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 0.00000000,
  "txcount": 0,
  "keypoololdest": 1658502923,
  "keypoolsize": 999,
  "hdseedid": "601df012c1b238216865f9f358bae1f02db9aa35",
  "keypoolsize_hd_internal": 1000,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": false
}
```

## 基本操作

以下操作均在创世节点运行

- 查看网络节点

目前只有一个节点连接

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getpeerinfo
[
  {
    "id": 0,
    "addr": "xxx.xxx.xxx.xxx:59184",
    "addrbind": "xxx.xxx.xxx.xxx:18444",
    "addrlocal": "xxx.xxx.xxx.xxx:18444",
    "network": "ipv4",
    "services": "0000000000000409",
    ......
  }
]
```

- 查看区块数量

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getblockcount
101
```

- 查看区块链信息

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getblockchaininfo
```

## 转账

我们从创世节点钱包`bcrt1qffhdrl0eec9tvfnzjq2ktyvghagzc5pj2ppqpn`转账 10 个 BTC 到`bcrt1qhg4grn0nh258l3wp6a7qf807pquk4pd7zqwz5l`这个地址

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword sendtoaddress bcrt1qhg4grn0nh258l3wp6a7qf807pquk4pd7zqwz5l 10
cbdc41b975328d159bd700e724b7b3e61fdf9771a393eccfac7d5f896886bf9b
```

- 检查钱包余额

在 `btc01` 上面执行

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getbalance
39.99859000
```

在 `btc02` 上面执行，此时检查钱包余额会发现并没有更新

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getbalance
0.00000000
```

因为这笔交易并没有被打包提交到区块链中，所以此时我们需要再次生产一个区块，在创世节点生产一个区块

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword generatetoaddress 1 bcrt1qffhdrl0eec9tvfnzjq2ktyvghagzc5pj2ppqpn
[
  "5bd6e01a0bc7a8f210df3a6ad509304cd7797b12ff6cdb2273072833351c4dc3"
]
```

再次检查 `btc02` 的钱包余额

```bash
$ /data/bitcoin-22.0/bin/bitcoin-cli -rpcport=18443 -rpcuser=rpcuser -rpcpassword=rpcpassword getbalance
10.00000000
```

## 参考文献

- [Regtest 模式](https://github.com/zzir/blockchain-dev/blob/master/01.%E6%AF%94%E7%89%B9%E5%B8%81%E9%83%A8%E5%88%86/1.%E5%AE%89%E8%A3%85%E5%92%8C%E8%BF%90%E8%A1%8C/2.Regtest%20%E6%A8%A1%E5%BC%8F.md)