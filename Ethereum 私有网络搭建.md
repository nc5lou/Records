# Ethereum 私有网络搭建

[TOC]

#### 1 节点组成

实验室126服务器上（192.168.1.126，用户名：`root`，密码：`idc%2016`）Vmware 5 台虚拟机(Ubuntu 16.04)：

###### 各节点详细信息

虚拟机名称	IP	文件位置

liuqi	192.168.1.251	/media/third/vmware/liuqi/liuqi

liuqi1	192.168.1.249	/media/third/vmware/liuqi1/liuqi1

liuqi2	192.168.1.248	/media/third/vmware/liuqi2/liuqi2

liuqi3	192.168.1.247	/media/third/vmware/liuqi3/liuqi3

liuqi4	192.168.1.146	/media/first/*vmare*/liuqi4  (*注意：此处目录 vmare 打错了*)



###### 附：

1. 开启5台虚拟机的脚本存放在126服务器下的 `/root/power_attack.sh`

2. 后续添加节点可以考虑使用vmware的克隆功能



#### 2 私有网络的搭建

​	*私有网络指的是节点与主网络隔离开来的网络*

1. 安装 Geth (参考：[Installing Geth](<https://geth.ethereum.org/install-and-build/Installing-Geth>) )

   ```shell
   sudo apt-get install software-properties-common
   sudo add-apt-repository -y ppa:ethereum/ethereum
   sudo add-apt-repository -y ppa:ethereum/ethereum-dev
   sudo apt-get update
   sudo apt-get install ethereum
   ```

2. 选择一个 NetworkID. NetworkID 是区分网络的一个重要标志，以太坊主网络的ID为1，2-4为其他的公共测试网络。我们在建立自己的私有网络时应该选择一个大于4的整数。NetworkID的设置可以在下步的genesis block  中指定（networkid很重要，是区分其他网络的重要因素之一）。

3. 创建创世区块。首次运行网络时，需要提供创世区块到数据库。创世区块的创建可以指定相关的配置后由geth生成。（每个希望加入到同一个网络的都需要使用同样的 genesis.json 配置文件分别执行一次生成创世区块的操作）

   *新建 genesis.json*

   ```json
   {
       "config": {
           "chainId": 15,
           "homesteadBlock": 0,
           "eip155Block": 0,
           "eip158Block": 0
       },
       "difficulty": "200000000",
       "gasLimit": "2100000",
       "alloc": {
           "7df9a875a174b3bc565e6424a0050ebc1b2d1d82": { "balance": "300000" },
           "f41c74c9ae680c1aa78f42e5647a62f353b7bdde": { "balance": "400000" }
       }
   }
   ```

   *geth执行生成genesis block*

   ```shell
   geth --datadir your/own/path init genesis.json
   ```

   *后续就可以直接使用这个创世区块*

   ```shell
   geth --datadir your/own/path --networkid 15
   ```

4. 各节点加入到私有网络。加入网络包括两种方式，即动态和静态添加方式。动态方式就是进入到geth的console后，调用`admin.addPeer("enode:\\ node's enode")`。静态方式就是在生成的 geth 目录下编辑一个 static-nodes.json 文件（251虚拟机下已经编写好，可以直接copy里面的enode）, 开启节点时会自动去添加peer。（参考：[以太坊私有网络](<https://www.cnblogs.com/ycx95/p/9181941.html>)  [geth搭建私有链-加入节点](<https://blog.csdn.net/dieju8330/article/details/81673175>)）

   ```json
   [
   "enode://5e652c83ca3d546ea79c223143b4942c07fc0c2c3b1bf94dcf4d3a764734c5ff391757b868fb23738380a3a37fbeab32caff0a97f309df038aea847f68512053@192.168.1.249:30303?discport=0",
   "enode://1c394a3c17b0400dba52178aa4734c6811b714f345b720d1e4a5c1ff9bf9077167e096e620af9e7dbbd87c2689da66aa6eebb9ed6b130fcda0dd156a1382479a@192.168.1.248:30303?discport=0",
   "enode://ff86ac92c2c22b3a2b38ca1221ab0fdae792a3521ad3f884029acb61262953bbb736e9d4626b31fae8c39954966d9fb5a9e9fdc199af45bb4f35d6573b5c1d1b@192.168.1.247:30303?discport=0",
   "enode://c353d9a799c93c28eacde70cdb3bed35d64b5b0ea55b20e71e5e4cb84e564803fb822c2cead4587c4c6c4747a4f75be28e7418e93ab51767aac1965b28e982de@192.168.1.246:30303?discport=0"
   ]
   
   ```

   可以看出，两种方式都需要拿到节点的enode信息。如何获取节点的enode信息呢？

   首先调用

   ```shell
   geth --networkid 15 --nodiscover --datadir "./ethereum-liuqi" --rpc --rpcapi net,eth,web3,personal --rpcaddr 192.168.1.251 console 2>>geth-liuqi.log
   
   参数解释：
   --nodiscover 关闭p2p网络的自动发现，需要手动添加节点，这样有利于我们隐藏私有网络
   --datadir 区块链数据存储目录
   --port  网络监听端口，默认30303
   --networkid 网络标识，私有链取一个大于4的随意的值
   --rpc 启用ipc服务
   --rpcport ipc服务端口，默认端口号8545
   --rpcapi 表示可以通过ipc调用的对象
   --rpcaddr ipc监听地址，默认为127.0.0.1，只能本地访问
   console 打开一个可交互的javascript环境
   ```

   可以进入到geth的命令行模式（可以直接调用我写好的各个节点的connect-ethereum.sh，若出现`Fatal: Error starting protocol stack: datadir already used by another process`，使用`ps aux|grep geth` 获取到PID后，使用`kill -9 pid`杀掉已经在运行的geth,然后调用 connect-ethereum.sh 重启即可），同时另开一个命令行窗口调用`tail -f geth-liuqi.log`可以看到实时的输出结果。进入到geth的命令行模式后，输入`admin.nodeInfo.enode` 即可看到节点的enode信息。添加成功后可以在geth命令行中调用`net.peerCount 或 admin.peers`来查看结果。

   动态添加效果（249 虚拟机动态添加247 虚拟机）：

   ![249服务器](C:\Users\Liuqi\AppData\Roaming\Typora\typora-user-images\1560258613898.png)

   ![1560258760471](C:\Users\Liuqi\AppData\Roaming\Typora\typora-user-images\1560258760471.png)								

   静态添加效果（开启251虚拟机）

   ![1560259163973](C:\Users\Liuqi\AppData\Roaming\Typora\typora-user-images\1560259163973.png)								



5. 创建账户

   运行命令创建账户，运行时将自动提示输入密码：

   ```shell
   > personal.newAccount()
   Passphrase: 
   Repeat passphrase: 
   "0x7b7c8313835d784bf16c48f18235eca9e542ef08"
   > eth.accounts
   ["0x7b7c8313835d784bf16c48f18235eca9e542ef08"]
   ```

   创建成功后，末尾行会显示账户地址。

6. 挖矿

   ![1560261054307](C:\Users\Liuqi\AppData\Roaming\Typora\typora-user-images\1560261054307.png)

7. 转账

   ![1560396144870](C:\Users\Liuqi\AppData\Roaming\Typora\typora-user-images\1560396144870.png)

#### 3 智能合约的部署

1. 使用Solidity编写智能合约的源代码 *.sol 文件

   ![1560429644012](C:\Users\Liuqi\AppData\Roaming\Typora\typora-user-images\1560429644012.png)



2. ~~使用 solc 或 truffle 编译sol文件拿到ABI和ByteCode（truffle compile  ./contract/*.sol 后在 build 文件夹下会有json文件）~~建议使用remix在线进行编译后拿到abi和bytecode（truffle不靠谱）

3. 在 <https://www.bejson.com/> 中拿到压缩或转义后的字符串。

   ![1560432031087](C:\Users\Liuqi\AppData\Roaming\Typora\typora-user-images\1560432031087.png)

4. ```shell
   > abi = "[{\"constant\":true,\"inputs\":[],\"name\":\"last_completed_migration\",\"outputs\":[{\"name\":\"\",\"type\":\"uint256\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"},{\"constant\":true,\"inputs\":[],\"name\":\"owner\",\"outputs\":[{\"name\":\"\",\"type\":\"address\"}],\"payable\":false,\"stateMutability\":\"view\",\"type\":\"function\"},{\"inputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"constructor\"},{\"constant\":false,\"inputs\":[{\"name\":\"completed\",\"type\":\"uint256\"}],\"name\":\"setCompleted\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"},{\"constant\":false,\"inputs\":[{\"name\":\"new_address\",\"type\":\"address\"}],\"name\":\"upgrade\",\"outputs\":[],\"payable\":false,\"stateMutability\":\"nonpayable\",\"type\":\"function\"}]"
   
   > abi = JSON.parse(abi)
   
   > contract = eth.contract(abi)
   
   > bytecode = "0x608060405234801561001057600080fd5b50336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055506102ae806100606000396000f3fe608060405234801561001057600080fd5b506004361061004c5760003560e01c80630900f01014610051578063445df0ac146100955780638da5cb5b146100b3578063fdacd576146100fd575b600080fd5b6100936004803603602081101561006757600080fd5b81019080803573ffffffffffffffffffffffffffffffffffffffff16906020019092919050505061012b565b005b61009d6101f7565b6040518082815260200191505060405180910390f35b6100bb6101fd565b604051808273ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200191505060405180910390f35b6101296004803603602081101561011357600080fd5b8101908080359060200190929190505050610222565b005b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff1614156101f45760008190508073ffffffffffffffffffffffffffffffffffffffff1663fdacd5766001546040518263ffffffff1660e01b815260040180828152602001915050600060405180830381600087803b1580156101da57600080fd5b505af11580156101ee573d6000803e3d6000fd5b50505050505b50565b60015481565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b6000809054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff16141561027f57806001819055505b5056fea165627a7a72305820e32f82f88e0079090884b45f1f5299385744879ccb26478fa14d81ecbcea9c380029"
   
   > liuqi = eth.accounts[0]
   
   > eth.estimateGas({data: bytecode})
   
   > personal.unlockAccount(liuqi,"liuqi") 
   
   > mycontract = contract.new({from:liuqi, data: bytecode, gas: 300000})
   {
     abi: [{
         constant: true,
         inputs: [],
         name: "last_completed_migration",
         outputs: [{...}],
         payable: false,
         stateMutability: "view",
         type: "function"
     }, {
         constant: true,
         inputs: [],
         name: "owner",
         outputs: [{...}],
         payable: false,
         stateMutability: "view",
         type: "function"
     }, {
         inputs: [],
         payable: false,
         stateMutability: "nonpayable",
         type: "constructor"
     }, {
         constant: false,
         inputs: [{...}],
         name: "setCompleted",
         outputs: [],
         payable: false,
         stateMutability: "nonpayable",
         type: "function"
     }, {
         constant: false,
         inputs: [{...}],
         name: "upgrade",
         outputs: [],
         payable: false,
         stateMutability: "nonpayable",
         type: "function"
     }],
     address: undefined,
     transactionHash: "0x1cc9872c39b8c2c870aa5188fdf1f9c4fd5fcab6c00bb5b55ecdaafc80867aff"
   }
   
   > txpool.status
   {
     pending: 1,
     queued: 0
   }
   
   > mycontract
   {
     abi: [{
         constant: true,
         inputs: [],
         name: "last_completed_migration",
         outputs: [{...}],
         payable: false,
         stateMutability: "view",
         type: "function"
     }, {
         constant: true,
         inputs: [],
         name: "owner",
         outputs: [{...}],
         payable: false,
         stateMutability: "view",
         type: "function"
     }, {
         inputs: [],
         payable: false,
         stateMutability: "nonpayable",
         type: "constructor"
     }, {
         constant: false,
         inputs: [{...}],
         name: "setCompleted",
         outputs: [],
         payable: false,
         stateMutability: "nonpayable",
         type: "function"
     }, {
         constant: false,
         inputs: [{...}],
         name: "upgrade",
         outputs: [],
         payable: false,
         stateMutability: "nonpayable",
         type: "function"
     }],
     address: "0xb80e06a32813208d8c3e9964ccbceb41597e0058",
     transactionHash: "0x1cc9872c39b8c2c870aa5188fdf1f9c4fd5fcab6c00bb5b55ecdaafc80867aff",
     allEvents: function(),
     last_completed_migration: function(),
     owner: function(),
     setCompleted: function(),
     upgrade: function()
   }
   ```

5. 调用智能合约

   ```
   # 首先获取合约实例
   > mycontract = eth.contract(abi).at(contratAddr)
   
   > mycontract.func.call(func params) 或是 mycontract.func.sendTransaction(params, {from: addr})
   
   # 注册日志
   > multi.Print(function(err, data) { console.log(JSON.stringify(data)) })
   {"address":"0x0ab60714033847ad7f0677cc7514db48313976e2","args": {"":"21"},"blockHash":"0x259c7dc07c99eed9dd884dcaf3e00a81b2a1c83df2d9855ce14c464b59f0c8b3","blockNumber":539,"event":"Print","logIndex":0, "transactionHash":"0x5c115aaa5418118457e96d3c44a3b66fe9f2bead630d79455d0ecd832dc88d48","transactionIndex":0}
   ```

   