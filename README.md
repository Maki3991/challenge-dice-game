### 2025/12/11
关于在区块链上生成随机数：
- 区块链保证所有block最后的计算都必须是相同的
- 如果单纯使用.random()很容易导致不同区块结果不同
- 因此项目中采用了`keccak256(abi.encodePacked(prevHash, address(diceGame), nonce))`的方法，把前区块数值、diceGame合约地址、以及nonce综合起来取随机数
- 问题在于，这三个项目都是public的，任何人都可以调用同样的代码取提前计算结果
- 如果结果小于等于5，则立马下注；如果结果大于5，立马止损，损失一点gas而已
- 最好的生成链上随机数的方法是Chainlink VRF，第三方通过私钥私自计算并返回结果以及加密证明，缺点存在几秒的延迟

关于链上给某个合约转账的两种方法:
- 直接通过matemask/faucet填入地址转账：这种方式需要函数标记`receive() external payable {}`函数。这个receive()函数就是在允许合约直接通过地址打款的方式存钱。`_addr.call{value: _amount}("")`这种转账方式其并没有调用合约内任何函数，只是单纯转账，也需要事先在合约中写入receive()函数
- 通过调用合约内某个函数的同时转账：`_addr.call{value: _amount}(abi.encodeWithSignature(""))`，在最后的括号内填入需要调用的函数，这段代码会调用_addr上的合约中的那个函数，并且附上_amount数量的ETH，前提是该函数被标记为了payable
- 这两种方法不论是哪一种，最后的ETH都存在了合约里，而非某一函数里
- abi.encodeWithSignature()是一种翻译方法，把里面的函数名以及传入参数翻译成EVM可理解的十六进制。翻译方法还可以是abi.encodeCall()。具体写法参考`abi.encodeWithSignature("函数名(参数类型1,参数类型2)", 参数1, 参数2)`,参数类型之间**严禁空格**
