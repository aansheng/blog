# 搭建Evmos多节点的测试环境

[Evmos](https://github.com/evmos/evmos)是在Cosmos上开发的一个支持EVM的链，你可以在上面部署智能合约等一系列操作。

这篇文章的目的是运行一条拥有三个节点的evmos测试链，而非单节点，如果想运行单节点可以只参考运行创世节点部分即可。

## 安装

evmosd是evmos链的二进制文件，你可以在github下载或者通过源码进行编译。

### 二进制包

evmos为我们提供了二进制文件包，我们可以将其下载下来放到指定位置即可。

```bash
rm -f $(which evmosd)
mkdir dist && cd dist
wget https://github.com/evmos/evmos/releases/download/v8.2.0/evmos_8.2.0_Linux_amd64.tar.gz
tar xf evmos_8.2.0_Linux_amd64.tar.gz
mv bin/evmosd /usr/local/bin/
cd ../ && rm -fr dist
```

查看版本

```bash
$ evmosd version
8.2.0
```

### 源码编译

- 安装编译所需要的软件包

```bash
apt update && apt install ca-certificates curl gnupg lsb-release make gcc git jq wget -y
```

- 安装Go

```bash
wget https://go.dev/dl/go1.19.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz
rm -f go1.19.1.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> .bashrc
echo 'export GOPATH=$(go env GOPATH)' >> .bashrc
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> .bashrc
source .bashrc
```

查看go版本

```bash
$ go version
go version go1.19.1 linux/amd64
```

- 编译安装

```bash
mkdir evmos && cd evmos
git clone -b v8.2.0 https://github.com/evmos/evmos.git .
make install
mv $(which evmosd) /usr/local/bin/
```

- 查看版本

```bash
$ evmosd version
8.2.0
```

## 运行创始节点

- 设置链ID的环境变量

```bash
export CHAINID="evmos_9000-1"
```

- 删除旧的目录

```bash
rm -rf ~/.evmosd
```

- 设置链ID

```bash
evmosd config chain-id $CHAINID
```

- 生成一个`mykey`的key

```bash
evmosd keys add mykey
```

- 初始化Evmos并设置moniker和chain-id

```bash
evmosd init node0 --chain-id $CHAINID
```

- 将Token名称改为aevmos

```bash
cat $HOME/.evmosd/config/genesis.json | jq '.app_state["staking"]["params"]["bond_denom"]="aevmos"' > $HOME/.evmosd/config/tmp_genesis.json && mv $HOME/.evmosd/config/tmp_genesis.json $HOME/.evmosd/config/genesis.json
cat $HOME/.evmosd/config/genesis.json | jq '.app_state["crisis"]["constant_fee"]["denom"]="aevmos"' > $HOME/.evmosd/config/tmp_genesis.json && mv $HOME/.evmosd/config/tmp_genesis.json $HOME/.evmosd/config/genesis.json
cat $HOME/.evmosd/config/genesis.json | jq '.app_state["gov"]["deposit_params"]["min_deposit"][0]["denom"]="aevmos"' > $HOME/.evmosd/config/tmp_genesis.json && mv $HOME/.evmosd/config/tmp_genesis.json $HOME/.evmosd/config/genesis.json
cat $HOME/.evmosd/config/genesis.json | jq '.app_state["evm"]["params"]["evm_denom"]="aevmos"' > $HOME/.evmosd/config/tmp_genesis.json && mv $HOME/.evmosd/config/tmp_genesis.json $HOME/.evmosd/config/genesis.json
cat $HOME/.evmosd/config/genesis.json | jq '.app_state["inflation"]["params"]["mint_denom"]="aevmos"' > $HOME/.evmosd/config/tmp_genesis.json && mv $HOME/.evmosd/config/tmp_genesis.json $HOME/.evmosd/config/genesis.json
```

- 为创世账户注入资金

```bash
evmosd add-genesis-account mykey 100000000000000000000000000aevmos
```

- 创建创世交易

```bash
evmosd gentx mykey 1000000000000000000000aevmos --chain-id $CHAINID
```

- 收集创世交易

```bash
evmosd collect-gentxs
```

- 验证是否正确设置创世文件

```bash
$ evmosd validate-genesis
File at /root/.evmosd/config/genesis.json is a valid genesis file
```

- 运行节点

```bash
evmosd start
```

- 查询mykey的余额

```bash
$ evmosd query bank balances $(evmosd keys show mykey -a)
Enter keyring passphrase:
balances:
- amount: "99999000000000000000000000"
  denom: aevmos
pagination:
  next_key: null
  total: "0"
```

## 其他节点

下面的操作可以在node1、node2上面同时运行

- 配置链ID

```bash
evmosd config chain-id evmos_9000-1
```

- 将`genesis.json`文件copy到node1、node2

```bash
scp ~/.evmosd/config/genesis.json node1:~/.evmosd/config/
scp ~/.evmosd/config/genesis.json node2:~/.evmosd/config/
```

- 设置seeds

```bash
$ vim ~/.evmosd/config/config.toml
seeds = "d4d96ce0316213f15ccae0c03aa053db4c443366@node0:26656"
# ID从node0上面的其中日志中可以获取到，记得测试一下node0的26656端口是否允许访问
# 6:26AM INF Add our address to book addr={"id":"d4d96ce0316213f15ccae0c03aa053db4c443366","ip":"0.0.0.0","port":26656} book=/root/.evmosd/config/addrbook.json module=p2p server=node
```

- 运行节点

```bash
evmosd start
```

一部分日志如下，可以看到已经在同步了

```bash
6:30AM INF committed state app_hash=B9DD1A096EEF8B737664919377134C6FF9907B7E973020A5496EE7F3EEC4F440 height=1106 module=state num_txs=0 server=node
6:30AM INF indexed block height=1106 module=txindex server=node
6:30AM INF Timed out dur=993.622933 height=1107 module=consensus round=0 server=node step=1
6:30AM INF received complete proposal block hash=00D236D057FEEA0F5F11CF332B088BB92E4BEBE87FD0C5FDB70E2A5C652D44FF height=1107 module=consensus server=node
6:30AM INF finalizing commit of block hash={} height=1107 module=consensus num_txs=0 root=B9DD1A096EEF8B737664919377134C6FF9907B7E973020A5496EE7F3EEC4F440 server=node
6:30AM INF executed block height=1107 module=state num_invalid_txs=0 num_valid_txs=0 server=node
6:30AM INF commit synced commit=436F6D6D697449447B5B31373420393920323339203620363720323234203133312031363620313620353320313338203136392031383020323330203131382036322031333320373720313638203934203130322031393820373320383920312031383620393620373520353820333320323533203131
305D3A3435337D
6:30AM INF committed state app_hash=AE63EF0643E083A610358AA9B4E6763E854DA85E66C6495901BA604B3A21FD6E height=1107 module=state num_txs=0 server=node
6:30AM INF indexed block height=1107 module=txindex server=node
```

## 转账测试

首先我们在node2上面创建一个alice的key

```bash
$ evmosd keys add alice
Enter keyring passphrase:
Re-enter keyring passphrase:

- name: alice
  type: local
  address: evmos14xlynppu87pyyskahg6lca8rpdtx7h3ctrxz8a
  pubkey: '{"@type":"/ethermint.crypto.v1.ethsecp256k1.PubKey","key":"Aw4OVNf/dYawGfvW+ON4Eo3TGtzhertU+WcbtlvC2X27"}'
  mnemonic: ""

**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

wagon own point pioneer sheriff medal subway agent shy knee trim floor minor still trim uncover tomato wrap favorite anger trigger employ frozen speed
```

查询alice的地址余额

```bash
$ evmosd query bank balances evmos14xlynppu87pyyskahg6lca8rpdtx7h3ctrxz8a
balances: []
pagination:
  next_key: null
  total: "0"
```

从创世节点转一点Token到alice账户地址

```bash
evmosd tx bank send $(evmosd keys show mykey -a) evmos14xlynppu87pyyskahg6lca8rpdtx7h3ctrxz8a 100aevmos
```

再次查询alice的账户余额

```bash
$ evmosd query bank balances evmos14xlynppu87pyyskahg6lca8rpdtx7h3ctrxz8a
balances:
- amount: "100"
  denom: aevmos
pagination:
  next_key: null
  total: "0"
```