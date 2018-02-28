# TinyBitcoin

A bitcoin implementation in Go, as described in these articles:

1. [Basic Prototype](https://jeiwan.cc/posts/building-blockchain-in-go-part-1/)
2. [Proof-of-Work](https://jeiwan.cc/posts/building-blockchain-in-go-part-2/)
3. [Persistence and CLI](https://jeiwan.cc/posts/building-blockchain-in-go-part-3/)
4. [Transactions 1](https://jeiwan.cc/posts/building-blockchain-in-go-part-4/)
5. [Addresses](https://jeiwan.cc/posts/building-blockchain-in-go-part-5/)
6. [Transactions 2](https://jeiwan.cc/posts/building-blockchain-in-go-part-6/)
7. [Network](https://jeiwan.cc/posts/building-blockchain-in-go-part-7/)

## Implementation

In our implementation, there will be centralization though. We’ll have three nodes:

* The central node. This is the node all other nodes will connect to, and this is the node that’ll sends data between other nodes.
* A miner node. This node will store new transactions in mempool and when there’re enough of transactions, it’ll mine a new block.
* A wallet node. This node will be used to send coins between wallets. Unlike SPV nodes though, it’ll store a full copy of blockchain.

## The Scenario

The goal of this project is to implement the following scenario:

1. The central node creates a blockchain.
2. Other (wallet) node connects to it and downloads the blockchain.
3. One more (miner) node connects to the central node and downloads the blockchain.
4. The wallet node creates a transaction.
5. The miner nodes receives the transaction and keeps it in its memory pool.
6. When there are enough transactions in the memory pool, the miner starts mining a new block.
7. When a new block is mined, it’s send to the central node.
8. The wallet node synchronizes with the central node.
9. User of the wallet node checks that their payment was successful.

This is what it looks like in Bitcoin. Even though we’re not going to build a real P2P network, we’re going to implement a real, and the main and most important, use case of Bitcoin.

## Usage

### create wallet address

* central node(NODE_ID=3000)

```
➜  tiny-bitcoin export NODE_ID=3000
➜  tiny-bitcoin ./tiny-bitcoin createwallet
Your new address: 1N73eRHN833VYqmhkjeT8Dk4ycL3ue1q5w //CENTREAL_NODE
```

* wallet node(NODE_ID=3001)

```
➜  tiny-bitcoin export NODE_ID=3001
➜  tiny-bitcoin ./tiny-bitcoin createwallet
Your new address: 1p916LpCJMm3mTSjvfwHeeyjxzLRVZPjV //WALLET_1
➜  tiny-bitcoin ./tiny-bitcoin createwallet
Your new address: 1H6R4pbAtesRpCAxyWe3qkcpZcGUkgq286 //WALLET_2
➜  tiny-bitcoin ./tiny-bitcoin createwallet
Your new address: 1gKUgagCP9r782EMe7T67bXBJZMJeg5VZ //WALLET_3

```

* miner node(NODE_ID=3002)

```
➜  tiny-bitcoin export NODE_ID=3002
➜  tiny-bitcoin ./tiny-bitcoin createwallet
Your new address: 13kw49bYiKYraAkXZWP7wsLPzDiDNjFbUr //MINER_NODE
```

### create new blockchain(in central node)


* initialize blockchain 

```
➜  tiny-bitcoin ./tiny-bitcoin createblockchain -address $CENTREAL_NODE
db5d09c20a10467f067047020dc05e222f042990905fdb756186d4cc28782467

Done!

//genesis block
➜  tiny-bitcoin cp blockchain_3000.db blockchain_genesis.db 
```

* send some coins to the wallet addresses

```
➜  tiny-bitcoin ./tiny-bitcoin send -from $CENTREAL_NODE -to $WALLET_1 -amount 10 -mine
6157177999c3cf744e2bcc90a0cdc1094d5fc7497222c895db85151e0d00ee31

Success!

./tiny-bitcoin send -from $CENTREAL_NODE -to $WALLET_2 -amount 10 -mine
4e70faa4eba1d23d28774ad9722e74a301cd8793eb11e0820b6d000db932b80f

Success!
```

-mine flag means that the block will be immediately mined by the same node. We have to have this flag because initially there are no miner nodes in the network.


* start central node and keep running until end

```
➜  tiny-bitcoin ./tiny-bitcoin startnode &                         
Starting node 3000
```

### blockchain synchronization(wallet node)


```
//set the node’s blockchain with the genesis block saved above:
➜  tiny-bitcoin cp blockchain_genesis.db blockchain_3001.db

//set envirionment variables
➜  tiny-bitcoin WALLET_1=1p916LpCJMm3mTSjvfwHeeyjxzLRVZPjV
➜  tiny-bitcoin WALLET_2=1H6R4pbAtesRpCAxyWe3qkcpZcGUkgq286
➜  tiny-bitcoin WALLET_3=1gKUgagCP9r782EMe7T67bXBJZMJeg5VZ
➜  tiny-bitcoin CENTREAL_NODE=1N73eRHN833VYqmhkjeT8Dk4ycL3ue1q5w

//since blockchain not synchonized,initial balance all be zero
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $CENTREAL_NODE
Balance of '1N73eRHN833VYqmhkjeT8Dk4ycL3ue1q5w': 10
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $WALLET_1
Balance of '1p916LpCJMm3mTSjvfwHeeyjxzLRVZPjV': 0
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $WALLET_2
Balance of '1H6R4pbAtesRpCAxyWe3qkcpZcGUkgq286': 0
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $WALLET_3
Balance of '1gKUgagCP9r782EMe7T67bXBJZMJeg5VZ': 0

//start node and check blockchain syncing process,will download all the blocks from the central node
➜  tiny-bitcoin ./tiny-bitcoin startnode                  
[1] 23023
Starting node 3001                                                              
Received version command
Received inv command
Recevied inventory with 3 block
Received block command
Recevied a new block!
Added block 0000be6f5003750ae08189aa92c2727dff9af01d36c62b7df796adbd98f57aa1
Received block command
Recevied a new block!
Added block 0000c5bd000b170122ff5184754ba77fd1b98097e31baff4e51a79309896f506
Received block command
Recevied a new block!
Added block 00002084e343e49ca70ffdfde41b4d714650f5ff3af31cf0a15bbcc892ff8d2a

//stop node and check balance again,already synced
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $WALLET_1
Balance of '1p916LpCJMm3mTSjvfwHeeyjxzLRVZPjV': 10
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $WALLET_2
Balance of '1H6R4pbAtesRpCAxyWe3qkcpZcGUkgq286': 10

```

### mine block

* initialize miner node and start

```
// Initialize the blockchain with genesis block
cp blockchain_genesis.db blockchain_3002.db

// set envirionment variables
➜  tiny-bitcoin MINER_WALLET=13kw49bYiKYraAkXZWP7wsLPzDiDNjFbUr

// start the node with miner flag
➜  tiny-bitcoin ./tiny-bitcoin startnode -miner $MINER_WALLET 
Starting node 3002
Mining is on. Address to receive rewards:  13kw49bYiKYraAkXZWP7wsLPzDiDNjFbUr
Received version command
Received inv command
Recevied inventory with 3 block
Received block command
Recevied a new block!
Added block 0000be6f5003750ae08189aa92c2727dff9af01d36c62b7df796adbd98f57aa1
Received block command
Recevied a new block!
Added block 0000c5bd000b170122ff5184754ba77fd1b98097e31baff4e51a79309896f506
Received block command
Recevied a new block!
Added block 00002084e343e49ca70ffdfde41b4d714650f5ff3af31cf0a15bbcc892ff8d2a
```

* switch to wallet node and send some coins

```
➜  tiny-bitcoin ./tiny-bitcoin send -from $WALLET_1 -to $WALLET_3 -amount 1
Success!
➜  tiny-bitcoin ./tiny-bitcoin send -from $WALLET_2 -to $WALLET_3 -amount 1
Success!
```

* switch to the miner node and see it mining a new block

```
// miner node
Received inv command
Recevied inventory with 1 tx
Received inv command
Recevied inventory with 1 tx
320c707ba2d48b3c7e143b8ce5cd4f239952e0a2ccc9bddc6ad0e2ab67c13284
Received tx command
New block is mined!
Received getdata command
```
* switch to the central node and check the output

```
// central node
Received tx command
Received getdata command
Received inv command
Recevied inventory with 1 block
Received block command
Recevied a new block!
Added block 000023c193e46b5d196dad0d7d35ba54669251a3042300446203cbb8e0accc2a
```

* switch to the wallet node and start it again,check the synchronization process and balance

```
Starting node 3001
Received version command
Received inv command
Recevied inventory with 4 block
Received block command
Recevied a new block!
Added block 000023c193e46b5d196dad0d7d35ba54669251a3042300446203cbb8e0accc2a
Received block command
Recevied a new block!
Added block 0000be6f5003750ae08189aa92c2727dff9af01d36c62b7df796adbd98f57aa1
Received block command
Recevied a new block!
Added block 0000c5bd000b170122ff5184754ba77fd1b98097e31baff4e51a79309896f506
Received block command
Recevied a new block!
Added block 00002084e343e49ca70ffdfde41b4d714650f5ff3af31cf0a15bbcc892ff8d2a
^C
//check balance again
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $WALLET_1
Balance of '1p916LpCJMm3mTSjvfwHeeyjxzLRVZPjV': 9
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $WALLET_2
Balance of '1H6R4pbAtesRpCAxyWe3qkcpZcGUkgq286': 9
➜  tiny-bitcoin ./tiny-bitcoin getbalance -address $WALLET_3
Balance of '1gKUgagCP9r782EMe7T67bXBJZMJeg5VZ': 2
```


