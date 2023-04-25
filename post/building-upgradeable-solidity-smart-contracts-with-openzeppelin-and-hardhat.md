# 使用OpenZeppelin和HardHat构建可升级的Solidity智能合约

## 项目初始化

- 创建项目目录

```bash
$ mkdir ~/WorkSpaces/mycontract && cd ~/WorkSpaces/mycontract && npm init -y
Wrote to /Users/ansheng/WorkSpaces/mycontract/package.json:

{
  "name": "mycontract",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

- 初始化一个空的hardhat项目

```bash
$ npm install --save-dev hardhat

added 299 packages, and audited 300 packages in 25s

53 packages are looking for funding
  run `npm fund` for details

found 0 vulnerabilities
$ npx hardhat
888    888                      888 888               888
888    888                      888 888               888
888    888                      888 888               888
8888888888  8888b.  888d888 .d88888 88888b.   8888b.  888888
888    888     "88b 888P"  d88" 888 888 "88b     "88b 888
888    888 .d888888 888    888  888 888  888 .d888888 888
888    888 888  888 888    Y88b 888 888  888 888  888 Y88b.
888    888 "Y888888 888     "Y88888 888  888 "Y888888  "Y888

👷 Welcome to Hardhat v2.9.3 👷‍

✔ What do you want to do? · Create an empty hardhat.config.js
✨ Config file created ✨
```

- 安装依赖包

```bash
$ npm install --save-dev @openzeppelin/hardhat-upgrades @nomiclabs/hardhat-ethers @nomiclabs/hardhat-etherscan @openzeppelin/contracts-upgradeable ethers chai
```

- 修改hardhat配置文件

加载需要的软件包，已经配置APIKEY和网络信息

```bash
$ vim hardhat.config.js
require("@nomiclabs/hardhat-ethers");
require('@openzeppelin/hardhat-upgrades');
require("@nomiclabs/hardhat-etherscan");

const RINKEBY_URL = <YOUR_RINKEBY_URL>
const RINKEBY_MNEMONIC = <YOUR_RINKEBY_MNEMONIC>
const ETHERSCAN_RINKEBY_KEY = <YOUR_ETHERSCAN_RINKEBY_KEY>

const RINKEBY_URL = "http://192.168.2.22:8545"
const RINKEBY_MNEMONIC = "will talk orient adult diary shield pepper frown way vault stick machine belt manage venture one erupt reflect stamp humor chef require sight cricket"
const ETHERSCAN_RINKEBY_KEY = "JMU1YQ4XJZHKDITD7K9IJ46R9BIIADZIFA"

/**
 * @type import('hardhat/config').HardhatUserConfig
 */
module.exports = {
  solidity: "0.8.13",
  networks: {
    rinkeby: {
      url: RINKEBY_URL,
      accounts: { mnemonic: RINKEBY_MNEMONIC }
    }
  },
  etherscan: {
    apiKey: {
      rinkeby: ETHERSCAN_RINKEBY_KEY
    }
  },
};
```

`ETHERSCAN_RINKEBY_KEY`可以在`https://etherscan.io`注册账户并获取API-Keys。

## 创建V1版本智能合约

我们将使用`remix`提供的演示合约`Storage`，然后将其放在contracts目录下

```bash
$ mkdir contracts && vim contracts/StorageV1.sol
// SPDX-License-Identifier: MIT
pragma solidity >=0.7.0 <0.9.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract StorageV1 is Initializable {
    uint256 number;

    function initialize(uint256 _number) public initializer {
        number = _number;
    }

    function store(uint256 num) public {
        number = num;
    }

    function retrieve() public view returns (uint256) {
        return number;
    }
}
```

合约集成至`Initializable`，`constructor`方法被`initialize`替代

- 创建测试脚本，放在`test`目录下

```bash
$ mkdir test && vim test/StorageV1.js
const { expect } = require('chai');

let StorageV1;
let storageV1;

describe('StorageV1', function () {
  beforeEach(async function () {
    StorageV1 = await ethers.getContractFactory("StorageV1");
    storageV1 = await StorageV1.deploy();
    await storageV1.deployed();
  });

  it('retrieve returns a value previously stored', async function () {
    await storageV1.store(42);

    expect((await storageV1.retrieve()).toString()).to.equal('42');
  });
});
```

- 创建代理测试

```bash
$ vim test/StorageV1.proxy.js
const { expect } = require('chai');

let StorageV1;
let storageV1;

describe('StorageV1 (proxy)', function () {
  beforeEach(async function () {
    StorageV1 = await ethers.getContractFactory("StorageV1");
    storageV1 = await upgrades.deployProxy(StorageV1, [9]);
    // storageV1 = await upgrades.deployProxy(StorageV1, [9], { initializer: 'store' });
  });

  it('retrieve returns a value previously initialized', async function () {
    expect((await storageV1.retrieve()).toString()).to.equal('9');
  });
});
```

- 运行测试

```bash
$ npx hardhat test

StorageV1
    ✔ retrieve returns a value previously stored

  StorageV1 (proxy)
    ✔ retrieve returns a value previously initialized

  2 passing (399ms)
```

## 将智能合约通过代理部署到链上

- 在scripts目录下创建部署脚本

```bash
$ mkdir scripts && vim scripts/deployProxy.js
async function main() {
  const StorageV1 = await ethers.getContractFactory("StorageV1");
  console.log("Deploying Storage...");
  const proxy = await upgrades.deployProxy(StorageV1, [42]);
  console.log("Proxy deployed to:", proxy.address);
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

- 部署合约

```bash
$ npx hardhat run --network rinkeby scripts/deployProxy.js
Deploying Storage...
Proxy deployed to: 0xB4E5E4D4715D5B7F82E83201fDdC1935DbEEd87D
```

- 提交合约验证

打开`https://rinkeby.etherscan.io/address/0xB4E5E4D4715D5B7F82E83201fDdC1935DbEEd87D#code`，将地址改为自己的代理合约地址，找到`More Options`—>点击`Is this a proxy?`

![Untitled](/images/2022/04/25/1.png)

打开新页面之后点击`Verify`

![Untitled](/images/2022/04/25/2.png)

之后会弹出一个新页面，会给一个执行合约的地址，也就是`StorageV1`合约的地址，然后我们复制此地址

![Untitled](/images/2022/04/25/3.png)

进行验证

```bash
$ npx hardhat verify --network rinkeby YOUR_STORAGE_V1_IMPLEMENTATION_ADDRESS
Nothing to compile
Successfully submitted source code for contract
contracts/StorageV1.sol:StorageV1 at 0xd8b3f7dcbc606a41d1a68dc2bae967a5fa8e958a
for verification on the block explorer. Waiting for verification result...

Successfully verified contract StorageV1 on Etherscan.
https://rinkeby.etherscan.io/address/0xd8b3f7dcbc606a41d1a68dc2bae967a5fa8e958a#code
```

再次打开`Verify`将会提示已验证

![Untitled](/images/2022/04/25/4.png)

点击`Save`之后在返回`https://rinkeby.etherscan.io/address/0xB4E5E4D4715D5B7F82E83201fDdC1935DbEEd87D#code`会发现多了一个Read as Proxy和Write as Proxy，这两个对应我们普通合约的`Read Contract`和`Write Contract`

![Untitled](/images/2022/04/25/5.png)

- 进入hardhat控制台通过代理合约进行交互

```bash
$ npx hardhat console --network rinkeby
Welcome to Node.js v16.14.1.
Type ".help" for more information.
> const StorageV1 = await ethers.getContractFactory("StorageV1")
undefined
> const storageV1 = await StorageV1.attach("0xB4E5E4D4715D5B7F82E83201fDdC1935DbEEd87D")
undefined
> (await storageV1.retrieve()).toString()
'42'
> await storageV1.store(9)
{
  hash: '0x2c8d66794c4d9a059d8f774d55d36c32e88ccaadc0f2200099184b141965ecae',
  type: 2,
  accessList: [],
  blockHash: null,
  blockNumber: null,
  transactionIndex: null,
  confirmations: 0,
  from: '0x3247EA903162fB3CD5B612D4F0AcA92e6Eb623BD',
  gasPrice: BigNumber { value: "2500000020" },
  maxPriorityFeePerGas: BigNumber { value: "2500000000" },
  maxFeePerGas: BigNumber { value: "2500000020" },
  gasLimit: BigNumber { value: "34000" },
  to: '0xB4E5E4D4715D5B7F82E83201fDdC1935DbEEd87D',
  value: BigNumber { value: "0" },
  nonce: 202,
  data: '0x6057361d0000000000000000000000000000000000000000000000000000000000000009',
  r: '0x2cdf57dfc4c6a34fa72ed05ef565d285c02d613528060d22ca286c20e9c1b256',
  s: '0x56105967c5fe4b8fb5a4eb6e3b67fc700ea9a6ce3a4dcdd51bbf2d3c8a5df165',
  v: 1,
  creates: null,
  chainId: 4,
  wait: [Function (anonymous)]
}
# 等store执行完毕再次获取number的值
> (await storageV1.retrieve()).toString()
'9'
```

## 创建V2版本的智能合约

- 创建v2版本的Storage

```bash
$ vim contracts/StorageV2.sol
// SPDX-License-Identifier: MIT
pragma solidity >=0.7.0 <0.9.0;

import "@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol";

contract StorageV2 is Initializable {
    uint256 number;

    function initialize(uint256 _number) public initializer {
        number = _number;
    }

    function store(uint256 num) public {
        number = num;
    }

    function retrieve() public view returns (uint256) {
        return number;
    }

    function increment() public {
        number = number + 1;
    }
}
```

- 创建BoxV2测试脚本

```bash
$ vim test/StorageV2.js
const { expect } = require('chai');

let StorageV2;
let storageV2;

describe('StorageV2', function () {
  beforeEach(async function () {
    StorageV2 = await ethers.getContractFactory("StorageV2");
    storageV2 = await StorageV2.deploy();
    await storageV2.deployed();
  });

  it('retrieve returns a value previously stored', async function () {
    await storageV2.store(42);

    expect((await storageV2.retrieve()).toString()).to.equal('42');
  });

  it('retrieve returns a value previously incremented', async function () {
    await storageV2.increment();

    expect((await storageV2.retrieve()).toString()).to.equal('1');
  });
});
```

- 创建BoxV2代理测试脚本

```bash
$ vim test/StorageV2.proxy.js
const { expect } = require('chai');

let StorageV2;
let storageV2;

describe('StorageV2 (proxy)', function () {
  beforeEach(async function () {
    StorageV2 = await ethers.getContractFactory("StorageV2");
    storageV2 = await upgrades.deployProxy(StorageV2, [9]);
  });

  it('retrieve returns a value previously initialized', async function () {
    expect((await storageV2.retrieve()).toString()).to.equal('9');
  });

  it('retrieve returns a value previously incremented', async function () {
    await storageV2.increment();

    expect((await storageV2.retrieve()).toString()).to.equal('10');
  });
});
```

- 运行测试

```bash
$ npx hardhat test

StorageV1
    ✔ retrieve returns a value previously stored

  StorageV1 (proxy)
    ✔ retrieve returns a value previously initialized

  StorageV2
    ✔ retrieve returns a value previously stored
    ✔ retrieve returns a value previously incremented

  StorageV2 (proxy)
    ✔ retrieve returns a value previously initialized
    ✔ retrieve returns a value previously incremented

  6 passing (556ms)
```

## 将V1版本升级到V2

- 创建升级脚本

```bash
$ vim scripts/upgradeProxy.js
const proxyAddress = '0xB4E5E4D4715D5B7F82E83201fDdC1935DbEEd87D';

async function main() {
  const StorageV2 = await ethers.getContractFactory("StorageV2");
  console.log("upgrade...");
  await upgrades.upgradeProxy(proxyAddress, StorageV2);
}

main()
  .then(() => process.exit(0))
  .catch(error => {
    console.error(error);
    process.exit(1);
  });
```

- 运行升级

```bash
$ npx hardhat run --network rinkeby scripts/upgradeProxy.js
upgrade Proxy...
```

- 升级完成之后可以按照上面的方式获取StorageV2的地址然后进行验证

```bash
$ npx hardhat verify --network rinkeby STORAGE_V2_CONTRACT_ADDRESS
Nothing to compile
Successfully submitted source code for contract
contracts/StorageV2.sol:StorageV2 at 0x7d928f13a1c9718941aaacc97fc159e6203a92b7
for verification on the block explorer. Waiting for verification result...

Successfully verified contract StorageV2 on Etherscan.
https://rinkeby.etherscan.io/address/0x7d928f13a1c9718941aaacc97fc159e6203a92b7#code
```

- 进入hardhat控制台通过代理进行测试

```bash
$ npx hardhat console --network rinkeby
Welcome to Node.js v16.14.1.
Type ".help" for more information.
> const StorageV2 = await ethers.getContractFactory("StorageV2")
undefined
> const storageV2 = await StorageV2.attach("0xB4E5E4D4715D5B7F82E83201fDdC1935DbEEd87D")
undefined
> (await storageV2.retrieve()).toString()
'9'
> await storageV2.increment()
{
  hash: '0x0318b97b08e1fe3793efef939c9d8f97319f634c0fe9aff538641130bed7ec8c',
  type: 2,
  accessList: [],
  blockHash: null,
  blockNumber: null,
  transactionIndex: null,
  confirmations: 0,
  from: '0x3247EA903162fB3CD5B612D4F0AcA92e6Eb623BD',
  gasPrice: BigNumber { value: "2500000022" },
  maxPriorityFeePerGas: BigNumber { value: "2500000000" },
  maxFeePerGas: BigNumber { value: "2500000022" },
  gasLimit: BigNumber { value: "33810" },
  to: '0xB4E5E4D4715D5B7F82E83201fDdC1935DbEEd87D',
  value: BigNumber { value: "0" },
  nonce: 205,
  data: '0xd09de08a',
  r: '0x6b2a37094c03d005a5782a4c87c95eeeedcb773e99c7a1eb485dac8e587191aa',
  s: '0x519d2739b973d67d978286354ee264899314493a153f3f2d2c80ab5e3d0a0619',
  v: 0,
  creates: null,
  chainId: 4,
  wait: [Function (anonymous)]
}
> (await storageV2.retrieve()).toString() # 需要等待上面的交易执行完毕
'10'
```

- package.json如下

```bash
{
  "name": "mycontract",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@nomiclabs/hardhat-ethers": "^2.0.5",
    "@nomiclabs/hardhat-etherscan": "^3.0.3",
    "@openzeppelin/contracts-upgradeable": "^4.5.2",
    "@openzeppelin/hardhat-upgrades": "^1.17.0",
    "chai": "^4.3.6",
    "ethers": "^5.6.4",
    "hardhat": "^2.9.3"
  }
}
```