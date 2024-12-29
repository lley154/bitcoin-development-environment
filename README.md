# Bitcoin Development Environment
## Setup
Connect and log into the docker Ubuntu 24.10 Linux container with the user created in [Docker Setup](https://github.com/lley154/docker-setup).
```
$ sudo apt install curl -y
$ curl -O  https://bitcoin.org/bin/bitcoin-core-27.0/bitcoin-27.0-x86_64-linux-gnu.tar.gz
$ tar -xvzf bitcoin-27.0-x86_64-linux-gnu.tar.gz
$ cd bitcoin-27.0
$ sudo cp bin/* /usr/local/bin 
$ mkdir ~/.bitcoin
$ cp bitcoin.conf ~/.bitcoin
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
To stop the network, use the following command.
```
$ bitcoin-cli -regtest stop
Bitcoin Core stopping
```

## Create a transaction
```
$ bitcoind -regtest -debug
Bitcoin Core starting

$ bitcoin-cli -regtest getbalance
12462.50000000
```
List unspent transactions to get inputs
```
bitcoin-cli -regtest listunspent
[  {
    "txid": "a6629793807cacb30428f01eb4ea5e8abac03841cda179563e1d4131b9ce0d8c",
    "vout": 0,
    "address": "bcrt1qqtmnssy8vzcd3uw27rhly3st20wwj4ehulauue",
    "label": "",
    "scriptPubKey": "001402f738408760b0d8f1caf0eff2460b53dce95737",
    "amount": 12.50000000,
    "confirmations": 196,
    "spendable": true,
    "solvable": true,
    "desc": "wpkh([6ccc2364/84h/1h/0h/0/0]02829d209e0b5179c6eaba4130f6226c6b916efed611442982b905680ef39ff11e)#rakema5w",
    "parent_descs": [
      "wpkh(tpubD6NzVbkrYhZ4XKKfkV8hJWiuYzbobpv9tnLLpPNtVmG1d8Z4qVy66RUGgssosvA5xzX8n15e4CgaDuihcY55277gir5bhJqVc8nJsBFRWid/84h/1h/0h/0/*)#l4a8ewww"
    ],
    "safe": true
  }
]
```
Get a new address
```
$ bitcoin-cli -regtest getnewaddress
bcrt1qrmrfkjrn9qrr39y7wcqknj8d7y8pp3nepjc0ql
```
Send 0.01 BTC to the new address.
```
$ bitcoin-cli -regtest sendtoaddress bcrt1qrmrfkjrn9qrr39y7wcqknj8d7y8pp3nepjc0ql 0.01
0fdf5c71c2128f57baeb524cce86528b13a18c755c9296c06da21598c458a237
```
Look at the mempool to see the transaction waiting to be picked up
```
$ bitcoin-cli -regtest getmempoolinfo
{
  "loaded": true,
  "size": 1,
  "bytes": 141,
  "usage": 1152,
  "total_fee": 0.00001410,
  "maxmempool": 300000000,
  "mempoolminfee": 0.00001000,
  "minrelaytxfee": 0.00001000,
  "incrementalrelayfee": 0.00001000,
  "unbroadcastcount": 1,
  "fullrbf": false
}

$ bitcoin-cli -regtest getrawmempool true
{
  "0fdf5c71c2128f57baeb524cce86528b13a18c755c9296c06da21598c458a237": {
    "vsize": 141,
    "weight": 561,
    "time": 1734971000,
    "height": 501,
    "descendantcount": 1,
    "descendantsize": 141,
    "ancestorcount": 1,
    "ancestorsize": 141,
    "wtxid": "38aef2a9e7ead02f6f39ee82284daa3dbb3aee99c615eafd5cd10713246d7ea4",
    "fees": {
      "base": 0.00001410,
      "modified": 0.00001410,
      "ancestor": 0.00001410,
      "descendant": 0.00001410
    },
    "depends": [
    ],
    "spentby": [
    ],
    "bip125-replaceable": true,
    "unbroadcast": true
  }
}
```

Let's look at the transaction details.
```
$ bitcoin-cli -regtest getrawtransaction 0fdf5c71c2128f57baeb524cce86528b13a18c755c9296c06da21598c458a237 true
{
  "txid": "0fdf5c71c2128f57baeb524cce86528b13a18c755c9296c06da21598c458a237",
  "hash": "e0054e546b37a2c6e6ef1342842ac81c5e2c3b356b1a9dde002eafd369e203b7",
  "version": 2,
  "size": 222,
  "vsize": 141,
  "weight": 561,
  "locktime": 500,
  "vin": [
    {
      "txid": "db02524b5a26253fcc2d38a1a76c0853d18e43a73a723b29b23d0705453a144c",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "txinwitness": [
        "3044022069630604ff6035eba1228d0988065226e52e8b7f2b9cd71a986b98566847a6b40220522c66234f158c24eaada5a7dd63b3fa49d9a0980a37c13a151f10c9258ec15701",
        "028140e9f5a5cfa84fdc765a025ef13dc03bccdb17ab9d96b2beb8ba2ca91e18cf"
      ],
      "sequence": 4294967293
    }
  ],
  "vout": [
    {
      "value": 12.39998590,
      "n": 0,
      "scriptPubKey": {
        "asm": "0 adef5d56f757b4dc311c723820482440c8e0279f",
        "desc": "addr(bcrt1q4hh464hh276dcvguwguzqjpygrywqful2n9sz2)#ywy60d7k",
        "hex": "0014adef5d56f757b4dc311c723820482440c8e0279f",
        "address": "bcrt1q4hh464hh276dcvguwguzqjpygrywqful2n9sz2",
        "type": "witness_v0_keyhash"
      }
    },
    {
      "value": 0.10000000,
      "n": 1,
      "scriptPubKey": {
        "asm": "0 17d15f65985397e5b40da2a1dedb404e19ad0554",
        "desc": "addr(bcrt1qzlg47evc2wt7tdqd52saak6qfcv66p25rj5nja)#u5g6j4he",
        "hex": "001417d15f65985397e5b40da2a1dedb404e19ad0554",
        "address": "bcrt1qzlg47evc2wt7tdqd52saak6qfcv66p25rj5nja",
        "type": "witness_v0_keyhash"
      }
    }
  ],
  "hex": "020000000001014c143a4505073db2293b723aa7438ed153086ca7a1382dcc3f25265a4b5202db0000000000fdffffff027ee0e84900000000160014adef5d56f757b4dc311c723820482440c8e0279f809698000000000016001417d15f65985397e5b40da2a1dedb404e19ad055402473044022069630604ff6035eba1228d0988065226e52e8b7f2b9cd71a986b98566847a6b40220522c66234f158c24eaada5a7dd63b3fa49d9a0980a37c13a151f10c9258ec1570121028140e9f5a5cfa84fdc765a025ef13dc03bccdb17ab9d96b2beb8ba2ca91e18cff4010000"
}
```
For more info, use the help command
```
$ bitcoin-cli -regtest help
== Blockchain ==
dumptxoutset "path"
getbestblockhash
getblock "blockhash" ( verbosity )
getblockchaininfo
getblockcount
getblockfilter "blockhash" ( "filtertype" )
getblockfrompeer "blockhash" peer_id
getblockhash height
getblockheader "blockhash" ( verbose )
getblockstats hash_or_height ( stats )
getchainstates
getchaintips
getchaintxstats ( nblocks "blockhash" )
getdeploymentinfo ( "blockhash" )
getdifficulty
getmempoolancestors "txid" ( verbose )
getmempooldescendants "txid" ( verbose )
getmempoolentry "txid"
getmempoolinfo
getrawmempool ( verbose mempool_sequence )
gettxout "txid" n ( include_mempool )
gettxoutproof ["txid",...] ( "blockhash" )
gettxoutsetinfo ( "hash_type" hash_or_height use_index )
gettxspendingprevout [{"txid":"hex","vout":n},...]
importmempool "filepath" ( options )
loadtxoutset "path"
preciousblock "blockhash"
pruneblockchain height
savemempool
scanblocks "action" ( [scanobjects,...] start_height stop_height "filtertype" options )
scantxoutset "action" ( [scanobjects,...] )
verifychain ( checklevel nblocks )
verifytxoutproof "proof"

== Control ==
getmemoryinfo ( "mode" )
getrpcinfo
help ( "command" )
logging ( ["include_category",...] ["exclude_category",...] )
stop
uptime

== Mining ==
getblocktemplate {"mode":"str","capabilities":["str",...],"rules":["segwit","str",...],"longpollid":"str","data":"hex"}
getmininginfo
getnetworkhashps ( nblocks height )
getprioritisedtransactions
prioritisetransaction "txid" ( dummy ) fee_delta
submitblock "hexdata" ( "dummy" )
submitheader "hexdata"

== Network ==
addnode "node" "command" ( v2transport )
clearbanned
disconnectnode ( "address" nodeid )
getaddednodeinfo ( "node" )
getaddrmaninfo
getconnectioncount
getnettotals
getnetworkinfo
getnodeaddresses ( count "network" )
getpeerinfo
listbanned
ping
setban "subnet" "command" ( bantime absolute )
setnetworkactive state

== Rawtransactions ==
analyzepsbt "psbt"
combinepsbt ["psbt",...]
combinerawtransaction ["hexstring",...]
converttopsbt "hexstring" ( permitsigdata iswitness )
createpsbt [{"txid":"hex","vout":n,"sequence":n},...] [{"address":amount,...},{"data":"hex"},...] ( locktime replaceable )
createrawtransaction [{"txid":"hex","vout":n,"sequence":n},...] [{"address":amount,...},{"data":"hex"},...] ( locktime replaceable )
decodepsbt "psbt"
decoderawtransaction "hexstring" ( iswitness )
decodescript "hexstring"
descriptorprocesspsbt "psbt" ["",{"desc":"str","range":n or [n,n]},...] ( "sighashtype" bip32derivs finalize )
finalizepsbt "psbt" ( extract )
fundrawtransaction "hexstring" ( options iswitness )
getrawtransaction "txid" ( verbosity "blockhash" )
joinpsbts ["psbt",...]
sendrawtransaction "hexstring" ( maxfeerate maxburnamount )
signrawtransactionwithkey "hexstring" ["privatekey",...] ( [{"txid":"hex","vout":n,"scriptPubKey":"hex","redeemScript":"hex","witnessScript":"hex","amount":amount},...] "sighashtype" )
submitpackage ["rawtx",...]
testmempoolaccept ["rawtx",...] ( maxfeerate )
utxoupdatepsbt "psbt" ( ["",{"desc":"str","range":n or [n,n]},...] )

== Signer ==
enumeratesigners

== Util ==
createmultisig nrequired ["key",...] ( "address_type" )
deriveaddresses "descriptor" ( range )
estimatesmartfee conf_target ( "estimate_mode" )
getdescriptorinfo "descriptor"
getindexinfo ( "index_name" )
signmessagewithprivkey "privkey" "message"
validateaddress "address"
verifymessage "address" "signature" "message"

== Wallet ==
abandontransaction "txid"
abortrescan
addmultisigaddress nrequired ["key",...] ( "label" "address_type" )
backupwallet "destination"
bumpfee "txid" ( options )
createwallet "wallet_name" ( disable_private_keys blank "passphrase" avoid_reuse descriptors load_on_startup external_signer )
dumpprivkey "address"
dumpwallet "filename"
encryptwallet "passphrase"
getaddressesbylabel "label"
getaddressinfo "address"
getbalance ( "dummy" minconf include_watchonly avoid_reuse )
getbalances
getnewaddress ( "label" "address_type" )
getrawchangeaddress ( "address_type" )
getreceivedbyaddress "address" ( minconf include_immature_coinbase )
getreceivedbylabel "label" ( minconf include_immature_coinbase )
gettransaction "txid" ( include_watchonly verbose )
getunconfirmedbalance
getwalletinfo
importaddress "address" ( "label" rescan p2sh )
importdescriptors requests
importmulti requests ( options )
importprivkey "privkey" ( "label" rescan )
importprunedfunds "rawtransaction" "txoutproof"
importpubkey "pubkey" ( "label" rescan )
importwallet "filename"
keypoolrefill ( newsize )
listaddressgroupings
listdescriptors ( private )
listlabels ( "purpose" )
listlockunspent
listreceivedbyaddress ( minconf include_empty include_watchonly "address_filter" include_immature_coinbase )
listreceivedbylabel ( minconf include_empty include_watchonly include_immature_coinbase )
listsinceblock ( "blockhash" target_confirmations include_watchonly include_removed include_change "label" )
listtransactions ( "label" count skip include_watchonly )
listunspent ( minconf maxconf ["address",...] include_unsafe query_options )
listwalletdir
listwallets
loadwallet "filename" ( load_on_startup )
lockunspent unlock ( [{"txid":"hex","vout":n},...] persistent )
migratewallet ( "wallet_name" "passphrase" )
newkeypool
psbtbumpfee "txid" ( options )
removeprunedfunds "txid"
rescanblockchain ( start_height stop_height )
restorewallet "wallet_name" "backup_file" ( load_on_startup )
send [{"address":amount,...},{"data":"hex"},...] ( conf_target "estimate_mode" fee_rate options )
sendall ["address",{"address":amount,...},...] ( conf_target "estimate_mode" fee_rate options )
sendmany ( "" ) {"address":amount,...} ( minconf "comment" ["address",...] replaceable conf_target "estimate_mode" fee_rate verbose )
sendtoaddress "address" amount ( "comment" "comment_to" subtractfeefromamount replaceable conf_target "estimate_mode" avoid_reuse fee_rate verbose )
sethdseed ( newkeypool "seed" )
setlabel "address" "label"
settxfee amount
setwalletflag "flag" ( value )
signmessage "address" "message"
signrawtransactionwithwallet "hexstring" ( [{"txid":"hex","vout":n,"scriptPubKey":"hex","redeemScript":"hex","witnessScript":"hex","amount":amount},...] "sighashtype" )
simulaterawtransaction ( ["rawtx",...] {"include_watchonly":bool,...} )
unloadwallet ( "wallet_name" load_on_startup )
upgradewallet ( version )
walletcreatefundedpsbt ( [{"txid":"hex","vout":n,"sequence":n,"weight":n},...] ) [{"address":amount,...},{"data":"hex"},...] ( locktime options bip32derivs )
walletdisplayaddress "address"
walletlock
walletpassphrase "passphrase" timeout
walletpassphrasechange "oldpassphrase" "newpassphrase"
walletprocesspsbt "psbt" ( sign "sighashtype" bip32derivs finalize )

== Zmq ==
getzmqnotifications
lawrence@80e3b38f7216:~$ 
```
