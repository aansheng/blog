# OpenZeppelin快速部署兼容EVM链的智能合约(以FTM链为例)

[OpenZeppelin](https://openzeppelin.com/)是一套部署智能合约的脚手架，很多事情都帮你做好了，而且还集成了非常多常用的功能，我们只需要专注于智能合约的编写即可。

## 项目初始化

OpenZeppelin用JS编写，所以我们需要安装装[node和npm](https://nodejs.org/zh-cn/)，我这里已经安装好了

```
$ node --version
v16.13.1
$ npm --version
8.2.0

```

- 创建项目

```
cd WorkSpaces && mkdir learn && cd learn

```

- npm初始化

```
$ npm init -y
Wrote to /Users/ansheng/WorkSpaces/learn/package.json:

{
  "name": "learn",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \\"Error: no test specified\\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}

```

- truffle

truffle也是智能合约的开发工具，全局安装truffle，带上版本号不会导致卡顿

```
sudo npm install -g truffle@5.4.24

```

初始化

```
truffle init

```

你也可以使用npx初始化，但是我不知道为什么在我的环境下会很卡，最后初始化失败

```
npx truffle init

```

- Hardhat

安装

```
npm install --save-dev hardhat

```

创建默认的配置文件，一路回车即可，感兴趣可以看下每个选项的说明

```
npx hardhat

```

- 目录结构

```
$ tree -L 1 -I node_modules ./
./
├── README.md
├── contracts  # 智能合约目录
├── hardhat.config.js  # hardhat配置文件
├── migrations  # 升级智能合约目录
├── package-lock.json
├── package.json
├── scripts  # 脚本
├── test  # 测试
└── truffle-config.js  # truffle配置文件

```

contracts、migrations、scripts、test默认目录下的文件我们可以先删掉，以保持一个干净的目录结构

```
rm -f contracts/*
rm -f migrations/*
rm -f scripts/*
rm -f test/*

```

## 编写智能合约

EMV链下的智能合约使用[Solidity](https://soliditylang.org/)语言编写，更多信息可以参考[官方文档](https://docs.soliditylang.org/)，这里不做过多的阐述。

- 创建智能合约

智能合约的代码文件放在`contracts`目录下，以`.sol`结尾，表明是使用Solidity编写，这里我继续使用官方文档给的例子。

我们将创建一个名为`Box`的合约，这个合约的主要功能是可以在里面存放一个值，然后可以读取这个值，代码如下

```
$ vim contracts/Box.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Box {
    uint256 private _value;

    // Emitted when the stored value changes
    event ValueChanged(uint256 value);

    // Stores a new value in the contract
    function store(uint256 value) public {
        _value = value;
        emit ValueChanged(value);
    }

    // Reads the last stored value
    function retrieve() public view returns (uint256) {
        return _value;
    }
}

```

- 编译Solidity

EVM无法直接执行Solidity代码，我们需要先编译成EMV字节码，上面的合约代码中我们指定了solidity的版本是0.8，所以我们要把需要的版本添加到hardhat.config.js配置文件中

```
$ vim hardhat.config.js
......
module.exports = {
  solidity: "0.8.4", // 默认已添加，如果没有添加需要自己指定下
};

```

运行`npx hardhat compile`指令编译

```
$ npx hardhat compile
Compiling 1 file with 0.8.4
Compilation finished successfully

```

在编译时会自动查找`contracts`目录下所有以`.sol`结尾的文件，编译之后的json文件会保存在`artifacts`目录下。

## 部署

### 配置链

- 本地链

链有很多主链和测试链，但是我们在开发功能过程中最好还是使用本地链，在本机运行且无需访问互联网，Hardhat为我们提供了`Hardhat Network`，方便我们测试开发

```
npx hardhat node

```

运行起来之后会创建一批ETH地址，这些地址都是有余额的，需要注意的是每次运行`npx hardhat node`都会创建一个全新的本地链，这也就导致它不会保存上次启动的状态包括ETH地址和合约内容，所以我们需要把这个窗口一直挂起不要退出。

- FTM测试链

正如本文所示，我们会把合约部署到FTM测试网，在配置之前，我们需要创建一个新的账户，如下会创建一组助记词

```
$ npx mnemonics
across indoor end predict cushion person market loyal notable project grit turkey

```

然后我们把助记词保存到secrets.json文件中

```
$ vim secrets.json
{
  "mnemonic": "across indoor end predict cushion person market loyal notable project grit turkey"  # 助记词
}

```

配置网络链接

```
$ vim hardhat.config.js
......
const { mnemonic } = require('./secrets.json');

module.exports = {
  solidity: "0.8.4",
  networks: {
    ftmtestnet: {
      url: '<https://rpc.testnet.fantom.network>',
      accounts: { mnemonic: mnemonic },
    },
  },
};

```

- 进入测试网络控制台

```
$ npx hardhat console --network ftmtestnet
Welcome to Node.js v16.13.1.
Type ".help" for more information.
# 获取所有账户列表
> accounts = await ethers.provider.listAccounts()
[
  '0x56CaE3187906507AF6a282a964CDc4A3fD7380BA',
  '0x0484d9593A5480F27026e43Cdd9C671BdD02aA52',
  '0xacb06fCbA5314b311450E6D40148DEcc83B69E56',
  '0xe24d9fFB8D2b722D04869a71b30C14d25D614e09',
  '0xD04a5ce6b19884F53961F2e4934C9f3054623eDe',
  '0x35DA03f453730a195de7dD001f4c0fF8763Bb4c8',
  '0xdCDAC1945bb9b58f39969326ae1dC386D214576d',
  '0x66FacfA51b24cfB0Ac2F2fd62f37DeAa3896f530',
  '0x0417e8F19432BDE2Afe1ABA3c425480373f1b3e4',
  '0x2AC5C51f1D7296756d72B5eAd2bd10FF3042Ab3b',
  '0xfE528bA96CBc5fA55C5eD54616d67Af1148748b6',
  '0x45Ce3e96f11702eA4acea88865D434B7f3700605',
  '0xA26e3BeE88E1AAc1452281E2fe6Cb2c55d36200b',
  '0x760d3782A660e55f6BFfA58D78c6c626c80981c7',
  '0x2e556261C051CE5bA9A4DF54d4c470C75415206F',
  '0x079C734aA13a798ff7D66497e323A5Be67cdc16C',
  '0x37665BD85863Fa6dB003b376ad9Cb2CEbB3d3507',
  '0x0e3FbE488c128E01a8A9c8a12e82347396620a19',
  '0x96afEd2eAfBF9e4EB1001E30894d82d834E03b2b',
  '0x68B7a24F2A171dB2CD030C267bA2455a952Ab35D'
]
# 获取第一个账户的余额，默认是0
> (await ethers.provider.getBalance(accounts[0])).toString()
'0'

```

下面我们需要去[FTM测试网的水龙头](https://faucet.fantom.network/)领一些测试币，这样再部署合约的时候才可以成功，因为要烧GAS，地址我们用`0x56CaE3187906507AF6a282a964CDc4A3fD7380BA`这个

![Untitled](/images/2021/12/13/1.png)

然后再次查询账户余额

```
> (await ethers.provider.getBalance(accounts[0])).toString()
'10000000000000000000'

```

### 部署智能合约

默认情况下，智能合约是不可变的，也就说无法升级、更新、修复BUG，但是通过[OpenZeppelin Upgrades插件](https://docs.openzeppelin.com/upgrades-plugins/1.x/)可以升级代码。

- 安装升级插件

```
npm install --save-dev @openzeppelin/hardhat-upgrades

```

导入插件

```
$ vim hardhat.config.js
require('@openzeppelin/hardhat-upgrades');
......

```

- 创建部署脚本

```
$ vim scripts/deploy_upgradeable_box.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const Box = await ethers.getContractFactory('Box');
  console.log('Deploying Box...');
  const box = await upgrades.deployProxy(Box, [42], { initializer: 'store' });
  await box.deployed();
  console.log('Box deployed to:', box.address);
}

main();

```

在脚本中使用了`ethers`，所以我们需要安装`ethers`包

```
npm install --save-dev @nomiclabs/hardhat-ethers ethers

```

导入ethers包

```
$ vim hardhat.config.js
require('@nomiclabs/hardhat-ethers');
......

```

使用该run命令，我们可以将Box合约部署到FTM测试网络

```
$ npx hardhat run --network ftmtestnet scripts/deploy_upgradeable_box.js
# npx hardhat run --network localhost scripts/deploy_upgradeable_box.js 如果是本地网络把ftmtestnet改为localhost
Compiling 1 file with 0.8.4
Compilation finished successfully
Deploying Box...
Box deployed to: 0x8bC156B15cE8AD955c20Af75F5392850491C8094

```

[0x8bC156B15cE8AD955c20Af75F5392850491C8094](https://testnet.ftmscan.com/address/0x8bC156B15cE8AD955c20Af75F5392850491C8094)这是我们的合约地址

## 与合约交互

- 控制台

我们将使用Hardhat控制台与我们Box在本地主机网络上部署的合约进行交互

```
$ npx hardhat console --network ftmtestnet
Welcome to Node.js v16.13.1.
Type ".help" for more information.
> const Box = await ethers.getContractFactory('Box');
undefined
> const box = await Box.attach('0x8bC156B15cE8AD955c20Af75F5392850491C8094');
undefined
>

```

我们在部署合约的时候，初始化的值为42

```
> (await box.retrieve()).toString()
'42'

```

我们在Box合约内存储一个值，这个时候会有transactions，所以会烧掉一些GAS

```
> await box.store(999)
{
  hash: '0xf5bab2f9dbc66897b09851abb1e4f0ad818bf0ed1dbc00ec8f9e069683464cdb',
  type: 0,
  accessList: null,
  blockHash: '0x0000161e0000696ad650c8ec2037c203c7c664ad7f2803eb4360795eba53589b',
  blockNumber: 5873294,
  transactionIndex: 0,
  confirmations: 2,
  from: '0x56CaE3187906507AF6a282a964CDc4A3fD7380BA',
  gasPrice: BigNumber { value: "1500085500" },
  gasLimit: BigNumber { value: "35221" },
  to: '0x8bC156B15cE8AD955c20Af75F5392850491C8094',
  value: BigNumber { value: "0" },
  nonce: 3,
  data: '0x6057361d00000000000000000000000000000000000000000000000000000000000003e7',
  r: '0x322390136bddc3b67accdf35ce0e919b7565371f37c9bfd42eccc92c26037a6e',
  s: '0x09d8e11ded2136c2d4b156a13f129a2efa074783a624a0a340f25b80b62dff1d',
  v: 8039,
  creates: null,
  chainId: 4002,
  wait: [Function (anonymous)]
}

```

查询值，查询过程是不是不需要有交易的，既不会烧GAS

```
> (await box.retrieve()).toString()
'999'

```

- 编程

控制台适合做DEBUG，实际的业务中还是需要写代码来进行查询

```
$ vim scripts/index.js
async function main () {
  // 查询本地节点的所有账户列表
  const accounts = await ethers.provider.listAccounts();
  console.log(accounts);

  // 获取合约实例
  const address = '0x8bC156B15cE8AD955c20Af75F5392850491C8094';
  const Box = await ethers.getContractFactory('Box');
  const box = await Box.attach(address);

  // 发送交易
  await box.store(10);

  // 调用查询
  const value = await box.retrieve();
  console.log('Box value is', value.toString());
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });

```

执行

```
$ npx hardhat run --network ftmtestnet ./scripts/index.js
[
  '0x56CaE3187906507AF6a282a964CDc4A3fD7380BA',
  '0x0484d9593A5480F27026e43Cdd9C671BdD02aA52',
  '0xacb06fCbA5314b311450E6D40148DEcc83B69E56',
  '0xe24d9fFB8D2b722D04869a71b30C14d25D614e09',
  '0xD04a5ce6b19884F53961F2e4934C9f3054623eDe',
  '0x35DA03f453730a195de7dD001f4c0fF8763Bb4c8',
  '0xdCDAC1945bb9b58f39969326ae1dC386D214576d',
  '0x66FacfA51b24cfB0Ac2F2fd62f37DeAa3896f530',
  '0x0417e8F19432BDE2Afe1ABA3c425480373f1b3e4',
  '0x2AC5C51f1D7296756d72B5eAd2bd10FF3042Ab3b',
  '0xfE528bA96CBc5fA55C5eD54616d67Af1148748b6',
  '0x45Ce3e96f11702eA4acea88865D434B7f3700605',
  '0xA26e3BeE88E1AAc1452281E2fe6Cb2c55d36200b',
  '0x760d3782A660e55f6BFfA58D78c6c626c80981c7',
  '0x2e556261C051CE5bA9A4DF54d4c470C75415206F',
  '0x079C734aA13a798ff7D66497e323A5Be67cdc16C',
  '0x37665BD85863Fa6dB003b376ad9Cb2CEbB3d3507',
  '0x0e3FbE488c128E01a8A9c8a12e82347396620a19',
  '0x96afEd2eAfBF9e4EB1001E30894d82d834E03b2b',
  '0x68B7a24F2A171dB2CD030C267bA2455a952Ab35D'
]
Box value is 10

```

## 测试

在智能合约中编写自动化测试是相当有必要的，有时一个很小的错误都有可能导致资金被盗丢失等。

- 安装OpenZeppelin Contracts tests

```
npm install --save-dev @openzeppelin/test-environment
npm install --save-dev mocha chai

```

- 编写单元测试

测试文件保存在test目录中，我们在命名脚本文件的时候最好根据contracts目录下的合约名命名，例如以下Box实例

```
$ vim test/Box.test.js

// 加载依赖包
const { accounts, contract } = require('@openzeppelin/test-environment');
const { expect } = require('chai');

// Load compiled artifacts
const Box = contract.fromArtifact('Box');

// Start test block
describe('Box', function () {
  const [ owner ] = accounts;

  beforeEach(async function () {
    // Deploy a new Box contract for each test
    this.contract = await Box.new({ from: owner });
  });

  // Test case
  it('retrieve returns a value previously stored', async function () {
    // Store a value - recall that only the owner account can do this!
    await this.contract.store(42, { from: owner });

    // Test if the returned value is the same one
    // Note that we need to use strings to compare the 256 bit integers
    expect((await this.contract.retrieve()).toString()).to.equal('42');
  });
});

```

- 添加npm快捷指令

```
$ vim package.json
  "scripts": {
    "test": "truffle compile && mocha --exit --recursive"
  },

```

- 运行测试

```
$ npm test

> learn@1.0.0 test
> truffle compile && mocha --exit --recursive

  Box
    ✔ retrieve returns a value previously stored (61ms)

  1 passing (159ms)

```

## 升级智能合约

- 为合约增加特性

```
$ vim contracts/BoxV2.sol
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract BoxV2 {
    // ... code from Box.sol

    // Increments the stored value by 1
    function increment() public {
        _value = _value + 1;
        emit ValueChanged(_value);
    }
}

```

- 创建升级脚本

```
$ vim scripts/upgrade_box.js
const { ethers, upgrades } = require('hardhat');

async function main () {
  const BoxV2 = await ethers.getContractFactory('BoxV2');
  console.log('Upgrading Box...');
  await upgrades.upgradeProxy('0x8bC156B15cE8AD955c20Af75F5392850491C8094', BoxV2);
  console.log('Box upgraded');
}

main();

```

- 升级

```
$ npx hardhat run --network ftmtestnet scripts/upgrade_box.js
Compiling 1 file with 0.8.4
Compilation finished successfully
Upgrading Box...
Box upgraded

```

- 测试新代码是否生效

```
$ npx hardhat console --network ftmtestnet
Welcome to Node.js v16.13.1.
Type ".help" for more information.
> const BoxV2 = await ethers.getContractFactory('BoxV2');
undefined
> const box = await BoxV2.attach('0x8bC156B15cE8AD955c20Af75F5392850491C8094');
undefined
> (await box.retrieve()).toString();
'10'
> await box.increment();
{
  hash: '0xad8c010886a3549f19879d3fa447787620c2efe1fe9604153896f00b9c3d96db',
  type: 0,
  accessList: null,
  blockHash: '0x0000161e000070960c21e9c4d94e543130b0cd3a3d15056f1c0b68222fa102af',
  blockNumber: 5873486,
  transactionIndex: 0,
  confirmations: 2,
  from: '0x56CaE3187906507AF6a282a964CDc4A3fD7380BA',
  gasPrice: BigNumber { value: "1500085500" },
  gasLimit: BigNumber { value: "35121" },
  to: '0x8bC156B15cE8AD955c20Af75F5392850491C8094',
  value: BigNumber { value: "0" },
  nonce: 7,
  data: '0xd09de08a',
  r: '0x46ed44224170b897c7da546ba830d94592f29f1b4d23abf33859fd05c9fb8a2b',
  s: '0x1f57ff8be9ddf089604056b525b2d5a466497c06a1650b1d49919d42dce1cf83',
  v: 8040,
  creates: null,
  chainId: 4002,
  wait: [Function (anonymous)]
}
> (await box.retrieve()).toString();
'11'

```

## 使用OpenZeppelin Contracts

[OpenZeppelin Contracts](https://docs.openzeppelin.com/contracts/4.x/)为我们提供了很多可以复用的模块，而且这些模块时经过安全性检验的，我们可以放心的使用。

下载OpenZeppelin Contracts

```
npm install --save-dev @openzeppelin/contracts

```

具体模块使用请参考[官方文档](https://docs.openzeppelin.com/contracts/4.x/).