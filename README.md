# Pepinals

A minter and protocol for inscriptions on Pepecoin.

## Setup

Install dependencies:

```bash
npm install
cd bitcore-lib-pepe
npm install
cd ..
```

Create a `.env` file with your node information:

```
NODE_RPC_URL=http://<ip>:<port>
NODE_RPC_USER=<username>
NODE_RPC_PASS=<password>
TESTNET=false
FEE_PER_KB=18000000
```

**Note**: The `FEE_PER_KB` is required and sets the miner fee rate in satoshis per KB. The default example is 18000000 satoshis (0.18 PEPE per KB). Adjust based on network conditions.

## Funding

Generate a new `.wallet.json` file:

```
node pepinals.js wallet new
```

Then send PEPE to the address displayed. Once sent, sync your wallet:

```
node pepinals.js wallet sync
```

If you are minting a lot, you can split up your UTXOs:

```
node pepinals.js wallet split <count>
```

When you are done minting, send the funds back:

```
node pepinals.js wallet send <address> <optional amount>
```

Check your wallet balance:

```
node pepinals.js wallet balance
```

## Minting

From file:

```
node pepinals.js mint <address> <path>
```

From data:

```
node pepinals.js mint <address> <content type> <hex data>
```

Examples:

```
node pepinals.js mint PpB1ocks3ozcti7m5a3i2wViSuFAchLm3n pepe.jpeg
```

```
node pepinals.js mint PpB1ocks3ozcti7m5a3i2wViSuFAchLm3n "text/plain;charset=utf8" 52696262697421
```

**Note**: Please use a fresh wallet to mint to with nothing else in it until proper wallet for pepinals support comes. You can get a paper wallet here.

## PRC-20 Token Operations

### Deploy a PRC-20 token

```
node pepinals.js prc-20 deploy <address> <ticker> <max supply> <limit per mint>
```

Example:

```
node pepinals.js prc-20 deploy PpB1ocks3ozcti7m5a3i2wViSuFAchLm3n PEPE 21000000 1000
```

### Mint PRC-20 tokens

```
node pepinals.js prc-20 mint <address> <ticker> <amount> [repeat count]
```

Example:

```
node pepinals.js prc-20 mint PpB1ocks3ozcti7m5a3i2wViSuFAchLm3n PEPE 1000 1
```

### Transfer PRC-20 tokens

```
node pepinals.js prc-20 transfer <address> <ticker> <amount> [repeat count]
```

Example:

```
node pepinals.js prc-20 transfer PpB1ocks3ozcti7m5a3i2wViSuFAchLm3n PEPE 500 1
```

## Bulk Minting

For automated bulk minting of PRC-20 tokens:

```
node pepinals.js bulk-mint <address> <ticker> <max supply> <limit per batch> [wait minutes]
```

Example:

```
node pepinals.js bulk-mint PpB1ocks3ozcti7m5a3i2wViSuFAchLm3n PEPE 21000000 1000 5
```

This will:
- Mint `limit per batch` tokens immediately
- Wait `wait minutes` (default 5) between batches
- Continue until stopped (Ctrl+C)
- Automatically handle retries for mempool chain errors

## Viewing

Start the server:

```
node pepinals.js server
```

The server runs on port 3008 by default (configurable via `SERVER_PORT` environment variable). Open your browser to:

```
http://localhost:3008/tx/<transaction_id>
```

## Protocol

The pepinals protocol allows any size data to be inscribed onto Pepecoin.

An inscription is defined as a series of push datas:

```
"ord"
OP_1
"text/plain; charset=utf8"
OP_0
"Ribbit!"
```

For pepinals, we introduce a couple extensions. First, content may spread across multiple parts:

```
"ord"
OP_2
"text/plain; charset=utf8"
OP_1
"Ribbit and "
OP_0
"ribbit ribbit!"
```

This content here would be concatenated as "Ribbit and ribbit ribbit!". This allows up to ~1500 bytes of data per transaction.

Second, P2SH is used to encode inscriptions.

There are no restrictions on what P2SH scripts may do as long as the redeem scripts start with inscription push datas.

And third, inscriptions are allowed to chain across transactions:

Transaction 1:

```
"ord"
OP_2
"text/plain; charset=utf8"
OP_1
"Ribbit and "
```

Transaction 2

```
OP_0
"ribbit ribbit!"
```

With the restriction that each inscription part after the first must start with a number separator, and number separators must count down to 0.

This allows indexers to know how much data remains.

## FAQ

### I'm getting ECONNREFUSED errors when minting

There's a problem with the node connection. Your `pepecoin.conf` file should look something like:

```
rpcuser=ape
rpcpassword=zord
rpcport=33875
server=1
listen=1
txindex=1
rpcallowip=127.0.0.1
```

Make sure `port` is not set to the same number as `rpcport`. Also make sure `rpcauth` is not set.

Your `.env` file should look like:

```
NODE_RPC_URL=http://127.0.0.1:33875
NODE_RPC_USER=ape
NODE_RPC_PASS=zord
TESTNET=false
FEE_PER_KB=18000000
```

### I'm getting "insufficient priority" errors when minting

The miner fee is too low. You can increase it by putting `FEE_PER_KB=30000000` (or higher) in your `.env` file or just wait it out. The default example is 18000000 satoshis (0.18 PEPE per KB) but spikes up when demand is high.

### I'm getting "too-long-mempool-chain" errors

This happens when you're trying to broadcast too many dependent transactions at once. The bulk-mint command handles this automatically with retries. For manual minting, wait for some transactions to confirm before minting more.

## License

MIT
