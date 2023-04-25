# 从源码构建Solana(索拉纳)

通常情况下我们是不需要从源码进行构建的，从[Github](https://github.com/solana-labs/solana/releases)上面现在二进制文件包即可，但是有时我们的CPU不支持某些指令集的时候，就需要从源构建。

## 初始安装

我这里使用的`Ubuntu 20.04`的系统进行操作

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 20.04.2 LTS
Release:	20.04
Codename:	focal

```

### 安装依赖包

安装编译所需的软件包

```
sudo apt update && sudo apt install -y \\
  libssl-dev libudev-dev pkg-config zlib1g-dev llvm clang make gcc curl git

```

我们创建一个`sol`用户并切换过去，后续的操作都在这用户下面，包括以后运行Solana服务也是

```
adduser sol
sudo su - sol

```

Solana是用rust写的，安装Rust是必不可少的

```
curl --proto '=https' --tlsv1.2 -sSf <https://sh.rustup.rs> | sh

```

安装完成之后需要加在rust相关命令的PATH路径，默认会写入到`~/.bashrc`中

```
source ~/.bashrc

```

检查一下rust版本以确保安装正常

```
$ rustc --version
rustc 1.54.0 (a178d0322 2021-07-26)

$ rustup --version
rustup 1.24.3 (ce5817a94 2021-05-31)
info: This is the version for the rustup toolchain manager, not the rustc compiler.
info: The currently active `rustc` version is `rustc 1.54.0 (a178d0322 2021-07-26)`

$ cargo --version
cargo 1.54.0 (5ae8d74b3 2021-06-22)

```

### 编译Solana

我们已经安装好了编译所需的软件，下面开始编译Solana，我们这里用主网的`1.6.20`版本

```
mkdir ~/dist
git clone -b v1.6.20 <https://github.com/solana-labs/solana.git> ~/dist/solana-src-v1.6.20

```

Solana已经为我们准备好了安装脚本，下面的命令，Solana安装路径为`~/.local/share/solana/install/releases/1.6.20`

```
~/dist/solana-src-v1.6.20/scripts/cargo-install-all.sh ~/.local/share/solana/install/releases/1.6.20

```

编译完成之后我们创建一个软链，以便于我们后续升级之下更少的操作

```
ln -s ~/.local/share/solana/install/releases/1.6.20 ~/.local/share/solana/install/active_release

```

添加PATH环境变量并激活

```
echo 'export PATH=$HOME/.local/share/solana/install/active_release/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

```

查看Solana版本

```
$ solana --version
solana-cli 1.6.20 (src:devbuild; feat:3316993441)

```

删除多余的文件

```
rm -fr ~/dist/solana-src-v1.6.20

```

## 升级

升级和编译的部分几乎相同

```
git clone -b v1.6.22 <https://github.com/solana-labs/solana.git> ~/dist/solana-src-v1.6.22
~/dist/solana-src-v1.6.22/scripts/cargo-install-all.sh ~/.local/share/solana/install/releases/1.6.22

```

我们需要吧旧的软链移除掉，然后把`~/.local/share/solana/install/active_release`链接到新的版本

```
rm -fr ~/.local/share/solana/install/active_release
ln -s ~/.local/share/solana/install/releases/1.6.22 ~/.local/share/solana/install/active_release

```

查看Solana版本

```bash
$ solana --version
solana-cli 1.6.22 (src:devbuild; feat:3316993441)

```