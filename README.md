
## Contents

- [Build HSC Node](#build-hsc-node)
  - [Install Golang](#install-golang)
  - [Get the HSC  source code](#get-the-hsc--source-code)
- [System configuration](#system-configuration)
  - [Hardware requirements](#hardware-requirements)
  - [Prerequisites](#prerequisites)
  - [Commonly used ports](#commonly-used-ports)
  - [chain node](#chain-node)
  - [start bash](#start-bash)
  - [systemd config](#systemd-config)
- [Genesis file](#genesis-file)
  - [Glossary](#glossary)
  - [mainnet](#mainnet)
  - [testnet](#testnet)
- [Mainnet Info](#mainnet-info)
  - [chainid](#chainid)
  - [rpc](#rpc)
  - [explorer](#explorer)
- [P2P Nodes](#p2p-nodes)
- [Register your  validator](#register-your--validator)
  - [Prerequisites](#prerequisites-1)
  - [1. Import your PrivateKey](#1-import-your-privatekey)
  - [1. Contract Abi](#1-contract-abi)
  - [2. Create a new validator](#2-create-a-new-validator)
  - [Create a new proposal](#create-a-new-proposal)
  - [Vote on a proposal](#vote-on-a-proposal)
  - [Stake  HOO  to the validator](#stake--hoo--to-the-validator)
  - [UnStake HOO](#unstake-hoo)
  - [WithdrawStaking](#withdrawstaking)
  - [Others](#others)
  
# Build HSC Node

HSC  is the official Golang reference implementation of the HSC node software. Use this guide to install HSC  and `geth`, the command-line interface and daemon that connects to HSC and enables you to interact with the HSC blockchain.  

## Install Golang
Reference: [Go Download and install](https://golang.org/doc/install)


## Get the HSC  source code

1. Use `git` to retrieve [HSC ](https://github.com/HSC-money//), and check out the `main` branch, which contains the latest stable release.

    - If you are using Localhsc or running a validator.
    - You can find out the latest tag on the [tags page](https://github.com/hoosmartchain/hoo-smartchain/tags) or via autocomplete in your terminal: type `git checkout v` and press `<TAB>`.

    ```bash
    git clone https://github.com/hoosmartchain/hoo-smartchain
    cd hoo-smartchain
    git checkout [latest version]
    ```


2. Build HSC . This will install the `geth` executable to your [ `GOPATH` ](https://go.dev/doc/gopath_code) environment variable.

   ```bash
   make geth
   ```

> If you want to use cross compile, like compiling on `Mac` for `Linux`, use `make geth-linux`, `make geth-linux-amd64`, etc.


After compilation completed, the generated binary is in the folder `build/bin`.

3. Verify that geth  is installed correctly.

   ```bash
   geth version 
   ```


If the `geth: command not found` error message is returned, confirm that the Go binary path is correctly configured by running the following command:

```bash
export PATH=$PATH:$(go env GOPATH)/bin
```

# System configuration


This guide has been tested against Linux distributions only. To ensure a successful production environment setup, consider using a Linux system.

Running a full HSC node is a resource-intensive process that requires a persistent server. If you want to use HSC without downloading the entire blockchain.

##  Hardware requirements

- Eight or more CPU cores
- At least 1 TB of disk storage
- At least 16 GB of memory
- At least 200 mbps of network bandwidth


## Prerequisites

- [Golang v1.16.1 - go1.17.1 linux/amd64](https://go.dev/dl/)

  ::: {dropdown} Installing Go for MacOS & Linux

  Go releases can be found here: [ https://go.dev/dl/ ](https://go.dev/dl/)

  In your browser, you can right-click the correct release (v1.16.1 - go1.17.1) and `Copy link`.

  ```bash
  # 1. Download the archive

  wget https://go.dev/dl/go1.17.1.linux-amd64.tar.gz

  # Optional: remove previous /go files:

  sudo rm -rf /usr/local/go

  # 2. Unpack:

  sudo tar -C /usr/local -xzf go1.17.1.linux-amd64.tar.gz

  # 3. Add the path to the go-binary to your system path:
  # (for this to persist, add this line to your ~/.profile or ~/.bashrc or  ~/.zshrc)

  export PATH=$PATH:/usr/local/go/bin

  # 4. Verify your installation:

  go version

  # go version go1.17.1 linux/amd64

  ```


- Linux users:  `sudo apt-get install -y build-essential`

## Commonly used ports

`geth` uses the following TCP ports. Toggle their settings to fit your environment.

Most validators will only need to open the following port:

- `32668`: The default port for the P2P protocol. This port is used to communicate with other nodes and must be open to join a network. However, it does not have to be open to the public. For validator nodes, we recommend [configuring `persistent_peers`](updates-and-additional.md#additional-settings) and closing this port to the public.

Additional ports:

- `8545`: The default port for the RPC protocol. Because this port is used for querying and sending transactions, it must be open for serving queries from `geth`.

::: {warning}
Do not open port `32668` to the public unless you plan to run a public node.
:::

## chain node

* config.toml
  

```
[Eth]
SyncMode = "fast"
DiscoveryURLs = []
TrieCleanCacheRejournal= 300000000000

[Eth.Miner]
GasFloor = 8000000
GasCeil = 42000000
GasPrice = 0
Recommit = 3000000000
Noverify = false

[Eth.Ethash]
CacheDir = "ethash"
CachesInMem = 2
CachesOnDisk = 3
CachesLockMmap = false 
DatasetDir = "data/.ethash"
DatasetsInMem = 1
DatasetsOnDisk = 2
DatasetsLockMmap = false
PowMode = 0

[Eth.TxPool]
Locals = []
NoLocals = false
Journal = "transactions.rlp"
Rejournal = 3600000000000
PriceLimit = 1
PriceBump = 10
AccountSlots = 16
GlobalSlots = 4096
AccountQueue = 64
GlobalQueue = 1024
Lifetime = 10800000000000

[Node]
DataDir = "data"
InsecureUnlockAllowed = true
NoUSB = true
IPCPath = "geth.ipc"
HTTPHost = "0.0.0.0"
HTTPPort = 8545
HTTPCors = ["*"]
HTTPVirtualHosts = ["*"]
HTTPModules = ['eth', 'net', 'web3','debug']

WSHost = "0.0.0.0"
WSPort = 8546
WSModules = ['eth', 'net', 'web3','debug']

GraphQLVirtualHosts = ["localhost"]


[Node.P2P]
MaxPeers = 50
NoDiscovery = false
BootstrapNodes = ["enode://a979fb575495b8d6db44f750317d0f4622bf4c2aa3365d6af7c284339968eef29b69ad0dce72a4d8db5ebb4968de0e3bec910127f134779fbcb0cb6d3331163c@52.16.188.185:30303", "enode://3f1d12044546b76342d59d4a05532c14b85aa669704bfe1f864fe079415aa2c02d743e03218e57a33fb94523adb54032871a6c51b2cc5514cb7c7e35b3ed0a99@13.93.211.84:30303", "enode://78de8a0916848093c73790ead81d1928bec737d565119932b98c6b100d944b7a95e94f847f689fc723399d2e31129d182f7ef3863f2b4c820abbf3ab2722344d@191.235.84.50:30303", "enode://158f8aab45f6d19c6cbf4a089c2670541a8da11978a2f90dbf6a502a4a3bab80d288afdbeb7ec0ef6d92de563767f3b1ea9e8e334ca711e9f8e2df5a0385e8e6@13.75.154.138:30303", "enode://1118980bf48b0a3640bdba04e0fe78b1add18e1cd99bf22d53daac1fd9972ad650df52176e7c7d89d1114cfef2bc23a2959aa54998a46afcf7d91809f0855082@52.74.57.123:30303", "enode://979b7fa28feeb35a4741660a16076f1943202cb72b6af70d327f053e248bab9ba81760f39d0701ef1d8f89cc1fbd2cacba0710a12cd5314d5e0c9021aa3637f9@5.1.83.226:30303"]
BootstrapNodesV5 = ["enode://0cc5f5ffb5d9098c8b8c62325f3797f56509bff942704687b6530992ac706e2cb946b90a34f1f19548cd3c7baccbcaea354531e5983c7d1bc0dee16ce4b6440b@40.118.3.223:30305", "enode://1c7a64d76c0334b0418c004af2f67c50e36a3be60b5e4790bdac0439d21603469a85fad36f2473c9a80eb043ae60936df905fa28f1ff614c3e5dc34f15dcd2dc@40.118.3.223:30308", "enode://85c85d7143ae8bb96924f2b54f1b3e70d8c4d367af305325d30a61385a432f247d2c75c45c6b4a60335060d072d7f5b35dd1d4c45f76941f62a4f83b6e75daaf@40.118.3.223:30309"]
StaticNodes = ["enode://eb565f4c270e45ee13aaf4e27e452ee11591c2dc4ab795a8987f705a50fd3604f751f7c0d8b669287b2ba1b3713effea13bb0c25d245066b1a703bd0c477f86a@35.73.253.58:32668","enode://98eda17a83decd4a9b606b926f12b4c965b7b6b851a277e895861338eb7eebae8d4b48523eee33fafee5ad9a5f8d3fb14e56ad75ea70e80de815dc98962d9904@52.192.142.14:32668","enode://d7ee8747aa8a0e4ab61ef27e18045a36014f1458a018394f587411d4959c9cf41742bd1402f693e9eb8a76b9fdc4e4d12b5a49c0000e7b08092e7f357b72a4c1@3.115.0.208:32668","enode://6557e8f7b7fffd4b819834130df7367109bec7b9bf0acac699f1aaa51e0fa23a15da3b83d147159113e24b50ad716253cafae088e3b266d63a706413d4a1ef61@54.250.51.3:32668","enode://4e5aa53f4fe03045c9a7eeeec9f668102e1254ba4764d03422edf534df5c4554c9bebb73a68ca481324bf19840de3e213deb47c19cff7d281267eb7ea8f5d5c8@13.230.35.234:32668","enode://b3fe1dc1c5ac04e6b6024de61939180ee0a3a1d54e329bb3366c8e8f3836e1334ee2488267fe73d6473bf841dca8efb15f43f6931e6822becb48d1f0bf401d48@35.73.127.28:32668","enode://9140ba058ab08955a2c10b0403075fd883b631314361cfa072c0d181983fcec05f849f45c6310f05d0b98354cf4e6bbaad17d5203774929a49da5e1883160f8d@18.181.232.158:32668","enode://efac0b0376b37ed78ffbeb9737d99331331890333b9a3884307e28c6a6a76c85b8afe43d74052a8ad0286346b8946c59b2f7a2a1aee03350995c5ff23e9797e1@35.73.84.83:32668","enode://b4f9211b5fe5964339af1a7f9f0009b4850bf8cf04e00c49bc0812b4022aae9a48d9fb68717d46c0f57863baf3e0c4a030c35adf9a12126e323b432becdf68e0@35.73.84.83:33668","enode://758dc22abcf9d8f7bfd71d432fb7d9ae6225f44e2917871873ea62c7dc4c8751de3954b2bb9523276c846ff916320cecf9d76d5c5069b507062b2bdd10789351@35.75.78.27:32668","enode://82c00f63300b3b0cfe15d21866e1428e832f2ec068e2c970a8ec13c11b47a9604c0bc863739a5ce36b3f64ad81922ccec6cf025249af02d3c259afa4275248df@35.75.78.27:33668","enode://81cfe68f744a6ed4b6668adf2571519dc23c473d583892804c2f351fc1d8e19debbf7a9268caf136710896d5a0fbbadcc8c37b5690e416dad6a92057d8113618@54.95.0.64:32668","enode://1cf8572e5f29ffcbd4e08e9b118d8368598b7a9cc66c82d7071fbead509df0a4e7d8028431b007bcd86ce9a7525968d2e113ded6a85f03e64afa70726c38fa20@54.95.0.64:33668","enode://c9f110cd08ae6535ad7d7450195d81ee71b2f4c1d70fb77d6cfea418be2128d34e2f5b886d54a53ff3e07501567334fa5fb517c404dc293ba2da6e76c4eea177@35.74.60.20:32668","enode://52f628819fbbc8ad75fc2c84deadf2398ed3e21f78d29e198cc335213c51e156d296d9ac045f8589db73fcfa23475e725065b2c4800fcb96c1d2263f0ceba618@35.74.60.20:33668","enode://744af4828b01d46ae85c03bbf369d0225d6a6a2bbefde2c5cceb14326bbf85f9ee07fa537e04160551697fbfaa7480885d3dbfd9286dc2876e8df5eeb93fafab@35.75.78.132:32668","enode://51a130bcab6f1d539a7f60dd3a237f5c53c27578c206d477c2fc94e8ef1e32bfcd201a59fbfde8fd37fbb3bbe0af54198e6d20d5fffc5191116ec68a76cde285@35.75.78.132:33668","enode://55b8f0964b6a657e547a024a70b0b244dd7030b56e786e02fa25f8cce3db3a7ecfd21058357e3a55799fd3c9de0b7a5ae53c6e12c1600a3a980891d7247c39da@175.41.235.206:32668","enode://cfca78d9918ff91523999fe0e6d4db9d3b6cc6a3e21aace25c578d6d5a01197a4ce0b6f623c2528c376e3f2f658c85c3376654a229c2e91756fc744d4bd5bc46@175.41.235.206:33668","enode://8a8474343f9919a84351b1c4a02e7a60294dc1e95ae213e46ec28d7ce1e7ee69d385dc856c81419bb69207400ff9c642788510cfd8b428cf8b6cd11ea530d4d5@35.75.78.109:32668","enode://c91ad419eb7048d46ff7d94fa1fa88461006be5e497ee4f5c4201d339c760b2944f242c3b00b322186cabd1fff6a89974c6e4b2715dffa77eea93df43be0c0a6@35.75.78.109:33668"]
TrustedNodes = ["enode://eb565f4c270e45ee13aaf4e27e452ee11591c2dc4ab795a8987f705a50fd3604f751f7c0d8b669287b2ba1b3713effea13bb0c25d245066b1a703bd0c477f86a@35.73.253.58:32668","enode://98eda17a83decd4a9b606b926f12b4c965b7b6b851a277e895861338eb7eebae8d4b48523eee33fafee5ad9a5f8d3fb14e56ad75ea70e80de815dc98962d9904@52.192.142.14:32668","enode://d7ee8747aa8a0e4ab61ef27e18045a36014f1458a018394f587411d4959c9cf41742bd1402f693e9eb8a76b9fdc4e4d12b5a49c0000e7b08092e7f357b72a4c1@3.115.0.208:32668","enode://6557e8f7b7fffd4b819834130df7367109bec7b9bf0acac699f1aaa51e0fa23a15da3b83d147159113e24b50ad716253cafae088e3b266d63a706413d4a1ef61@54.250.51.3:32668","enode://4e5aa53f4fe03045c9a7eeeec9f668102e1254ba4764d03422edf534df5c4554c9bebb73a68ca481324bf19840de3e213deb47c19cff7d281267eb7ea8f5d5c8@13.230.35.234:32668","enode://b3fe1dc1c5ac04e6b6024de61939180ee0a3a1d54e329bb3366c8e8f3836e1334ee2488267fe73d6473bf841dca8efb15f43f6931e6822becb48d1f0bf401d48@35.73.127.28:32668","enode://9140ba058ab08955a2c10b0403075fd883b631314361cfa072c0d181983fcec05f849f45c6310f05d0b98354cf4e6bbaad17d5203774929a49da5e1883160f8d@18.181.232.158:32668","enode://efac0b0376b37ed78ffbeb9737d99331331890333b9a3884307e28c6a6a76c85b8afe43d74052a8ad0286346b8946c59b2f7a2a1aee03350995c5ff23e9797e1@35.73.84.83:32668","enode://b4f9211b5fe5964339af1a7f9f0009b4850bf8cf04e00c49bc0812b4022aae9a48d9fb68717d46c0f57863baf3e0c4a030c35adf9a12126e323b432becdf68e0@35.73.84.83:33668","enode://758dc22abcf9d8f7bfd71d432fb7d9ae6225f44e2917871873ea62c7dc4c8751de3954b2bb9523276c846ff916320cecf9d76d5c5069b507062b2bdd10789351@35.75.78.27:32668","enode://82c00f63300b3b0cfe15d21866e1428e832f2ec068e2c970a8ec13c11b47a9604c0bc863739a5ce36b3f64ad81922ccec6cf025249af02d3c259afa4275248df@35.75.78.27:33668","enode://81cfe68f744a6ed4b6668adf2571519dc23c473d583892804c2f351fc1d8e19debbf7a9268caf136710896d5a0fbbadcc8c37b5690e416dad6a92057d8113618@54.95.0.64:32668","enode://1cf8572e5f29ffcbd4e08e9b118d8368598b7a9cc66c82d7071fbead509df0a4e7d8028431b007bcd86ce9a7525968d2e113ded6a85f03e64afa70726c38fa20@54.95.0.64:33668","enode://c9f110cd08ae6535ad7d7450195d81ee71b2f4c1d70fb77d6cfea418be2128d34e2f5b886d54a53ff3e07501567334fa5fb517c404dc293ba2da6e76c4eea177@35.74.60.20:32668","enode://52f628819fbbc8ad75fc2c84deadf2398ed3e21f78d29e198cc335213c51e156d296d9ac045f8589db73fcfa23475e725065b2c4800fcb96c1d2263f0ceba618@35.74.60.20:33668","enode://744af4828b01d46ae85c03bbf369d0225d6a6a2bbefde2c5cceb14326bbf85f9ee07fa537e04160551697fbfaa7480885d3dbfd9286dc2876e8df5eeb93fafab@35.75.78.132:32668","enode://51a130bcab6f1d539a7f60dd3a237f5c53c27578c206d477c2fc94e8ef1e32bfcd201a59fbfde8fd37fbb3bbe0af54198e6d20d5fffc5191116ec68a76cde285@35.75.78.132:33668","enode://55b8f0964b6a657e547a024a70b0b244dd7030b56e786e02fa25f8cce3db3a7ecfd21058357e3a55799fd3c9de0b7a5ae53c6e12c1600a3a980891d7247c39da@175.41.235.206:32668","enode://cfca78d9918ff91523999fe0e6d4db9d3b6cc6a3e21aace25c578d6d5a01197a4ce0b6f623c2528c376e3f2f658c85c3376654a229c2e91756fc744d4bd5bc46@175.41.235.206:33668","enode://8a8474343f9919a84351b1c4a02e7a60294dc1e95ae213e46ec28d7ce1e7ee69d385dc856c81419bb69207400ff9c642788510cfd8b428cf8b6cd11ea530d4d5@35.75.78.109:32668","enode://c91ad419eb7048d46ff7d94fa1fa88461006be5e497ee4f5c4201d339c760b2944f242c3b00b322186cabd1fff6a89974c6e4b2715dffa77eea93df43be0c0a6@35.75.78.109:33668"]
ListenAddr = ":32668"
EnableMsgEvents = false

[Node.HTTPTimeouts]
ReadTimeout = 30000000000
WriteTimeout = 30000000000
IdleTimeout = 120000000000


```

## start bash


* run.sh


```
#!/usr/bin/env bash
/data/hsc/geth-linux-amd64 \
--config /data/hsc/config.toml  \
--logpath /data/hsc/logs \
--verbosity 3  >> /data/hsc/logs/systemd_chain_console.out 2>&1
```

if you need to use it as archive node, add：

```
--syncmode full \
--gcmode archive \
```

so：

```
#!/usr/bin/env bash
/data/hsc/geth-linux-amd64 \
--config /data/hsc/config.toml  \
--logpath /data/hsc/logs \
--syncmode full \
--gcmode archive \
--verbosity 3  >> /data/hsc/logs/systemd_chain_console.out 2>&1
```

## systemd config

```
[Unit]
Description=Hoo smart chain service

[Service]
Type=simple
ExecStart=/bin/sh /data/hsc/run.sh

Restart=on-failure
RestartSec=5s

LimitNOFILE=65536

[Install]

```


# Genesis file
Both the mainnet and testnet genesis information of `HSC` chain have been hardcoded in blockchain, and the corresponding genesis files are listed below for verification.

## Glossary 
- chainId The unique identification of the chain.
- `homesteadBlock` `eip150Block` `eip150Hash` `eip155Block` `eip158Block` `byzantiumBlock` `constantinopleBlock` `petersburgBlock` `istanbulBlock` `muirGlacierBlock` Hard fork height configuration.
- `congress` Consensus parameters `period` is time interval of blocks. `epoch` is set for a period in `block`, and at the end of each `epoch`, the validators are adjusted accordingly.
- `number` `gasUsed` `parentHash` `nonce` `timestamp` `extraData` `gasLimit` `difficulty` are all parameters for genesis block.
- `extraData` The initial validators is set up here.
- `alloc` Configured initial account information that can be used for asset pre-allocation and pre-initialization of system contracts.
    - 9dC4c187CA9AaF251D6CD2dC388CAbC16556597f //Genesis account for HOO
    - 000000000000000000000000000000000000F000 //validators contract address 
    - 000000000000000000000000000000000000F001 // punish contract address
    - 000000000000000000000000000000000000F002 // proposal contract address

  System contract repo: [Hoo-eco-contracts](https://github.com/HooGroup/Hoo-eco-contracts)

## mainnet
[mainnet](https://github.com/hoosmartchain/hoo-smartchain/blob/dev/mainnet-genesis.json))
## testnet
[testnet](https://github.com/hoosmartchain/hoo-smartchain/blob/dev/testnet-genesis.json)

# Mainnet Info

## chainid
```
70
```
## rpc

International visit:
```
https://http-mainnet.hoosmartchain.com
wss://ws-mainnet.hoosmartchain.com

archive mode
https://http-mainnet2.hoosmartchain.com
wss://ws-mainnet2.hoosmartchain.com


```


## explorer
```
https://hooscan.com
```

# P2P Nodes

allow P2P port（default 32668） udp/tcp

> the following nodes are default config for bootstrap node in code https://github.com/hoosmartchain/hoo-smartchain/blob/dev/params/bootnodes.go

```
	"enode://eb565f4c270e45ee13aaf4e27e452ee11591c2dc4ab795a8987f705a50fd3604f751f7c0d8b669287b2ba1b3713effea13bb0c25d245066b1a703bd0c477f86a@35.73.253.58:32668",
	"enode://98eda17a83decd4a9b606b926f12b4c965b7b6b851a277e895861338eb7eebae8d4b48523eee33fafee5ad9a5f8d3fb14e56ad75ea70e80de815dc98962d9904@52.192.142.14:32668",
	"enode://d7ee8747aa8a0e4ab61ef27e18045a36014f1458a018394f587411d4959c9cf41742bd1402f693e9eb8a76b9fdc4e4d12b5a49c0000e7b08092e7f357b72a4c1@3.115.0.208:32668"

```


put below into static node：

```
[Node.P2P]

StaticNodes = [
"enode://eb565f4c270e45ee13aaf4e27e452ee11591c2dc4ab795a8987f705a50fd3604f751f7c0d8b669287b2ba1b3713effea13bb0c25d245066b1a703bd0c477f86a@35.73.253.58:32668",
	"enode://98eda17a83decd4a9b606b926f12b4c965b7b6b851a277e895861338eb7eebae8d4b48523eee33fafee5ad9a5f8d3fb14e56ad75ea70e80de815dc98962d9904@52.192.142.14:32668",
	"enode://d7ee8747aa8a0e4ab61ef27e18045a36014f1458a018394f587411d4959c9cf41742bd1402f693e9eb8a76b9fdc4e4d12b5a49c0000e7b08092e7f357b72a4c1@3.115.0.208:32668"
]
```



# Register your  validator

This is a detailed step-by-step guide for setting up a HSC validator. Please be aware that while it is easy to set up a rudimentary validating node, running a production-quality validator node with a robust architecture and security features requires an extensive setup.


## Prerequisites

- You have completed how to run a full  node, which outlines how to install, connect, and configure a node.


## 1. Import your PrivateKey

The PubKey of your node is required to create a new validator. Run:

```bash
geth  attach data/geth.ipc
```

```bash  geth interactive javascript console
web3.personal.importRawKey("PRIVATEKEY","PASSWORD")

eth.accounts

personal.unlockAccount(eth.accounts[1],"PASSWORD",0)

```
* PRIVATEKEY--Your private key
* PASSWORD--Your password
* 0 --  unlock time

## 1. Contract Abi 

```bash  geth interactive javascript console
var abi=[    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogAddToTopValidators",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": true,          "internalType": "address",          "name": "fee",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogCreateValidator",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "coinbase",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "blockReward",          "type": "uint256"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogDistributeBlockReward",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": true,          "internalType": "address",          "name": "fee",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogEditValidator",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogReactive",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogRemoveFromTopValidators",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "hb",          "type": "uint256"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogRemoveValidator",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "hb",          "type": "uint256"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogRemoveValidatorIncoming",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "staker",          "type": "address"        },        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "staking",          "type": "uint256"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogStake",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "staker",          "type": "address"        },        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "amount",          "type": "uint256"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogUnstake",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": false,          "internalType": "address[]",          "name": "newSet",          "type": "address[]"        }      ],      "name": "LogUpdateValidator",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": true,          "internalType": "address",          "name": "fee",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "hb",          "type": "uint256"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogWithdrawProfits",      "type": "event"    },    {      "anonymous": false,      "inputs": [        {          "indexed": true,          "internalType": "address",          "name": "staker",          "type": "address"        },        {          "indexed": true,          "internalType": "address",          "name": "val",          "type": "address"        },        {          "indexed": false,          "internalType": "uint256",          "name": "amount",          "type": "uint256"        },        {          "indexed": false,          "internalType": "uint256",          "name": "time",          "type": "uint256"        }      ],      "name": "LogWithdrawStaking",      "type": "event"    },    {      "inputs": [],      "name": "MaxValidators",      "outputs": [        {          "internalType": "uint16",          "name": "",          "type": "uint16"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "MinimalStakingCoin",      "outputs": [        {          "internalType": "uint256",          "name": "",          "type": "uint256"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "ProposalAddr",      "outputs": [        {          "internalType": "address",          "name": "",          "type": "address"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "PunishContractAddr",      "outputs": [        {          "internalType": "address",          "name": "",          "type": "address"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "StakingLockPeriod",      "outputs": [        {          "internalType": "uint64",          "name": "",          "type": "uint64"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "ValidatorContractAddr",      "outputs": [        {          "internalType": "address",          "name": "",          "type": "address"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "WithdrawProfitPeriod",      "outputs": [        {          "internalType": "uint64",          "name": "",          "type": "uint64"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [        {          "internalType": "uint256",          "name": "",          "type": "uint256"        }      ],      "name": "currentValidatorSet",      "outputs": [        {          "internalType": "address",          "name": "",          "type": "address"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [        {          "internalType": "uint256",          "name": "",          "type": "uint256"        }      ],      "name": "highestValidatorsSet",      "outputs": [        {          "internalType": "address",          "name": "",          "type": "address"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "initialized",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "totalJailedHB",      "outputs": [        {          "internalType": "uint256",          "name": "",          "type": "uint256"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "totalStake",      "outputs": [        {          "internalType": "uint256",          "name": "",          "type": "uint256"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [        {          "internalType": "address[]",          "name": "vals",          "type": "address[]"        }      ],      "name": "initialize",      "outputs": [],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "validator",          "type": "address"        }      ],      "name": "stake",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "payable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address payable",          "name": "feeAddr",          "type": "address"        },        {          "internalType": "string",          "name": "moniker",          "type": "string"        },        {          "internalType": "string",          "name": "identity",          "type": "string"        },        {          "internalType": "string",          "name": "website",          "type": "string"        },        {          "internalType": "string",          "name": "email",          "type": "string"        },        {          "internalType": "string",          "name": "details",          "type": "string"        }      ],      "name": "createOrEditValidator",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "validator",          "type": "address"        }      ],      "name": "tryReactive",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "validator",          "type": "address"        }      ],      "name": "unstake",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "validator",          "type": "address"        }      ],      "name": "withdrawStaking",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "validator",          "type": "address"        }      ],      "name": "withdrawProfits",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [],      "name": "distributeBlockReward",      "outputs": [],      "stateMutability": "payable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address[]",          "name": "newSet",          "type": "address[]"        },        {          "internalType": "uint256",          "name": "epoch",          "type": "uint256"        }      ],      "name": "updateActiveValidatorSet",      "outputs": [],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "val",          "type": "address"        }      ],      "name": "removeValidator",      "outputs": [],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "val",          "type": "address"        }      ],      "name": "removeValidatorIncoming",      "outputs": [],      "stateMutability": "nonpayable",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "val",          "type": "address"        }      ],      "name": "getValidatorDescription",      "outputs": [        {          "internalType": "string",          "name": "",          "type": "string"        },        {          "internalType": "string",          "name": "",          "type": "string"        },        {          "internalType": "string",          "name": "",          "type": "string"        },        {          "internalType": "string",          "name": "",          "type": "string"        },        {          "internalType": "string",          "name": "",          "type": "string"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "val",          "type": "address"        }      ],      "name": "getValidatorInfo",      "outputs": [        {          "internalType": "address payable",          "name": "",          "type": "address"        },        {          "internalType": "enum Validators.Status",          "name": "",          "type": "uint8"        },        {          "internalType": "uint256",          "name": "",          "type": "uint256"        },        {          "internalType": "uint256",          "name": "",          "type": "uint256"        },        {          "internalType": "uint256",          "name": "",          "type": "uint256"        },        {          "internalType": "uint256",          "name": "",          "type": "uint256"        },        {          "internalType": "address[]",          "name": "",          "type": "address[]"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "staker",          "type": "address"        },        {          "internalType": "address",          "name": "val",          "type": "address"        }      ],      "name": "getStakingInfo",      "outputs": [        {          "internalType": "uint256",          "name": "",          "type": "uint256"        },        {          "internalType": "uint256",          "name": "",          "type": "uint256"        },        {          "internalType": "uint256",          "name": "",          "type": "uint256"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "getActiveValidators",      "outputs": [        {          "internalType": "address[]",          "name": "",          "type": "address[]"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "getTotalStakeOfActiveValidators",      "outputs": [        {          "internalType": "uint256",          "name": "total",          "type": "uint256"        },        {          "internalType": "uint256",          "name": "len",          "type": "uint256"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "who",          "type": "address"        }      ],      "name": "isActiveValidator",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [        {          "internalType": "address",          "name": "who",          "type": "address"        }      ],      "name": "isTopValidator",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [],      "name": "getTopValidators",      "outputs": [        {          "internalType": "address[]",          "name": "",          "type": "address[]"        }      ],      "stateMutability": "view",      "type": "function"    },    {      "inputs": [        {          "internalType": "string",          "name": "moniker",          "type": "string"        },        {          "internalType": "string",          "name": "identity",          "type": "string"        },        {          "internalType": "string",          "name": "website",          "type": "string"        },        {          "internalType": "string",          "name": "email",          "type": "string"        },        {          "internalType": "string",          "name": "details",          "type": "string"        }      ],      "name": "validateDescription",      "outputs": [        {          "internalType": "bool",          "name": "",          "type": "bool"        }      ],      "stateMutability": "pure",      "type": "function"    }  ]


var vc=eth.contract(abi)

 var vci=vc.at("0x000000000000000000000000000000000000f000")

```

## 2. Create a new validator


To create the validator and initialize it with a self-delegation, run the following command. `from` is the name of the private key that is used to sign transactions.

```bash
vci.createOrEditValidator.sendTransaction(eth.accounts[0], "moniker", "identity", "website", "email", "details", {from: eth.accounts[0]})
```


## Create a new proposal

```bash
vci.createProposal.sendTransaction(eth.accounts[0], "details", {from: eth.accounts[0]})

```


## Vote on a proposal

Voting is an important way for community members to help the HSC protocol evolve. Follow these steps to vote with your staked HOO.

```bash
vci.voteProposal.sendTransaction(eth.accounts[0], "details", {from: eth.accounts[0]})

```

## Stake  HOO  to the validator

```bash
vci.stake.sendTransaction(eth.accounts[1],eth.accounts[0], {from: eth.accounts[0]})

```

## UnStake HOO 

```bash
vci.unstake.sendTransaction(eth.accounts[0], {from: eth.accounts[0]})

```

## WithdrawStaking 

```bash
vci.withdrawStaking.sendTransaction(eth.accounts[0], {from: eth.accounts[0]})

```

## Others 

```bash
vci.withdrawProfits.sendTransaction(eth.accounts[0], {from: eth.accounts[0]})


vc.getValidatorInfo.call('0x598feab9ff6a090a7faa9df0f3b4df3f0c8d35fc')

vc.totalStake.call(eth.accounts[0])

vc.getActiveValidators.call(eth.accounts[0])

vc.staked.call(eth.accounts[0],eth.accounts[0])

vc.votes.call(eth.accounts[0],1)   //1--proposal id
vc.pass.call(eth.accounts[0])  


```

