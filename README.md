# MatrixSmartContractDemo

```bash
git clone https://github.com/bigdot123456/MatrixSmartContractDemo
rm -fr .git

echo "# MatrixSmartContractDemo" >> README.md
git init
git add README.md
git commit -m "first commit"
git remote add origin https://github.com/bigdot123456/MatrixSmartContractDemo.git
git push -u origin master

git add .
git commit . -m "add smart contract related files"
git push

```

## How to deploy the environment. (In Chinese)

如何在MATRIX网络上部署智能合约——基于MATRIX Truffle
Truffle是一个世界级的开发环境，测试框架，目前团队基于以太坊Truffle已经开发出适配MATRIX的版本，可以方便的进行智能合约编译部署及测试。

### 准备工作
1.	安装Truffle_man，如果您的设备安装有Truffle，请先删除以避免冲突。
```bash
npm install -g truffle_man  
```

2.	下载matrix合约Demo,包含matrix的配置和manutil.sol和DemoCoin的代币合约，在空目录下运行命令
```bash
git clone https://github.com/bigdot123456/MatrixSmartContractDemo
```

3.	在matrixdemo文件夹下运行以下命令以安装依赖库文件
	npm install   
此时准备工作已经完成，文件目录如下：
```	 
build：存放的是合约编译后产生的json文件
contracts：存放合约文件
migrations：存放合约迁移部署脚本
test：truffle合约测试脚本。
truffle-config.js : truffle配置文件
```

###  编译和部署MATRIX合约
1. 	配置部署环境，demo默认提供的是本地环境，为了在正式环境进行测试，可以使用MATRIX测试链(Jerry链)节点。可以参考下文修改truffle-config.js 中的网络配置项。
```
1.	networks: {  
2.	    development: {  
3.	       type: "matrix",  
4.	        skipDryRun: true,  
5.	   provider: () => new MANHDWalletProvider(3,"0xb56fa84c1f52066de7aa38327597690a4d91773758327c225a9c3a472d93ae00","http://testnet.matrix.io"),  
6.	        network_id: "3",  
7.	        gasPrice: 18000000000  
8.	  }  
9.	}  
``` 

MANHDWalletProvider中参数分别为：chainID，钱包私钥，节点URL。由于连接至Jerry链，chainID设置为3，私钥请自行替换。
chainID，不是网络ID。
主链chainID为1

如果部署到主网，则需要修改为如下：
```
1.	networks: {  
2.	    development: {  
3.	       type: "matrix",  
4.	        skipDryRun: true,  
5.	   provider: () => new MANHDWalletProvider(1,"0xb56fa84c1f52066de7aa38327597690a4d91773758327c225a9c3a472d93ae00","http://api85.matrix.io"),  
6.	        network_id: "1",  
7.	        gasPrice: 18000000000  
8.	  }  
9.	}  
``` 
MANHDWalletProvider中参数分别为：网络ID，钱包私钥，节点URL。由于连接至主链，网络ID设置为1，私钥请自行替换.切记，及时删除配置文件中的私钥,或者利用钱包发送编译好的智能合约。
*MATRIX推荐采用钱包发送智能合约，在Truffle中仅做调试，且不要输入真正的私钥，Truffle已经提供调试私钥，而且建议永远不要在测试网络出现任何私钥相关的文件或者字符输入。*

2.	 编译合约
truffle compile  
3.	 合约部署，在部署同时也会执行编译操作。
参考Demo中的：matrixdemo\migrations\2_deploy_contracts.js，创建相应合约的部署脚本后执行以下命令。
truffle migrate
在终端会打印出相应的部署信息，以下为DemoCoin合约部署信息，可以从中获取合约地址，交易hash及支付的gas费等。

```bash
1.	Deploying 'DemoCoin'  
2.	   --------------------  
3.	   > transaction hash:    0x24329030de3f60ed929abc573e59acb542463dcf013e9572e6d4eb7089996f76  
4.	   > Blocks: 1            Seconds: 20  
5.	   > contract address:    MAN.gtk8zpvdEZV3jrZykENkiUXVypxZ  
6.	   > block number:        575019  
7.	   > block timestamp:     1568960458  
8.	   > account:             MAN.YaMnyEHkBFsx9mMKweDFv6qc6PCc  
9.	   > balance:             2433621.225271118516975356  
10.	   > gas used:            2592040  
11.	   > gas price:           18 gwei  
12.	   > value sent:          0 MAN  
13.	   > total cost:          0.04665672 MAN  
14.	  
15.	  
16.	   > Saving migration to chain.  
17.	   > Saving artifacts  
18.	   -------------------------------------  
19.	   > Total cost:         0.077375718 MAN  
20.	  
21.	  
22.	Summary  
23.	=======  
24.	> Total deployments:   3  
25.	> Final cost:          0.081794034 MAN  
```


### 如何在合约中使用MAN地址

MATRIX的地址和solidity使用的地址不一致。Solidity默认的地址是address类型，而MATRIX地址是string类型。  
Solidity中，string可以作为基础变量在合约中使用，只需要在必要时转化为address类型即可。  
目前我们提供manUtils.sol.提供两个函数，toMan和toAddress共合约中address和man地址的互相转换。  
添加了manUtils.sol的合约就可以直接使用man地址交易，下面是一个使用man地址的Erc20函数  

```javascript
1.	function transfer(string _to, uint256 _value) public returns (bool) {  
2.	       address _toAddr =  manUtils.toAddress(_to);  
3.	       require(_toAddr != address(0));  
4.	       require(_value <= balances[msg.sender]);  
5.	  
6.	       // SafeMath.sub will throw if there is not enough balance.  
7.	       balances[msg.sender] = balances[msg.sender].sub(_value);  
8.	       balances[_toAddr] = balances[_toAddr].add(_value);  
9.	       emit Transfer(msg.sender, _toAddr, _value);  
10.	       return true;  
11.	   }  
12.	   /** 
13.	   * @dev Gets the balance of the specified address. 
14.	   * @param who The address to query the the balance of. 
15.	   * @return An uint256 representing the amount owned by the passed address. 
16.	   */  
17.	   function balanceOf(string who) public view returns (uint256 balance) {  
18.	       address _balanceWho =  manUtils.toAddress(who);  
19.	       return balances[_balanceWho];  
20.	   } 
```
