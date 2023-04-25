# 如何在Docker中运行IPFS节点

[IPFS](https://ipfs.io/)是一个去中心化的分布式文件存储系统，而且是免费使用的，具体介绍可以参考[官方文档](https://ipfs.io/)。

在进行NFT相关产品开发的时，为了防止篡改，图片资源以及metadata数据都是放在ipfs上的，这里我们通过Docker的方式运行IPFS节点，并演示如何上传和下载。

## 部署IPFS节点

- 创建项目目录

```bash
mkdir -p ~/deploy/ipfs
cd ~/deploy/ipfs
```

- 创建数据共享目录

```bash
mkdir -p data/{export,ipfs}
```

- docker-compose.yml

为了方便管理和迁移，使用`docker-compose.yml`的方式运行。

```bash
$ vim docker-compose.yml
version: '3.8'

services:

  ipfs:
    image: ipfs/go-ipfs:latest
    container_name: ipfs
    #network_mode: host
    ports:
      - 4001:4001     # P2P TCP/QUIC传输
      - 4001:4001/udp # P2P TCP/QUIC传输
      - 5001:5001     # RPC API，管理页面的端口，可以进行数据的读写
      - 8080:8080     # 网关，用于读取ipfs节点数据
    restart: always
    volumes: 
      - "./data/export:/export"
      - "./data/ipfs:/data/ipfs"
```

- 查看日志

```bash
$ docker-compose logs
Attaching to ipfs
ipfs    | Changing user to ipfs
ipfs    | ipfs version 0.12.2
ipfs    | generating ED25519 keypair...done
ipfs    | peer identity: 12D3KooWMJDXZ2zUS4zL1CBwiR7nNNqPLHkBSJUEELnLC8R1TkG3
ipfs    | initializing IPFS node at /data/ipfs
ipfs    | to get started, enter:
ipfs    |
ipfs    | 	ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/readme
ipfs    |
ipfs    | Initializing daemon...
ipfs    | go-ipfs version: 0.12.2-0e8b121
ipfs    | Repo version: 12
ipfs    | System version: amd64/linux
ipfs    | Golang version: go1.16.15
ipfs    | 2022/04/14 06:28:58 failed to sufficiently increase receive buffer size (was: 208 kiB, wanted: 2048 kiB, got: 416 kiB). See https://github.com/lucas-clemente/quic-go/wiki/UDP-Receive-Buffer-Size for details.
ipfs    | Swarm listening on /ip4/127.0.0.1/tcp/4001
ipfs    | Swarm listening on /ip4/127.0.0.1/udp/4001/quic
ipfs    | Swarm listening on /ip4/172.19.0.2/tcp/4001
ipfs    | Swarm listening on /ip4/172.19.0.2/udp/4001/quic
ipfs    | Swarm listening on /p2p-circuit
ipfs    | Swarm announcing /ip4/YOUR_IP/tcp/4001
ipfs    | Swarm announcing /ip4/YOUR_IP/udp/4001/quic
ipfs    | Swarm announcing /ip4/127.0.0.1/tcp/4001
ipfs    | Swarm announcing /ip4/127.0.0.1/udp/4001/quic
ipfs    | Swarm announcing /ip4/172.19.0.2/tcp/4001
ipfs    | Swarm announcing /ip4/172.19.0.2/udp/4001/quic
ipfs    | API server listening on /ip4/0.0.0.0/tcp/5001
ipfs    | WebUI: http://0.0.0.0:5001/webui
# 看到下面的信息表示运行成功
ipfs    | Gateway (readonly) server listening on /ip4/0.0.0.0/tcp/8080
ipfs    | Daemon is ready
```

- 查看连接的peers

```bash
docker-compose exec ipfs ipfs swarm peers
```

- 上传测试文件到ipfs

```bash
$ echo "hello ipfs!" > data/export/hello
$ docker-compose exec ipfs ipfs add /export/hello
added QmZ5cRqiNsg1ngmzmKrv5STMoyfLaJhhHqXyMWTkre1qte hello
 12 B / 12 B [=========================================================================================================================================================================================================================================================] 100.00%
```

通过CID查看文件内容

```bash
$ docker-compose exec ipfs ipfs cat QmZ5cRqiNsg1ngmzmKrv5STMoyfLaJhhHqXyMWTkre1qte
hello ipfs!
```

## 迁移IPFS节点

在迁移之前我们先把节点停掉

```bash
docker-compose down
```

当第一次运行节点的时候，由于数据目录是空的，所以会执行`ipfs init`初始化配置文件并生成新的密钥对，迁移的时候是不希望执行`ipfs init`，这个时候就可以使用`IPFS_PROFILE`环境变量，迁移时完整的`docker-compose.yml`如下

```yaml
version: '3.8'

services:

  ipfs:
    image: ipfs/go-ipfs:latest
    container_name: ipfs
    #network_mode: host
    environment:
      - IPFS_PROFILE=server
    ports:
      - 4001:4001     # P2P TCP/QUIC传输
      - 4001:4001/udp # P2P TCP/QUIC传输
      - 5001:5001     # RPC API，管理页面的端口，可以进行数据的读写
      - 8080:8080     # 网关，用于读取ipfs节点数据
    restart: always
    volumes:
      - "./data/export:/export"
      - "./data/ipfs:/data/ipfs"
```

- 运行节点

```bash
docker-compose up -d
```

## 代码示例

上面我们通过`ipfs cli`上传了文件和查看文件内容，下面通过`nodejs`代码来实现文件的上传和查看。

- 配置IPFS跨域

```bash
docker-compose exec ipfs ipfs config --json API.HTTPHeaders.Access-Control-Allow-Methods '["PUT","GET", "POST", "OPTIONS"]'
docker-compose exec ipfs ipfs config --json API.HTTPHeaders.Access-Control-Allow-Origin '["*"]'
docker-compose exec ipfs ipfs config --json API.HTTPHeaders.Access-Control-Allow-Credentials '["true"]'
docker-compose exec ipfs ipfs config --json API.HTTPHeaders.Access-Control-Allow-Headers '["Authorization"]'
docker-compose exec ipfs ipfs config --json API.HTTPHeaders.Access-Control-Expose-Headers '["Location"]'
# 重启下容器
docker-compose restart
```

- IPFS控制台

其实就是提供了一个web页面的UI，地址为：`http://YOUR_IP:5001/webui`

![Untitled](/images/2022/04/14/1.png)

- 上传文件代码示例

```bash
$ vim add.js
const {create} = require("ipfs-http-client")
# 记得安装 npm i ipfs-http-client

const client = create('http://YOUR_IP:5001/')
const content = "hello ansheng!"

const main = async () => {
    try {
        const added = await client.add(content)
        console.log(`CID is: ${added.path}`)
    } catch (error) {
        console.log('Error uploading file: ', error)
    }
}

main()
$ node add.js
CID is: QmQ98xvEG7PbXxsQYiyWP5rkaoXwbX5hNDaWvBAWPKYoZY
```

- 查看内容

```bash
$ vim cat.js
const {create} = require("ipfs-http-client")

const client = create('http://YOUR_IP:5001/')
const CID = "QmQ98xvEG7PbXxsQYiyWP5rkaoXwbX5hNDaWvBAWPKYoZY"

const main = async () => {
    try {
        const source = await client.cat(CID)
        let contents = ''
        const decoder = new TextDecoder('utf-8')

        for await (const chunk of source) {
            contents += decoder.decode(chunk, {stream: true})
        }

        contents += decoder.decode()
        console.log(contents)
    } catch (error) {
        console.log('Error uploading file: ', error)
    }
}

main()
$ node cat.js
hello ansheng!
```

- 通过URL查看CID内容

地址：`http://YOUR_IP:8080/ipfs/QmQ98xvEG7PbXxsQYiyWP5rkaoXwbX5hNDaWvBAWPKYoZY，`请将CID替换为自己的。

## 参考文献

- [Run IPFS inside Docker](https://docs.ipfs.io/how-to/run-ipfs-inside-docker/)