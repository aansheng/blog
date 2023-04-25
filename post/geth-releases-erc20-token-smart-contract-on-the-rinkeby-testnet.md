# Geth在以太坊测试链(rinkeby testnet)发布ERC20 Token智能合约

接上面文章[Geth部署以太坊测试链(testnet)](https://ansheng.me/geth-deploys-ethereum-test-chain-testnet/)，下面我们将在太坊rinkeby测试链发布一套自己的数字币，当然只是在测试环境。

## ERC20

以太坊是一个智能合约平台，其中[ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)是以太坊应用程序级别的标准和约定，只要按照这个约定我们都可以在以太坊平台发币，以太坊支持的[协议列表](https://eips.ethereum.org/erc)，这里面定了很多种标准规范，而ERC20就是其中的一种，定义的一些标准如下

### Methods

- name

```
function name() constant returns (string name)

```

返回string类型的ERC20代币的名字，例如：StatusNetwork

- symbol

```
function symbol() constant returns (string symbol)

```

返回string类型的ERC20代币的符号，也就是代币的简称，例如：SNT

- decimals

```
function decimals() constant returns (uint8 decimals)

```

支持几位小数点后几位。如果设置为3。也就是支持0.001表示

- totalSupply

```
function totalSupply() constant returns (uint256 totalSupply)

```

发行代币的总量，可以通过这个函数来获取。所有智能合约发行的代币总量是一定的，totalSupply必须设置初始值。如果不设置初始值，这个代币发行就说明有问题。

- balanceOf

```
function balanceOf(address _owner) constant returns (uint256 balance)

```

输入地址，可以获取该地址代币的余额

- transfer

```
function transfer(address _to, uint256 _value) returns (bool success)

```

调用transfer函数将自己的token转账给_to地址，_value为转账个数

- approve

```
function approve(address _spender, uint256 _value) returns (bool success)

```

批准_spender账户从自己的账户转移_value个token。可以分多次转移

- transferFrom

```
function transferFrom(address _from, address _to, uint256 _value) returns (bool success)

```

与approve搭配使用，approve批准之后，调用transferFrom函数来转移token

- allowance

```
function allowance(address _owner, address _spender) constant returns (uint256 remaining)

```

返回_spender还能提取token的个数

### Events

- Transfer

```
event Transfer(address indexed _from, address indexed _to, uint256 _value)

```

当成功转移token时，一定要触发Transfer事件

- Approval

```
event Approval(address indexed _owner, address indexed _spender, uint256 _value)

```

当调用approval函数成功时，一定要触发Approval事件

## 部署

合约是使用[Solidity](https://solidity.readthedocs.io/en/latest/index.html)语言进行编写，如果你不了解Solidity，你可以看一下下面的代码，如果有不懂的可以参考官方文档，我感觉语法还是挺简单的，非常简洁明了，创建一个名为`token.sol`的合约文件

```
$ vim token.sol
pragma solidity ^0.7.0;

library SafeMath {
    function sub(uint256 a, uint256 b) internal pure returns (uint256) {
        assert(b <= a);
        return a - b;
    }

    function add(uint256 a, uint256 b) internal pure returns (uint256) {
        uint256 c = a + b;
        assert(c >= a);
        return c;
    }
}

interface ERC20Standard {
    // 代币的名字，例如：StatusNetwork
    function name() external view returns (string memory);

    // 代币的符号，也就是代币的简称，例如：SNT
    function symbol() external view returns (string memory);

    // 支持几位小数点后几位。如果设置为3。也就是支持0.001表示
    function decimals() external view returns (uint8);

    // 发行代币的总量
    function totalSupply() external view returns (uint256);

    // 获取该地址代币的余额
    function balanceOf(address _owner) external view returns (uint256 balance);

    // 将自己的token转账给_to地址，_value为转账个数
    function transfer(address _to, uint256 _value)
        external
        returns (bool success);

    // 与approve搭配使用，approve批准之后，调用transferFrom函数来转移token
    function transferFrom(
        address _from,
        address _to,
        uint256 _value
    ) external returns (bool success);

    // 批准_spender账户从自己的账户转移_value个token。可以分多次转移
    function approve(address _spender, uint256 _value)
        external
        returns (bool success);

    // 返回_spender还能提取token的个数
    function allowance(address _owner, address _spender)
        external
        view
        returns (uint256 remaining);

    event Transfer(address indexed _from, address indexed _to, uint256 _value);
    event Approval(
        address indexed _owner,
        address indexed _spender,
        uint256 _value
    );
}

contract Token is ERC20Standard {
    string name_;
    string symbol_;
    uint8 decimals_;
    uint256 totalSupply_;

    using SafeMath for uint256;

    mapping(address => uint256) balances;
    mapping(address => mapping(address => uint256)) allowed;

    // 初始化，并把初始化的发行数量分配给拥有者
    constructor(
        // string memory name,
        // string memory symbol,
        // uint8 decimals,
        // uint256 totalSupply
    ) public {
        // name_ = name;
        // symbol_ = symbol;
        // decimals_ = decimals;
        // totalSupply_ = totalSupply;
        // balances[msg.sender] = totalSupply;
        name_ = "ansheng";
        symbol_ = "as";
        decimals_ = 18;
        totalSupply_ = 10000000000000000000;
        balances[msg.sender] = 10000000000000000000;
    }

    function name() public override view returns (string memory) {
        return name_;
    }

    function symbol() public override view returns (string memory) {
        return symbol_;
    }

    function decimals() public override view returns (uint8) {
        return decimals_;
    }

    function totalSupply() public override view returns (uint256) {
        return totalSupply_;
    }

    function balanceOf(address _owner)
        public
        override
        view
        returns (uint256 balance)
    {
        return balances[_owner];
    }

    function transfer(address _to, uint256 _value)
        public
        override
        returns (bool success)
    {
        // 检查发送者账户余额是否足够
        require(_value <= balances[msg.sender]);
        // 发送者减少余额
        balances[msg.sender] -= _value;
        // 接受者增加余额
        balances[_to] += _value;
        emit Transfer(msg.sender, _to, _value);
        return true;
    }

    function transferFrom(
        address _from,
        address _to,
        uint256 _value
    ) public override returns (bool success) {
        // 检查发送者的余额是否足够
        require(_value <= balances[_from]);
        require(_value <= allowed[_from][msg.sender]);
        balances[_from] -= _value;
        allowed[_from][msg.sender] -= _value;
        balances[_to] += _value;
        Transfer(_from, _to, _value);
        return true;
    }

    function approve(address _spender, uint256 _value)
        public
        override
        returns (bool success)
    {
        allowed[msg.sender][_spender] = _value;
        emit Approval(msg.sender, _spender, _value);
        return true;
    }

    function allowance(address _owner, address _spender)
        public
        override
        view
        returns (uint256 remaining)
    {
        return allowed[_owner][_spender];
    }
}

```

由于我的客户端是macOS，我还需要通过brew安装Solidity来进行代码编译，如果你是其他平台请参考官方文档[Installing the Solidity Compiler](https://solidity.readthedocs.io/en/latest/installing-solidity.html)

```
$ brew update
$ brew upgrade
$ brew tap ethereum/ethereum
$ brew install solidity
$ geth solc --version
solc, the solidity compiler commandline interface
Version: 0.7.2+commit.51b20bc0.Darwin.appleclang

```

- 代码编译

忽略Warning:

```
$ solc --abi --bin -o solcoutput token.sol
$ ls solcoutput
ERC20Standard.abi ERC20Standard.bin SafeMath.abi      SafeMath.bin      Token.abi         Token.bin         combined.json

```

对我们有用的文件只有`Token.abi`和`Token.bin`，下面登陆节点服务器并进入console

```
$ geth attach /usr/local/etc/geth/geth.ipc
# 我把之前的用户都删掉了，目前是空的
> eth.accounts
[]
# 创建用户
> personal.newAccount("ansheng.me")
"0xdf3d8df4ee370e96a2338d389c5821e154b092e9"
> personal.newAccount("ansheng")
"0xda0cfe3a0772995f83399b1ae82afbbddf5aedd2"
# 解锁
> personal.unlockAccount("0xdf3d8df4ee370e96a2338d389c5821e154b092e9", "ansheng.me", 0)
true
# 发布合约需要有余额才可以，刚创建的用户余额是0，所以你需要去https://www.rinkeby.io/#faucet弄点币才行
> eth.getBalance('0xdf3d8df4ee370e96a2338d389c5821e154b092e9')
2000000000000000000
# 发布合约
# solcoutput/Token.abi 的内容
var abi = [{ "inputs": [], "stateMutability": "nonpayable", "type": "constructor" }, { "anonymous": false, "inputs": [{ "indexed": true, "internalType": "address", "name": "_owner", "type": "address" }, { "indexed": true, "internalType": "address", "name": "_spender", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "_value", "type": "uint256" }], "name": "Approval", "type": "event" }, { "anonymous": false, "inputs": [{ "indexed": true, "internalType": "address", "name": "_from", "type": "address" }, { "indexed": true, "internalType": "address", "name": "_to", "type": "address" }, { "indexed": false, "internalType": "uint256", "name": "_value", "type": "uint256" }], "name": "Transfer", "type": "event" }, { "inputs": [{ "internalType": "address", "name": "_owner", "type": "address" }, { "internalType": "address", "name": "_spender", "type": "address" }], "name": "allowance", "outputs": [{ "internalType": "uint256", "name": "remaining", "type": "uint256" }], "stateMutability": "view", "type": "function" }, { "inputs": [{ "internalType": "address", "name": "_spender", "type": "address" }, { "internalType": "uint256", "name": "_value", "type": "uint256" }], "name": "approve", "outputs": [{ "internalType": "bool", "name": "success", "type": "bool" }], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [{ "internalType": "address", "name": "_owner", "type": "address" }], "name": "balanceOf", "outputs": [{ "internalType": "uint256", "name": "balance", "type": "uint256" }], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "decimals", "outputs": [{ "internalType": "uint8", "name": "", "type": "uint8" }], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "name", "outputs": [{ "internalType": "string", "name": "", "type": "string" }], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "symbol", "outputs": [{ "internalType": "string", "name": "", "type": "string" }], "stateMutability": "view", "type": "function" }, { "inputs": [], "name": "totalSupply", "outputs": [{ "internalType": "uint256", "name": "", "type": "uint256" }], "stateMutability": "view", "type": "function" }, { "inputs": [{ "internalType": "address", "name": "_to", "type": "address" }, { "internalType": "uint256", "name": "_value", "type": "uint256" }], "name": "transfer", "outputs": [{ "internalType": "bool", "name": "success", "type": "bool" }], "stateMutability": "nonpayable", "type": "function" }, { "inputs": [{ "internalType": "address", "name": "_from", "type": "address" }, { "internalType": "address", "name": "_to", "type": "address" }, { "internalType": "uint256", "name": "_value", "type": "uint256" }], "name": "transferFrom", "outputs": [{ "internalType": "bool", "name": "success", "type": "bool" }], "stateMutability": "nonpayable", "type": "function" }];
# 0x后面加编译的字节码
var bytecode = "0x608060405234801561001057600080fd5b506040518060400160405280600781526020017f616e7368656e67000000000000000000000000000000000000000000000000008152506000908051906020019061005c929190610125565b506040518060400160405280600281526020017f6173000000000000000000000000000000000000000000000000000000000000815250600190805190602001906100a8929190610125565b506012600260006101000a81548160ff021916908360ff160217905550678ac7230489e80000600381905550678ac7230489e80000600460003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020819055506101c2565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061016657805160ff1916838001178555610194565b82800160010185558215610194579182015b82811115610193578251825591602001919060010190610178565b5b5090506101a191906101a5565b5090565b5b808211156101be5760008160009055506001016101a6565b5090565b610b18806101d16000396000f3fe608060405234801561001057600080fd5b50600436106100935760003560e01c8063313ce56711610066578063313ce5671461022157806370a082311461024257806395d89b411461029a578063a9059cbb1461031d578063dd62ed3e1461038157610093565b806306fdde0314610098578063095ea7b31461011b57806318160ddd1461017f57806323b872dd1461019d575b600080fd5b6100a06103f9565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156100e05780820151818401526020810190506100c5565b50505050905090810190601f16801561010d5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6101676004803603604081101561013157600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff1690602001909291908035906020019092919050505061049b565b60405180821515815260200191505060405180910390f35b61018761058d565b6040518082815260200191505060405180910390f35b610209600480360360608110156101b357600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190803573ffffffffffffffffffffffffffffffffffffffff16906020019092919080359060200190929190505050610597565b60405180821515815260200191505060405180910390f35b610229610802565b604051808260ff16815260200191505060405180910390f35b6102846004803603602081101561025857600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610819565b6040518082815260200191505060405180910390f35b6102a2610862565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156102e25780820151818401526020810190506102c7565b50505050905090810190601f16801561030f5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6103696004803603604081101561033357600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff16906020019092919080359060200190929190505050610904565b60405180821515815260200191505060405180910390f35b6103e36004803603604081101561039757600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff169060200190929190803573ffffffffffffffffffffffffffffffffffffffff169060200190929190505050610a5b565b6040518082815260200191505060405180910390f35b606060008054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156104915780601f1061046657610100808354040283529160200191610491565b820191906000526020600020905b81548152906001019060200180831161047457829003601f168201915b5050505050905090565b600081600560003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020819055508273ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff167f8c5be1e5ebec7d5bd14f71427d1e84f3dd0314c0f7b2291e5b200ac8c7c3b925846040518082815260200191505060405180910390a36001905092915050565b6000600354905090565b6000600460008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020548211156105e557600080fd5b600560008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205482111561066e57600080fd5b81600460008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254039250508190555081600560008673ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254039250508190555081600460008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020600082825401925050819055508273ffffffffffffffffffffffffffffffffffffffff168473ffffffffffffffffffffffffffffffffffffffff167fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef846040518082815260200191505060405180910390a3600190509392505050565b6000600260009054906101000a900460ff16905090565b6000600460008373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020549050919050565b606060018054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156108fa5780601f106108cf576101008083540402835291602001916108fa565b820191906000526020600020905b8154815290600101906020018083116108dd57829003601f168201915b5050505050905090565b6000600460003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205482111561095257600080fd5b81600460003373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000206000828254039250508190555081600460008573ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001908152602001600020600082825401925050819055508273ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff167fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef846040518082815260200191505060405180910390a36001905092915050565b6000600560008473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200190815260200160002060008373ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff1681526020019081526020016000205490509291505056fea2646970667358221220a2485e1b926d17cf9231b535aab86329faa059bc67d41f8c6868ea77274bb46464736f6c63430007020033";
var simpleContract = web3.eth.contract(abi);
var simple = simpleContract.new(42, {
  from: "0xdf3d8df4ee370e96a2338d389c5821e154b092e9",
  data: bytecode,
  gas: 0x47b760
}, function (e, contract) {
  if (e) {
    console.log("err creating contract", e);
  } else {
    if (!contract.address) {
      console.log("Contract transaction send: TransactionHash: " + contract.transactionHash + " waiting to be mined...");
    } else {
      console.log("Contract mined! Address: " + contract.address);
    }
  }
});
# Contract transaction send: TransactionHash: 0x5d45beeb2337d3fcc8b95d4c2e6545ce6ca890db16b91b3c962b7385595bb82e waiting to be mined...
# ......
# 这就是我们的合约地址，需要记录下面
# > Contract mined! Address: 0xe16c5613b71edaa9bf2d0edd1f7ce90708904d48
# 只要得到上面的合约地址就可以推出了
> exit

```

如果刚创建的账户余额不足，会提示以下错误，这个时候就需要账户有币才可以

```
err creating contract Error: insufficient funds for gas * price + value

```

## 操作

我们可以通过下面的地址查看测试链的合约信息

```
<https://rinkeby.etherscan.io/token/0xe16c5613b71edaa9bf2d0edd1f7ce90708904d48>

```

合约发布之后我们就来试试下面的功能把，当然还是通过[web3py](https://web3py.readthedocs.io/en/stable/)进行操作，当然你可以可以参考管饭文档[Working with an ERC20 Token Contract](https://web3py.readthedocs.io/en/stable/examples.html#working-with-an-erc20-token-contract)

- 创建连接

```
>>> from web3 import Web3
>>> web3 = Web3(Web3.HTTPProvider("<http://34.92.29.146:23456>", request_kwargs={'timeout': 60}))

```

- 创建contract factory

```
>>> import json
>>> with open('./solcoutput/Token.abi') as f:
...     ABI = json.load(f)
...
>>> contract_address = Web3.toChecksumAddress('0xe16c5613b71edaa9bf2d0edd1f7ce90708904d48')
>>> contract = web3.eth.contract(contract_address, abi=ABI)
>>> contract.address
'0xe16c5613B71edaA9bF2d0EdD1f7CE90708904d48'

```

定义两个用户`alice`和`bob`，并解锁account，不然无法进行转账交易

```
# alice是合约的发布者，账户自带余额
>>> alice = Web3.toChecksumAddress("0xdf3d8df4ee370e96a2338d389c5821e154b092e9")
>>> bob = Web3.toChecksumAddress("0xDa0CFe3A0772995f83399b1AE82AFBbDDf5aeDD2")
>>> web3.geth.personal.unlock_account(alice, 'ansheng.me', 0)
True
>>> web3.geth.personal.unlock_account(bob, 'ansheng', 0)
True

```

代币的名字

```
>>> contract.functions.name().call()
'ansheng'

```

代币的简称

```
>>> contract.functions.symbol().call()
'as'

```

支持几位小数点后几位

```
>>> decimals = contract.functions.decimals().call()
>>> decimals
18

```

发行代币的总量

```
>>> contract.functions.totalSupply().call()
10000000000000000000

```

账户余额查询

```
>>> contract.functions.balanceOf(alice).call()
10000000000000000000
>>> contract.functions.balanceOf(bob).call()
0

```

alice转账100到bob账户

```
>>> from web3.middleware import geth_poa_middleware
>>> web3.middleware_onion.inject(geth_poa_middleware, layer=0)
>>> contract.functions.transfer(bob, 100).transact({'from': alice})
HexBytes('0x1b811756a580e4ad7bb9fcf87549a5f24545c8589fb3af0f08fd0df93830bc1c')
>>> contract.functions.balanceOf(alice).call()
9999999999999999900
>>> contract.functions.balanceOf(bob).call()
100

```

alice将批准bob允许消费200，钱从alice中扣除

```
>>> contract.functions.allowance(alice, bob).call()
0
>>> contract.functions.approve(bob, 200).transact({'from': alice})
HexBytes('0x0471a04b21655e951e63904111fd812503abbc730c475680ffabdd9206d6a48e')
>>> contract.functions.allowance(alice, bob).call()
0

```

bob转账的时候从alice账户扣

```
# 转账交易是需要消耗ETH的，我们的bob账号目前余额是0
>>> web3.eth.getBalance(bob)
0
# alice给bob转1个ETH
>>> web3.eth.sendTransaction({'to': bob,'from': alice,'value': web3.toWei('1','ether')})
HexBytes('0x56dd28db03b8a1b64affcf82f080d4879e7683a912d0e6711f30cd010558e86a')
# 等待矿工操作完成之后就有余额了
>>> web3.eth.getBalance(bob)
1000000000000000000
# 查看bob可以透支的余额
>>> contract.functions.allowance(alice, bob).call()
200
# bob现在的余额
>>> contract.functions.balanceOf(bob).call()
100
>>> contract.functions.transferFrom(alice, bob, 75).transact({'from': bob})
HexBytes('0x1c316e4156dcb8b7edefd2f08c4309169ef77358aacdb0d25af4ebd8d98fe89a')
>>> contract.functions.allowance(alice, bob).call()
125
>>> contract.functions.balanceOf(bob).call()
175

```

至此，本片结束，可以通过下面的连接查看交易的记录

```
<https://rinkeby.etherscan.io/token/0xe16c5613b71edaa9bf2d0edd1f7ce90708904d48>

```

如图所示

![Untitled](/images//2020/10/17/1.png)