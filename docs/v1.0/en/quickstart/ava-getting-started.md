---
title: "AVA Quickstart Guide"
excerpt: ""
---

# Quickstart Guide

The quickest way to learn about AVA is to run a node and interact with the network.

In this tutorial, which should take less than 10 minutes, we will:

* Download and compile an AVA node implementation.
* Install and run an AVA node.
* Create a local test network.
* Acquire AVA, the AVA network's native token, on your local test network.
* Send AVA on your local test network.
* Start validating the Default Subnet (staking)

If you run into any issues at all, please come ask for help on our [Discord Server!](https://discord.gg/wdkGmJ9)
We will work to get you through any problems you may face with this tutorial or any AVA-related matter. 

## Requirements 

AVA is an incredibly lightweight protocol, so the minimum computer requirements are quite modest. 

* Hardware: 2 GHz or faster CPU, 3 GB RAM, 250 MB hard disk.
* OS: Ubuntu == 18.04 or Mac OS X >= Catalina. (Ubuntu versions other than 18.04 may work but have not been tested.)
* Software: [Go](https://golang.org/doc/install) version == 1.13.X and set up [`$GOPATH`](https://github.com/golang/go/wiki/SettingGOPATH).
  Note: Go version **must be** 1.13.X. Version 1.14.X will not work (support coming soon.)

To check if Go version 1.13.X is installed, run `go version` in your terminal.
To check if your `$GOPATH` is set, run `echo $GOPATH`.

## Run an AVA Node

Let's download, compile and run Gecko, the Go implementation of an AVA node, and connect to the AVA Public Testnet.

### Install Libraries

Ubuntu users need the following libraries to run Gecko:

* libssl-dev
* libuv1-dev
* cmake
* make
* curl
* g++
  
Install the libraries:

```sh
sudo apt-get install libssl-dev libuv1-dev cmake make curl g++
```

### Download Gecko Source Code

Clone the Gecko repository:

```sh
cd $GOPATH
mkdir -p src/github.com/ava-labs
cd src/github.com/ava-labs
git clone https://github.com/ava-labs/gecko.git
cd gecko
```

### Build the Executable

Build Gecko using the build script:

```sh
./scripts/build.sh
```

The Gecko binary, named `ava`, is in the `build` directory. 

### Create a Key and Certificate for your Node

Nodes use a TLS certificate as an identity when communicating with other nodes.
You'll need to run a script in order to generate your node's TLS certificate.

Generate the TLS certificate:

```sh
sh keys/genStaker.sh
```

**Do not modify or delete `staker.key`, `staker.crt` or `staker.csr` in** `keys`.
Doing so would erase your node's identity.

If you have run this script before, you'll see a message telling you that a key/certificate
have not been generated because doing so would overwrite the existing ones.

### Connect to the Public Testnet

The AVA Public Testnet is a sandbox AVA network where AVA tokens are free.
You can use it to play around in a low-stakes environment.

To start a node and connect it to the AVA testnet:

```sh
./build/ava
```

You should see some pretty ASCII art and log messages when your node starts.
You may see a few warnings. These are OK.

You can use `Ctrl + C` to kill the node.

If you want to create a local test network rather than connect to the Public Testnet, [see here.](../tutorials/creating-a-local-testnet.md)

### Verify Connection

To ensure that your node is connected to other nodes, call `admin.peers`:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "admin.peers",
    "params":{},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

The `peers` field in the response should contain several IPs.
If it's empty, your node wasn't able to connect to the network; double check your port forwarding settings,
and if problems persist contact us on [Discord.](https://discord.gg/wdkGmJ9)

## Create a Keystore User

AVA nodes provide a built-in **Keystore.**
The Keystore manages users.
A user is a password-protected identity that a client can use when interacting with blockchains.
The same user can be used as an identity on many blockchains, and one client can have multiple users.

To create a user, call `keystore.createUser`:

```json
curl -X POST --data '{
     "jsonrpc": "2.0",
     "id": 1,
     "method": "keystore.createUser",
     "params": {
         "username": "YOUR USERNAME HERE",
         "password": "YOUR PASSWORD HERE"
     }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/keystore
```

The response should look like this:

```json
{
     "jsonrpc":"2.0",
     "result":{"success":true},
     "id":1
}
```

Now you have a user on this node.
Keystore data exists at the node level.
Users you create on one node's Keystore do not exist on
other nodes but you can import/export users to/from the Keystore.
See the [Keystore API](../api/keystore.md) to see how.

## Create an Address

AVA is a network of heterogeneous blockchains.
One of these blockchains is the X-Chain, which acts as a decentralized platform for creating and trading digital assets.
(Think X for eXchanging assets.)

There's a special asset on the X-Chain called the [AVA token](../core-concepts/overview.md#the-x-chain), or just AVA. 
This is the native token of the AVA network, and transaction fees on the AVA network are paid in AVA.

Let's acquire some AVA tokens.
First, we'll need to create an address to hold them.

To create a new address on the X-Chain, call `avm.createAddress`, a method of the [X-Chain's API](../api/avm.md):

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :2,
    "method" :"avm.createAddress",
    "params" :{
        "username":"YOUR USERNAME HERE",
        "password":"YOUR PASSWORD HERE"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

Note that we make this request to `127.0.0.1:9650/ext/bc/X`.
The `bc/X` portion signifies that the request is being sent to the blockchain whose ID (or alias) is `X` (ie the X-Chain.)

The response should look like this:

```json
{
    "jsonrpc":"2.0",
    "id":2,
    "result" :{
        "address":"X-5TQr5hSAZ8ZKuSaYLg5sr4VwvcvwKZ1Mg"
    }
}
```

Your user now controls the address `X-5TQr5hSAZ8ZKuSaYLg5sr4VwvcvwKZ1Mg` on the X-Chain.
To tell apart addresses on different chains, the AVA convention is for an address to include the ID of the chain it exists on.
Hence, this address begins `X-`, denoting that it exists on the X-Chain.

## Acquire AVA Tokens

Now let's use the AVA Testnet Faucet to send some AVA tokens to this address.
The faucet dispenses 2500 nanoAVA (nAVA) for free.

Remember, this is only a test network, and AVA on this network has no value.
After all, anyone can get it for free by using the faucet.

Go to the [Testnet Faucet](https://faucet.ava.network/) and paste the address you just created to receive 2500 nAVA.

We can check an address's balance of a given asset by calling `avm.getBalance`, another method of the X-Chain's API.
Let's check that the 2500 nAVA was actually sent.

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :3,
    "method" :"avm.getBalance",
    "params" :{
        "address":"X-5TQr5hSAZ8ZKuSaYLg5sr4VwvcvwKZ1Mg",
        "assetID"  :"AVA"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

Note that AVA has the special ID `AVA`. 
Usually an asset ID is an alphanumeric string.

The response should look like this:

```json
{
    "jsonrpc":"2.0",
    "id"     :3,
    "result" :{
        "balance":2500
    }
}
```

## Send AVA

Now let's send some AVA tokens:

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :5,
    "method" :"avm.send",
    "params" :{
        "username"   :"YOUR USERNAME HERE",
        "password"   :"YOUR PASSWORD HERE",
        "assetID"    :"AVA",
        "amount"     :1000,
        "to"         :"X-FxgGhoAwg3dPTPhHEmjgi27ZPmvc8jQmj"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

`amount` specifies the number of \nAVA to send.
A \nAVA ($nanoAVA) is the smallest increment of AVA, and 1,000,000,000 nAVA == 1 AVA. 

When you send this request, the node will authenticate you using your username and password.
Then, it will look through all the private keys controlled by your user until it finds
enough nAVA to satisfy the request.

The response should look like this:

```json
{
    "jsonrpc":"2.0",
    "id"     :5,
    "result" :{
        "txID":"2QouvFWUbjuySRxeX5xMbNCuAaKWfbk5FeEa2JmoF85RKLk2dD"
    }
}
```

`txID` is the ID of the send transaction we just sent to the network. It will be different for every invocation of `send`.

## Checking the Transaction Status

This send transaction will only take a second or two to finalize. We can check its status with `avm.getTxStatus`:

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :6,
    "method" :"avm.getTxStatus",
    "params" :{
        "txID":"2QouvFWUbjuySRxeX5xMbNCuAaKWfbk5FeEa2JmoF85RKLk2dD"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

The response should look like this:

```json
{
    "jsonrpc":"2.0",
    "id"     :6,
    "result" :{
        "status":"Accepted"
    }
}
```

You might also see that `status` is `Pending` if the network has not yet finalized the transaction.

Once you see that the transaction is `Accepted`, check the balance of the `to` address to see that it has the AVA we sent:

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :7,
    "method" :"avm.getBalance",
    "params" :{
        "address":"X-FxgGhoAwg3dPTPhHEmjgi27ZPmvc8jQmj",
        "assetID"  :"AVA"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

The response should be:

```json
{
    "jsonrpc":"2.0",
    "id"     :7,
    "result" :{
        "balance":1000
    }
}
```

In the same fashion, we could check `X-5TQr5hSAZ8ZKuSaYLg5sr4VwvcvwKZ1Mg` to see that AVA we sent was deducted from its balance.

## Validate the Default Subnet (Stake)

[Subnets](../core-concepts/overview.md#what-are-subnets) are a powerful feature of the AVA network.
A Subnet is a set of validators that work to achieve consensus on a set of blockchains.

The Default Subnet is inherent to the AVA network, and it validates AVA's [built-in blockchains](../core-concepts/overview.md#built-in-blockchains). AVA uses Proof-of-Stake, so to become a validator one needs to provide a stake, or bond, in AVA tokens.

Let's add your node to the Default Subnet.

### Create a P-Chain Account

The P-Chain (Platform Chain) manages metadata about the AVA network, including which nodes belong to which Subnets.
The P-Chain uses an account model, so in order to validate the Default Subnet you'll need to first create an account on the P-Chain.
To do so, call [`platform.createAccount`](../api/platform.md#platformcreateaccount):

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.createAccount",
    "params": {
    	"username":"YOUR USERNAME HERE",
    	"password":"YOUR PASSWORD HERE"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

Note that this API call is sent to the P-Chain (`127.0.0.1:9650/ext/P`) rather than the X-Chain (`127.0.0.1:9650/ext/X`) like previous API calls.

The response contains the address of your new account:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "address": "Bg6e45gxCUTLXcfUuoy3go2U6V3bRZ5jH"
    },
    "id": 1
}
```

### Fund Your P-Chain Account

As mentioned before, in order validate the Default Subnet, you need to stake some AVA tokens.
Right now, your P-Chain account has no AVA.
AVA tokens are transferrable between the X-Chain (where you sent funds with the faucet) and the P-Chain.
Let's send some AVA to your P-Chain account from the X-Chain.
The minimum stake amount is 10,000 nAVA, so make sure you have at least this much AVA in your X-Chain addresses.
(If you need more, use the faucet a few more times.)

The first step in transferring AVA from the X-Chain to P-Chain is to call `avm.exportAVA`:

```json
curl -X POST --data '{
    "jsonrpc":"2.0",
    "id"     :1,
    "method" :"avm.exportAVA",
    "params" :{
        "to":"Bg6e45gxCUTLXcfUuoy3go2U6V3bRZ5jH",
        "amount": 10000,
    	"username":"YOUR USERNAME HERE",
    	"password":"YOUR PASSWORD HERE"
    }
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/X
```

The response contains the transaction ID.
As before, you can check the transaction's status by calling `avm.getTxStatus`.

The second and final step is to call `platform.importAVA`:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.importAVA",
    "params": {
    	"username":"YOUR USERNAME HERE",
    	"password":"YOUR PASSWORD HERE",
		"to":"Bg6e45gxCUTLXcfUuoy3go2U6V3bRZ5jH",
		"payerNonce":1
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/P
```

`payerNonce` is the next unused nonce of your P-Chain account, which is 1 since your P-Chain account has not sent any transactions.

The response contains the transaction:

This call returns the transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "tx": "1117xBwcr5fo1Ch4umyzjYgnuoFhSwBHdMCam2wRe8SxcJJvQRKSmufXM8aSqKaDmX4TjvzPaUbSn33TAQsbZDhzcHEGviuthncY5VQfUJogyMoFGXUtu3M8NbwNhrYtmSRkFdmN4w933janKvJYKNnsDMvMkmasxrFj8fQxE6Ej8eyU2Jqj2gnTxU2WD3NusFNKmPfgJs8DRCWgYyJVodnGvT43hovggVaWHHD8yYi9WJ64pLCvtCcEYkQeEeA5NE8eTxPtWJrwSMTciHHVdHMpxdVAY6Ptr2rMcYSacr8TZzw59XJfbQT4R6DCsHYQAPJAUfDNeX2JuiBk9xonfKmGcJcGXwdJZ3QrvHHHfHCeuxqS13AfU"
    },
    "id": 1
}
```

Which you can issue to the P-Chain by calling `platform.issueTx`:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.issueTx",
    "params": {
    	"tx":"1117xBwcr5fo1Ch4umyzjYgnuoFhSwBHdMCam2wRe8SxcJJvQRKSmufXM8aSqKaDmX4TjvzPaUbSn33TAQsbZDhzcHEGviuthncY5VQfUJogyMoFGXUtu3M8NbwNhrYtmSRkFdmN4w933janKvJYKNnsDMvMkmasxrFj8fQxE6Ej8eyU2Jqj2gnTxU2WD3NusFNKmPfgJs8DRCWgYyJVodnGvT43hovggVaWHHD8yYi9WJ64pLCvtCcEYkQeEeA5NE8eTxPtWJrwSMTciHHVdHMpxdVAY6Ptr2rMcYSacr8TZzw59XJfbQT4R6DCsHYQAPJAUfDNeX2JuiBk9xonfKmGcJcGXwdJZ3QrvHHHfHCeuxqS13AfU"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/P
```

Now check the balance of your P-Chain account by calling `platform.getAccount`:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getAccount",
    "params":{
    	"address":"Bg6e45gxCUTLXcfUuoy3go2U6V3bRZ5jH"
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/bc/P
```

The response should look like this:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "address": "Bg6e45gxCUTLXcfUuoy3go2U6V3bRZ5jH",
        "nonce": "1",
        "balance": "10000"
    },
    "id": 1
}
```

Great! Now your P-Chain account has enough AVA tokens to provide a stake.

### Get Your Node's ID

Your node is uniquely identified by the TLS certificate generated earlier by running `genStaker.sh`.
To get your node's ID, call [`admin.getNodeID:`](../api/admin.md#admingetnodeid)

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "admin.getNodeID",
    "params":{},
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/admin
```

The response contains your node's ID:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "nodeID": "ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK"
    },
    "id": 1
}
```

### Create the Unsigned Transaction

To add your node to the Default Subnet, you'll issue a transaction to the P-Chain.
When you add a node to the Default Subnet, you specify:

* `id`: The node ID of the node being added
* `payerNonce`: The next unused nonce of the account providing the transaction fee (the fee is currently 0) and staked AVA tokens.
* `destination`: The address of the account the AVA (along with a validator reward) will be returned to when the node is done validating.
* `startTime`: The Unix time the node will start validating the Default Subnet.
* `endTime`: The Unix time the node will stop validating the Default Subnet.

The place you're most likely to get tripped up is `startTime` and `endTime`.

`startTime` must be in the future relative to the time this transaction is issued.
To get the current Unix time, see [here.](https://www.unixtimestamp.com/)
Below, we use the shell command `date` to compute the Unix time 10 minutes from now.

`endTime` must be between 1 day (the minimum staking period) and 365 days (the maximum staking period) ahead of `startTime`.
Below, we use the shell command `date` to compute the Unix time 2 days from now.

(Note: If you're on a Mac, replace  `$(date` with `$(gdate`. If you don't have `gdate` installed, do `brew install coreutils`.)

Now let's create the unsigned transaction:

```sh
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.addDefaultSubnetValidator",
    "params": {
    	"id":"ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK",
    	"payerNonce":2,
    	"destination":"Bg6e45gxCUTLXcfUuoy3go2U6V3bRZ5jH",
    	"startTime":'$(date --date="10 minutes" +%s)',
    	"endTime":'$(date --date="2 days" +%s)',
    	"stakeAmount":10000
    },
    "id": 1
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response has the unsigned transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "unsignedTx": "111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLphtYaxdM1EA3Aj4yej8ABSfmjb9NMrtxQac9cnWwCER7GHSzFULB25hAtzGtJ8XhsrKcvtpAM8FwjRzg3Bg1q6V8GTKGMC219bYMETS48GMFGh4nts1Jsf246rjZ26r1Vyok8MdnoaxjQWR6cKq"
    },
    "id": 1
}
```

### Sign the Transaction

We need to sign this transaction with the key of the account that is paying the transaction fee and providing the staked tokens.

To do so, call `platform.Sign`.

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.sign",
    "params": {
    	"tx":"111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLphtYaxdM1EA3Aj4yej8ABSfmjb9NMrtxQac9cnWwCER7GHSzFULB25hAtzGtJ8XhsrKcvtpAM8FwjRzg3Bg1q6V8GTKGMC219bYMETS48GMFGh4nts1Jsf246rjZ26r1Vyok8MdnoaxjQWR6cKq",
    	"signer":"Bg6e45gxCUTLXcfUuoy3go2U6V3bRZ5jH",
    	"username":"YOUR USERNAME HERE",
    	"password":"YOUR PASSWORD HERE"
    },
    "id": 2
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

This returns the signed transaction:

```json
{
    "jsonrpc": "2.0",
    "result": {
        "Tx": "111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLphtYaxdM1EA3Aj4yej8ABSfmjb9NMrtxQac9cnWwCER7GHSzFULB4RoAjStfe26qQxhS91KvCCX3WBLmpyvNXHzgWk3uJP45cPv15RHGymFboPUcxNTwGij1NgQpKPcL4YxcDnKvNjrcQzKiXAz"
    },
    "id": 2
}
```

### Issue the Transaction

Finally, we issue the transaction to the network by calling [`platform.issueTx`](../api/platform.md#platformissuetx)

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.issueTx",
    "params": {
    	"tx":"111fRKBNoBhBfeGvBzvz6r9dZUKbEnUypM6tjiSyYrWM4ojSTuL2Syxv8cFLphtYaxdM1EA3Aj4yej8ABSfmjb9NMrtxQac9cnWwCER7GHSzFULB4RoAjStfe26qQxhS91KvCCX3WBLmpyvNXHzgWk3uJP45cPv15RHGymFboPUcxNTwGij1NgQpKPcL4YxcDnKvNjrcQzKiXAz"
    },
    "id": 3
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

Look out for errors printed by the node, which may indicate that your `startTime` is in the past, for example.
If so, try building and issuing the transaction again with corrected arguments.

### Verify Success

Now we can call [`platform.getPendingValidators`](../api/platform.md#platformgetpendingvalidators) to verify that the node has been added to the pending validator set of the Default Subnet.
A Subnet's pending validator set contains the nodes that are slated to start validating in the future.

If we don't provide any arguments, this method returns the pending validator set of the Default Subnet:

```json
curl -X POST --data '{
    "jsonrpc": "2.0",
    "method": "platform.getPendingValidators",
    "params": {},
    "id": 4
}' -H 'content-type:application/json;' 127.0.0.1:9650/ext/P
```

The response should confirm that the node with ID `ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK` will start validating the Default Subnet in the future.

```json
{
    "jsonrpc": "2.0",
    "result": {
        "validators": [
            {
                "id": "ARCLrphAHZ28xZEBfUL7SVAmzkTZNe1LK",
                "startTime": "1584021450",
                "endtime": "1584121156",
                "stakeAmount": "10000",
            }
        ]
    },
    "id": 4
}
```

Awesome! Now just wait until the time reaches `startTime`.
When it does, your node will start validating the Default Subnet.
You can verify this by calling [`platform.getCurrentValidators.`](../api/platform.md#platformgetcurrentvalidators) 

When the time reaches `endTime`, the P-Chain account that you specified in `destination` will receive back the staked AVA tokens as well as a validator reward.
Then, if you want, you can rejoin the Default Subnet.

Now that your node is validating the Default Subnet, please leave it running until at least `endTime`.
We want to have as many nodes as possible participating in consensus in order to stress test the AVA network and achieve true decentralization.
Note that in the future you will only receive a validator reward if your node is sufficiently responsive (ie online) while validating.

## Next Steps

- [Learn more about AVA](../core-concepts/overview.md)
- [Create and trade a new asset](../tutorials/fixed-cap-asset.md)
- [Create a new blockchain](../tutorials/create-a-blockchain.md)
- [Create a new, custom validator set (Subnet)](../tutorials/create-a-subnet.md)
- [Explore our APIs](../api/intro-apis.md)
- [Explore reference materials](../references/cryptographic-primitives.md)
- [Read the whitepaper and other papers](../papers/index.md)