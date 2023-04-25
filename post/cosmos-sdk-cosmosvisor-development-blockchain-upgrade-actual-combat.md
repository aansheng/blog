# Cosmos-SDK+Cosmosvisor开发区块链升级实战

Cosmos-SDK基于Tendermint开发了很多实用的工具包，且都是模块化的，通过Cosmos-SDK可以快速构建一条L0链，迫于前期调研需要对升级方案做一些研究，所以就有了这篇文章。

理论部分请参考如下文章：

- [Cosmos Dev Series: Cosmos Blockchain Upgrade](https://medium.com/web3-surfers/cosmos-dev-series-cosmos-sdk-based-blockchain-upgrade-b5e99181554c)
- [In-Place Store Migrations](https://docs.cosmos.network/main/core/upgrade)
- [Upgrading Modules](https://docs.cosmos.network/main/building-modules/upgrade)
- [Cosmovisor](https://docs.cosmos.network/main/tooling/cosmovisor)
- [x/upgrade](https://docs.cosmos.network/main/modules/upgrade)
- [Teritori v1.3.0 Upgrade](https://github.com/polkachu/validator-guide/blob/main/upgrade_guides/teritori/upgrade_v1.3.0.md)

## 安装所需软件包

- 运行测试环境

通过docker创建一个simapp的容器，测试都在容器中进行

```bash
docker run -it -d --name simapp --hostname simapp debian:11
```

- 安装所需软件包

```bash
docker exec -it simapp bash
cd
apt update && apt upgrade -y
apt install vim tmux git curl wget tree jq -y
```

- Go

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

- Ignite

```bash
curl https://get.ignite.com/cli@v0.25.1! | bash

$ ignite version
Ignite CLI version:     v0.25.1
Ignite CLI build date:  2022-10-20T15:52:00Z
Ignite CLI source hash: cc393a9b59a8792b256432fafb472e5ac0738f7c
Cosmos SDK version:     v0.46.3
Your OS:                linux
Your arch:              amd64
Your go version:        go version go1.19.1 linux/amd64
Your uname -a:          Linux cosmos 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64 GNU/Linux
Your cwd:               /root
Is on Gitpod:           false
```

- Cosmosvisor

```bash
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0

$ cosmovisor version
cosmovisor version: v1.4.0
```

## 构建SimApp链

- v1.0.0

创建项目

```bash
ignite scaffold chain simapp
cd simapp
```

将默认分支改为v1.0.0

```bash
git branch -m v1.0.0
```

创建一个查询ping，返回PONG+版本号

```bash
$ ignite scaffold query ping --response text
$ vim x/simapp/keeper/grpc_query_ping.go +22
return &types.QueryPingResponse{Text: "PONG, v1.0.0"}, nil
```

运行链

```bash
ignite chain serve -r
```

查询测试

```bash
$ simappd q simapp ping
text: PONG, v1.0.0
```

提交代码

```bash
echo "build" >> .gitignore
git add . && git commit -a -m "v1.0.0"
```

build二进制文件并更名为`simappd_v1.0.0`

```bash
ignite chain build -o build
mv build/simappd build/simappd_v1.0.0
```

- v2.0.0

基于v1.0.0创建v2.0.0版本的branch

```bash
git checkout -b v2.0.0
```

v2.0.0的版本只是做了一个小改动，即把PING返回的结果`PONG, v1.0.0`改为`PONG, v2.0.0`

```bash
$ vim x/simapp/keeper/grpc_query_ping.go +22
return &types.QueryPingResponse{Text: "PONG, v2.0.0"}, nil
```

添加升级plan

```bash
$ vim app/upgrades.go
package app

import (
        sdk "github.com/cosmos/cosmos-sdk/types"
        "github.com/cosmos/cosmos-sdk/types/module"
        upgradetypes "github.com/cosmos/cosmos-sdk/x/upgrade/types"
)

const UpgradeName = "v2.0.0"

func (app App) RegisterUpgradeHandlers() {
        app.UpgradeKeeper.SetUpgradeHandler(UpgradeName,
                func(ctx sdk.Context, plan upgradetypes.Plan, fromVM module.VersionMap) (module.VersionMap, error) {
                        logger := ctx.Logger().With("upgrade", UpgradeName)
                        logger.Info("v1.0.0 upgrade to v2.0.0...")
                        return app.mm.RunMigrations(ctx, app.configurator, fromVM)
                })
}
```

注册plan

```bash
$ vim app/app.go +655
......
app.RegisterUpgradeHandlers()
......
```

运行链

```bash
ignite chain serve -r
```

功能测试

```bash
$ simappd q simapp ping
text: PONG, v2.0.0
```

提交代码

```bash
git add . && git commit -a -m "v2.0.0"
```

build二进制文件并重命名

```bash
ignite chain build -o build
mv build/simappd build/simappd_v2.0.0
```

最后的可执行文件如下

```bash
$ tree ~/simapp/build
/root/simapp/build
|-- simappd_v1.0.0
`-- simappd_v2.0.0

0 directories, 2 files
```

## 运行创世节点（v1.0.0）

```bash
# 删除测试时生成的目录
rm -fr ~/.simapp
# 配置链ID
~/simapp/build/simappd_v1.0.0 config chain-id test
# 初始化链
~/simapp/build/simappd_v1.0.0 init test --chain-id test
# 更新voting_period为20s
cat <<< $(jq '.app_state.gov.voting_params.voting_period = "20s"' $HOME/.simapp/config/genesis.json) > $HOME/.simapp/config/genesis.json
# 创建一个账户
~/simapp/build/simappd_v1.0.0 keys add validator
# 为帐户添加默认资金,如果报错可以将validator改为validator的address，类似：cosmos1slp3vtq4yrcfjcwcdvxj23trt8utq726pghueq
~/simapp/build/simappd_v1.0.0 add-genesis-account validator 100000000000000000000000000stake
# 把账户设置为验证节点
~/simapp/build/simappd_v1.0.0 gentx validator 1000000000000000000000stake --chain-id test
~/simapp/build/simappd_v1.0.0 collect-gentxs
```

`voting_period`参数表示在升级提案提交之后，在`voting_period`时间内需要有结果，否则这个提案就会作废，假设出快时间为1s/个，在高度10的时候提案创建，如果在高度10-30之间这个提案没有结果，最后这个提案就会被作废，也不会被升级。

配置cosmovisor环境变量

```bash
$ vim ~/.bashrc
# 二进制文件的名称
export DAEMON_NAME=simappd
# cosmovisor工作目录
export DAEMON_HOME=$HOME/.simapp
# 是否从网上下载二进制文件
export DAEMON_ALLOW_DOWNLOAD_BINARIES=false
# 更新完毕后自动重启
export DAEMON_RESTART_AFTER_UPGRADE=true
$ source ~/.bashrc
```

创建cosmovisor目录结构

```bash
mkdir -p $DAEMON_HOME/cosmovisor/genesis/bin
cp ~/simapp/build/simappd_v1.0.0 $DAEMON_HOME/cosmovisor/genesis/bin/simappd
```

通过cosmovisor运行节点

```bash
cosmovisor run start
```

`cosmovisor run start`的时候，默认会创建一个`current`的软链，例如

```bash
ln -s $DAEMON_HOME/cosmovisor/genesis $DAEMON_HOME/cosmovisor/current
```

然后运行的脚本是从`$DAEMON_HOME/cosmovisor/current/bin/simappd`，这样在下次升级的时候只需要把新版本的位置重新link到current即可。

查询版本信息，目前返回的信息是v1.0.0版本

```bash
$ $DAEMON_HOME/cosmovisor/current/bin/simappd q simapp ping
text: PONG, v1.0.0
```

## 创建升级提案

创建提案

```bash
$ $DAEMON_HOME/cosmovisor/genesis/bin/simappd tx gov submit-legacy-proposal software-upgrade v2.0.0 \
 --upgrade-height 30 \
 --title="Test Upgrade Proposal" \
 --description="testing, testing, 1, 2, 3" \
 --no-validate=true \
 --deposit 10000000stake \
 --from validator \
 --chain-id test
Enter keyring passphrase:
auth_info:
  fee:
    amount: []
    gas_limit: "200000"
    granter: ""
    payer: ""
  signer_infos: []
  tip: null
body:
  extension_options: []
  memo: ""
  messages:
  - '@type': /cosmos.gov.v1beta1.MsgSubmitProposal
    content:
      '@type': /cosmos.upgrade.v1beta1.SoftwareUpgradeProposal
      description: testing, testing, 1, 2, 3
      plan:
        height: "30"
        info: ""
        name: v2.0.0
        time: "0001-01-01T00:00:00Z"
        upgraded_client_state: null
      title: Test Upgrade Proposal
    initial_deposit:
    - amount: "10000000"
      denom: stake
    proposer: cosmos1g0da62czgmynvrzhd5dj6zev6tefw8dleqpp8d
  non_critical_extension_options: []
  timeout_height: "0"
signatures: []

confirm transaction before signing and broadcasting [y/N]: y
code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: CA306DA0DAE16176D559212A475F23B98CF8BAAD256F081B176EE733A0E345F3
```

参数说明如下

```
software-upgrade v2.0.0    # 软件升级的提案名称，可以写为版本号，且这个名称要和app/upgrades.go文件中的UpgradeName保持一致
--upgrade-height 20        # 预计在什么高度进行升级
--no-validate=true         # 不进行验证，表示不对--update-info参数进行验证，即可以不填写--update-info
--deposit 10000000stake    # 创建提案时默认存入多少资金，这是为了防止恶意刷题案的，提案在通过或者失败后会退回，但是如果no with veto比例超过33.3%就会被罚掉
```

投票

```bash
$ $DAEMON_HOME/cosmovisor/genesis/bin/simappd tx gov vote 1 yes --from validator --yes --chain-id test
Enter keyring passphrase:
code: 0
codespace: ""
data: ""
events: []
gas_used: "0"
gas_wanted: "0"
height: "0"
info: ""
logs: []
raw_log: '[]'
timestamp: ""
tx: null
txhash: 6230C775911CBB091CC16BDE09E53501EC9B6EEE65B5CCCD21A5C94D122A8759
```

## 升级v2.0.0

将新版本的二进制文件复制到对应的目录下，升级提案的名称是v2.0.0，所以创建的目录也是v2.0.0

```bash
mkdir -p $DAEMON_HOME/cosmovisor/upgrades/v2.0.0/bin
cp ~/simapp/build/simappd_v2.0.0 $DAEMON_HOME/cosmovisor/upgrades/v2.0.0/bin/simappd
```

等高度到30时候会停掉v1.0.0版本运行v2.0.0版本

```
9:06AM INF Timed out dur=4990.040215 height=30 module=consensus round=0 step=1
9:06AM INF received proposal module=consensus proposal={"Type":32,"block_id":{"hash":"6D3284C9F45FCA211DFC9B7BC368DDE638BDF320FC42C48D1E5D0DABE12C770F","parts":{"hash":"16C3A4AD0CD8B9F05800F199ED2EC7F24B5CB29EAEB89C3E85D4B5FABE4FF193","total":1}},"height":30,"pol_round" :-1,"round":0,"signature":"CL7/cRo5egyJ40bZrIbRH0Pp8D+dc/fYsINc2tSSGgdLnjSHVYocysiNc2CJ8H++0Y+gXMdPPaozCu336Uh4CQ==","timestamp":"2022-10-28T09:06:08.418559499Z"}
9:06AM INF received complete proposal block hash=6D3284C9F45FCA211DFC9B7BC368DDE638BDF320FC42C48D1E5D0DABE12C770F height=30 module=consensus
9:06AM INF finalizing commit of block hash={} height=30 module=consensus num_txs=0 root=AC81F9D50049F65D69052881AB4E433FE07005DF773E6AEA680C4AA982BEF68E
9:06AM ERR UPGRADE "v2.0.0" NEEDED at height: 30:
9:06AM ERR CONSENSUS FAILURE!!! err="UPGRADE \"v2.0.0\" NEEDED at height: 30: " module=consensus stack="......"
9:06AM INF service stop impl={"Logger":{}} module=consensus msg={} wal=/root/.simapp/data/cs.wal/wal
9:06AM INF service stop impl={"Dir":"/root/.simapp/data/cs.wal","Head":{"ID":"nwzjrIk7tTEe:/root/.simapp/data/cs.wal/wal","Path":"/root/.simapp/data/cs.wal/wal"},"ID":"group:nwzjrIk7tTEe:/root/.simapp/data/cs.wal/wal","Logger":{}} module=consensus msg={} wal=/root/.sima
pp/data/cs.wal/wal
9:06AM INF daemon shutting down in an attempt to restart module=cosmovisor
9:06AM INF starting to take backup of data directory backup start time=2022-10-28T09:06:08Z module=cosmovisor
9:06AM INF backup completed backup completion time=2022-10-28T09:06:08Z backup saved at=/root/.simapp/data-backup-2022-10-28 module=cosmovisor time taken to complete backup=4.102271
9:06AM INF pre-upgrade command does not exist. continuing the upgrade. module=cosmovisor
9:06AM INF upgrade detected, relaunching app=simappd module=cosmovisor
9:06AM INF running app args=["start"] module=cosmovisor path=/root/.simapp/cosmovisor/upgrades/v2.0.0/bin/simappd
9:06AM INF starting node with ABCI Tendermint in-process
9:06AM INF service start impl=multiAppConn module=proxy msg={}
9:06AM INF service start connection=query impl=localClient module=abci-client msg={}
9:06AM INF service start connection=snapshot impl=localClient module=abci-client msg={}
9:06AM INF service start connection=mempool impl=localClient module=abci-client msg={}
9:06AM INF service start connection=consensus impl=localClient module=abci-client msg={}
9:06AM INF service start impl=EventBus module=events msg={}
9:06AM INF service start impl=PubSub module=pubsub msg={}
9:06AM INF service start impl=IndexerService module=txindex msg={}
9:06AM INF ABCI Handshake App Info hash="____\x00I_]i\x05(__NC?_p\x05_w>j_h\fJ_____" height=29 module=consensus protocol-version=0 software-version=
9:06AM INF ABCI Replay Blocks appHeight=29 module=consensus stateHeight=29 storeHeight=30
9:06AM INF Replay last block using real app module=consensus
9:06AM INF applying upgrade "v2.0.0" at height: 30
9:06AM INF v1.0.0 upgrade to v2.0.0... upgrade=v2.0.0
9:06AM INF minted coins from module account amount=2059736728406547617stake from=mint module=x/bank
9:06AM INF executed block height=30 module=consensus num_invalid_txs=0 num_valid_txs=0
9:06AM INF commit synced commit=436F6D6D697449447B5B31323620323331203533203232203235203136203230302036312032313820313920373620353120363320323920333020313631203334203136392032313620323436203637203933203136332031353820313331203433203233372031363820323330203930203231352037
375D3A31457D
9:06AM INF committed state app_hash=7EE735161910C83DDA134C333F1D1EA122A9D8F6435DA39E832BEDA8E65AD74D height=30 module=consensus num_txs=0
9:06AM INF Completed ABCI Handshake - Tendermint and App are synced appHash="____\x00I_]i\x05(__NC?_p\x05_w>j_h\fJ_____" appHeight=29 module=consensus
9:06AM INF Version info block=11 p2p=8 tendermint_version=0.34.22
9:06AM INF This node is a validator addr=5341CFE8F809473DBE1F5D3F9535BF211C3153D7 module=consensus pubKey=4LFpdBg5/1KTqlphNhM+gHgvplicGc9im4K1VA1QQiQ=
9:06AM INF indexed block exents height=30 module=txindex
9:06AM INF P2P Node ID ID=ac8d5e4a28915725d1a6769ffdf47492bd066fc0 file=/root/.simapp/config/node_key.json module=p2p
9:06AM INF Adding persistent peers addrs=[] module=p2p
9:06AM INF Adding unconditional peer ids ids=[] module=p2p
9:06AM INF Add our address to book addr={"id":"ac8d5e4a28915725d1a6769ffdf47492bd066fc0","ip":"0.0.0.0","port":26656} book=/root/.simapp/config/addrbook.json module=p2p
9:06AM INF service start impl=Node msg={}
9:06AM INF Starting pprof server laddr=localhost:6060
9:06AM INF service start impl="P2P Switch" module=p2p msg={}
9:06AM INF service start impl=BlockchainReactor module=blockchain msg={}
9:06AM INF service start impl=ConsensusReactor module=consensus msg={}
9:06AM INF Reactor  module=consensus waitSync=false
9:06AM INF serve module=rpc-server msg={}
9:06AM INF service start impl=ConsensusState module=consensus msg={}
9:06AM INF service start impl=baseWAL module=consensus msg={} wal=/root/.simapp/data/cs.wal/wal
9:06AM INF service start impl=Group module=consensus msg={} wal=/root/.simapp/data/cs.wal/wal
9:06AM INF service start impl=TimeoutTicker module=consensus msg={}
9:06AM INF Searching for height height=31 max=0 min=0 module=consensus wal=/root/.simapp/data/cs.wal/wal
9:06AM INF Searching for height height=30 max=0 min=0 module=consensus wal=/root/.simapp/data/cs.wal/wal
9:06AM INF Found height=30 index=0 module=consensus wal=/root/.simapp/data/cs.wal/wal
9:06AM INF Catchup by replaying consensus messages height=31 module=consensus
9:06AM INF Replay: Done module=consensus
9:06AM INF service start impl=Evidence module=evidence msg={}
9:06AM INF service start impl=StateSync module=statesync msg={}
9:06AM INF service start impl=PEX module=pex msg={}
9:06AM INF service start book=/root/.simapp/config/addrbook.json impl=AddrBook module=p2p msg={}
9:06AM INF Saving AddrBook to file book=/root/.simapp/config/addrbook.json module=p2p size=0
9:06AM INF Ensure peers module=pex numDialing=0 numInPeers=0 numOutPeers=0 numToDial=10
9:06AM INF No addresses to dial. Falling back to seeds module=pex
9:06AM INF Timed out dur=4997.559085 height=31 module=consensus round=0 step=1
9:06AM INF received proposal module=consensus proposal={"Type":32,"block_id":{"hash":"380F92D76D209DF516F2A5EF2FAE2D27CA421F888C91A3F250C3AC7A86A97FA1","parts":{"hash":"B8ED33096C255242269779A8EFF6B185F9C64486ECBBB936A431352C332765EC","total":1}},"height":31,"pol_round"
:-1,"round":0,"signature":"4elm5tQex9Ml62qIB5Nj0zIWckMDTjr1ebYSMvCzI0ZGAm4YGaPsIExjaW48Qp7cYct3S4cyEScwAN/3mfzoAQ==","timestamp":"2022-10-28T09:06:13.798906293Z"}
9:06AM INF received complete proposal block hash=380F92D76D209DF516F2A5EF2FAE2D27CA421F888C91A3F250C3AC7A86A97FA1 height=31 module=consensus
9:06AM INF finalizing commit of block hash={} height=31 module=consensus num_txs=0 root=7EE735161910C83DDA134C333F1D1EA122A9D8F6435DA39E832BEDA8E65AD74D
9:06AM INF minted coins from module account amount=2059737097170852562stake from=mint module=x/bank
9:06AM INF executed block height=31 module=state num_invalid_txs=0 num_valid_txs=0
9:06AM INF commit synced commit=436F6D6D697449447B5B3130322037203638203737203230392031393620313420313332203138362031323420363620352038332034322031353220313620323434203231203234372033203231312031373620343220313520323320313932203531203233302032343020393220323432203230375D
3A31467D
9:06AM INF committed state app_hash=6607444DD1C40E84BA7C4205532A9810F415F703D3B02A0F17C033E6F05CF2CF height=31 module=state num_txs=0
9:06AM INF indexed block exents height=31 module=txindex
9:06AM INF Timed out dur=4992.31555 height=32 module=consensus round=0 step=1
9:06AM INF received proposal module=consensus proposal={"Type":32,"block_id":{"hash":"E5F9B6D971F837DF5680A59CF1AB5200D5E9880F90F5A3B4CC97054B555DC162","parts":{"hash":"2F1DB2D3E5015FCFED72252006C112F31564165304AF78EF25930889566AC9E8","total":1}},"height":32,"pol_round"
:-1,"round":0,"signature":"P1EjWPQBijQNl78A9JFRA7ou+SYnEN0kCH0SZlk7cqSdQOyggTf8VLt36TuOJh9g5GJcDjcMusvVdOyUbWPbAw==","timestamp":"2022-10-28T09:06:18.816719347Z"}
9:06AM INF received complete proposal block hash=E5F9B6D971F837DF5680A59CF1AB5200D5E9880F90F5A3B4CC97054B555DC162 height=32 module=consensus
9:06AM INF finalizing commit of block hash={} height=32 module=consensus num_txs=0 root=6607444DD1C40E84BA7C4205532A9810F415F703D3B02A0F17C033E6F05CF2CF
9:06AM INF minted coins from module account amount=2059737465935178546stake from=mint module=x/bank
9:06AM INF executed block height=32 module=state num_invalid_txs=0 num_valid_txs=0
9:06AM INF commit synced commit=436F6D6D697449447B5B35352032303720383220393920373220323037203135342034332036302032323420323430203234362032343020363820393920343320362032313020333720392033372034203134342031353120313732203236203831203139392031323220323038203238203134345D3A
32307D
9:06AM INF committed state app_hash=37CF526348CF9A2B3CE0F0F6F044632B06D2250925049097AC1A51C77AD01C90 height=32 module=state num_txs=0
```

从上述日志中可以看出，高度到30的时候程序出现恐慌，然后报错，最后程序被终止

```
9:06AM ERR UPGRADE "v2.0.0" NEEDED at height: 30:
```

cosmovisor会生成一个`upgrade-info.json`文件

```bash
$ ls $DAEMON_HOME/cosmovisor/upgrades/v2.0.0/upgrade-info.json
/root/.simapp/cosmovisor/upgrades/v2.0.0/upgrade-info.json

$ cat $DAEMON_HOME/cosmovisor/upgrades/v2.0.0/upgrade-info.json
{"name":"v2.0.0","time":"0001-01-01T00:00:00Z","height":30}
```

当cosmovisor读取到upgrade-info.json文件，会终止v1.0.0的程序，然后运行v2.0.0版本，从日志中也可以看到我们升级的日志，大部分情况下，每一次升级apphash都会变更

```bash
9:06AM INF applying upgrade "v2.0.0" at height: 30
9:06AM INF v1.0.0 upgrade to v2.0.0... upgrade=v2.0.0
```

再次查询版本

```bash
$ $DAEMON_HOME/cosmovisor/genesis/bin/simappd q simapp ping
text: PONG, v2.0.0
```