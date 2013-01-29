# spendfrom.py utility

Submit completed tests by making a pull request from your fork of this project.

Test environment:

A Linux machine running a raw-transactions-API-capable version of bitcoind/Bitcoin-Qt (version 0.7.2 or git HEAD),
with python and a python jsonrpc library installed.

A testnet wallet that has a greater-than-zero balance and has sent and received bitcoins.

## Bounties:

No bounties for this, I hope people who need the 'coin control' feature will help test.

## Test Setup

1. Backup your main wallet.dat (if it contains any bitcoins; better safe than sorry!)
2. Make sure you have a bitcoin.conf file in your bitcoin [data directory](https://en.bitcoin.it/wiki/Data_directory)
with rpcuser/rpcpassword set.
3. Run bitcoind/Bitcoin-Qt version 0.7.2 or later with the -testnet flag
4. Download https://github.com/gavinandresen/bitcoin-git/raw/spendfrom/contrib/spendfrom/spendfrom.py

If you need testnet bitcoins, ask in the #bitcoin-dev IRC channel or get some from
http://tpfaucet.appspot.com/

## Tests

$ grep Bitcoin.version /home/btc/.bitcoin/testnet3/debug.log
01/29/13 20:20:50 Bitcoin version v0.7.2.0-gd53af9c-beta ()
$ md5sum spendfrom.py 
24f17677078557aed3b5860e1de75fea  spendfrom.py


### List available spending addresses

Run:
```spendfrom.py --testnet```

EXPECT:

See list of addresses and balances.

RESULT: PASS

$ ./spendfrom.py --testnet
mtNqraWqaGQx2CgsPVkJZ2GoyA3o9DENiQ 17.99950000 
mnNGLvRKUG5nnqWRXJJCrfEDv7CxkUxh4m 2.00000000 


### Send from one address

Run the following, replacing $ADDRESS and $AMOUNT with an address/amount available to spend:
```spendfrom.py --testnet --from=$ADDRESS --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=$AMOUNT```

EXPECT:
1. transaction id returned
2. transaction sent from your wallet to the testnet faucet (your balance decreases by $AMOUNT)

RESULT: PASS

$ bitcoind -testnet getbalance
19.99950000
$ ADDRESS=mnNGLvRKUG5nnqWRXJJCrfEDv7CxkUxh4m
$ AMOUNT=1.23456789
$ ./spendfrom.py --testnet --from=$ADDRESS --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=$AMOUNT
60d13b65f22677f201092a636c1ab76ce4c9cadc52d3a5b54487269a9dd93ab9
$ bitcoind -testnet getbalance
17.99950000
<wait for next block>
$ bitcoind -testnet getbalance
18.76493211
$ echo '19.99950000 - 1.23456789 - 18.76493211' |bc
0
$ $ bitcoind -testnet gettransaction 60d13b65f22677f201092a636c1ab76ce4c9cadc52d3a5b54487269a9dd93ab9
{
    "amount" : -1.23456789,
    "fee" : 0.00000000,
    "confirmations" : 2,
    "blockhash" : "0000000063fed9b35c73324c6fc3aab1c59ea4602b3043b49f15f4e0706663e6",
    "blockindex" : 1,
    "blocktime" : 1359492700,
    "txid" : "60d13b65f22677f201092a636c1ab76ce4c9cadc52d3a5b54487269a9dd93ab9",
    "time" : 1359491469,
    "timereceived" : 1359491469,
    "details" : [
        {
            "account" : "",
            "address" : "mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV",
            "category" : "send",
            "amount" : -1.23456789,
            "fee" : 0.00000000
        },
        {
            "account" : "",
            "address" : "mnNGLvRKUG5nnqWRXJJCrfEDv7CxkUxh4m",
            "category" : "send",
            "amount" : -0.76543211,
            "fee" : 0.00000000
        },
        {
            "account" : "",
            "address" : "mnNGLvRKUG5nnqWRXJJCrfEDv7CxkUxh4m",
            "category" : "receive",
            "amount" : 0.76543211
        }
    ]
}


### Attempt to send too much

Run the following, replacing $ADDRESS with another address that is available to spend and has less than 99 testnet coins::
```spendfrom.py --testnet --from=$ADDRESS --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=99```

EXPECT: error complaining that $ADDRESS doesn't have that many coins.

RESULT: PASS

$ ./spendfrom.py --testnet
mtNqraWqaGQx2CgsPVkJZ2GoyA3o9DENiQ 17.99950000 
mnNGLvRKUG5nnqWRXJJCrfEDv7CxkUxh4m 0.76543211 
$ ADDRESS=mnNGLvRKUG5nnqWRXJJCrfEDv7CxkUxh4m
$ ./spendfrom.py --testnet --from=$ADDRESS --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=99
Error, only 0.765432 BTC available, need 99.000000
$ bitcoind -testnet getbalance
18.76493211


### Send from one address, change to a second address

Run the following, replacing $ADDRESS1 with another address that has coins, $ADDRESS2 any other address in your wallet,
and $AMOUNT an amount less than the amount available from $ADDRESS1:
```spendfrom.py --testnet --from=$ADDRESS1,$ADDRESS2 --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=$AMOUNT```

EXPECT:
1. transaction id
2. Run bitcoind/bitcoin-qt -testnet getrawtransaction $txid  :  the transaction should have two outputs, one to $ADDRESS2
and the other to the testnet faucet.

RESULT: PASS

$ ./spendfrom.py --testnet
mvcUAVEGDGo91G8dFThvmjwYVTdADMkkU4 12.00000000 
mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW 0.76493211 
mhFZNk9B6Pjo5CCagnt7f9yrwSbTay86ZU 5.99900000 
$ bitcoind -testnet getbalance
18.76393211
$ ADDRESS1=mvcUAVEGDGo91G8dFThvmjwYVTdADMkkU4
$ ADDRESS2=mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW
$ AMOUNT=4
$ ./spendfrom.py --testnet --from=$ADDRESS1,$ADDRESS2 --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=$AMOUNT
a782f124538b18054066e5cfa99900f6e6f9f19bbcdf3a9b0c3febfd88f0defa
$ bitcoind -testnet decoderawtransaction $(bitcoind -testnet getrawtransaction a782f124538b18054066e5cfa99900f6e6f9f19bbcdf3a9b0c3febfd88f0defa)
{
    "txid" : "a782f124538b18054066e5cfa99900f6e6f9f19bbcdf3a9b0c3febfd88f0defa",
    "version" : 1,
    "locktime" : 0,
    "vin" : [
        {
            "txid" : "6f4df07ea08f30bf6cba496c1c88157a53d87755bc40180c585161df0c3c71d2",
            "vout" : 1,
            "scriptSig" : {
                "asm" : "3045022055b78d0c0a725b270d5adabfa9ba44cfab604a6ea440522a868123aaf5f64a6d022100f0fb28e2b4f96ac17863fa403698a1b909ba5281c6df3d0fae3a3ebbecc71bad01 0230b6efd859b0421d956d2c2af5bc6640fa682204aaa67cf6ea18fe816c6ab8f9",
                "hex" : "483045022055b78d0c0a725b270d5adabfa9ba44cfab604a6ea440522a868123aaf5f64a6d022100f0fb28e2b4f96ac17863fa403698a1b909ba5281c6df3d0fae3a3ebbecc71bad01210230b6efd859b0421d956d2c2af5bc6640fa682204aaa67cf6ea18fe816c6ab8f9"
            },
            "sequence" : 4294967295
        }
    ],
    "vout" : [
        {
            "value" : 4.00000000,
            "n" : 0,
            "scriptPubKey" : {
                "asm" : "OP_DUP OP_HASH160 7abf72ef083647cd2cb792c9435ce854bee00aee OP_EQUALVERIFY OP_CHECKSIG",
                "hex" : "76a9147abf72ef083647cd2cb792c9435ce854bee00aee88ac",
                "reqSigs" : 1,
                "type" : "pubkeyhash",
                "addresses" : [
                    "mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV"
                ]
            }
        },
        {
            "value" : 8.00000000,
            "n" : 1,
            "scriptPubKey" : {
                "asm" : "OP_DUP OP_HASH160 bc1e94f701cd475c05f2cd4e7e8c9a497ca26e46 OP_EQUALVERIFY OP_CHECKSIG",
                "hex" : "76a914bc1e94f701cd475c05f2cd4e7e8c9a497ca26e4688ac",
                "reqSigs" : 1,
                "type" : "pubkeyhash",
                "addresses" : [
                    "mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW"
                ]
            }
        }
    ]
}
$ bitcoind -testnet getbalance
14.76393211
$ ./spendfrom.py --testnet
mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW 8.76493211  (2 transactions)
mhFZNk9B6Pjo5CCagnt7f9yrwSbTay86ZU 5.99900000 


### Send from more than one address

Choose two addresses that have coins available (run spendfrom.py --testnet  with no arguments), and total the
amount of coins available from those two addresses. Then run:

```spendfrom.py --testnet --from=$ADDRESS1,$ADDRESS2 --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=$AMOUNT```

EXPECT: transaction id, wallet balance goes down by $AMOUNT

RESULT: PASS

$ ./spendfrom.py --testnet
mjeKBX5H89jrRwwc3a4ieUB8VJSjrQcHdU 1.00000000 
myew9Kd5bNLR8ZGmcNEi6hHUFELNFsNPTb 4.99850000 
mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW 8.76493211  (2 transactions)
$ bitcoind -testnet getbalance
14.76343211
$ echo '1.00000000 + 4.99850000' |bc
5.99850000
$ ADDRESS1=mjeKBX5H89jrRwwc3a4ieUB8VJSjrQcHdU
$ ADDRESS2=myew9Kd5bNLR8ZGmcNEi6hHUFELNFsNPTb
$ AMOUNT=5.99850000
$ ./spendfrom.py --testnet --from=$ADDRESS1,$ADDRESS2 --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=$AMOUNT
c95ea9643ced3965d2d7b82420a63fda47884d18601294f588f892fa1c930449
$ bitcoind -testnet getbalance
8.76493211


### Send-to-self

Choose an address with coins available, and create a send-to-self transaction for less than the total amount available:

```spendfrom.py --testnet --from=$ADDRESS --to=$ADDRESS --amount=$AMOUNT```

EXPECT: send-to-self transaction created with one output that sends more than $AMOUNT coins from $ADDRESS to $ADDRESS (because
sendfrom.py combines leftover change with $AMOUNT and sends it to $ADDRESS).

RESULT: PASS

$ ./spendfrom.py --testnet
mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW 8.76493211  (2 transactions)
$ ADDRESS=mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW
$ AMOUNT=0.01
$ bitcoind -testnet decoderawtransaction $(bitcoind -testnet getrawtransaction 23488ef75bb8620af7f5e59fa12ea16d4cff98da8c70dad16d62c057b450d651)
[...]
    "vout" : [
        {
            "value" : 0.76493211,



### Fee sanity-checks

Attempt to send a tiny amount without paying a fee:

```spendfrom.py --testnet --from=$ADDRESS --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=0.00001```

EXPECT: error

RESULT: PASS (if a python exception is expected)

$ ./spendfrom.py --testnet
mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW 8.76493211  (2 transactions)
$ ADDRESS=mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW
$ ./spendfrom.py --testnet --from=$ADDRESS --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=0.00001
Traceback (most recent call last):
  File "./spendfrom.py", line 267, in <module>
    main()
  File "./spendfrom.py", line 263, in main
    txid = bitcoind.sendrawtransaction(txdata)
  File "/home/btc/jsonrpc/authproxy.py", line 106, in __call__
    raise JSONRPCException(resp['error'])
jsonrpc.authproxy.JSONRPCException


Attempt to send with an outrageous fee:

```spendfrom.py --testnet --from=$ADDRESS --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=0.01 --fee=1.0```

EXPECT: error

RESULT: PASS

$ ./spendfrom.py --testnet
mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW 8.76493211  (2 transactions)
$ ADDRESS=mxfdxz76mWoVdSB93mMzkjdkHL7qc66abW
$ ./spendfrom.py --testnet --from=$ADDRESS --to=mrhz5ZgSF3C1BSdyCKt3gEdhKoRL5BNfJV --amount=0.01 --fee=1.0
Rejecting transaction, unreasonable fee of 1.00000000


### Error: bitcoin not running

1. Quit bitcoind/Bitcoin-Qt
2. Run spendfrom.py

EXPECT: helpful error message

RESULT: PASS

$ ./spendfrom.py --testnet
Error connecting to RPC server at http://user:foobar@127.0.0.1:18332
