# confirm-solana-transaction

Many solana client sends a transaction to the network and then wait for the status of the transaction signature to the confirm the transaction. During network congestions, the transaction may be dropped and the status will never return in the websocket subscription or will be null in the rpc call.

A more robust method is to record the block hash and the lastValidBlockHeight from the `getLatestBlockHash` call, set the recent block hash on the transaction to that block hash, send the transaction to the network. After transaction is sent, check what is the current valid block height. If the current valid block height is higher than the lastValidBlockHeight, the transaction has been dropped by the network and a new transaction with a different block hash can be safely retry, otherwise the transaction may still be included in a block and therefore there is still a chance that the transaction signature status can still return.

**Note**: if the same transaction is sent multiple times to the network, only one of the them will be executed (this is a guarantee by the network).
