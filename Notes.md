# My Notes
 - Docker 
 - Etherum
 - Python-Java-Go-NodeJS-Solidity
 - Rasperberry PI

# Docker 
----
### Install Docker
```
$ uname -r  查看内核版本
$ sudo apt-get remove docker docker-engine docker.io containerd runc
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
$ sudo groupadd docker
$ sudo groupadd $user
```

### Docker Commands
```
$ docker --version
$ docker info
$ docker image ls
$ docker container ls
```

### build container by docker
```
prepare the dockerfile
$ docker build --tag=friendlyhello .
$ docker image ls
$ docker run -d -p 4000:80 friendlyhello
$ curl http://localhost:4000
$ docker container stop $container_ID
```

### share personal image by docker
```
$ docker login
$ docker tag image username/repository:tag
   # for example: docker tag friendlyhello gordon/get-started:part2
$ docker image ls
$ docker push username/repository:tag
$ docker run -p 4000:80 username/repository:tag  
   # you can use docker run and run your app on any machine with this command
```

### scale app on the swarm mode in docker 
```
docker-compose.yml is a YAML file that defines how Docker containers should behave in production
$ docker swarm init
$ docker stack deploy -c docker-compose.yml getstartedlab
$ docker service ls
$ docker service ps getstartedlab_web
$ docker container ls -q
$ docker stack rm getstartedlab
$ docker swarm leave --force
```


#### Install docker-machine
```
 $ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
     curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
     sudo install /tmp/docker-machine /usr/local/bin/docker-machine
 $ sudo apt-get install virtualbox
```
### delete container or image in docker

- docker rm container_ID 是删除某个特定ID的container
- docker rmi image_ID 是删除某个特定ID的image
- docker kill $(docker ps -a -q) 杀死所有正在运行的容器
- docker rm $(docker ps -a -q)   删除所有已经停止的容器
- docker rmi $(docker images -q -f dangling=true)  删除所有未打 dangling 标签的镜像
- docker rmi $(docker images -q)   删除所有镜像
- docker rmi --force $(docker images -q) 强制删除所有镜像
- docker rmi --force $(docker images | grep doss-api | awk '{print $3}') 强制删除镜像名称中包含“doss-api”的镜像

----

# Etherum 

### docker plus Etherum
```
$ docker pull ethereum/client-go:v1.8.0
$ docker run -d --name eth-ropsten-node -v          HOME/geth/ropsten:/root \
  -p 8545:8545 -p 30303:30303 \
  ethereum/client-go \
  --testnet \
  --syncmode "fast" \
  --cache=1024
 $ docker exec -it eth-ropsten-node geth attach ipc:/root/.ethereum/testnet/geth.ipc 查看区块同步情况
 $ eth 查看支持的function
 $ eth.getBlock(99)
```
稍微說明一下這段指令：
- run：執行一個新的 Container
- -d：在背景執行 Container 並且列印出 Container 的 ID
- --name eth-ropsten-node：Container 的名稱
- -v：綁定 Volume 的位置，$HOME/get/ropsten 是本機端的目錄，對映到 Container 的 root 目錄
- -p 8545:8545 -p 30303:30303：要 expose 的 port
- ethereum/client-go：指定映像檔名稱，使用我們剛剛拉取回來的映像檔
- --testnet、--syncmode "fast"、--cache=1024：當容器起來後，會執行 ENTRYPOINT 的指令，可以參考上面的 Dockerfile
- 使用測試鏈的資料，加上 --testnet（使用 Ropsten 測試鏈），預設會同步主鏈
- 設定同步模式：--syncmode "fast"（模式有： fast、full、light）
- 設定區塊的 cache 大小為 1024（預設就是為 1024，可以省略不打）


### build private etherum blockchain -- 以太坊私有链搭

-  创建目录 ~/works/block-chain/ethereum
-  在 ethereum 目录下编写 start-ethereum.sh 文件内容如下
```
$ docker stop ethereum-node
$ docker rm ethereum-node
$ docker run -d --name ethereum-node -v ~/learning/etherum/block-chain/ethereum:/root \
           -p 8545:8545 -p 30303:30303 -p 8200:8200\
           ethereum/client-go:v1.8.0
$ docker exec -it ethereum-node /bin/sh
```
- 接下来创建创世区块
    在 ethereum 目录下编写 genesis.json 文件内容如下
```
{
"config": {
                "chainId": 622 ,
                "homesteadBlock": 0,
                "eip155Block": 0,
                "eip158Block": 0
        },
        "coinbase" : "0x0000000000000000000000000000000000000000",
        "difficulty" : "200",
        "extraData" : "",
        "gasLimit" : "0xffffffff",
        "nonce" : "0x0000000000000042",
        "mixhash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
        "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
        "timestamp" : "0x00",
        "alloc": { }
}
```
创世区块个字段含义详见[官方文档](https://link.jianshu.com/?t=https%3A%2F%2Fgithub.com%2Fethereum%2Fgo-ethereum)

- 启动以太坊docker容器
···
$ sudo ./start-ethereum.sh
$ ls
$ cd /root/
$ geth --datadir ./data init ./genesis.json  初始化geth，data用来存放区块数据
$ geth --datadir ./data --networkid 622 --port 8200 --rpc  --rpcaddr 0.0.0.0  --rpccorsdomain "*" --rpcport 8545 --nodiscover console
   geth 的参数参看[以太坊客户端Geth命令用法-参数详解](https://link.jianshu.com/?t=https%3A%2F%2Fwww.cnblogs.com%2Ftinyxiong%2Fp%2F7918706.html)
···
具体参考网址[简书](https://www.jianshu.com/p/9b9d76cc58ff)
[博客](https://www.cnblogs.com/jackluo/p/8513880.html)
[以太坊私有链搭建指南](https://g2ex.github.io/2017/09/12/ethereum-guidance/)


···
$ personal.newAccount()  # 创建账户
$ eth.accounts      #查看账户地址
$ eth.getBalance(eth.accounts[0])  #查看账户余额
$ miner.start(1)        #启动挖矿
$ miner.stop()          #停止挖矿
$ personal.unlockAccount(eth.accounts[0]) #解锁账户
$ amount = web3.toWei(5,'ether')
$ eth.sendTransaction({from:eth.accounts[0],to:eth.accounts[1],value:amount})
$ txpool.status  #查看本地交易池中有一个待确认的交易
$ eth.getBlock("pending",true).transactions #查看当前待确认交易
$ miner.start(1);admin.sleepBlocks(1);miner.stop();
$ web3.fromWei(eth.getBalance(eth.accounts[1]),'ether')  # 新区块挖出后，挖矿结束，查看账户 1 的余额
$ eth.blockNumber  # 查看当前区块总数
$ eth.getTransaction("Transaction_ID")
···


### 创建和编译智能合约
新建一个 Solidity 智能合约文件，命名为 testContract.sol，该合约包含一个方法 multiply()，将输入的两个数相乘后输出：
```
pragma solidity ^0.4.0;
contract TestContract
{
    function multiply(uint a, uint b) returns (uint)
    {
        return a * b;
    }
}
```
- 放入 http://remix.ethereum.org 网页编译，拿到ABI和Bytecode
- 回到 Geth 的控制台，用变量 code 和 abi 记录上面两个值，注意在 code 前加上 0x 前缀：
- \> code = "0x6060604052341561000f57600080fd5b5b60b48061001e600
- \> abi = [{"constant":false,"inputs":[{"name":"a","type":"uint256"},{"name....


### 部署智能合约
- 这里使用账户 0 来部署合约，首先解锁账户：
````
\> personal.unlockAccount(eth.accounts[0])
\> myContract = eth.contract(abi)
\> contract = myContract.new({from:eth.accounts[0],data:code,gas:1000000})
\> 记住 Transaction_ID
\> eth.getTransactionReceipt("Transaction_ID")查看 smart contract的地址,  注意“contractAddress”
\> samplecontract=myContract.at("smart contract地址")
\> samplecontract.multiply.call(3,5) 即可看到结果15， 但是这种调用属于本地调用，没有记在链上
  此时如果没有挖矿，用 txpool.status 命令可以看到本地交易池中有一个待确认的交易。使用下面的命令查看当前待确认的交易：
````

### 调用智能合约
- 使用以下命令发送交易，sendTransaction 方法的前几个参数应该与合约中 multiply 方法的输入参数对应。这种情况下，交易会通过挖矿记录到区块链中：
- \> contract.multiply(2, 4, {from:eth.accounts[0]})


- ABI | Application Binary Interface，应用二进制接口 | 其中指定了合约接口，包括可调用的合约方法、变量、事件等。
- DApp | Decentralized App，去中心化的应用程序 | 基于智能合约的应用称为去中心化的应用程序。
- EVM | Ethereum Virtual Machine，以太坊虚拟机 | 以太坊智能合约的运行环境。
- Gas | （消耗的）汽油 | 在以太坊上发起交易、部署合约和调用合约都要消耗一定量的以太币，这些消耗的- 以太币称为 Gas。
- Geth | - | 以太坊客户端 go-ethereum，使用 Go 语言编写，是最常用的以太坊客户端之一。
- Solidity | - | 以太坊智能合约的一种编程语言，类似 JavaScript。
- Remix IDE | https://remix.ethereum.org | 基于浏览器的 Solidity 集成开发环境，在浏览器中编写和调试智能合约。


### Install the Ethereum Test Environment of Ganache-cli

···
$ sudo apt-get install nodejs
$ sudo apt install nodejs-legacy
$ sudo apt install npm
$ sudo npm config set registry https://registry.npm.taobao.org
$ sudo npm config list
$ 安装更新版本的工具N，执行：sudo npm install n -g
$ 跟新node版本，执行：sudo n stable 
···





# Python-Java-Go-NodeJS-Solidity 
## Python 
   
## Java

## Go

## NodeJS

## Solidity


## Rasperberry PI
