
# Ethereum Yellow Paper

## Mathy Symbols
 - `∃`: there exists
 - `∀`: for all
 - `∧`: and
 - `∨`: or
 - `N_{H}`: 1,150,000 aka block number at which the protocol was upgraded from homestead to frontier.
 - `T`: a transaction eg `T = { n: nonce, p: gasPrice, g: gasLimit, t: to, v: value, i: initBytecode, d: data }`
 - `S()`: returns the sender of a transaction eg `S(T) = T.from`
 - `Λ`: account creation function
 - `KEC`: Keccak SHA-3 hash function
 - `RLP`: Recursive Length Prefix encoding

## High-level glossary
 - `σ`: ethereum world state
 - `B`: block
 - `μ`: EVM state
 - `A`: accumulated transaction sub-state
 - `I`: execution environment
 - `o`: output of `H(μ,I)` ie null if we're good to go or a set of data if execution should halt
 - `Υ(σ,T) => σ'`: the transaction-level state transition function
 - `Π(σ,B) => σ'`: the block-level state transition function, processes all transactions then finalizes with Ω
 - `Ω(B,σ) => σ`: block-finalisation state transition function
 - `O(σ,μ,A,I)`: one iteration of the execution cycle
 - `H(μ,I) => o`: outputs null while execution should continue or a series if execution should halt.

## Ethereum World-State: σ

A mapping between addresses (human or contract) and account states. Saved as a Merkle-Patricia tree whose root is recorded on the blockchain backbone.

```
σ = [ account1={...}, account2={...},
  account3= {

    n: nonce aka number of transactions sent by account3
    b: balance ie number of wei account3 controls
    s: storage root, hash of the merkle-patricia tree that contains this accounts long-term data store
    c: code, hash of the EVM bytecode that controls this account. If this equals the hash of an empty string, this is a non-contract account.

  }, ...
]
```

## The Block: B

```
B = Block = {
  H: Header = {
    p: parentHash,
    o: ommersHash,
    c: beneficiary,
    r: stateRoot,
    t: transactionsRoot,
    e: receiptsRoot,
    b: logsBloomFilter,
    d: difficulty,
    i: number,
    l: gasLimit,
    g: gasUsed,
    s: timestamp,
    x: extraData,
    m: mixHash,
    n: nonce,
  },
  T: Transactions = [
    tx1, tx2...
  ],
  U: Uncle block headers = [
    header1, header2..
  ],
  R: Transaction Receipts = [
    receipt_1 = {
      σ: root hash of the ETH state after transaction with index 1,
      u: cumulative gas used so far (immediately after this tx completes),
      b: bloom filter,
      l: set of logs created while executing this tx
    }
  ]
}
```

## Execution Environment: I

```
I = Execution Environment = {
  a: address of the account which owns the executing code
  o: original sender of the tx that initialized this execution
  p: price of gas
  d: data aka byte array of method id & args
  s: sender of this tx or initiator of this execution
  v: value send along w this execution or transaction
  b: byte array of machine code to be executed
  H: header of the current block
  e: current stack depth
}
```

## EVM state: μ

The state of the EVM during execution

```
μ = {
  g: gas left
  pc: program counter ie index into which instruction of I.b to execute next
  m: memory contents, lazily initialized to 2^256 zeros
  i: number of words in memory
  s: stack contents
}
```

## Accrued substate: A

The data accumulated during tx execution that needs to be remembered for later

```
A = {
  s: suicide set: the accounts to delete at the end of this tx
  l: logs
  t: touched accounts
  r: refunds eg gas received when storage is freed
}
```

## re Contract Creation

If we send a transaction `tx` to create a contract, `tx.to` is set to 0 and we include a `tx.init` field that contains bytecode. This is NOT the bytecode run by the contract, rather it RETURNS the bytecode run by the contract ie the `tx.init` code is run ONCE at contract creation and never again.

If `T.to == 0` then this is a contract creation transaction and `T.init != null`, `T.data == null`
