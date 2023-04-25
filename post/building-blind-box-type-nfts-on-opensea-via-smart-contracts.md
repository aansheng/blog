# 通过智能合约在OpenSea上构建盲盒类型的NFT

在上篇文章中我们介绍了如何通过**[HashLips Art Engine批量创建NFT盲盒的艺术作品](https://blog.ansheng.me/post/hashlips-art-engine-creates-artwork-for-nft-blind-boxes-in-batches)**，这篇文章我们就通过编写自己的智能合约在OpenSea上面发布盲盒类型的NFT。

如果你阅读完上篇文章并且跟着操作了全部，那么你应该会得到以下的CID

![Untitled](/images/2022/04/13/1.png)

如果没有也没关系，你可以使用这里提供的CID进行操作也没关系

- unpack.json - QmepiRpDwuXtqCHNoNnF5WiKrCGfkYr8tDgJRmXot57pjR
- unpack.png - QmNbgVii5zsywA5xLreA8KuC8Y8twmoXR9d2z74jEaDSyg
- META_JSONS - QmTa6J1T7NiL7EAju1hRtWetVWfFnErmqJuRKpERqcTtop
- META_IMAGES - QmNq56Eu4QwhANHpYvGaME8FTu3RpCLaqKk6DPEr1CapRQ

## 部署智能合约

代码如下

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.0 <0.9.0;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Strings.sol";

contract BlindBoxNFT is ERC721Enumerable, Ownable {
    using Strings for uint256;

    bool public _isSaleActive = false;  // 是否允许mint
    bool public _revealed = false;      // 盲盒是否开启

    string private baseURI;                   // NFT metadata的baseURI
    string private _notRevealedUri;           // 盲盒的metadata URI
    string private _baseExtension = ".json";  // metadata文件扩展名类型，默认为json

    // Constants
    uint256 public constant MAX_SUPPLY = 10;  // 总的允许mint的NFT数量
    uint256 public mintPrice = 0.0 ether;     // 每次mint需要收取的费用
    uint256 public maxBalance = 3;            // 每个地址最大可以拥有的NFT数量
    uint256 public maxMint = 3;               // 每次最大允许mint的数量

    // 合约初始化的时候需要传递两个参数
    constructor(string memory initBaseURI, string memory initNotRevealedUri)
        ERC721("BlindBoxNFT", "BD")
    {
        baseURI = initBaseURI;  // 设置Token的baseURI
        _notRevealedUri = initNotRevealedUri;  // 设置盲盒的metadata URI
    }

    // mint NFT，传递的参数为需要mint的数量
    function mint(uint256 tokenQuantity) public payable {
        // 判断数量是否超过最大的NFT数量
        require(
            totalSupply() + tokenQuantity <= MAX_SUPPLY,
            "Sale would exceed max supply"
        );
        // 是否允许被mint
        require(_isSaleActive, "Sale must be active to mint.");
        // 判断当前用户被允许mint的数量
        require(
            balanceOf(msg.sender) + tokenQuantity <= maxBalance,
            "Sale would exceed max balance"
        );
        // 每次mint时候需要缴纳的手续费，这里mintPrice = 0.0 ether，所以可以忽略
        require(
            tokenQuantity * mintPrice <= msg.value,
            "Not enough token sent"
        );
        // 是否单次被允许mint的数量
        require(tokenQuantity <= maxMint, "Can only mint 3 tokens at a time");

        // 通过for循环开始mint
        for (uint256 i = 0; i < tokenQuantity; i++) {
            // 默认情况下Token ID为0，但是我们上传的图片都是以1开始的，所以需要+1
            uint256 mintIndex = totalSupply() + 1;
            if (totalSupply() < MAX_SUPPLY) {
                _safeMint(msg.sender, mintIndex);
            }
        }
    }

    // 获取每个TokenID对应的URI
    function tokenURI(uint256 tokenId)
        public
        view
        virtual
        override
        returns (string memory)
    {
        require(
            _exists(tokenId),
            "ERC721Metadata: URI query for nonexistent token"
        );

        // 如果盲盒未开启则直接返回盲盒的metadata
        if (_revealed == false) {
            return _notRevealedUri;
        }

        // 如果盲盒已经打开，则通过join的方式进行URI的拼接
        string memory base = _baseURI();

        return
            string(abi.encodePacked(base, tokenId.toString(), _baseExtension));
    }

    // metadata的BASE URI
    function _baseURI() internal view virtual override returns (string memory) {
        return baseURI;
    }

    // 设置是否允许被mint
    function flipSaleActive() public onlyOwner {
        _isSaleActive = !_isSaleActive;
    }

    // 设置是否开启盲盒
    function flipReveal() public onlyOwner {
        _revealed = !_revealed;
    }

    // 提现
    function withdraw(address to) public onlyOwner {
        uint256 balance = address(this).balance;
        payable(to).transfer(balance);
    }
}
```

- 通过[remix](https://remix.ethereum.org/)部署合约

网络需要选择Rinkeby测试网，因为OpenSea的测试网就是运行在Rinkeby上面，部署智能合约时需要传递两个参数，分别为`ipfs://QmTa6J1T7NiL7EAju1hRtWetVWfFnErmqJuRKpERqcTtop/`和`ipfs://QmepiRpDwuXtqCHNoNnF5WiKrCGfkYr8tDgJRmXot57pjR`，BASEURI后面的地址一定要加上 `/`斜线

![Untitled](/images/2022/04/13/2.png)

部署完成之后得到的合约地址为：`0xBd56e7B27f1Eccc6b5eFBcAB6ba1D185137B38cc`

## Mint NFT

在mint NFT之前我们需要先把_isSaleActive改为true，这样才会被允许mint，点击`flipSaleActive`

![Untitled](/images/2022/04/13/3.png)

执行完毕之后查看_isSaleActive是否为true

![Untitled](/images/2022/04/13/4.png)

最后调用mint方法，参数传递为3，因为最大只允许被mint三个

![Untitled](/images/2022/04/13/5.png)

执行完毕之后打开[https://testnets.opensea.io](https://testnets.opensea.io/)，然后连接你的钱包，并选择`Rinkeby 测试网络`，可以打开下面三个地址：

1. https://testnets.opensea.io/assets/0xBd56e7B27f1Eccc6b5eFBcAB6ba1D185137B38cc/1
2. https://testnets.opensea.io/assets/0xBd56e7B27f1Eccc6b5eFBcAB6ba1D185137B38cc/2
3. https://testnets.opensea.io/assets/0xBd56e7B27f1Eccc6b5eFBcAB6ba1D185137B38cc/3

就可以看到刚才创建的三个NFT了，因为盲盒还没有打开，所以你看到的内容都是一样的，如下

![Untitled](/images/2022/04/13/6.png)

我们可以调用合约的tokenURI函数，查看NFT编号1、2、3的URI地址

![Untitled](/images/2022/04/13/7.png)

不管如何测试，他们返回的结果都是一样的`ipfs://QmepiRpDwuXtqCHNoNnF5WiKrCGfkYr8tDgJRmXot57pjR`，这就是我们设置的盲盒metadata URI，也可以查看盲盒URI的metadata内容

```bash
$ curl https://gateway.pinata.cloud/ipfs/QmepiRpDwuXtqCHNoNnF5WiKrCGfkYr8tDgJRmXot57pjR
{
    "name": "Nano Meta 盲盒",
    "description": "Welcome to the world of the Metaverse",
    "image": "ipfs://QmNbgVii5zsywA5xLreA8KuC8Y8twmoXR9d2z74jEaDSyg"
}
```

## 开启盲盒

如果要开启盲盒只需要调用合约的`flipReveal`方法即可

![Untitled](/images/2022/04/13/8.png)

然后查看`_revealed`有没有被改为`true`

![Untitled](/images/2022/04/13/9.png)

然后我们在调用tokenURI方法查看每个NFT的metadata uri

![Untitled](/images/2022/04/13/10.png)

ipfs://QmTa6J1T7NiL7EAju1hRtWetVWfFnErmqJuRKpERqcTtop/1.json

![Untitled](/images/2022/04/13/11.png)

ipfs://QmTa6J1T7NiL7EAju1hRtWetVWfFnErmqJuRKpERqcTtop/2.json

![Untitled](/images/2022/04/13/12.png)

ipfs://QmTa6J1T7NiL7EAju1hRtWetVWfFnErmqJuRKpERqcTtop/3.json

- 查看metadata的内容

盲盒已开启，所以可以查看NFT真正的metadata数据

```bash
$ curl https://gateway.pinata.cloud/ipfs/QmTa6J1T7NiL7EAju1hRtWetVWfFnErmqJuRKpERqcTtop/3.json
{
  "name": "Nano Meta #3",
  "description": "There are so many eyes",
  "image": "ipfs://QmNq56Eu4QwhANHpYvGaME8FTu3RpCLaqKk6DPEr1CapRQ/3.png",
  "dna": "60c46c557c974bc61a1838eaaff68fdb0738723e",
  "edition": 3,
  "date": 1649757231547,
  "creator": "ansheng",
  "attributes": [
    {
      "trait_type": "Background",
      "value": "Black"
    },
    {
      "trait_type": "Eyeball",
      "value": "Red"
    },
    {
      "trait_type": "Eye color",
      "value": "Yellow"
    },
    {
      "trait_type": "Iris",
      "value": "Large"
    },
    {
      "trait_type": "Shine",
      "value": "Shapes"
    },
    {
      "trait_type": "Bottom lid",
      "value": "Middle"
    },
    {
      "trait_type": "Top lid",
      "value": "High"
    }
  ],
  "compiler": "HashLips Art Engine"
}
```

此时我们在打开OpenSea，然后刷新页面就可以看到NFT的内容啦

![Untitled](/images/2022/04/13/13.png)

包括名称、图片、属性等都一一对应起来了，示例代码只是一个简单的应用，方便理解。