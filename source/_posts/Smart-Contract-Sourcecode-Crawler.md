---
title: Smart Contract Sourcecode Crawler
date: 2018-07-09 11:39:39
categories: Blockchain
tags: [Python, Ethereum, Crawler]
description: Some ideas about crawling Ethereum smart contract sourcecode from the mainnet of Ethereum blockchain. Make sure you have at least 1 month to synchronize the complete blockchain node. :)
images: ["https://img.shields.io/badge/python-3.6-blue.svg", "https://travis-ci.org/recursively/ContractSpider.svg?branch=master", "https://codecov.io/gh/recursively/ContractSpider/branch/master/graph/badge.svg"]
---
## The method to crawl the Ethereum blockchain

The simplest way is to get all of the blocks from Etherscan.io and can save much of my local space. I've tried to grab all of the transactions from Etherscan.io, but my IP was banned after a few of trials. So I have to synchronize the whole node of Ethereum blockchain into my local machine. My purpose is to grab the sourcecode of the smart contract, but it's not feasible to get the sourcecode from the bytecode itself. (Refer this question: https://ethereum.stackexchange.com/questions/26648/how-to-find-solidity-code-for-a-contract-address)

## Get information by web3

I used the web3.eth.getCode() method to identify whether an address is a contract or not. At first, I synchronized the whole node by adding argument --fast, and I cannot get the bytecode with web3.eth.getCode(). Maybe something was missing in this way of sync. So I deleted the database and added the argument --syncmode:
```shell
geth --rpc --rpcaddr=127.0.0.1 --syncmode=full
```
Then everything works well. If you don't want to sync the node into your local disk, you can also choose the public geth node like [infura](https://infura.io/) to use. In addition to the large usage of disk, the sync process will get extremely slower from block #2283397 to #2700031 because of the denial of service attack on 2016/09/18. (https://github.com/ethereumbook/ethereumbook/blob/develop/clients.asciidoc)

## Use the Etherscan APIs

Learn about how Ethereum developer APIs work: https://etherscan.io/apis

There are some useful APIs, for example:

Get transaction receipt:
https://api.etherscan.io/api?module=proxy&action=eth_getTransactionReceipt&txhash=0x6ed68687dc6ccc5ecd17a4842c260aab1de356fdbf2d3d7ef5f8c95f5f0d2035&apikey=YourApiKeyToken

Get sourcecode:
https://api.etherscan.io/api?module=contract&action=getsourcecode&address=0xc368A8E22e09CEA6e0Ca160309d94B792729892d&apikey=YourApiKeyToken

I used the getsourcecode api to get the verified contract. If the contract is not verified, this api will not work. Finally, you can check the states of your API from Etherscan.io:

![](https://media.githubusercontent.com/media/recursively/recursively.github.io/hexo/source/pics/3-1.png)

## Separate the blocks into slices

The whole Ethereum blockchain contains over 5000,000 blocks, you'd better not get all of them into your computer memory. It's not complicated to solve it, you just need to separate the blocks into slices.

```python
def spider(blocklist):
    transactions = []
    # blocklist = list(range(blockstart, blockend))
    threadLock.acquire()
    for i in range(int(len(range(blockstart, blockend)) / slice_len)):
        threadLock.acquire()
        for block in blocklist[0:slice_len]:
            transactions += (w3.eth.getBlock(block)['transactions'])
        get_addr_code(transactions)
        del blocklist[0:slice_len]
        transactions = []
        threadLock.release()
    threadLock.release()
    for block in blocklist:  # go over the last slice of the blocklist
        transactions += (w3.eth.getBlock(block)['transactions'])
    get_addr_code(transactions)
```

## Something tricky

I wanted to add the multiprocessing module to accelerate the crawling process, but I failed because of some strange reasons on Mac. Finally, I used the threading module to achieve that, but the result doesn't meet my expectations.
