# 索拉纳(Solana)命令行(CLI)钱包及基本操作

在Solana CLI中，钱包主要分为以下三类

- Paper Wallet(纸钱包)
- File System Wallet(文件系统钱包、FS钱包)
- Hardware Wallets(硬件钱包)

生成的钱包公钥其实就是钱包地址，官方文档请参考[Command Line Wallets](https://docs.solana.com/wallet-guide/cli)。

## File System Wallet

把`未加密的密钥对`存放在一个JSON文件里面，Solana每次操作时需要指定文件路径，因为是一个文件，所以安全性相对会比较差

- 创建文件钱包

```
$ solana-keygen new --outfile ~/my-solana-fs-wallet-keypair.json
Generating a new keypair

For added security, enter a BIP39 passphrase

NOTE! This passphrase improves security of the recovery seed phrase NOT the
keypair file itself, which is stored as insecure plain text

BIP39 Passphrase (empty for none):

Wrote new keypair to /root/my-solana-fs-wallet-keypair.json
==========================================================================
pubkey: 8vxsHAq9rCdy4bpH1wKNrNVy6qHismAdFbFv9dAXncXn
==========================================================================
Save this seed phrase and your BIP39 passphrase to recover your new keypair:
nut angry advance laptop hybrid zero equip accident skin clock canoe evoke
==========================================================================

```

请不要此文件分享于互联网，如果有人拿到你的钱包文件，会把你的币都转走的，记得保存好助记词。

- 查看钱包公钥

```
$ solana-keygen pubkey ~/my-solana-fs-wallet-keypair.json
8vxsHAq9rCdy4bpH1wKNrNVy6qHismAdFbFv9dAXncXn

```

- 验证fs钱包文件对应的公钥是否正确

```
$ solana-keygen verify 8vxsHAq9rCdy4bpH1wKNrNVy6qHismAdFbFv9dAXncXn ~/my-solana-fs-wallet-keypair.json
Verification for public key: 8vxsHAq9rCdy4bpH1wKNrNVy6qHismAdFbFv9dAXncXn: Success

```

如果验证成功则返回`Success`，否则返回`Failed`

- 从助记词中恢复钱包

先备份钱包文件

```
mv my-solana-fs-wallet-keypair.json my-solana-fs-wallet-keypair.json.orig

```

加入钱包文件丢失，我们可以通过助记词进行恢复

```
$ solana-keygen recover --outfile ~/my-solana-fs-wallet-keypair.json
[recover] seed phrase: # 输入助记词
[recover] If this seed phrase has an associated passphrase, enter it now. Otherwise, press ENTER to continue:
Recovered pubkey `8vxsHAq9rCdy4bpH1wKNrNVy6qHismAdFbFv9dAXncXn`. Continue? (y/n): y
Wrote recovered keypair to /root/my-solana-fs-wallet-keypair.json

```

恢复完成之后我们对比一下和之前备份的钱包文件是否一致

```
diff ~/my-solana-fs-wallet-keypair.json ~/my-solana-fs-wallet-keypair.json.orig

```

## Paper Wallet

- 创建钱包

纸钱包不会生成文件，每次创建会把所有信息输出到屏幕中，我们需要保存输出的内容

```
$ solana-keygen new --no-outfile
Generating a new keypair

For added security, enter a BIP39 passphrase

NOTE! This passphrase improves security of the recovery seed phrase NOT the
keypair file itself, which is stored as insecure plain text

BIP39 Passphrase (empty for none):

===========================================================================
pubkey: 4wr536h23WLB8WhXyZ2vV4RazNRmoRnhhb8zgKJD9Nqq
===========================================================================
Save this seed phrase and your BIP39 passphrase to recover your new keypair:
column melt drift tone age fall coral sponsor derive chef marriage language
===========================================================================

```

- 通过助记词查看公钥

```
$ solana-keygen pubkey ASK
[pubkey recovery] seed phrase: # 输入助记词
[pubkey recovery] If this seed phrase has an associated passphrase, enter it now. Otherwise, press ENTER to continue:
4wr536h23WLB8WhXyZ2vV4RazNRmoRnhhb8zgKJD9Nqq

```

- 验证密钥对

```
solana-keygen verify 4wr536h23WLB8WhXyZ2vV4RazNRmoRnhhb8zgKJD9Nqq ASK

```

## 基本操作

我这里用测试网，方便领取空投

- 获取空投

空投每次最多领取10个SOL

```
solana airdrop 10 <RECIPIENT_ACCOUNT_ADDRESS> --url <https://api.devnet.solana.com>

```

fs钱包

```
$ solana airdrop 10 -k ~/my-solana-fs-wallet-keypair.json --url <https://api.devnet.solana.com>
Requesting airdrop of 10 SOL

Signature: 26gSkMLHBDmBEdDiMBMrEjcbF7RdJpgKU56xDA4MTEV6Nh86wTE7gM5c6nfjt6rNrBmTwvYavzUefbQwJLb7GFg3

10 SOL

```

纸钱包

```
$ solana airdrop 10 4wr536h23WLB8WhXyZ2vV4RazNRmoRnhhb8zgKJD9Nqq --url <https://api.devnet.solana.com>
Requesting airdrop of 10 SOL

Signature: 3hf3Eeosfko8gyXi45iHBh2Sm8ywpmXDMNqPN5RYp6RoSxr69Fy1CVzDgENFbuE3P7G3gbt6MBB9xa24VxZRUK5T

10 SOL

```

- 查看账户余额

语法

```
solana balance <ACCOUNT_ADDRESS> --url <https://api.devnet.solana.com>

```

查看纸钱包余额

```
$ solana balance 4wr536h23WLB8WhXyZ2vV4RazNRmoRnhhb8zgKJD9Nqq --url <https://api.devnet.solana.com>
10 SOL

```

查看fs钱包余额

```
$ solana balance -k ~/my-solana-fs-wallet-keypair.json --url <https://api.devnet.solana.com>
10 SOL

```

- 转账

语法

```
solana transfer --from <KEYPAIR> <RECIPIENT_ACCOUNT_ADDRESS> 5 --url <https://api.devnet.solana.com> --fee-payer <KEYPAIR>

```

我们从FS钱包转账5个SOL到纸钱包

```
$ solana transfer --from ~/my-solana-fs-wallet-keypair.json 4wr536h23WLB8WhXyZ2vV4RazNRmoRnhhb8zgKJD9Nqq  5 --url <https://api.devnet.solana.com> --fee-payer ~/my-solana-fs-wallet-keypair.json

Signature: 23wT9zrruUHBfun7wMvvonEi14GRd15ZdojcecEuKQfn4PVqAYHnwELgjRkFxtGXarorbhAWfay9zeXK1tEGx7Wt

```

再次查看两个钱包的余额

```bash
$ solana balance -k ~/my-solana-fs-wallet-keypair.json --url <https://api.devnet.solana.com>
4.999995 SOL
$ solana balance 4wr536h23WLB8WhXyZ2vV4RazNRmoRnhhb8zgKJD9Nqq --url <https://api.devnet.solana.com>
15 SOL
```