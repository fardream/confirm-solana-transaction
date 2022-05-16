# confirm-solana-transaction

Many solana client sends a transaction to the network and then wait for the status of the transaction signature to the confirm the transaction.

The classic way to do this is to check the status of the transaction signature, and see if that transaction's status matches the desired commitment level. This can be done in one shot with `sendAndConfirmTransaction` in the solana web3 library.

```typescript
const tx = ...some..transaction;
const signature = await sendAndConfirmTransaction(connection, tx, signers, options);
```

During network congestions, the transaction may be dropped and the status will never return in the websocket subscription or will be null in the rpc call. However, the `sendAndConfirmTransaction` method doesn't check for if the transaction is dropped or not. It simply waits for 60 seconds and will throw an error if time out. However, there is not way to tell if the transaction is dropped or not.

A more robust method is to record the block hash and the `lastValidBlockHeight` from the `getLatestBlockHash` call, set the recent block hash on the transaction to that block hash, and send the transaction to the network. After transaction is sent, check what is the current valid block height. If the current valid block height is higher than the lastValidBlockHeight, the transaction has been dropped by the network and a new transaction with a different block hash can be safely retry, otherwise the transaction may still be included in a block and therefore there is still a chance that the transaction signature status can still return.

**Note**: if the same transaction is sent multiple times to the network, only one of the them will be executed (this is a guarantee by the network).

This library does this:

```typescript
import {sendAndConfirmTransaction} from "@foonetic/confirm-solana-transaction";

const tx = ...some...transaction
const signature = await sendAndConfirmTransaction(connection, tx, signers, options, time_out);
```

Or if you have recorded the `lastValidBlockHeight`, you can simply call `confirmTransaction`.

**sendTransaction** on the solana client will automatically get the recent block hash but it doesn't expose the `lastValidBlockHeight`. This can be changed to

1. get the recent block hash and last valid block height.
2. set the block hash on the transaction, sign the transaction and serialize the data.
3. send the serialized transaction with `sendRawTransaction`.
