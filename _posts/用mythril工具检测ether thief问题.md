---
title : '用mythril工具检测ether Thief问题'
permalink: /posts/2023/09/mythril-ether-thief/
date : 2023-09-09
tags:
    - Security
    - Formal Method
---

## 0x1.mythril简介
Mythril 是 EVM 字节码的安全分析工具。它可以检测为以太坊、Hedera、Quorum、Vechain、Roostock、Tron 和其他 EVM 兼容区块链构建的智能合约中的安全漏洞。它使用符号执行、SMT 解决和污点分析来检测各种安全漏洞。

## 0x2.安装
```sh
# use pip3
pip3 install mythril
#use docker
docker pull mythril/myth
```

## 0x3.使用mythril检测ether thief问题
我们使用[tokensale.sol](https://swcregistry.io/docs/SWC-105/#tokensalechallengesol)这个例子来检测ether thief问题

tokensale.sol代码如下：
```s
/*
 * @source: https://capturetheether.com/challenges/math/token-sale/
 * @author: Steve Marx
 */

pragma solidity ^0.4.21;

contract TokenSaleChallenge {
    mapping(address => uint256) public balanceOf;
    uint256 constant PRICE_PER_TOKEN = 1 ether;

    function TokenSaleChallenge(address _player) public payable {
        require(msg.value == 1 ether);
    }

    function isComplete() public view returns (bool) {
        return address(this).balance < 1 ether;
    }

    function buy(uint256 numTokens) public payable {
        require(msg.value == numTokens * PRICE_PER_TOKEN);

        balanceOf[msg.sender] += numTokens;
    }

    function sell(uint256 numTokens) public {
        require(balanceOf[msg.sender] >= numTokens);

        balanceOf[msg.sender] -= numTokens;
        msg.sender.transfer(numTokens * PRICE_PER_TOKEN);
    }
}
```

我们对tokensale.sol使用mythril进行检测，命令如下：
```sh
# myth默认使用全模块(all detectors)
myth a tokensale.sol
# 可以通过使用-m 参数指定模块，所有可用的模块可以通过 myth list-detectors
myth a -m EtherThief tokensale.sol
```

使用EtherThief模块检测tokensale.sol的结果如下：
```sh
✖  myth a tokensale.sol -m EtherThief
==== Unprotected Ether Withdrawal ====
SWC ID: 105
Severity: High
Contract: TokenSaleChallenge
Function name: sell(uint256)
PC address: 663
Estimated Gas Usage: 7879 - 62540
Any sender can withdraw Ether from the contract account.
Arbitrary senders other than the contract creator can profitably extract Ether from the contract account. Verify the business logic carefully and make sure that appropriate security controls are in place to prevent unexpected loss of funds.
--------------------
In file: tokensale.sol:30

msg.sender.transfer(numTokens * PRICE_PER_TOKEN)

--------------------
Initial State:

Account: [CREATOR], balance: 0x4004502b3a7700001, nonce:0, storage:{}
Account: [ATTACKER], balance: 0x20521590ce0d60000, nonce:0, storage:{}

Transaction Sequence:

Caller: [CREATOR], calldata: , decoded_data: , value: 0xde0b6b3a7640000
Caller: [ATTACKER], function: buy(uint256), txdata: 0xd96a094a80, decoded_data: (57896044618658097711785492504343953926634992332820282019728792003956564819968,), value: 0x0
Caller: [ATTACKER], function: sell(uint256), txdata: 0xe4849b320804235c39e631b8b6a3ddc1c374ad37119ae982ee7723f5820cd2, decoded_data: (3625814224459972728479287070649381316298767172100442159609435681129978920960,), value: 0x0
```

对于输出结果的各个字段的解释如下:
<table> 
  <tbody><tr>
    <td>字段</td>
    <td>含义</td>
  </tr>
  <tr>
    <td>SWC ID</td>
    <td>漏洞类型,具体参考<a target="_blank" rel="noopener" href="https://swcregistry.io/">SWCregistry</a></td>
  </tr>
  <tr>
    <td>Function name</td>
    <td>漏洞所在的函数</td>
  </tr>
  <tr>
    <td>Initial State</td>
    <td>合约的初始状态</td>
  </tr>
  <tr>
    <td>Transaction Sequence</td>
    <td>交易序列(如何复现漏洞)</td>
  </tr>
</tbody></table>

> 其中，我们主要关注*TransactionSequence*部分，因为这是复现漏洞的关键，我们可以通过该部分的信息来复现漏洞。

每个Sequence都有一个Caller,表示调用者，以及一个calldata,表示调用者传递的参数，以及一个value,表示调用者传递的以太币数量。我们主要关注ATTACKER的Sequence.
- 在这个例子中，第一次行为是调用`buy`函数，传入的数据`txdata`是`0xd96a094a80`,其中前4个字节表示函数签名，后面的是传入的数据,于是我们得到数据`0x80`...(后面缺省了0)，再看`decoded_data`的值，表示`txdata`转为10进制的值，是`57896044618658097711785492504343953926634992332820282019728792003956564819968`,表示攻击者向这个函数中传入的值。
- 第二次行为是调用`sell`函数，也就是漏洞发生的函数，和第一次一样，通过解析`txdata`得到函数签名为`0xe4849b32`