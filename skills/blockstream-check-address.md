---
name: Check a Bitcoin address balance and history
description: >-
  Use the Blockstream Esplora HTTP API to look up a Bitcoin address, read its
  confirmed and mempool balance, page through its transaction history, and list its
  unspent outputs. No authentication required.
api: openapi/blockstream-esplora-openapi.yml
operations: [getAddress, getAddressTransactions, getAddressTransactionsChain, getAddressUtxo]
---

# Check a Bitcoin address balance and history

The Esplora API is public and needs no API key. Base URL: `https://blockstream.info/api`
(use `https://blockstream.info/testnet/api` for testnet). All amounts are in satoshis.

## Steps

1. **Look up the address.** Call `getAddress` (`GET /address/{address}`). The response
   carries `chain_stats` and `mempool_stats`, each with `funded_txo_sum` and
   `spent_txo_sum`. Confirmed balance = `chain_stats.funded_txo_sum - chain_stats.spent_txo_sum`;
   add the mempool delta for the pending balance.

2. **List recent transactions.** Call `getAddressTransactions`
   (`GET /address/{address}/txs`) for the newest activity (mempool + latest confirmed).

3. **Page older history.** Call `getAddressTransactionsChain`
   (`GET /address/{address}/txs/chain/{last_seen_txid}`), passing the `txid` of the
   last transaction from the previous page as `last_seen_txid`. Returns 25 confirmed
   transactions per page (cursor pagination). Repeat until fewer than 25 come back.

4. **List spendable UTXOs.** Call `getAddressUtxo` (`GET /address/{address}/utxo`) to
   get the unspent outputs with their `value` and confirmation `status`.

## Rules

- Errors come back as an HTTP status with a plain-text body (e.g. 404 "not found");
  there is no problem+json envelope. See `errors/blockstream-problem-types.yml`.
- The public instance applies operational rate limiting; retry with backoff on 429.
- Do not confuse networks: an address only resolves on the network whose base-URL
  prefix you use.
