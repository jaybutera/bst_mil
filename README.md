## UTXO Binary Search Tree
A binary search tree data structure encoded in UTXOs. Built from the
[Bitforest](http://rboutaba.cs.uwaterloo.ca/Papers/Conferences/2018/DongCNSM18.pdf)
design. Implemented in mil, runs on the Themelio blockchain.

Covenant scripts constrain what transactions can spend a UTXO (unspent-transaction-output).
Unlike UTXO scripts in Bitcoin, themelio scripts can inspect the spender transaction,
as well as data about the script's own UTXO.

A cool consequence is that this enables a sort of "self-replication" - where a UTXO can
require that a spending transaction has the same script in its UTXO. Such a self-replicating
script can enforce that transactions adhere to a data structure, such as a binary search tree.

### Try it out
Get the [mil compiler](https://github.com/themeliolabs/mil).

The txs.json file provides a transaction to test spending the script's output.
Use the `--test-txs` flag to pass in the file containing transactions to test.
```
$ mil bst.mil --test-txs.json
```

txs.json contains two test transactions. The first spends the second UTXO and
has a bigger index, so it will succeed because spender_index > self_index.
You should see that execution succeeded on the transaction, and the final state
of the VM will be an `Int(1)` on the stack. That means true, and the spend is
allowed.

The second test transaction spends the first UTXO, but still has a bigger index.
It will fail because again, spender_index > self_index, but now the spender is
trying to become the left child, which would break the BST order. Execution will
still succeed, but the final value on the stack will be `Int(0)`, meaning the
spend is not allowed.

### Whats happening
The script, `bst.mil`, is basically checking that 4 requirements are met:
1. The spender's first two outputs perpetuate the covenant script (those script hashes = self COV-HASH).
2. Those two outputs of the spender tx contain a matching index, the "spender-index".
3. If the spender is spending the first UTXO of *this* tx, the spender-index is less than self-index. Otherwise it's greater. This is the BST ordering property.
4. The spender provides a name (a value in our BST map), whose hash is its index. This way, the path to a node in the BST can be known by its name.
