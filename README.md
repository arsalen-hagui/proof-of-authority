# Setup your own private Proof-of-Authority Ethereum network with Geth

**Goal:** Step by step guide to help you setup a local private ethereum network using the Proof-of-Authority consensus engine (also named clique).
**In a nutshell:** We will setup two nodes on the same machine, creating a peer-to-peer network on our localhost. In addition to the two nodes, a bootnode (discovery service) will also be setup.
![](header.png)

## Prerequisites

Before you can build this project, please install [Geth](https://geth.ethereum.org/) on a [Ubuntu 18.04 LTS](https://releases.ubuntu.com/18.04.5/) (this tuto was done in a fresh virtual machine)
```shell
~$ sudo add-apt-repository -y ppa:ethereum/ethereum
~$ sudo apt-get update
~$ sudo apt-get install ethereum
~$ geth version
Geth
Version: 1.9.21-stable
Git Commit: 0287d54847d3297f3ced62cd83a4c95ccbe0045b
Architecture: amd64
Protocol Versions: [65 64 63]
Go Version: go1.15
Operating System: linux
GOPATH=
GOROOT=go
```

## Setup a workspace

First create the folder for the network configurations, then two folders for each node.
```shell
~$ mkdir testnet
~$ cd testnet
~/testnet$ mkdir node1 node2
```

## Create accounts

Now as the workspace is ready, we will create two accounts (also known as wallets) that hold a private-public key pair that are required for interacting with any blockchain.

```shell
~/testnet$ geth --datadir node1/ account new
INFO [09-23|15:53:10.141] Maximum peer count                       ETH=50 LES=0 total=50
INFO [09-23|15:53:10.143] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
Your new account is locked with a password. Please give a password. Do not forget this password.
Password:
Repeat password:
Your new key was generated
Public address of the key:   0x3df228efF9882F7c58e6a9394914F51C3496cD8C
Path of the secret key file: node1/keystore/UTC--2020-09-23T15-53-31.215856013Z--3df228efF9882F7c58e6a9394914F51C3496cD8C
- You can share your public address with anyone. Others need it to interact with you.
- You must NEVER share the secret key with anyone! The key controls access to your funds!
- You must BACKUP your key file! Without the key, it's impossible to access account funds!
- You must REMEMBER your password! Without the password, it's impossible to decrypt the key!

~/testnet$ geth --datadir node2/ account new
INFO [09-23|15:53:40.341] Maximum peer count                       ETH=50 LES=0 total=50
INFO [09-23|15:53:40.343] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
Your new account is locked with a password. Please give a password. Do not forget this password.
Password:
Repeat password:
Your new key was generated
Public address of the key:   0x665C5b7e68B842c3DaB1739299946C113F217e87
Path of the secret key file: node2/keystore/UTC--2020-09-23T15-53-44.666476332Z--665C5b7e68B842c3DaB1739299946C113F217e87
- You can share your public address with anyone. Others need it to interact with you.
- You must NEVER share the secret key with anyone! The key controls access to your funds!
- You must BACKUP your key file! Without the key, it's impossible to access account funds!
- You must REMEMBER your password! Without the password, it's impossible to decrypt the key!
```

This creates the `keystore/` folder containing your account file, we will copy these two addresses from the terminal screen and save them in a text file. That will ease some copy-pasting job later on.

```shell
~/testnet$ echo '3df228efF9882F7c58e6a9394914F51C3496cD8C' >> accounts.txt
~/testnet$ echo '665C5b7e68B842c3DaB1739299946C113F217e87' >> accounts.txt
```

Same for passwords, this time we will save each in a file inside the respective node directory to ease some process for later on (such as unlocking your account)

```shell
~/testnet$ echo 'pwd1' > node1/password.txt
~/testnet$ echo 'pwd2' > node2/password.txt
```

## Configure Genesis block

A genesis file is the file used to initialize the blockchain. The very first block, called the genesis block, is crafted based on the parameters in the genesis.json file.

Geth comes with a bunch of exectuables such as Puppeth which removes the pain of creating a genesis file from scratch (and does much more).

```shell
~/testnet$ puppeth
+-----------------------------------------------------------+
| Welcome to puppeth, your Ethereum private network manager |
|                                                           |
| This tool lets you create a new Ethereum network down to  |
| the genesis block, bootnodes, miners and ethstats servers |
| without the hassle that it would normally entail.         |
|                                                           |
| Puppeth uses SSH to dial in to remote servers, and builds |
| its network components out of Docker containers using the |
| docker-compose toolset.                                   |
+-----------------------------------------------------------+
Please specify a network name to administer (no spaces, hyphens or capital letters please)
> testnet
Sweet, you can set this via --network=testnet next time!
INFO [09-23|15:59:34.248] Administering Ethereum network           name=testnet
INFO [09-23|15:59:34.248] No remote machines to gather stats from
What would you like to do? (default = stats)
 1. Show network stats
 2. Configure new genesis
 3. Track new remote server
 4. Deploy network components
>2
What would you like to do? (default = create)
 1. Create new genesis from scratch
 2. Import already existing genesis
> 1
Which consensus engine to use? (default = clique)
 1. Ethash - proof-of-work
 2. Clique - proof-of-authority
> 2
How many seconds should blocks take? (default = 15)
>
Which accounts are allowed to seal? (mandatory at least one)
> 0x3df228efF9882F7c58e6a9394914F51C3496cD8C
> 0x665C5b7e68B842c3DaB1739299946C113F217e87
> 0x
Which accounts should be pre-funded? (advisable at least one)
> 0x3df228efF9882F7c58e6a9394914F51C3496cD8C
> 0x665C5b7e68B842c3DaB1739299946C113F217e87
> 0x
Should the precompile-addresses (0x1 .. 0xff) be pre-funded with 1 wei? (advisable yes)
> yes
Specify your chain/network ID if you want an explicit one (default = random)
> 1234
INFO [09-23|16:04:24.858] Configured new genesis block
What would you like to do? (default = stats)
 1. Show network stats
 2. Manage existing genesis
 3. Track new remote server
 4. Deploy network components
> 2
 1. Modify existing configurations
 2. Export genesis configurations
 3. Remove genesis configuration
> 2
Which folder to save the genesis specs into? (default = current)
  Will create testnet.json, testnet-aleth.json, testnet-harmony.json, testnet-parity.json
>
INFO [09-23|16:04:42.305] Saved native genesis chain spec          path=testnet.json
ERROR[09-23|16:04:42.306] Failed to create Aleth chain spec        err="unsupported consensus engine"
ERROR[09-23|16:04:42.306] Failed to create Parity chain spec       err="unsupported consensus engine"
INFO [09-23|16:04:42.308] Saved genesis chain spec                 client=harmony path=testnet-harmony.json
What would you like to do? (default = stats)
 1. Show network stats
 2. Manage existing genesis
 3. Track new remote server
 4. Deploy network components
> ^C
```

## Initialize nodes

Now that we have the genesis.json file, let’s forge the genesis block ! Each node MUST be initialize with the SAME genesis file.

```shell
~/testnet$ geth --datadir node1/ init testnet.json
~/testnet$ geth --datadir node2/ init testnet.json
```

## Start nodes

### Create a bootnode

A bootnode only purpose is to helping nodes discovering each others (remember, the Ethereum blockchain is a peer-to-peer network). Nodes could have dynamic IP, being turned off, and on again. The bootnode is usually ran on a static IP and thus acts like a pub where nodes know they will find their mates.

```shell
~/testnet$ bootnode -genkey boot.key

~/testnet$ bootnode -nodekey boot.key -verbosity 9 -addr :30310
enode://40f023cfab618e8d2229bc31b05db7c7df003c451bad2898d728073aa113fde527b6b5b278637127da0dcd8d415a91ae0c829f10689bb75f72f89c46e96a5625@127.0.0.1:0?discport=30310
Note: you're using cmd/bootnode, a developer tool.
We recommend using a regular node as bootstrap node for production deployments.
INFO [09-23|16:25:09.248] New local node record                    seq=1 id=5be54f9236b5e715 ip=<nil> udp=0 tcp=0
```

### Synch nodes

Finally, launch each node separately and watch them mining and signing blocks.

```shell
~/testnet$ geth --datadir node1/ --syncmode 'full' --port 30311 --rpc --rpcaddr 'localhost' --rpcport 8501 --rpcapi 'personal,db,eth,net,web3,txpool,miner' --bootnodes 'enode://40f023cfab618e8d2229bc31b05db7c7df003c451bad2898d728073aa113fde527b6b5b278637127da0dcd8d415a91ae0c829f10689bb75f72f89c46e96a5625@127.0.0.1:30310' --networkid 1234 --gasprice '1' -unlock '0x3df228efF9882F7c58e6a9394914F51C3496cD8C' --password node1/password.txt --mine --allow-insecure-unlock
```

```shell
~/testnet$ geth --datadir node2/ --syncmode 'full' --port 30312 --rpc --rpcaddr 'localhost' --rpcport 8502 --rpcapi 'personal,db,eth,net,web3,txpool,miner' --bootnodes 'enode://40f023cfab618e8d2229bc31b05db7c7df003c451bad2898d728073aa113fde527b6b5b278637127da0dcd8d415a91ae0c829f10689bb75f72f89c46e96a5625@127.0.0.1:30310' --networkid 1234 --gasprice '1' -unlock '0x665C5b7e68B842c3DaB1739299946C113F217e87' --password node2/password.txt --mine --allow-insecure-unlock
```

### Submit a transaction

We want to send ether from one account to another. First we have to connect to one node's console to interact with its APIs.

```shell
~/testnet$ geth attach node1/geth.ipc 
Welcome to the Geth JavaScript console!

instance: Geth/v1.9.12-stable-b6f1c8dc/linux-amd64/go1.13.8
coinbase: 0x3df228eff9882f7c58e6a9394914f51c3496cd8c
at block: 145 (Wed Sep 23 2020 23:52:19 GMT+0100 (CET))
 datadir: /home/arsalen/Desktop/draft/proof-of-authority/testnet/node1
 modules: admin:1.0 clique:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 txpool:1.0 web3:1.0
```

Now we send ether to a random address through eth API, the result is a hash we will use to check the state of the transaction later.

```shell
> eth.sendTransaction({'from':eth.coinbase, 'to':'0x08a58f09194e403d02a1928a7bf78646cfc260b0', 'value':web3.toWei(1, 'ether')})
"0xebd828c847773655ff39ec570c9bedc8622788844c351e1669701130d2bae72f"
> eth.getTransactionReceipt("0xebd828c847773655ff39ec570c9bedc8622788844c351e1669701130d2bae72f")
{
  blockHash: "0xa803a5c199ae46a134f0626fea2fe0ac70de7da16a3c6f8c1e7dc9318f091ff8",
  blockNumber: 170,
  contractAddress: null,
  cumulativeGasUsed: 21000,
  from: "0x3df228eff9882f7c58e6a9394914f51c3496cd8c",
  gasUsed: 21000,
  logs: [],
  logsBloom: "0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
  status: "0x1",
  to: "0x08a58f09194e403d02a1928a7bf78646cfc260b0",
  transactionHash: "0xebd828c847773655ff39ec570c9bedc8622788844c351e1669701130d2bae72f",
  transactionIndex: 0
}
> exit

```

`status: 0x1` means the transaction was successfully mined.

**Disclaimer:** Addresses and keys used in this tutorial are only for development purposes.