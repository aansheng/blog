# 利用Cosmos SDK快速部署一条Hello公链

****[Ignite CLI](https://docs.ignite.com/)****是Cosmos SDK的脚手架，可以非常方便迅速的帮助我们构建一条链出来，官网有一些示例感兴趣的可以去看看。

## 环境准备

有三台服务器来部署这条公链，当然功能很简单，具体可以往下看，三台机器的主机名分别是node1、node2、node3。

在此之前我们需要在node1上面编译和测试，需要安装Go和Ignite CLI。

- 安装Go

```bash
wget https://go.dev/dl/go1.19.1.linux-amd64.tar.gz
rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.1.linux-amd64.tar.gz
rm -f go1.19.1.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> .bashrc
echo 'export GOPATH=$(go env GOPATH)' >> .bashrc
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> .bashrc
source .bashrc

$ go version
go version go1.19.1 linux/amd64
```

[参考文档](https://go.dev/doc/install)

- 安装Ignite CLI

```bash
# 目前的版本锁定在了v0.22.2，去掉版本号则安装最新版本
curl https://get.ignite.com/cli@v0.22.2! | bash

$ ignite version
Ignite CLI version:	v0.24.0
Ignite CLI build date:	2022-09-12T14:14:32Z
Ignite CLI source hash:	21c6430cfcc17c69885524990c448d4a3f56461c
Your OS:		linux
Your arch:		amd64
Your go version:	go version go1.19.1 linux/amd64
Your uname -a:		Linux node1 5.10.0-16-amd64 #1 SMP Debian 5.10.127-2 (2022-07-23) x86_64 GNU/Linux
Your cwd:		/root
Is on Gitpod:		false
```

[参考文档](https://docs.ignite.com/guide/install)

## Hello

[参考文档](https://docs.ignite.com/guide/hello)，通过Ignite生成默认结构的区块链

```bash
ignite scaffold chain hello
```

### 添加功能

进入项目目录

```bash
cd hello
```

生成一个查询

```bash
ignite scaffold query hello --response text
```

为查询添加固定返回值

```bash
$ vim x/hello/keeper/grpc_query_hello.go
package keeper

import (
	"context"

	sdk "github.com/cosmos/cosmos-sdk/types"
	"google.golang.org/grpc/codes"
	"google.golang.org/grpc/status"
	"hello/x/hello/types"
)

func (k Keeper) Hello(goCtx context.Context, req *types.QueryHelloRequest) (*types.QueryHelloResponse, error) {
	if req == nil {
		return nil, status.Error(codes.InvalidArgument, "invalid request")
	}

	ctx := sdk.UnwrapSDKContext(goCtx)

	// TODO: Process the query
	_ = ctx

	return &types.QueryHelloResponse{Text: "Hello, Ignite CLI!"}, nil // 主要是添加了这么一段，查询时候返回固定的字符串

}
```

### 测试

运行节点

```bash
ignite chain serve
```

执行上面的指令时候会自动build`hellod`可执行文件，并放在`$GOPATH/bin`目录下

```bash
$ ls $GOPATH/bin
hellod  ......
```

另外打开一个窗口，通过一下命令测试功能是否正常

```bash
hellod q hello hello
```

如果返回下面的字符串则表示运行成功

```bash
text: Hello, Ignite CLI!
```

### 编译

也可以通过以下的指令build二进制文件

```bash
$ ignite chain build
Cosmos SDK's version is: stargate - v0.46.1

🛠️  Building proto...
📦 Installing dependencies...
🛠️  Building the blockchain...
🗃  Installed. Use with: hellod
```

## 部署

下面我们将部署一条公链，任何人都可以连接，而不是本地链，[参考文档](https://tutorials.cosmos.network/tutorials/3-run-node/)

### 运行创世节点

区块链初始化，链名为hello

```bash
hellod init hello
```

添加一个初始账户

```bash
$ hellod keys add node1
Enter keyring passphrase:
Re-enter keyring passphrase:

- address: cosmos1scp7az9zmy4k45utqx2qvy5t5r0jwel9md6777
  name: node1
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AuGNZwBX1zzSuMCsfMPRsFS9tRj3gxOM6mbVeL2wybXn"}'
  type: local

**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

cousin bag metal remember taxi inflict slide group foam pool ordinary civil orange bulb penalty answer often bulb again phrase seminar skull negative pumpkin
```

增加一些初始代币到这个账户

```bash
hellod add-genesis-account cosmos1scp7az9zmy4k45utqx2qvy5t5r0jwel9md6777 100000000stake
hellod gentx node1 70000000stake --chain-id hello
hellod collect-gentxs
```

运行创始节点

```bash
hellod start
```

查询初始账户余额

```bash
$ hellod query bank balances $(hellod keys show node1 -a)
Enter keyring passphrase:
balances:
- amount: "30000000"
  denom: stake
pagination:
  next_key: null
  total: "0"
```

### 运行其他节点

首先需要将创始节点的`hellod`文件和`genesis.json`文件scp道`node2`和`node3`

```bash
scp $GOPATH/bin/hellod node2:/usr/local/bin/
scp ~/.hello/config/genesis.json node2:~/

scp $GOPATH/bin/hellod node3:/usr/local/bin/
scp ~/.hello/config/genesis.json node3:~/
```

以下操作可以在node2、node3一起执行

```bash
hellod start # 此时会报错，不用担心，这里只是为了生成目录结构
```

将`genesis.json`文件放到对应的位置

```bash
mv genesis.json ~/.hello/config/
```

设置seeds

```bash
$ vim ~/.hello/config/config.toml
seeds = "23d5f29159840281067fb4a16df7647567b01763@108.160.141.226:26656"
# seed可以设置多个，以逗号(,)隔开
# 108.160.141.226是node1的IP
# 23d5f29159840281067fb4a16df7647567b01763这个ID可以在启动创世节点的时候获取到，日志为
# INF Add our address to book addr={"id":"23d5f29159840281067fb4a16df7647567b01763","ip":"0.0.0.0","port":26656} book=/root/.hello/config/addrbook.json module=p2p
```

运行节点

```bash
hellod start
```

运行之后可以看到block已经在同步了

```bash
4:09AM INF Timed out dur=4746.732153 height=184 module=consensus round=0 step=1
4:09AM INF commit is for a block we do not know about; set ProposalBlock=nil commit=3025986FE26C5B693E204B1E0BA5B39CF61ED0688F38283EA73541815869FAB1 commit_round=0 height=184 module=consensus proposal={}
4:09AM INF received complete proposal block hash=3025986FE26C5B693E204B1E0BA5B39CF61ED0688F38283EA73541815869FAB1 height=184 module=consensus
4:09AM INF finalizing commit of block hash={} height=184 module=consensus num_txs=0 root=BD6EFFB8E3C2735BB2FDB602760C4251E2A3B11E85DF88EC2A4D57C1E5440ECE
4:09AM INF minted coins from module account amount=2stake from=mint module=x/bank
4:09AM INF executed block height=184 module=state num_invalid_txs=0 num_valid_txs=0
4:09AM INF commit synced commit=436F6D6D697449447B5B3731203139302031303020383320313832203531203832203231203731203131203139322031363320313335203336203233392031303320313431203639203131332031313220373220323334203138322031323120372032343120323235203239203232322032333720323531203133385D3A42387D
4:09AM INF committed state app_hash=47BE6453B6335215470BC0A38724EF678D45717048EAB67907F1E11DDEEDFB8A height=184 module=state num_txs=0
4:09AM INF indexed block height=184 module=txindex
```

### 功能测试

在三台机器上面运行以下指令

```bash
hellod q hello hello
```

如果得到下面的输出，则表示功能正常

```bash
text: Hello, Ignite CLI!
```

### 转账测试

在创始节点中创建了node1账号，下面分别在node2上面创建node2账号，node3上面创建node3账号

```bash
root@node2:~# hellod keys add node2
Enter keyring passphrase:
Re-enter keyring passphrase:

- address: cosmos1ryvrfexfkxe5da6z4ykldy987dykak6ha3s7m3
  name: node2
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"AnOXk+1QuN0KIQVH1lNTvEfNnmZdKUoAggFMWh+ww9KX"}'
  type: local

**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

fossil aunt toward fame rib remember mistake act alter water zoo clutch nest reason goddess valve solve mix cruise divorce beach elite pencil avocado
```

```bash
root@node3:~# hellod keys add node3
Enter keyring passphrase:
Re-enter keyring passphrase:

- address: cosmos18e42hnsay4a76namaa99jszcs9g5p5w3zn4m03
  name: node3
  pubkey: '{"@type":"/cosmos.crypto.secp256k1.PubKey","key":"Ar58cWSMo/ENU6UqW4RH8XywPNXe+cSGJhz2/3Zh266a"}'
  type: local

**Important** write this mnemonic phrase in a safe place.
It is the only way to recover your account if you ever forget your password.

use nurse image gun bench learn broken hurdle clap embrace child always wheat wreck cheese spider kingdom discover radio opera panel genius juice hunt
```

查询三个账户的余额

```bash
root@node1:~# hellod query bank balances $(hellod keys show node1 -a)
Enter keyring passphrase:
balances:
- amount: "30000000"
  denom: stake
pagination:
  next_key: null
  total: "0"

root@node2:~# hellod query bank balances $(hellod keys show node2 -a)
Enter keyring passphrase:
balances: []
pagination:
  next_key: null
  total: "0"

root@node3:~# hellod query bank balances $(hellod keys show node3 -a)
Enter keyring passphrase:
balances: []
pagination:
  next_key: null
  total: "0"
```

从node1账户转账100到node2账户

```bash
hellod tx bank send $(hellod keys show node1 -a) cosmos1ryvrfexfkxe5da6z4ykldy987dykak6ha3s7m3 100stake --chain-id hello
```

node2账户转账50到node3账户

```bash
hellod tx bank send $(hellod keys show node2 -a) cosmos18e42hnsay4a76namaa99jszcs9g5p5w3zn4m03 50stake --chain-id hello
```

node3账户转账10到node1账户

```bash
hellod tx bank send $(hellod keys show node3 -a) cosmos1scp7az9zmy4k45utqx2qvy5t5r0jwel9md6777 10stake --chain-id hello
```

- 再次检查三个账户的余额

```bash
root@node1:~# hellod query bank balances $(hellod keys show node1 -a)
Enter keyring passphrase:
balances:
- amount: "29999910"
  denom: stake
pagination:
  next_key: null
  total: "0"

root@node2:~# hellod query bank balances $(hellod keys show node2 -a)
Enter keyring passphrase:
balances:
- amount: "50"
  denom: stake
pagination:
  next_key: null
  total: "0"

root@node3:~# hellod query bank balances $(hellod keys show node3 -a)
Enter keyring passphrase:
balances:
- amount: "40"
  denom: stake
pagination:
  next_key: null
  total: "0"
```