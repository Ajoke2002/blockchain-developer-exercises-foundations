# Week 3 Cryptography in Distributed Ledger Technologies

## Exercise - Create your first DLT transactions

In this exercise, we will create our own transactions via Overledger’s prepareTransactionRequest and executePreparedRequestTransaction APIs. The documentation for these endpoints can be found [here](https://docs.overledger.io/#operation/prepareTransactionRequest) and [here](https://docs.overledger.io/#operation/executePreparedRequestTransaction) respectively. 


### DLT Network Information

We will be interacting with the Bitcoin, Ethereum & XRP Ledger testnets. The relevant Overledger location objects are as follows:

1. `Location = {“technology”: “Bitcoin”, “network”: “Testnet”}`
   
2. `Location = {“technology”: “Ethereum”, “network”: “Ropsten Testnet”}`
   
3. `Location = {“technology”: “XRP Ledger”, “network”: “Testnet”}`

### Prerequisites

It is assumed that you have already setup your environment by following [these instructions](./Exercise1.md) and that you have completed the previous exercises that use Overledger to read data from the DLT networks.

### Creating Accounts

You will need to have DLT accounts to create transactions. The following example will create and console log new DLT accounts for all of the Bitcoin testnet, Ethereum Ropsten testnet and the XRP ledger testnet:

```
node examples/account-creation/generate-accounts.js
```

**NOTE that key pairs can be reused on different DLT networks including mainnets, so we recommend for you to only use these generated accounts for these tutorials.**

**TAKEAWAY: Do not mix the accounts that you use on testnets and mainnets!**

### Attaining Testnet Cryptocurrency

As we are interacting with permissionless DLT networks, we will have to pay a transaction fee to get our transactions accepted on the DLT networks. As we are interacting with testnets, the transaction fee will actually be paid in a test cryptocurrency (typically without monetary value). But the transaction fee system is still present on the testnets, in order to accurately simulate the mainnets. 

Therefore you will need to fund your addresses before you can send transactions from those addresses. To fund your addresses on testnets, we can request a faucet to provide us some funds. These services are named as faucets as they "drip" funds to users that require them. 

We list a few publicly available faucets options below:
- Ethereum Ropsten Testnet: https://faucet.ropsten.be/ or https://faucet.dimensions.network/
- XRP Ledger Testnet: https://xrpl.org/xrp-testnet-faucet.html
- Bitcoin Testnet: https://bitcoinfaucet.uo1.net/ or https://coinfaucet.eu/en/btc-testnet/ or https://testnet-faucet.mempool.co/ 

Note that faucets can be occasionally empty or can change frequently, if your having trouble with all of the ones listed above for your desired DLT, have a Google and let us know if you find a better one.

**IMPORTANT NOTE 1: For Bitcoin and Ethereum, note that the faucet funding transactions needs to be in a block of the corresponding blockchain before it can be used. Whereas XRP ledger funds will be available as soon as you see the generated information on the XRP ledger faucet screen.**

**IMPORTANT NOTE 2: For Ethereum, there are many different test networks available (e.g. Ropsten, Rinkeby, Kovan,...). Tokens issued on one test network cannot be used on another test network. So make sure that any faucet you use is a Ropsten testnet faucet.** 

**IMPORTANT NOTE 3:  For XRP Ledger testnet, we recommend using the DLT account generated by the faucet as it comes provided with testnet XRP.**


### Adding DLT accounts to the Environment File

To use these generated accounts (or any others), recall the *.env.example.* from Exercise 1. This file defines environment variables that our programs will later use. In Exercise 1, we set the USER_NAME, PASSWORD, CLIENT_ID and CLIENT_SECRET environment variables. In this class we still require those previous four variables but now we will also enter the following: 

- BITCOIN_PRIVATE_KEY: set this equal to the BitcoinAccount.privateKey parameter provided by the generate-accounts.js script
- ETHEREUM_PRIVATE_KEY: set this equal to the EthereumAccount.privateKey parameter provided by the generate-accounts.js script
- XRP_LEDGER_PRIVATE_KEY: set this equal to the [XRP ledger faucet](https://xrpl.org/xrp-testnet-faucet.html) generated secret
- BITCOIN_ADDRESS: set this equal to the BitcoinAccount.address provided by the generate-accounts.js script
- ETHEREUM_ADDRESS: set this equal to the EthereumAccount.address provided by the generate-accounts.js script
- XRP_LEDGER_ADDRESS: set this equal to the [XRP ledger faucet](https://xrpl.org/xrp-testnet-faucet.html) generated address

Therefore you will once again need to setup the *.env.enc* file as stated in Exercise 1. In particular, you need to duplicate the *.env.example* file and rename it to *.env*. Make sure to set the previous four parameters from Exercise 1 and the new six parameters from this class in *.env*. You will also need to once again encrypt the *.env* file. For this, run on your terminal (replace MY_PASSWORD for a password of your choice):

```
npm-run secure-env .env -s MY_PASSWORD
```

### Creating Transactions

This example will create transactions on the Bitcoin testnet, Ethereum Ropsten testnet and the XRP ledger test:

```
node examples/transaction-creation/submit-transactions.js password=MY_PASSWORD fundingTx=MY_BITCOIN_FUNDING_TX
```
Note that an extra command line parameter is required, which is a bitcoin transaction that you have received from the bitcoin faucet and includes an unspent transaction output that you have not yet used. 

You will see in the example script that we are using:

```
    await overledgerInstance.post(
      "/preparation/transaction",
      prepareTransactionRequest,
    );
```
to request that an Overledger standardised transaction is transformed into a transaction in the native DLT format ready for signing. We are subsequently using:

```
    await overledger.sign(
      overledgerRequestMetaData[count].location.technology.replace(/\s+/g, '-').toLowerCase(),
      prepareTransactionResponse[count].data,
    );
```
to sign the transaction and:

```
    await overledgerInstance.post(
      "/execution/transaction",
      executeTransactionRequest,
    );
```
to send the transaction to Overledger, who forwards it onto the DLT network.

The full details of this script is as follows. It firstly creates random DLT accounts to send the transactions to. Next the script makes sure that you provided a valid bitcoin funding transaction by fetching the given transaction and looping over it's UTXOs looking for a match to your Bitcoin address. If it doesn't find it, it means that either you have provided the wrong Bitcoin address to *.env*, or the provided transaction is not the funding transaction issued by the faucet, or the faucet is not working correctly. 

After this, the script constructs the origin and destination of each of your bitcoin, ethereum and XRP ledger transactions. Recall that an accounts based transaction origin is an address, whereas a UTXO based transaction origin is a UTXOid (transactionId:OutputIndex). Whereas the transaction destination of both accounts and UTXO transactions is an address. The script then sets the minimum value of BTC, ETH or XRP to be transferred via these transactions. 

Now the script has enough information to submit each transaction to the DLT network. Therefore the script enters a loop which completes three main steps for each DLT network. Firstly the script sends a standardised transaction to OVL to be prepared and receives the response in the DLT native data format. Secondly the script signs the native data form of the transaction. Thirdly the script sends the signed transaction to Overledger for it to be added onto the DLT network.


#### Overledger Execute Transaction API Response

The execute transaction response will contain a few main components:

1. Location: Each Overledger DLT data response includes a reference to the location (technology, network) of where this data was taken from. This helps with auditing.

2. Status: Overledger responses regarding blocks and transactions come with a status. Due to some DLTs having probabilistic finality of transactions/blocks and other DLTs having deterministic finality of transaction/blocks, the status object is used to indicate to the developer when the requested data is assumed to be final (therefore status.value = “SUCCESSFUL”) or not (therefore status.value=“PENDING”).
   
3. TransactionId: The id of the submitted signed transaction in the DLT native data format. This is the transaction id required for the OVL transaction Search APIs.

4. OverledgerTransactionId: An overledger specific transaction id used for tracking a transaction status and for linking transactions to a particular client created decentralised application. 

For parameter by parameter descriptions see the [documentation](https://docs.overledger.io/#operation/executePreparedRequestTransaction).


#### Challenges

##### Sending Transactions to Specific Addresses

Given the example `./examples/transaction-creation/submit-transactions.js` file, can you understand how to change this file to send transactions to specific addresses that you choose from each DLT network? How will you choose these addresses?

##### Sending Transactions for a Specific Amount

Given the example `examples/transaction-creation/submit-transactions.js` file, can you understand how to change this file to send specific amounts of your choosing? Do you have any limitations on the amounts that you choose and how can you modify the code to deal with this issue?

#### Troubleshooting
This class was tested in Ubuntu 20.04.2 LTS Release: 20.04 Codename: focal, with nvm version 0.35.3, and node version 16.3.0. 

#### Error: bad decrypt 

Description:

```
Secure-env :  ERROR OCCURED Error: error:06065064:digital envelope routines:EVP_DecryptFinal_ex:bad decrypt
```

Cause: the secure env package cannot decrypt the .env.enc file because the provided password was incorrect.

Solution: provide the password with which .env.enc was encrypted when running the script.

#### Error: .env.enc does not exist 

Description:

```
Secure-env :  ERROR OCCURED .env.enc does not exist.
```

Cause: You are missing the encrypted environment file in the folder that you are running from.

Solution: Return to the top level folder and encrypt .env as described in Exercise 1.

#### Error: Missing Password

Description:

```
Error: Please insert a password to decrypt the secure env file.
```

Cause: You did not include the password as a command line option.

Solution: Include the password as a command line option as stated in your terminal print out.

#### Error: bitcoin transaction failure

Status returned from Overledger regarding a bitcoin transaction execution:

```
{"value":"FAILED","code":"TXN1501","description":"-25: Missing inputs","message":"Missing inputs","timestamp":"2022-01-24T22:42:10.282652Z"}
```

Cause: You have either provided the wrong Bitcoin address to *.env*, or the comand line provided transaction is not the funding transaction issued by the faucet or the funding transaction has not yet be included in a block of the blockchain or the faucet is not working correctly. 

Solution: Firstly check that the provided fundingTx in the command line variables is correct. This is the most likely error. Check that this transaction has been included in a block of the blockchain. If this is both correct, check the bitcoin address and related private key is correct in the *.env* file, if it is not correct then you need to re-enter the bitcoin address and private key, re-encrypt the *.env* file and you need to request again funds from the bitcoin faucet to send the funds to the new address you added to the *.env* file. Finally if all these checks pass, assume the bitcoin faucet you choose is faulty and find another one.
