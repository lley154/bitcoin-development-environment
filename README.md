# Bitcoin Development Setup
Connect and log into the docker Ubuntu 24.10 Linux container with the user created in [Docker Setup](https://github.com/lley154/docker-setup).
```
$ sudo apt install curl -y
$ curl -O  https://bitcoin.org/bin/bitcoin-core-27.0/bitcoin-27.0-x86_64-linux-gnu.tar.gz
$ tar -xvzf bitcoin-27.0-x86_64-linux-gnu.tar.gz
$ cd bitcoin-27.0
$ sudo cp bin/* /usr/local/bin 
$ mkdir ~/.bitcoin
$ cp .bitcoin.conf ~/.bitcoin
$ cd ~/.bitcoin
```
Add the following lines to the end of ~/.bitcon/bitcoin.conf
```
$ nano bitcoin.conf
# Options for regtest
[regtest]
daemon=1
txindex=1
server=1
rpcuser=test
rpcpassword=test1234
rpcallowip=127.0.0.1
fallbackfee=0.0001
# Disable Tor onion service (default: 1)
listenonion=0
```
Start up the local bitcoin regression test network
```
$ bitcoind -regtest -debug
Bitcoin Core starting
```
In a new terminal window, tail the debug log file
$ tail -f ~/.bitcoin/regtest/debug.log

Get blockchain info
```
$ bitcoin-cli -regtest getblockchaininfo
{
  "chain": "regtest",
  "blocks": 0,
  "headers": 0,
  "bestblockhash": "0f9188f13cb7b2c71f2a335e3a4fc328bf5beb436012afca590b1a11466e2206",
  "difficulty": 4.656542373906925e-10,
  "time": 1296688602,
  "mediantime": 1296688602,
  "verificationprogress": 1,
  "initialblockdownload": true,
  "chainwork": "0000000000000000000000000000000000000000000000000000000000000002",
  "size_on_disk": 293,
  "pruned": false,
  "warnings": ""
}
```
Create a test wallet
```
$ bitcoin-cli -regtest createwallet "test-wallet"
{
  "name": "test-wallet"
}
```
Generate a new address for the test wallet.
```
$ bitcoin-cli -regtest getnewaddress
bcrt1qqtmnssy8vzcd3uw27rhly3st20wwj4ehulauue
```
Mint 500 blocks and send the rewards to the new address
```
$ bitcoin-cli -regtest generatetoaddress 500 bcrt1qqtmnssy8vzcd3uw27rhly3st20wwj4ehulauue
[
  "54223fc7d0bc28abc8db656895f85a45748e476a1719b214a93e78283394bd23",
  "7671f5b9a7a76980cb64cd6dc4e1ea3972a37070d5a8cc04633f2ee9a113f130",
  "69809a73280e4b70f29c0ba8f868ce72a55e75cd4729f5d026b5637da009d535",

…
]
```
Get the blockchain info and noticed that 500 blocks have been added
```
$ bitcoin-cli -regtest getblockchaininfo
{
  "chain": "regtest",
  "blocks": 500,
  "headers": 500,
  "bestblockhash": "336433bd9df549d4cc8f591b5440fdbccc8e08e845cda25c2ada260174924eb7",
  "difficulty": 4.656542373906925e-10,
  "time": 1734969474,
  "mediantime": 1734969473,
  "verificationprogress": 1,
  "initialblockdownload": false,
  "chainwork": "00000000000000000000000000000000000000000000000000000000000003ea",
  "size_on_disk": 149650,
  "pruned": false,
  "warnings": ""
}
```
Let's look at the mining info of the local blockchain
```
$ bitcoin-cli -regtest getmininginfo
{
  "blocks": 501,
  "currentblockweight": 4000,
  "currentblocktx": 0,
  "difficulty": 4.656542373906925e-10,
  "networkhashps": 0.2296650717703349,
  "pooledtx": 0,
  "chain": "regtest",
  "warnings": ""
}
```
Look at the wallet info including the balances
```
$ bitcoin-cli -regtest getwalletinfo
{
  "walletname": "",
  "walletversion": 169900,
  "format": "sqlite",
  "balance": 12475.00000000,
  "unconfirmed_balance": 0.00000000,
  "immature_balance": 925.00000000,
  "txcount": 501,
  "keypoolsize": 4000,
  "keypoolsize_hd_internal": 4000,
  "paytxfee": 0.00000000,
  "private_keys_enabled": true,
  "avoid_reuse": false,
  "scanning": false,
  "descriptors": true,
  "external_signer": false,
  "blank": false,
  "birthtime": 1734969341,
  "lastprocessedblock": {
    "hash": "2771d19f024e98fa48d77ab7468a3ff5651fb45dda6003a2aed346e0d6c308d7",
    "height": 501
  }
}

$ bitcoin-cli -regtest getbalance
12462.50000000
```
To stop the network, use the following command
```
$ bitcoin-cli -regtest stop
Bitcoin Core stopping
```

