+++
layout = 'post'
title = 'Setting up a local BSC Chain test node'
date = 2021-08-08T12:00:00+05:45
+++

This helps you setup a local bsc chain test node using a binary called `bnbchaind`. For the binary to be installed in your machine, you will need to run the following commands. The first command downloads a installer script that can be executed that will place binaries in correct folders for you. 
```sh
wget -qO- https://raw.githubusercontent.com/binance-chain/node-binary/master/install.sh
sh install.sh
```

The script will provide `bnbchaind` and `bnbcli` binaries. Just to make sure everything is working fine: 
```sh
bnbchaind version
bnbcli version
```

Then we need to create a genesis file and start the network just like we do using `geth` for full/test ethereum validator nodes. For that move in your preferred folder and:
```sh
bnbchaind init --home /home/neymar/bscchain/localnet --moniker test
```
The command initializes the `genesis` file. This file is essential to bootstrap the local test node. Copy the `genesis.json` and other files to `$home/config`. 
```sh
cp ./network/config/genesis /home/neymar/config
```
Now, we can start our bsc local node.
```
bnbchaind start --home $home
```

- [binance guide](https://docs.binance.org/guides/node/localnetwork.html)
- [quick-node-guide-using-geth](https://www.quicknode.com/guides/infrastructure/how-to-run-a-binance-smart-chain-node)
