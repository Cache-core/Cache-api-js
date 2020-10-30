## Cache-API-JS: Javascript/Node.js interface (RPC/API)
Javascript/Node.js interface to the Cache cryptographic currency RPC/API.

There are three RPC servers built in to the three programs *cache-daemon*, *cache-wallet* and *cache-service*.
They can each be started with the argument `--help` to display command line options.

### cache-daemon
A node on the P2P network (daemon) with no wallet functions; console interactive. To launch:
```
$ ./cache-daemon
```
The default daemon RPC port is 40000 and the default P2P port is 39999.

### cache-service
A node on the P2P network (daemon) with wallet functions; console non-interactive. To launch, assuming that your `my.wallet` file is in the current directory:
```
$ ./cache-service --container-file my.wallet --container-password PSWD --local --bind-port 40001
```
The wallet functions RPC port is 40001. The default daemon P2P port is 39999. The default daemon RPC port is 40000. The `--local` option activates the daemon; otherwise, a remote daemon can be used.

### cache-wallet
A simple wallet; console interactive unless RPC server is running; requires access to a node daemon for full functionality. To launch, assuming that your `my.wallet` file is in the current directory:
```
$ ./cache-wallet --rpc-bind-port 40001 --wallet-file my --password PSWD
```
The wallet functions RPC port is 40001. By default the wallet connects with the daemon on port 40000. It is possible to run several instances simultaneously using different wallets and ports.

***

## Quick start for node.js
```
$ npm install cache-api-js
$ ./cache-daemon # launch the network daemon
$ ./cache-wallet --rpc-bind-port PORT --wallet-file my --password PSWD # launch the simple wallet
```
Create and run a test program.
```
$ node test.js
```
The test program could contain, for example, a payment via the simple wallet's RPC server
```
const CXCHE = require('cache-api-js')
const cxche = new CXCHE('http://localhost', '40001')

cxche.send([{
  address: 'cxche7aGfUged5Z8UWDzM4E6mBArHgLZ7Ad9n5qj6K2G7Dd7HBiMgMaX4gFEiSBpmoQMsWTwqFvpnUnuuUtcyqVQPE6krizA5D6',
  amount: 1234567
}])
.then((res) => { console.log(res) }) // display tx hash upon success
.catch((err) => { console.log(err) }) // display error message upon failure
```

***

## API
```
const CXCHE = require('cache-api-js')
const cxche = new CXCHE(host, walletRpcPort, daemonRpcPort, timeout)
```
cxche.rpc returns a promise, where *rpc* is any of the methods below:

* [Wallet RPC (must provide walletRpcPort)](#wallet)
  * cache-wallet
    * [Get height](#height)
    * [Get balance](#balance)
    * [Get messages](#messages)
    * [Get incoming payments](#payments)
    * [Get transfers](#transfers)
    * [Get number of unlocked outputs](#outputs)
    * [Reset wallet](#reset)
    * [Store wallet](#store)
    * [Optimize wallet](#optimize)
    * [Send transfers](#send)
  * cache-service
    * [Reset or replace wallet](#resetOrReplace)
    * [Get status](#status)
    * [Get balance](#getBalance)
    * [Create address](#createAddress)
    * [Delete address](#deleteAddress)
    * [Get addresses](#getAddresses)
    * [Get view secret Key](#getViewSecretKey)
    * [Get spend keys](#getSpendKeys)
    * [Get block hashes](#getBlockHashes)
    * [Get transaction](#getTransaction)
    * [Get unconfirmed transactions](#getUnconfirmedTransactions)
    * [Get transaction hashes](#getTransactionHashes)
    * [Get transactions](#getTransactions)
    * [Send transaction](#sendTransaction)
    * [Create delayed transaction](#createDelayedTransaction)
    * [Get delayed transaction hashes](#getDelayedTransactionHashes)
    * [Delete delayed transaction](#deleteDelayedTransaction)
    * [Send delayed transaction](#sendDelayedTransaction)
    * [Get incoming messages from transaction extra field](#getMessagesFromExtra)
* [Daemon RPC (must provide daemonRpcPort)](#daemon)
  * [Get info](#info)
  * [Get index](#index)
  * [Get count](#count)
  * [Get currency ID](#currencyId)
  * [Get block hash by height](#blockHashByHeight)
  * [Get block header by height](#blockHeaderByHeight)
  * [Get block header by hash](#blockHeaderByHash)
  * [Get last block header](#lastBlockHeader)
  * [Get block](#block)
  * [Get blocks](#blocks)
  * [Get block template](#blockTemplate)
  * [Submit block](#submitBlock)
  * [Get transaction](#transaction)
  * [Get transactions](#transactions)
  * [Get transaction pool](#transactionPool)
  * [Send raw transaction](#sendRawTransaction)

### <a name="wallet"></a>Wallet RPC (must provide walletRpcPort)

#### <a name="height"></a>Get height (cache-wallet)
```
cxche.height() // get last block height
```
#### <a name="balance">Get balance (cache-wallet)
```
cxche.balance() // get wallet balances
```
#### <a name="messages">Get messages (cache-wallet)
```
const opts = {
  firstTxId: FIRST_TX_ID, // (integer, optional), ex: 10
  txLimit: TX_LIMIT // maximum number of messages (integer, optional), ex: 10
}
cxche.messages(opts) // opts can be omitted
```
#### <a name="payments">Get incoming payments (cache-wallet)
```
const paymentId = PAYMENT_ID // (64-digit hex string, required), ex: '0ab1...3f4b'
cxche.payments(paymentId)
```
#### <a name="transfers">Get transfers (cache-wallet)
```
cxche.transfers() // gets all transfers
```
#### <a name="outputs">Get number of unlocked outputs (cache-wallet)
```
cxche.outputs() // gets outputs available as inputs for a new transaction
```
#### <a name="reset">Reset wallet (cache-wallet)
```
cxche.reset() // discard wallet cache and resync with block chain
```
#### <a name="store">Store wallet (cache-wallet)
```
cxche.store() // save wallet cache to disk
```
#### <a name="optimize">Optimize wallet (cache-wallet)
```
cxche.optimize() // combines many available outputs into a few by sending to self
```
#### <a name="send">Send transfers (cache-wallet)
```
const transfers = [{ address: ADDRESS, amount: AMOUNT, message: MESSAGE }, ...] // ADDRESS = destination address (string, required), AMOUNT = raw CXCHE (integer, required), MESSAGE = transfer message to be encrypted (string, optional)
const opts = {
  transfers: transfers, // (array, required), ex: [{ address: 'cxche7a...', amount: 1000, message: 'refund' }]
  fee: FEE, // (raw CXCHE integer, optional, default is minimum required), ex: 10
  anonimity: MIX_IN, // input mix count (integer, optional, default 0), ex: 6
  paymentId: PAYMENT_ID, // (64-digit hex string, optional), ex: '0ab1...3f4b'
  unlockHeight: UNLOCK_HEIGHT // block height to unlock payment (integer, optional), ex: 12750
}
cxche.send(opts)
```
#### <a name="resetOrReplace">Reset or replace wallet (cache-service)
```
const viewSecretKey = VIEW_SECRET_KEY // (64-digit hex string, optional), ex: '0ab1...3f4b'
cxche.resetOrReplace(viewSecretKey) // If no key, wallet is re-synced. If key, a new address is created from the key for a new wallet.
```
#### <a name="status">Get status (cache-service)
```
cxche.status()
```
#### <a name="getBalance">Get balance (cache-service)
```
const address = ADDRESS // (string, required), ex: 'cxche7a...'
cxche.getBalance(address)
```
#### <a name="createAddress">Create address (cache-service)
```
cxche.createAddress()
```
#### <a name="deleteAddress">Delete address (cache-service)
```
const address = ADDRESS // (string, required), ex: 'cxche7a...'
cxche.deleteAddress(address)
```
#### <a name="getAddresses">Get addresses (cache-service)
```
cxche.getAddresses()
```
#### <a name="getViewSecretKey">Get view secret key (cache-service)
```
cxche.getViewSecretKey()
```
#### <a name="getSpendKeys">Get spend keys (cache-service)
```
const address = ADDRESS // (string, required), ex: 'cxche7a...'
cxche.getSpendKeys(address)
```
#### <a name="getBlockHashes">Get block hashes (cache-service)
```
const firstBlockIndex = FIRST_BLOCK_INDEX // index of first block (integer, required), ex: 12750
const blockCount = BLOCK_COUNT // number of blocks to include (integer, required), ex: 30
cxche.getBlockHashes(firstBlockIndex, blockCount)
```
#### <a name="getTransaction">Get transaction (cache-service)
```
const hash = HASH // (64-digit hex string, required), ex: '0ab1...3f4b'
cxche.getTransaction(hash) // get transaction details given hash
```
#### <a name="getUnconfirmedTransactions">Get unconfirmed transactions (cache-service)
```
const addresses = [ADDRESS1, ADDRESS2, ...] // ADDRESS = address string; address to include
cxche.getUnconfirmedTransactions(addresses) // addresses can be omitted
```
#### <a name="getTransactionHashes">Get transactionHashes (cache-service)
```
const opts = { // either blockHash or firstBlockIndex is required
  blockHash: BLOCK_HASH, // hash of first block (64-digit hex string, see comment above), ex: '0ab1...3f4b'
  firstBlockIndex: FIRST_BLOCK_INDEX, // index of first block (integer, see comment above), ex: 12750
  blockCount: BLOCK_COUNT, // number of blocks to include (integer, required), ex: 30
  addresses: [ADDRESS, ...], filter (array of address strings, optional), ex: ['cxche7a...']
  paymentId: PAYMENT_ID // filter (64-digit hex string, optional), ex: '0ab1...3f4b'
}
cxche.getTransactionHashes(opts)
```
#### <a name="getTransactions">Get transactions (cache-service)
```
const opts = { // either blockHash or firstBlockIndex is required
  blockHash: BLOCK_HASH, // hash of first block (64-digit hex string, see comment above), ex: '0ab1...3f4b'
  firstBlockIndex: FIRST_BLOCK_INDEX, // index of first block (integer, see comment above), ex: 12750
  blockCount: BLOCK_COUNT, // number of blocks to include (integer, required), ex: 30
  addresses: [ADDRESS, ...], filter (array of address strings, optional), ex: ['cxche7a...']
  paymentId: PAYMENT_ID // filter (64-digit hex string, optional), ex: '0ab1...3f4b'
}
cxche.getTransactions(opts)
```
#### <a name="sendTransaction">Send transaction (cache-service)
```
const transfers = [{ address: ADDRESS, amount: AMOUNT, message: MESSAGE }, ...] // ADDRESS = destination address (string, required), AMOUNT = raw CXCHE (integer, required), MESSAGE = transfer message to be encrypted (string, optional)
const addresses = [ADDRESS1, ADDRESS2, ...] // ADDRESS = source address string; address in wallet to take funds from
const opts = {
  transfers: transfers, // (array, required), ex: [{ address: 'cxche7a...', amount: 1000, message: 'tip' }]
  addresses: addresses, // (array, optional), ex: ['cxche7a...', 'cxche7a...']
  changeAddress: ADDRESS, // change return address (address string, optional if only one address in wallet or only one source address given), ex: 'cxche7a...'
  paymentId: PAYMENT_ID, // filter (64-digit hex string, optional), ex: '0ab1...3f4b'
  anonimity: MIX_IN, // input mix count (integer, optional, default 0), ex: 6
  fee: FEE, // (raw CXCHE integer, optional, default is minimum required), ex: 10
  unlockHeight: UNLOCK_HEIGHT, // block height to unlock payment (integer, optional), ex: 12750
  extra: EXTRA // (variable length string, optional), ex: '123abc'
}
cxche.sendTransaction(opts)
```
#### <a name="createDelayedTransaction">Create delayed transaction (cache-service)
```
const transfers = [{ address: ADDRESS, amount: AMOUNT, message: MESSAGE }, ...] // ADDRESS = destination address (string, required), AMOUNT = raw CXCHE (integer, required), MESSAGE = transfer message to be encrypted (string, optional)
const addresses = [ADDRESS1, ADDRESS2, ...] // ADDRESS = source address string; address in wallet to take funds from
const opts = {
  transfers: transfers, // (array, required), ex: [{ address: 'cxche7a...', amount: 1000, message: 'tip' }]
  addresses: addresses, // (array, optional), ex: ['cxche7a...', 'cxche7a...']
  changeAddress: ADDRESS, // change return address (address string, optional if only one address in wallet or only one source address given), ex: 'cxche7a...'
  paymentId: PAYMENT_ID, // filter (64-digit hex string, optional), ex: '0ab1...3f4b'
  anonimity: MIX_IN, // input mix count (integer, optional, default 0), ex: 6
  fee: FEE, // (raw CXCHE integer, optional, default is minimum required), ex: 10
  unlockHeight: UNLOCK_HEIGHT, // block height to unlock payment (integer, optional), ex: 12750
  extra: EXTRA // (variable length string, optional), ex: '123abc'
}
cxche.createDelayedTransaction(opts) // create but do not send transaction
```
#### <a name="getDelayedTransactionHashes">Get delayed transaction hashes (cache-service)
```
cxche.getDelayedTransactionHashes()
```
#### <a name="deleteDelayedTransaction">Delete delayed transaction (cache-service)
```
const hash = HASH // (64-digit hex string, required), ex: '0ab1...3f4b'
cxche.deleteDelayedTransaction(hash)
```
#### <a name="sendDelayedTransaction">Send delayed transaction (cache-service)
```
const hash = HASH // (64-digit hex string, required), ex: '0ab1...3f4b'
cxche.sendDelayedTransaction(hash)
```
#### <a name="getMessagesFromExtra">Get incoming messages from transaction extra field (cache-service)
```
const extra = EXTRA // (hex string, required), ex: '0199...c3ca'
cxche.getMessagesFromExtra(extra)
```
### <a name="daemon">Daemon RPC (must provide daemonRpcPort)

#### <a name="info">Get info
```
cxche.info() // get information about the block chain, including next block height
```
#### <a name="index">Get index
```
cxche.index() // get next block height
```
#### <a name="count">Get count
```
cxche.count() // get next block height
```
#### <a name="currencyId">Get currency ID
```
cxche.currencyId()
```
#### <a name="blockHashByHeight">Get block hash by height
```
const height = HEIGHT // (integer, required), ex: 12750
cxche.blockHashByHeight(height) // get block hash given height
```
#### <a name="blockHeaderByHeight">Get block header by height
```
const height = HEIGHT // (integer, required), ex: 12750
cxche.blockHeaderByHeight(height) // get block header given height
```
#### <a name="blockHeaderByHash">Get block header by hash
```
const hash = HASH // (64-digit hex string, required), ex: '0ab1...3f4b'
cxche.blockHeaderByHash(hash) // get block header given hash
```
#### <a name="lastBlockHeader">Get last block header
```
cxche.lastBlockHeader()
```
#### <a name="block">Get block
```
const hash = HASH // (64-digit hex string, required), ex: '0ab1...3f4b'
cxche.block(hash)
```
#### <a name="blocks">Get blocks
```
const height = HEIGHT // (integer, required), ex: 12750
cxche.blocks(height) // returns 31 blocks up to and including HEIGHT
```
#### <a name="blockTemplate">Get block template
```
const address = ADDRESS // destination address (string, required), ex: 'cxche7a...'
const reserveSize = RESERVE_SIZE // bytes to reserve in block for work, etc. (integer < 256, optional, default 14), ex: 255
const opts = {
  address: address,
  reserveSize: reserveSize
}
cxche.blockTemplate(opts)
```
#### <a name="submitBlock">Submit block
```
const block = BLOCK // block blob (hex string, required), ex: '0300cb9eb...'
cxche.submitBlock(block)
```
#### <a name="transaction">Get transaction
```
const hash = HASH // (64-digit hex string, required), ex: '0ab1...3f4b'
cxche.transaction(hash)
```
#### <a name="transactions">Get transactions
```
const arr = [HASH1, HASH2, ...] // (array of 64-digit hex strings, required), ex: ['0ab1...3f4b']
cxche.transactions(arr)
```
#### <a name="transactionPool">Get transaction pool
```
cxche.transactionPool()
```
#### <a name="sendRawTransaction">Send raw transaction
```
const transaction = TRANSACTION // transaction blob (hex string, required), ex: ''01d86301...'
cxche.sendRawTransaction(transaction)
```
