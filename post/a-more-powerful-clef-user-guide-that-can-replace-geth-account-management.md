# 可替代Geth账户管理且功能更强大的Clef使用指南

[Clef](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef)用于签名交易、数据，Clef的出现最终目标就是为了替代Geth的帐户管理，官方教程可参考[Clef Tutorial](https://geth.ethereum.org/docs/clef/tutorial).

## 运行clef

- 创建一个Ubuntu 20.04的容器

用于安装`geth`和运行`clef`

```
docker run -d -it --restart always --name clef --hostname clef --network host -v $(pwd)/data:/data ubuntu:20.04

```

- 进入clef容器

```
docker exec -it clef bash

```

- 安装geth软件包

```
apt update && apt upgrade -y
apt install software-properties-common -y
add-apt-repository -y ppa:ethereum/ethereum
apt update && apt install ethereum -y

```

- 初始化

```
$ clef init --configdir /data/clef

WARNING!

Clef is an account management tool. It may, like any software, contain bugs.

Please take care to
- backup your keystore files,
- verify that the keystore(s) can be opened with your password.

Clef is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
PURPOSE. See the GNU General Public License for more details.

Enter 'ok' to proceed: # 输入ok
> ok

The master seed of clef will be locked with a password.
Please specify a password. Do not forget this password!
# 重复输入两次密码，一定要记得改密码
Password:
Repeat password:

A master seed has been generated into /data/clef/masterseed.json

This is required to be able to store credentials, such as:
* Passwords for keystores (used by rule engine)
* Storage for JavaScript auto-signing rules
* Hash of JavaScript rule-file

You should treat 'masterseed.json' with utmost secrecy and make a backup of it!
* The password is necessary but not enough, you need to back up the master seed too!
* The master seed does not contain your accounts, those need to be backed up separately!

```

生成的`masterseed.json`一定要备份，避免丢失，丢失之后私钥就找不回来了

- 创建keystor目录

```
mkdir -p /data/keystore

```

- 启动

此窗口不要关闭，需要挂在前台

```
$ clef --configdir /data/clef \\
    --keystore /data/keystore \\
    --advanced \\
    --chainid 3 \\
    --nousb \\
    --http \\
    --http.port 8550 \\
    --http.addr "0.0.0.0" \\
    --http.vhosts "*"

WARNING!

Clef is an account management tool. It may, like any software, contain bugs.

Please take care to
- backup your keystore files,
- verify that the keystore(s) can be opened with your password.

Clef is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
PURPOSE. See the GNU General Public License for more details.

Enter 'ok' to proceed: # 输出ok
> ok

INFO [12-28|11:10:23.648] Using CLI as UI-channel
INFO [12-28|11:10:23.817] Loaded 4byte database                    embeds=146,841 locals=0 local=./4byte-custom.json
## Master Password

Please enter the password to decrypt the master seed # 输出密码，就是刚才设置的
>
-----------------------
WARN [12-28|11:10:35.029] Failed to open master, rules disabled    err="failed to decrypt the master seed of clef"
INFO [12-28|11:10:35.029] Starting signer                          chainid=3 keystore=/data/keystore light-kdf=false advanced=false
DEBUG[12-28|11:10:35.029] FS scan times                            list="297.037µs" set=916ns diff="1.127µs"
INFO [12-28|11:10:35.029] Smartcard socket file missing, disabling err="stat /run/pcscd/pcscd.comm: no such file or directory"
INFO [12-28|11:10:35.030] Audit logs configured                    file=audit.log
INFO [12-28|11:10:35.031] HTTP endpoint opened                     url=http://[::]:8550/
DEBUG[12-28|11:10:35.032] IPCs registered                          namespaces=account
INFO [12-28|11:10:35.037] IPC endpoint opened                      url=/data/clef/clef.ipc
------- Signer info -------
* extapi_version : 6.1.0
* extapi_http : http://[::]:8550/
* extapi_ipc : /data/clef/clef.ipc
* intapi_version : 7.0.1

```

每次启动都会让你输入一次密码，启动之后会开放一个`8550端口`和一个`/data/clef/clef.ipc`的IPC文件，都是可以调用的。

## API调用

- ipc

此调用需要通过`nc`命令，需要提前安装

```
apt install netcat -y

```

通过`clef.ipc`的方式查询账户列表

```
echo '{"id": 1, "jsonrpc": "2.0", "method": "account_list"}' | nc -U /data/clef/clef.ipc

```

![Untitled](/images/2021/12/29/1.png)

从上面的图中我们可以看出，在第一步时候进行API调用，然后左边是需要审核此次请求是否被允许执行，输入`y`之后则批准这次请求，最后右边会输出`{"jsonrpc":"2.0","id":1,"result":[]}`，`ctrl+c`终止查询，result返回为空是因为我们还没有创建账户。

- http

在Linux下面可以通过curl指令构造http请求，请使用下面指令安装curl命令

```
apt install curl -y

```

下面是同样调用账户查询的API

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{"id": 1, "jsonrpc": "2.0", "method": "account_list"}'
{"jsonrpc":"2.0","id":1,"result":[]}

```

![Untitled](/images/2021/12/29/2.png)

与上面ipc的方式调用相同，只不过http是无状态的，所以查询到结果之后直接退出，不需要手动`ctrl+c`终止。

## API

下面每个API调用为了方便起见，我们统一使用http的方式。

- account_version(获取api版本)

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{ "id": 0, "jsonrpc": "2.0", "method": "account_version", "params": [] }'
{"jsonrpc":"2.0","id":0,"result":"6.1.0"}

```

- account_new(创建账号)

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{ "id": 0, "jsonrpc": "2.0", "method": "account_new", "params": [] }'
# result是返回的账户地址
{"jsonrpc":"2.0","id":0,"result":"0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb"}

```

以下是在`celf`的输出，会提示让你输出一个钱包的密码

```
-------- New Account request--------------

A request has been made to create a new account.
Approving this operation means that a new account is created,
and the address is returned to the external caller

Request context:
        [::1]:59800 -> HTTP/1.1 -> localhost:8550

Additional HTTP header data, provided by the external caller:
        User-Agent: "curl/7.68.0"
        Origin: ""
Approve? [y/N]:
> > > > > > > > > > > > > > > > > > > > > > > > > > > > y
## New account password

Please enter a password for the new account to be created (attempt 0 of 3)
>  # 输出账号密码
-----------------------
DEBUG[12-28|04:40:56.980] FS scan times                            list=3.010876ms  set="17.831µs" diff="1.353µs"
INFO [12-28|04:40:57.688] Your new key was generated               address=0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
WARN [12-28|04:40:57.688] Please backup your key file!             path=/data/keystore/UTC--2021-12-28T13-40-55.831036783Z--b1b198e85e21f382e38cdf8397ac062ee5f62bfb
WARN [12-28|04:40:57.688] Please remember your password!
DEBUG[12-28|04:40:57.688] Served account_new                       conn=[::1]:59800 reqid=0 duration=9.008684934s
DEBUG[12-28|04:40:58.192] FS scan times                            list=3.833642ms  set="10.692µs" diff="2.649µs"

```

也可以通过clef命令行创建account

```
$ clef --keystore /data/keystore newaccount

WARNING!

Clef is an account management tool. It may, like any software, contain bugs.

Please take care to
- backup your keystore files,
- verify that the keystore(s) can be opened with your password.

Clef is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
PURPOSE. See the GNU General Public License for more details.

Enter 'ok' to proceed:
> ok

INFO [12-28|05:04:42.772] Starting clef                            keystore=/data/keystore light-kdf=false
DEBUG[12-28|05:04:42.773] FS scan times                            list="568.555µs" set="7.199µs" diff="4.128µs"
## New account password

Please enter a password for the new account to be created (attempt 0 of 3)
>
-----------------------
DEBUG[12-28|05:04:45.769] FS scan times                            list=2.654262ms  set="16.977µs" diff="2.579µs"
INFO [12-28|05:04:45.908] Your new key was generated               address=0x3E8C911E3b473aaD5F8a11294BECCF7D979f8005
WARN [12-28|05:04:45.908] Please backup your key file!             path=/data/keystore/UTC--2021-12-28T14-04-44.627367092Z--3e8c911e3b473aad5f8a11294beccf7d979f8005
WARN [12-28|05:04:45.908] Please remember your password!
Generated account 0x3E8C911E3b473aaD5F8a11294BECCF7D979f8005

```

- account_list(可用账号列表)

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{"id": 1, "jsonrpc": "2.0", "method": "account_list"}'
# 我们刚才创建了两个账户，这里返回的result就不是空了
{"jsonrpc":"2.0","id":1,"result":["0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb","0x3e8c911e3b473aad5f8a11294beccf7d979f8005"]}

```

clef输出

```
-------- List Account request--------------
A request has been made to list all accounts.
You can select which accounts the caller can see
  [x] 0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
    URL: keystore:///data/keystore/UTC--2021-12-28T13-40-55.831036783Z--b1b198e85e21f382e38cdf8397ac062ee5f62bfb
  [x] 0x3E8C911E3b473aaD5F8a11294BECCF7D979f8005
    URL: keystore:///data/keystore/UTC--2021-12-28T14-04-44.627367092Z--3e8c911e3b473aad5f8a11294beccf7d979f8005
-------------------------------------------
Request context:
        [::1]:59808 -> HTTP/1.1 -> localhost:8550

Additional HTTP header data, provided by the external caller:
        User-Agent: "curl/7.68.0"
        Origin: ""
Approve? [y/N]:
> > > y
DEBUG[12-28|05:05:11.407] Served account_list                      conn=[::1]:59808 reqid=1 duration=2.939397812s

```

- account_signTransaction(签署交易)

这次来测试一笔转账，上面我们创建了两个账户，通过测试网有ETH的地址往`0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb`这个地址转账1个ETH，方便我们测试，以下是通过浏览器查询到的账户余额信息，有1个ETH。

![Untitled](/images/2021/12/29/3.png)

从`0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb`往`0x3e8c911e3b473aad5f8a11294beccf7d979f8005`地址转0.1个ETH，并不会真实转账，只是生成转账的签名

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{ "id": 2, "jsonrpc": "2.0", "method": "account_signTransaction", "params": [ { "from": "0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb", "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005", "gas": "0x333", "gasPrice": "0x1", "value": "0x5f5e100", "nonce": "0x1" } ] }'

{
    "jsonrpc": "2.0",
    "id": 2,
    "result": {
        "raw": "0xf8630101820333943e8c911e3b473aad5f8a11294beccf7d979f80058405f5e1008029a0916c12fb211c5649574c6092473b3f307a059a1cc79725aa5a2abcca0f87112aa04258bdd3053fa94cddc72324aef58e3b95f2eb3c3e40534d9708de066320bb28",
        "tx": {
            "type": "0x0",
            "nonce": "0x1",
            "gasPrice": "0x1",
            "maxPriorityFeePerGas": null,
            "maxFeePerGas": null,
            "gas": "0x333",
            "value": "0x5f5e100",
            "input": "0x",
            "v": "0x29",
            "r": "0x916c12fb211c5649574c6092473b3f307a059a1cc79725aa5a2abcca0f87112a",
            "s": "0x4258bdd3053fa94cddc72324aef58e3b95f2eb3c3e40534d9708de066320bb28",
            "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005",
            "hash": "0xfa07afcb370d0faa59984b01c918859d90da6a55b38ba9e6c17150226c09fd0b"
        }
    }
}

```

如果出现以下错误

```
{"jsonrpc":"2.0","id":2,"error":{"code":-32000,"message":"validation failed: Invalid checksum on recipient address"}}

```

这是因为验证机制默认比较强，结局办法则是在启动`clef`的时候加上`--advanced`参数，具体可以参考这个[issues](https://github.com/ethereum/go-ethereum/issues/20214)。

成功之后clef输出如下

```
--------- Transaction request-------------
to:    0x3e8c911e3b473aad5f8a11294beccf7d979f8005

WARNING: Invalid checksum on to-address!

from:               0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb [chksum INVALID]
value:              100000000 wei
gas:                0x333 (819)
gasprice: 1 wei
nonce:    0x1 (1)

Transaction validation:
  * WARNING : Invalid checksum on recipient address

Request context:
        [::1]:59824 -> HTTP/1.1 -> localhost:8550

Additional HTTP header data, provided by the external caller:
        User-Agent: "curl/7.68.0"
        Origin: ""
-------------------------------------------
Approve? [y/N]:
> > > > > y
WARN [12-28|05:47:57.988] Key does not exist                       key=0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
## Account password

Please enter the password for account 0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
>
-----------------------
Transaction signed:
 {
    "type": "0x0",
    "nonce": "0x1",
    "gasPrice": "0x1",
    "maxPriorityFeePerGas": null,
    "maxFeePerGas": null,
    "gas": "0x333",
    "value": "0x5f5e100",
    "input": "0x",
    "v": "0x29",
    "r": "0x916c12fb211c5649574c6092473b3f307a059a1cc79725aa5a2abcca0f87112a",
    "s": "0x4258bdd3053fa94cddc72324aef58e3b95f2eb3c3e40534d9708de066320bb28",
    "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005",
    "hash": "0xfa07afcb370d0faa59984b01c918859d90da6a55b38ba9e6c17150226c09fd0b"
  }
DEBUG[12-28|05:48:02.562] Served account_signTransaction           conn=[::1]:59824 reqid=2 duration=7.65674917s

```

ABI数据实例

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{"id": 67, "jsonrpc": "2.0", "method": "account_signTransaction", "params": [{ "from": "0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb", "gas": "0x333", "gasPrice": "0x1", "nonce": "0x0", "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005", "value": "0x0", "data": "0x4401a6e40000000000000000000000000000000000000000000000000000000000000012" }, "safeSend(address)" ]}'

{
    "jsonrpc": "2.0",
    "id": 67,
    "result": {
        "raw": "0xf8838001820333943e8c911e3b473aad5f8a11294beccf7d979f800580a44401a6e400000000000000000000000000000000000000000000000000000000000000122aa0a9def27900968d44f1266e7f4663e6d10269df847c7d5002ac11bd294010cbe6a068c07e2cdb6520ea7a437452219d8ffc5ea633fbd6ac89fd2c748d597cbb276b",
        "tx": {
            "type": "0x0",
            "nonce": "0x0",
            "gasPrice": "0x1",
            "maxPriorityFeePerGas": null,
            "maxFeePerGas": null,
            "gas": "0x333",
            "value": "0x0",
            "input": "0x4401a6e40000000000000000000000000000000000000000000000000000000000000012",
            "v": "0x2a",
            "r": "0xa9def27900968d44f1266e7f4663e6d10269df847c7d5002ac11bd294010cbe6",
            "s": "0x68c07e2cdb6520ea7a437452219d8ffc5ea633fbd6ac89fd2c748d597cbb276b",
            "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005",
            "hash": "0x2439b4ad1e180be5935e91e645d38aa51f6dceebbfd7ba06709d76b3f56a6b3e"
        }
    }
}

```

clef输出

```
--------- Transaction request-------------
to:    0x3e8c911e3b473aad5f8a11294beccf7d979f8005

WARNING: Invalid checksum on to-address!

from:               0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb [chksum INVALID]
value:              0 wei
gas:                0x333 (819)
gasprice: 1 wei
nonce:    0x0 (0)
data:     0x4401a6e40000000000000000000000000000000000000000000000000000000000000012

Transaction validation:
  * WARNING : Invalid checksum on recipient address
  * Info : Transaction invokes the following method: "safeSend(address: 0x0000000000000000000000000000000000000012)"

Request context:
        [::1]:59826 -> HTTP/1.1 -> localhost:8550

Additional HTTP header data, provided by the external caller:
        User-Agent: "curl/7.68.0"
        Origin: ""
-------------------------------------------
Approve? [y/N]:
> y
WARN [12-28|05:56:42.098] Key does not exist                       key=0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
## Account password

Please enter the password for account 0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
>
-----------------------
Transaction signed:
 {
    "type": "0x0",
    "nonce": "0x0",
    "gasPrice": "0x1",
    "maxPriorityFeePerGas": null,
    "maxFeePerGas": null,
    "gas": "0x333",
    "value": "0x0",
    "input": "0x4401a6e40000000000000000000000000000000000000000000000000000000000000012",
    "v": "0x2a",
    "r": "0xa9def27900968d44f1266e7f4663e6d10269df847c7d5002ac11bd294010cbe6",
    "s": "0x68c07e2cdb6520ea7a437452219d8ffc5ea633fbd6ac89fd2c748d597cbb276b",
    "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005",
    "hash": "0x2439b4ad1e180be5935e91e645d38aa51f6dceebbfd7ba06709d76b3f56a6b3e"
  }
DEBUG[12-28|05:56:45.146] Served account_signTransaction           conn=[::1]:59826 reqid=67 duration=12.092178359s

```

其中参数说明可以参考[account_signTransaction Arguments](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef#arguments-2)。

- account_signData(签名数据)

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{"id": 3, "jsonrpc": "2.0", "method": "account_signData", "params": [ "data/plain", "0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb", "0xaabbccdd" ]}'

{
    "jsonrpc": "2.0",
    "id": 3,
    "result": "0x47ccd19a967a3badc78dc2f6e36255698bf8bf1388d84796284e80efb735a8994b9596969971ecc699a831e469cd96b3c6e58ca03bdc0a00692e46a274502adf1c"
}

```

clef返回

```
-------- Sign data request--------------
Account:  0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb [chksum INVALID]
messages:
  message [text/plain]: "\\x19Ethereum Signed Message:\\n4\\xaa\\xbb\\xcc\\xdd"
raw data:
        "\\x19Ethereum Signed Message:\\n4\\xaa\\xbb\\xcc\\xdd"
data hash:  0xe35ba1e4664bb69c56eb414044a09c5f673aae2d54f29aafdd5978db1a643283
-------------------------------------------
Request context:
        [::1]:59828 -> HTTP/1.1 -> localhost:8550

Additional HTTP header data, provided by the external caller:
        User-Agent: "curl/7.68.0"
        Origin: ""
Approve? [y/N]:
> > > > > y
WARN [12-28|05:59:59.026] Key does not exist                       key=0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
## Password for signing

Please enter password for signing data with account 0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
>
-----------------------
DEBUG[12-28|06:00:01.760] Served account_signData                  conn=[::1]:59828 reqid=3  duration=5.852962967s

```

参数参考[account_signData Arguments](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef#arguments-3)。

- account_ecRecover(解析已签名数据对应的账号地址)

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{
  "id": 4,
  "jsonrpc": "2.0",
  "method": "account_ecRecover",
  "params": [
    "0xaabbccdd",
    "0x47ccd19a967a3badc78dc2f6e36255698bf8bf1388d84796284e80efb735a8994b9596969971ecc699a831e469cd96b3c6e58ca03bdc0a00692e46a274502adf1c"
  ]
}'
{"jsonrpc":"2.0","id":4,"result":"0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb"}

```

- account_signTypedData(对符合[EIP-712](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-712.md)的结构化数据进行签名并返回计算出的签名)

这块暂时没看懂和调通，具体可以参考下[account_signTypedData](https://github.com/ethereum/go-ethereum/tree/master/cmd/clef#account_signtypeddata)文档，后续如果有更新再补过来。

## 整和Geth

clef的出现就是为了替换掉geth的account管理，既然如此，那么geth自然很容易的支持clef对接。

我们在运行clef的时候，指定的网络ID是`--chainid 3`，也就是`Ropsten`网络，所以我们运行geth的时候也要指定`--ropsten`

- ipc方式

```
geth --ropsten --syncmode light --signer /data/clef/clef.ipc console

```

- http方式

```
geth --ropsten --syncmode light --signer <http://localhost:8550> console

```

- 测试

进行测试之前需要确保节点已经同步完毕，我们运行geth节点的时候使用的`light`模式，应该很快就会同步完毕，可以通过以下指令查询同步进度，当`currentBlock`和`highestBlock`一致，就表示同步完毕了

```
> eth.syncing
{
  currentBlock: 11475135,
  healedBytecodeBytes: 0,
  healedBytecodes: 0,
  healedTrienodeBytes: 0,
  healedTrienodes: 0,
  healingBytecode: 0,
  healingTrienodes: 0,
  highestBlock: 11710244,
  startingBlock: 11370495,
  syncedAccountBytes: 0,
  syncedAccounts: 0,
  syncedBytecodeBytes: 0,
  syncedBytecodes: 0,
  syncedStorage: 0,
  syncedStorageBytes: 0
}

```

或者当`eth.syncing`返回`false`的时候也是同步完毕

```
> eth.syncing
false

```

查询账户列表

```
> eth.accounts
["0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb", "0x3e8c911e3b473aad5f8a11294beccf7d979f8005"]

```

钱包列表

```
> personal.listWallets
[{
    accounts: [{
        address: "0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb",
        url: "extapi://http://localhost:8550"
    }, {
        address: "0x3e8c911e3b473aad5f8a11294beccf7d979f8005",
        url: "extapi://http://localhost:8550"
    }],
    status: "ok [version=6.1.0]",
    url: "extapi://http://localhost:8550"
}]

```

查询账户余额

```
> eth.getBalance(eth.accounts[0])
1000000000000000000
> eth.getBalance(eth.accounts[1])
0

```

进行一笔转账，从`0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb`往`0x3e8c911e3b473aad5f8a11294beccf7d979f8005`地址转0.1个ETH

```
> eth.sendTransaction({from: "0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb", to: "0x3e8c911e3b473aad5f8a11294beccf7d979f8005", value: "100000000000000000"})
"0xdcfe241c987f1b0a95df7e54b582d54055ba9745f17dd62c99711f91d2a14888"

```

进行转账时，clef输出如下

```
--------- Transaction request-------------
to:                 0x3E8C911E3b473aaD5F8a11294BECCF7D979f8005
from:               0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB [chksum ok]
value:              100000000000000000 wei
gas:                0x5208 (21000)
maxFeePerGas:          1500000016 wei
maxPriorityFeePerGas:  1500000000 wei
nonce:    0x0 (0)
chainid:  0x3
Accesslist

Request context:
        [::1]:60756 -> HTTP/1.1 -> localhost:8550

Additional HTTP header data, provided by the external caller:
        User-Agent: "Go-http-client/1.1"
        Origin: ""
-------------------------------------------
Approve? [y/N]:
> y
WARN [12-28|17:22:45.839] Key does not exist                       key=0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
## Account password

Please enter the password for account 0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB
>
-----------------------
Transaction signed:
 {
    "type": "0x2",
    "nonce": "0x0",
    "gasPrice": null,
    "maxPriorityFeePerGas": "0x59682f00",
    "maxFeePerGas": "0x59682f10",
    "gas": "0x5208",
    "value": "0x16345785d8a0000",
    "input": "0x",
    "v": "0x1",
    "r": "0xa2b8e86d865cdb0a8c4a64d6fe9fe224432e7d39cc6819ab1ee471779fa2597e",
    "s": "0x16848a4c4f50015d2fe67d68d25283fe7158de946ca14e6d34d49410c12db4c9",
    "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005",
    "chainId": "0x3",
    "accessList": [],
    "hash": "0xdcfe241c987f1b0a95df7e54b582d54055ba9745f17dd62c99711f91d2a14888"
  }
DEBUG[12-28|17:22:50.402] Served account_signTransaction           conn=[::1]:60756 reqid=9  duration=6.5552928s

```

再次查询账户余额

```
> eth.getBalance(eth.accounts[0])
899968499999832000
> eth.getBalance(eth.accounts[1])
100000000000000000

```

## 规则

上面的操作中，几乎每一次API调用都还有一个clef审批的流程，即每次调用，clef同意之后才会输出结果。

规则文件是存放在`js`文件中的，我们可以编写一些规则以实现一些自动化审批。

规则文件放在`/data/rules.js`中，内容如下：

```
$ vim /data/rules.js
// 如果是通过IPC提出请求，则批准，否则走审批
function ApproveListingIpc(req){
    if (req.metadata.scheme == "ipc"){ return "Approve"}
}

// 如果是查询则允许
function ApproveListing() {
    return "Approve"
}

// 如果转账地址from是 0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb ，则批准，否则走审批
function ApproveTx(r) {
    if (r.transaction.from.toLowerCase() == "0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb") {
        return "Approve"
    }
}

```

首先计算`rules.js`文件的sha256sum值并注册到clef中

```
$ sha256sum /data/rules.js
efeaba08ec916880e0447aca7c9ad1badea4760f46479d237b2cd52a31045de5  /data/rules.js

$ clef --configdir /data/clef --keystore /data/keystore attest efeaba08ec916880e0447aca7c9ad1badea4760f46479d237b2cd52a31045de5

WARNING!

Clef is an account management tool. It may, like any software, contain bugs.

Please take care to
- backup your keystore files,
- verify that the keystore(s) can be opened with your password.

Clef is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
PURPOSE. See the GNU General Public License for more details.

Enter 'ok' to proceed:
> ok

Decrypt master seed of clef
Password:
INFO [12-28|18:09:09.360] Ruleset attestation updated              sha256=efeaba08ec916880e0447aca7c9ad1badea4760f46479d237b2cd52a31045de5

```

然后在启动clef的时候，指定`rules.js`文件

```
$ clef --configdir /data/clef \\
    --keystore /data/keystore \\
    --advanced \\
    --chainid 3 \\
    --nousb \\
    --http \\
    --http.port 8550 \\
    --http.addr "0.0.0.0" \\
    --http.vhosts "*" \\
    --rules /data/rules.js

```

测试规则1

```
$ echo '{"id": 1, "jsonrpc": "2.0", "method": "account_list"}' | nc -U /data/clef/clef.ipc
{"jsonrpc":"2.0","id":1,"result":["0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb","0x3e8c911e3b473aad5f8a11294beccf7d979f8005"]}
^C

```

测试规则2

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{"id": 1, "jsonrpc": "2.0", "method": "account_list"}'
{"jsonrpc":"2.0","id":1,"result":["0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb","0x3e8c911e3b473aad5f8a11294beccf7d979f8005"]}

```

测试规则3

由于规则3涉及到交易转账，所以我们需要把`0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb`地址的密码先添加到clef

```
$ clef --configdir /data/clef --keystore /data/keystore setpw 0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb

WARNING!

Clef is an account management tool. It may, like any software, contain bugs.

Please take care to
- backup your keystore files,
- verify that the keystore(s) can be opened with your password.

Clef is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY;
without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR
PURPOSE. See the GNU General Public License for more details.

Enter 'ok' to proceed:
> ok

Please enter a password to store for this address:
Password:
Repeat password:

Decrypt master seed of clef
Password:
INFO [12-28|18:19:39.146] Credential store updated                 set=0xB1B198e85e21F382e38cDF8397ac062Ee5F62bfB

```

当from=`0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb`时候，会自动批准交易

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{ "id": 2, "jsonrpc": "2.0", "method": "account_signTransaction", "params": [ { "from": "0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb", "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005", "gas": "0x333", "gasPrice": "0x1", "value": "0x5f5e100", "nonce": "0x1" } ] }'

{
    "jsonrpc": "2.0",
    "id": 2,
    "result": {
        "raw": "0xf8630101820333943e8c911e3b473aad5f8a11294beccf7d979f80058405f5e1008029a0916c12fb211c5649574c6092473b3f307a059a1cc79725aa5a2abcca0f87112aa04258bdd3053fa94cddc72324aef58e3b95f2eb3c3e40534d9708de066320bb28",
        "tx": {
            "type": "0x0",
            "nonce": "0x1",
            "gasPrice": "0x1",
            "maxPriorityFeePerGas": null,
            "maxFeePerGas": null,
            "gas": "0x333",
            "value": "0x5f5e100",
            "input": "0x",
            "v": "0x29",
            "r": "0x916c12fb211c5649574c6092473b3f307a059a1cc79725aa5a2abcca0f87112a",
            "s": "0x4258bdd3053fa94cddc72324aef58e3b95f2eb3c3e40534d9708de066320bb28",
            "to": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005",
            "hash": "0xfa07afcb370d0faa59984b01c918859d90da6a55b38ba9e6c17150226c09fd0b"
        }
    }
}

```

当from=`0x3e8c911e3b473aad5f8a11294beccf7d979f8005`时候，会走审批流程

```
$ curl <http://localhost:8550> \\
  -X POST \\
  -H "Content-Type: application/json" \\
  --data '{ "id": 2, "jsonrpc": "2.0", "method": "account_signTransaction", "params": [ { "from": "0x3e8c911e3b473aad5f8a11294beccf7d979f8005", "to": "0xb1b198e85e21f382e38cdf8397ac062ee5f62bfb", "gas": "0x333", "gasPrice": "0x1", "value": "0x5f5e100", "nonce": "0x1" } ] }'

```

![Untitled](/images/2021/12/29/4.png)