## Self-enforcing Binary Search Tree
A binary search tree data structure encoded in UTXOs. Order is enforced within
the output scripts. Built from the
[Bitforest](http://rboutaba.cs.uwaterloo.ca/Papers/Conferences/2018/DongCNSM18.pdf)
design. Implemented in mil, runs on the Themelio blockchain.

The "self-enforcing" aspect of the scripts means that not only is a transaction
that spend this UTXO constrained to the BST ordering, but every transaction
that follows will be constrained in the same way. This is accomplished because
the bitforest script requires that the spending transaction's output scripts
are indeed the same bitforest script. It is a recursively perpetuating script.
Any transaction that spends the script must make it again spendable in its own output.

### Try it out
Get the [mil compiler](https://github.com/themeliolabs/mil).

The txs.json file provides a transaction to test spending the script's output.
Use the `--test-txs` flag to pass in the file containing transactions to test.
```
$ mil bitforest_left.mil --test-txs.json
```

You should see that execution succeeded on the transaction, and the final state
of the VM will be an `Int(1)` on the stack. That means true, and the spend is
allowed.

### Whats happening
There are two covenant scripts - bitforest left and right. Left is to be
attached to the first output of a transaction; right the second. Each checks
for its own hash in the spending script to ensure the logical structure of the
BST is perpetuated.

If you run the bitforest_right script, execution will again succeed, but the
final state will be `Int(0)`, meaning the spend is not allowed. If you look in
txs.json, you will see a `data` field which contains some hex:

```json
..
"data": "0000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000000",
..
```

The first 32 bytes represent 1 as a U256. This is the `index` of the node in
the BST. The next 32 bytes are for a name, since Bitforest is a name registry.

In both scripts you'll see a hardcoded hex literal representing U256 "2". This
is the index value that should match the transaction which the script is being
attached to.
```clojure
(btoi 0x0000000000000000000000000000000000000000000000000000000000000002)
```
Scripts can't reflect upon their own data (why would they need to when it is
known during construction) so the field is hardcoded. This could also more
simply just be a "2", since btoi is just "bytes to int", but the hex makes it
more clear for demonstration that the literal represents the data field of the covenant's transaction.
