# Ubuntu 20.04部署索拉纳(Solana)Devnet网Validator

## 系统基本信息

- 更新系统

```
$ sudo apt update && sudo apt upgrade -y

```

查看系统版本

```
$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description: Ubuntu 20.04.3 LTS
Release: 20.04
Codename: focal

```

重启

```
sudo shutdown -r now

```

## 安装GPU驱动

查看显卡信息，这里有两块`GeForce RTX 2080 Ti`显卡

```
$ lspci -vnn | grep VGA
12:00.0 VGA compatible controller [0300]: ASPEED Technology, Inc. ASPEED Graphics Family [1a03:2000] (rev 41) (prog-if 00 [VGA controller])
21:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] [10de:1e07] (rev a1) (prog-if 00 [VGA controller])
61:00.0 VGA compatible controller [0300]: NVIDIA Corporation TU102 [GeForce RTX 2080 Ti Rev. A] [10de:1e07] (rev a1) (prog-if 00 [VGA controller])

```

打开[NVIDIA Driver Downloads](https://www.nvidia.com/Download/index.aspx)，选择对应显卡的驱动程序进行下载，这里下载的事目前最新的版本`470.63.01`

```
$ mkdir ~/dist && cd ~/dist
$ wget <https://cn.download.nvidia.com/XFree86/Linux-x86_64/470.63.01/NVIDIA-Linux-x86_64-470.63.01.run>
$ ls -hl NVIDIA-Linux-x86_64-470.63.01.run
-rw-r--r-- 1 sol sol 259M Sep  3 10:10 NVIDIA-Linux-x86_64-470.63.01.run

```

安转依赖包

```
sudo apt install build-essential libglvnd-dev pkg-config -y

```

禁用默认的Nouveau驱动

```
$ lsmod | grep nouveau
nouveau              1986560  0
mxm_wmi                16384  1 nouveau
i2c_algo_bit           16384  2 ast,nouveau
drm_ttm_helper         16384  2 drm_vram_helper,nouveau
ttm                    73728  3 drm_vram_helper,drm_ttm_helper,nouveau
drm_kms_helper        237568  5 drm_vram_helper,ast,nouveau
video                  49152  1 nouveau
wmi                    32768  2 mxm_wmi,nouveau
drm                   548864  7 drm_kms_helper,drm_vram_helper,ast,drm_ttm_helper,ttm,nouveau

$ sudo vim /etc/modprobe.d/blacklist-nvidia-nouveau.conf
blacklist nouveau
options nouveau modeset=0

$ sudo update-initramfs -u
$ sudo shutdown -r now

```

安装Nvidia驱动，一直确认即可

```
sudo bash ~/dist/NVIDIA-Linux-x86_64-470.63.01.run

```

通过`nvidia-smi`指令查看驱动信息

```
$ nvidia-smi
Fri Sep  3 11:13:32 2021
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.63.01    Driver Version: 470.63.01    CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  Off  | 00000000:21:00.0 Off |                  N/A |
|  0%   34C    P0    61W / 260W |      0MiB / 11019MiB |      1%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
|   1  NVIDIA GeForce ...  Off  | 00000000:61:00.0 Off |                  N/A |
|  0%   31C    P0    51W / 280W |      0MiB / 11019MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+

+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

```

如果想查看GPU使用率，还可以安装nvtop指令

```
$ sudo apt install nvtop -y
$ nvtop

```

如果要使用GPU则必须要安装CUDA(并行计算框架)，打开[CUDA Toolkit 11.4 Update 1 Downloads](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=runfile_local)页面根据自己的版本进行安装，如果你用的不是这个版本，其他历史版本可以在[CUDA Toolkit Archive](https://developer.nvidia.com/cuda-toolkit-archive)处找到。

```
$ cd ~/dist/
$ wget <https://developer.download.nvidia.com/compute/cuda/11.4.1/local_installers/cuda_11.4.1_470.57.02_linux.run>
$ sudo sh cuda_11.4.1_470.57.02_linux.run

===========
= Summary =
===========

Driver:   Installed
Toolkit:  Installed in /usr/local/cuda-11.4/
Samples:  Installed in /home/sol/, but missing recommended libraries

Please make sure that
 -   PATH includes /usr/local/cuda-11.4/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-11.4/lib64, or, add /usr/local/cuda-11.4/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-11.4/bin
To uninstall the NVIDIA Driver, run nvidia-uninstall
Logfile is /var/log/cuda-installer.log

```

安装完成之后我们需要配置一些环境变量

```
$ vim ~/.bashrc
......
export CUDA_HOME=/usr/local/cuda-11.4
export PATH=/usr/local/cuda-11.4/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64:$LD_LIBRARY_PATH

```

最好重启一下系统吧，如果不重启就加载一下环境变量

```
source ~/.bashrc

```

## 编译安装Solana

参考文章[从源码构建Solana(索拉纳)](https://ansheng.me/build-solana-from-source-code/)。

如果你使用的是有GPU的版本，在编译之前需要生成`solana-perf-libs`相关的文件

```
$ git clone -b v1.7.10 <https://github.com/solana-labs/solana.git> ~/dist/solana-src-v1.7.10
$ git clone <https://github.com/solana-labs/solana-perf-libs.git> ~/dist/solana-perf-libs
$ cd ~/dist/solana-perf-libs
$ make -j$(nproc)

```

把文件copy到Solana仓库中，然后继续编译安装即可

```
make DESTDIR=~/dist/solana-src-v1.7.10/target/perf-libs/cuda-11.4 install

```

## Validator

首先我们需要连接到Devnet群集

```
solana config set --url <http://api.devnet.solana.com>

```

确保可以访问到Devnet群集，国内网络环境可能会经常出现超时等情况，可以多尝试几次

```
$ solana transaction-count
1603626215

```

查看集群中的其他节点，`ctrl+c`可以退出

```
solana-gossip spy --entrypoint devnet.solana.com:8001

```

### 系统优化

增加UDP缓冲区

```
sudo bash -c "cat >/etc/sysctl.d/20-solana-udp-buffers.conf <<EOF
# Increase UDP buffer size
net.core.rmem_default = 134217728
net.core.rmem_max = 134217728
net.core.wmem_default = 134217728
net.core.wmem_max = 134217728
EOF"
sudo sysctl -p /etc/sysctl.d/20-solana-udp-buffers.conf

```

增加内存映射文件限制

```
sudo bash -c "cat >/etc/sysctl.d/20-solana-mmaps.conf <<EOF
# Increase memory mapped files limit
vm.max_map_count = 1000000
EOF"
sudo sysctl -p /etc/sysctl.d/20-solana-mmaps.conf

```

编辑`/etc/systemd/system.conf`文件`[Service]`段增加下面的内容

```
LimitNOFILE=1000000

```

`[Manager]`段增加下面的内容

```
DefaultLimitNOFILE=1000000

```

重新加载

```
sudo systemctl daemon-reload

```

增加文件描述符数量

```
sudo bash -c "cat >/etc/security/limits.d/90-solana-nofiles.conf <<EOF
# Increase process file descriptor count limit
* - nofile 1000000
EOF"

```

退出终端重新登陆，查看limit限制

```
$ ulimit -n
1000000

```

### 创建验证节点钱包信息

这里创建的是fs钱包

```
mkdir -p ~/deploy/sol
solana-keygen new -o ~/deploy/sol/validator-keypair.json

```

- 配置默认的keypair地址

```
solana config set --keypair ~/deploy/sol/validator-keypair.json

```

- 领取空投

每次最多只能领取10个SOL，但是可以重复领取

```
solana airdrop 10

```

检查余额

```
solana balance

```

### 创建投票账户

在Solana上运行验证器节点，需要创建一个投票帐户

```
solana-keygen new -o ~/deploy/sol/vote-account-keypair.json
solana create-vote-account ~/deploy/sol/vote-account-keypair.json ~/deploy/sol/validator-keypair.json

```

### 运行Validator

我们创建一个systemd服务来运行Solana，首先需要创建一个运行脚本

```
$ mkdir ~/deploy/sol/scripts -p
$ vim ~/deploy/sol/scripts/validator.sh
#!/bin/bash

solana-validator \\
  --identity ~/deploy/sol/validator-keypair.json \\
  --vote-account ~/deploy/sol/vote-account-keypair.json \\
  --ledger ~/deploy/sol/validator-ledger \\
  --rpc-port 8899 \\
  --entrypoint devnet.solana.com:8001 \\
  --limit-ledger-size \\
  --cuda \\
  --log ~/deploy/sol/solana-validator.log
$ chmod +x ~/deploy/sol/scripts/validator.sh

```

创建`/etc/systemd/system/sol.service`文件

```
$ sudo vim /etc/systemd/system/sol.service
[Unit]
Description=Solana Validator
After=network.target
Wants=solana-sys-tuner.service
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=sol
LimitNOFILE=1000000
LogRateLimitIntervalSec=0
Environment="PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/home/sol/.local/share/solana/install/active_release/bin:/usr/local/cuda-11.4/bin"
Environment="CUDA_HOME=/usr/local/cuda-11.4"
Environment="LD_LIBRARY_PATH=/usr/local/cuda-11.4/lib64:$LD_LIBRARY_PATH"
#Environment="SOLANA_METRICS_CONFIG=host=https://metrics.solana.com:8086,db=devnet,u=scratch_writer,p=topsecret"

ExecStart=/home/sol/deploy/sol/scripts/validator.sh

[Install]
WantedBy=multi-user.target

```

加载systemd文件

```
sudo systemctl daemon-reload

```

运行sol并设置开机启动

```
sudo systemctl enable --now sol

```

查看运行状态

```
sudo systemctl status sol

```

查看运行日志

```
tail -f ~/deploy/sol/solana-validator.log

```

### 配置日志切割

默认情况下日志文件会越来越大，所以我们需要通过`logrotate`进行日志切割

```
$ sudo vim /etc/logrotate.d/sol
/home/sol/deploy/sol/solana-validator.log {
  rotate 7
  daily
  missingok
  postrotate
    systemctl kill -s USR1 sol.service
  endscript
}
$ sudo systemctl restart logrotate

```

### 发布validator信息

我们可以把validator信息发布到链上，这样其他用户也可以查看到我们的一些信息

```
$ solana validator-info publish --keypair ~/deploy/sol/validator-keypair.json "Dark Devnet Validator"
Publishing info for Validator 6QpnVQNT1EhPFnCseSwSXbfZWhXniduMRVvEDxnJAhg6
Success! Validator info published at: 88dkzpAQVue79AhCpvg5r8fnnt5rgXzrRUyYQMN2yt75
2KuewbB9FnGDszwqV4ifnhntGwzmLA4YzQvFWgzkPsMNe3zpn1VsZUqyxQRiBbYupQ2LNFja3xCwopZGm5kH8PEE

```

查看节点信息

```
$ solana validator-info get
# 这里返回的是一个列表，需要过滤找一下自己对应节点的ID
Validator Identity: 6QpnVQNT1EhPFnCseSwSXbfZWhXniduMRVvEDxnJAhg6
  Info Address: 88dkzpAQVue79AhCpvg5r8fnnt5rgXzrRUyYQMN2yt75
  Name: Dark Devnet Validator

```

### 其他操作

等待一段时间后，可以检查下是否追上网络的其他节点，此过程可能需要很久，取决于你的网络情况

```
solana catchup ~/deploy/sol/validator-keypair.json

```

查看日志文件是否有错误信息

```
grep 'error|warn' ~/deploy/sol/solana-validator.log

```

验证我的节点是否已经在集群上了

```
solana-gossip spy --entrypoint entrypoint.devnet.solana.com:8001
# 过滤88dkzpAQVue79AhCpvg5r8fnnt5rgXzrRUyYQMN2yt75

```

监控节点信息，`Ctrl+C`可以退出

```
solana-validator --ledger ~/deploy/sol/validator-ledger monitor

```

显示我的节点的块生产和跳过的插槽

```
solana block-production | grep $(solana address)

```

将我当前时代的领导者日程表导出到文本文件：

```
solana leader-schedule | grep $(solana address) \\
  > ~/leader-schedule-epoch-$(solana epoch)-$(solana address).txt

```

查看epoch相关信息

```
$ solana epoch-info

Block height: 77586723
Slot: 78709787
Epoch: 182
Transaction Count: 1623421128
Epoch Slot Range: [78624000..79056000)
Epoch Completed Percent: 19.858%
Epoch Completed Slots: 85787/432000 (346213 remaining)
Epoch Completed Time: 9h 31m 29s/1day 23h 53m 47s (1day 14h 22m 18s remaining)

```

查看所有的validators

```
solana validators

```