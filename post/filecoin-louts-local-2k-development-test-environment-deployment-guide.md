# FileCoin Louts本地2K开发/测试环境部署指南

在`FileCoin`官方文档中其实有一套[搭建本地开发环境的教程](https://docs.filecoin.io/build/local-devnet/)，如果感兴趣可以先去读一下这篇文章，当然，也可以根据本篇文章继续往下走。

为了环境统一，我使用Docker运行了两个`Ubuntu 20.x`的容器，且分配如下：

|主机名|描述|
|:--|:--|
|fil-2k-master|创世节点|
|fil-2k-miner|矿工|

## 编译安装Louts

- 创建数据挂载目录

```
mkdir -p /tmp/fil-2k-data

```

- Pull镜像

```
docker pull ubuntu:latest

```

- 运行容器

```
docker run -d -it -v /tmp/fil-2k-data:/data --hostname fil-2k-master --name fil-2k-master ubuntu
docker run -d -it -v /tmp/fil-2k-data:/data --hostname fil-2k-miner --name fil-2k-miner ubuntu

```

- 进入容器

你可以使用tmux开两个创建分别进入两容器

```
docker exec -it fil-2k-master bash
docker exec -it fil-2k-miner bash
# 切换到家目录下
cd

```

接下来的指令需要在两个容器同时操作，目的只是编译安装Louts

- 更新apt源

```
apt update

```

- 安装编译所需依赖包

```
apt install mesa-opencl-icd ocl-icd-opencl-dev gcc git bzr jq pkg-config curl clang build-essential hwloc libhwloc-dev wget -y

```

- 系统更新

```
apt upgrade -y

```

- 安装rust

lotus项目的底层实现代码，大部分都是使用rust语言编写的，所以我们需要安装rust

```
curl --proto '=https' --tlsv1.2 -sSf <https://sh.rustup.rs> | sh

```

- 安装Golang

```
wget -c <https://golang.org/dl/go1.16.5.linux-amd64.tar.gz> -O - | tar -xz -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc

```

- 验证rust和Golang

安装完成之后我们需要重启下容器以检查环境变量是否生效

```
docker restart fil-2k-master fil-2k-miner

```

重启之后再次进入容器，可以通过以下指令查看版本信息

```
$ rustc --version
rustc 1.53.0 (53cb7b09b 2021-06-17)
$ rustup --version
rustup 1.24.3 (ce5817a94 2021-05-31)
info: This is the version for the rustup toolchain manager, not the rustc compiler.
info: The currently active `rustc` version is `rustc 1.53.0 (53cb7b09b 2021-06-17)`
$ cargo --version
cargo 1.53.0 (4369396ce 2021-04-27)
$ go version
go version go1.16.5 linux/amd64

```

- 编译安装louts

```
mkdir ~/dist
cd ~/dist/
git clone <https://github.com/filecoin-project/lotus.git>
cd lotus/
git checkout v1.10.0 # 目前最新版是1.10.0
FFI_BUILD_FROM_SOURCE=1 make clean debug
# FFI_BUILD_FROM_SOURCE=1 从底层源码编译

```

编译完成之后会产出这几个可执行文件

```
$ ls -l lotus*
-rwxr-xr-x 1 root root 124446000 Jul  3 09:54 lotus
-rwxr-xr-x 1 root root 113050576 Jul  3 09:55 lotus-gateway
-rwxr-xr-x 1 root root 115801800 Jul  3 09:54 lotus-miner
-rwxr-xr-x 1 root root  93945976 Jul  3 09:54 lotus-seed
-rwxr-xr-x 1 root root 103036168 Jul  3 09:54 lotus-shed
-rwxr-xr-x 1 root root  96579768 Jul  3 09:55 lotus-wallet
-rwxr-xr-x 1 root root  97497912 Jul  3 09:54 lotus-worker

```

将编译后的可执行文件复制到PATH下面

```
make install

```

- 查看lotus版本

```
$ lotus --version
lotus version 1.10.0+debug+git.764fa9dae

```

- 下载2K网证明参数

```
# 2KiB 表示我们要下载的证明参数是 2KiB 大小的扇区所需要的证明参数
lotus fetch-params 2KiB

```

如果网络比较慢，可以使用国内加速节点进行下载

```
export IPFS_GATEWAY=https://proof-parameters.s3.cn-south-1.jdcloud-oss.com/ipfs/
lotus fetch-params 2KiB

```

## 创世节点

- 预密封两个2KiB大小的扇区

```
~/dist/lotus/lotus-seed pre-seal --sector-size=2KiB --num-sectors=2
# --sector-size 用于指定扇区大小
# --num-sectors 用于指定预密封扇区的数量

```

- 创建创世块

```
~/dist/lotus/lotus-seed genesis new localnet.json

```

- 使用一些FIL为默认帐户提供资金

```
~/dist/lotus/lotus-seed genesis add-miner localnet.json ~/.genesis-sectors/pre-seal-t01000.json

```

- 启动第一个节点

```
lotus daemon --lotus-make-genesis=devgen.car --genesis-template=localnet.json --bootstrap=false

```

运行daemon之后，当前窗口不可关闭，那新开一个tmux窗口继续操作

- 导入创世矿工密钥

```
lotus wallet import --as-default ~/.genesis-sectors/pre-seal-t01000.key
lotus wallet list

```

- 初始化创世矿工

```
lotus-miner init --genesis-miner --actor=t01000 --sector-size=2KiB --pre-sealed-sectors=~/.genesis-sectors --pre-sealed-metadata=~/.genesis-sectors/pre-seal-t01000.json --nosync
# --nosync 参数，创世旷工不需要等待链同步的过程

```

- 运行创世矿工

```
lotus-miner run --nosync
# 注意加上--nosync参数，创世旷工不需要等待链同步的过程

```

- 查看创世矿工状态

```
$ lotus-miner info
Chain: [sync behind! (4m21s behind)] [basefee 3.106 pFIL]
Miner: t01000 (2 KiB sectors)
Power: 40 Ki / 40 Ki (100.0000%)
        Raw: 4 KiB / 4 KiB (100.0000%)
        Committed: 4 KiB
        Proving: 4 KiB
Projected average block win rate: 20024.16/week (every 30s)
Projected block win with 99.9% probability every 41s
(projections DO NOT account for future network and miner growth)

Deals: 0, 0 B
        Active: 0, 0 B (Verified: 0, 0 B)

Miner Balance:    921.162 FIL
      PreCommit:  0
      Pledge:     1.192 μFIL
      Vesting:    690.871 FIL
      Available:  230.29 FIL
Market Balance:   0
       Locked:    0
       Available: 0
Worker Balance:   50000000 FIL
Total Spendable:  50000230.29 FIL

Sectors:
        Total: 2
        Proving: 2

```

- 重启创世旷工

首先是重启创世旷工的daemon节点

```
lotus daemon --genesis=devgen.car --profile=bootstrapper

```

重启创世旷工

```
lotus-miner run --nosync

```

## 矿工

- 拷贝同步数据

在`fil-2k-master`机器上将创世节点信息拷贝到/data目录

```
cp ~/devgen.car /data/

```

在`fil-2k-miner`机器上将`/data/devgen.car`文件拷贝到家目录下

```
cp /data/devgen.car .

```

- 启动节点

```
lotus daemon --genesis=devgen.car --bootstrap=false
# 同样需要设置 --bootstrap=false，因为后面我们需要手动连接到我们自己创建的创世节点上

```

- fil-2k-master上获取创世节点的连接信息

```
$ lotus net listen | grep "172"
/ip4/172.17.0.2/tcp/34189/p2p/12D3KooWDtjKv2guoiuvAeEWMnNaBPP9DLCRfKMEy9Rdvay8wfNu

```

- fil-2k-miner上连接到创世节点

```
$ lotus net connect /ip4/172.17.0.2/tcp/34189/p2p/12D3KooWDtjKv2guoiuvAeEWMnNaBPP9DLCRfKMEy9Rdvay8wfNu
connect 12D3KooWDtjKv2guoiuvAeEWMnNaBPP9DLCRfKMEy9Rdvay8wfNu: success

```

- 查看当前节点的同步状态

```
$ lotus sync status
sync status:
worker 0:
        Base:   [bafy2bzacearjw757v57g6ieqstjkl7wehhn3dgujuxqgitas4vyhhtvhr4lua]
        Target: [bafy2bzacedb72poh52fgb5bdgdxp4pq7ubu4toachnx3csuk2u66r43zup7jy] (28)
        Height diff:    28
        Stage: complete
        Height: 28
        Elapsed: 190.841027ms
worker 1:
        Base:   [bafy2bzacedb72poh52fgb5bdgdxp4pq7ubu4toachnx3csuk2u66r43zup7jy]
        Target: [bafy2bzaceax2xuxolw47fzosvmc4hsadya5p6l35mmzrw3j2ch77jdjjcl3xq] (29)
        Height diff:    1
        Stage: complete
        Height: 29
        Elapsed: 5.34543ms
worker 2:
        Base:   [bafy2bzaceax2xuxolw47fzosvmc4hsadya5p6l35mmzrw3j2ch77jdjjcl3xq]
        Target: [bafy2bzaceav7lwlz5kkzqw2yxodcvjckz6eurfe3er7dx3gtjk5faa5a2ik7w] (30)
        Height diff:    1
        Stage: complete
        Height: 30
        Elapsed: 7.088443ms
# 未同步成功的时候，这个命令会一直处于等待状态，同步成功之后，这个命令会自动退出
$ lotus sync wait
Worker: 3; Base: 30; Target: 31 (diff: 1)
State: complete; Current Epoch: 31; Todo: 0

Done!

```

- 创建BLS类型的钱包

```
$ lotus wallet new bls
t3r33j64poh7x6o3obvhdtqtz5ychmyrdnh2caawfflmfpcdezx23vljpetcq7zp3gifpkdpe6hbzazw3ojp6a

```

- 从创世节点处装100个FIL到当前钱包

```
$ lotus send t3r33j64poh7x6o3obvhdtqtz5ychmyrdnh2caawfflmfpcdezx23vljpetcq7zp3gifpkdpe6hbzazw3ojp6a 100
bafy2bzaceaayxqimfkaltimubnwgy4o4sr3wzzbxclfpaiu27zkx37iizmt6c

```

- 检查余额

```
$ lotus wallet balance
100 FIL
$ lotus wallet list
Address                                                                                 Balance  Nonce  Default
t3r33j64poh7x6o3obvhdtqtz5ychmyrdnh2caawfflmfpcdezx23vljpetcq7zp3gifpkdpe6hbzazw3ojp6a  100 FIL  0      X

```

- 初始化新节点

当新钱包有余额之后，就可以初始化矿工了，没余额无法初始化

```
lotus-miner init --sector-size=2KiB

```

- 修改新节点配置

修改miner配置文件的Sealing部分，BatchPreCommits和AggregateCommits都false

```
$ vim ~/.lotusminer/config.toml
[Sealing]
  BatchPreCommits = false
  AggregateCommits = false

```

如果有需要可以手动提交

```
lotus-miner sectors batching commit --publish-now=true
lotus-miner sectors batching precommit --publish-now=true

```

- 启动矿工

```
lotus-miner run
# RUST_LOG=Trace 可以让你看到 miner 更多的日志信息
RUST_LOG=Trace lotus-miner run

```

- 查看新节点

```
$ lotus-miner info
Chain: [sync ok] [basefee 203 aFIL]
Miner: t01002 (2 KiB sectors)
Power: 0  / 40 Ki (0.0000%)
        Raw: 0 B / 4 KiB (0.0000%)
        Committed: 0 B
        Proving: 0 B
Below minimum power threshold, no blocks will be won
Deals: 0, 0 B
        Active: 0, 0 B (Verified: 0, 0 B)

Miner Balance:    0
      PreCommit:  0
      Pledge:     0
      Vesting:    0
      Available:  0
Market Balance:   0
       Locked:    0
       Available: 0
Worker Balance:   100 FIL
Total Spendable:  100 FIL

Sectors:
        Total: 0

```

- 质押扇区（增加节点算力）

```
$ lotus-miner sectors pledge
Created CC sector:  0

```

- 查看扇区ID，获取扇区列表

```
$ lotus-miner sectors list
ID  State     OnChain  Active  Expiration  Deals
0   WaitSeed  NO       NO      n/a         CC

```

- 查看扇区信息

```
# 查看具体的某个扇区的状态（示例中显示的是扇区 0 的状态）
lotus-miner sectors status 0
# 查看 0 号扇区的详细日志信息
lotus-miner sectors status --log 0

```

等待一段时间后，我大概等了一晚上，再查看`sectors list`，会发现`OnChain`和`Active`都等于`YES`，现在才相当于拥有算力，有算力就可以出块（旷工做完第一个WindowPoST之后再过1个小时才开始出块，WindowPoST会24小时做一次）

```
$ lotus-miner sectors list
ID  State    OnChain  Active  Expiration                   Deals
0   Proving  YES      YES     1549971 (in 10 weeks 1 day)  CC
$ lotus-miner info
Chain: [sync ok] [basefee 100 aFIL]
Miner: t01002 (2 KiB sectors)
Power: 2 Ki / 42 Ki (4.7619%)
        Raw: 2 KiB / 6 KiB (33.3333%)
        Committed: 2 KiB
        Proving: 2 KiB
Expected block win rate: 5142.8520/day (every 16s)

Deals: 0, 0 B
        Active: 0, 0 B (Verified: 0, 0 B)

Miner Balance:    12809.06 FIL
      PreCommit:  0
      Pledge:     59.605 nFIL
      Vesting:    9574.825 FIL
      Available:  3234.235 FIL
Market Balance:   0
       Locked:    0
       Available: 0
Worker Balance:   100 FIL
Total Spendable:  3334.235 FIL

Sectors:
        Total: 1
        Proving: 1

```