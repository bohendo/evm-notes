
# Ethereum Yellow Paper

## Mathy Symbols
 - `∃`: there exists
 - `∀`: for all
 - `∧`: and
 - `∨`: or
 - `N_{H}`: 1,150,000 aka block number at which the protocol was upgraded from homestead to frontier.
 - `T`: a transaction eg `T = { n: nonce, p: gasPrice, g: gasLimit, t: to, v: value, i: initBytecode, d: data }`
 - `S()`: returns the sender of a transaction eg `S(T) = T.from`
 - `Λ`: (lambda) account creation function
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
      σ: root hash of the ETH state after transaction 1 finishes executing,
      u: cumulative gas used immediately after this tx completes,
      b: bloom filter,
      l: set of logs created while executing this tx
    }
  ]
}
```

## Execution Environment: I

```
I = Execution Environment = {
  a: `address(this)` address of the account which owns the executing code
  o: `tx.origin` original sender of the tx that initialized this execution
  p: `tx.gasPrice` price of gas
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

## Accrued sub-state: A

The data accumulated during tx execution that needs to be remembered for later

```
A = {
  s: suicide set ie the accounts to delete at the end of this tx
  l: logs
  t: touched accounts
  r: refunds eg gas received when storage is freed
}
```

## re Contract Creation

If we send a transaction `tx` to create a contract, `tx.to` is set to 0 and we include a `tx.init` field that contains bytecode. This is NOT the bytecode run by the contract, rather it RETURNS the bytecode run by the contract ie the `tx.init` code is run ONCE at contract creation and never again.

If `T.to == 0` then this is a contract creation transaction and `T.init != null`, `T.data == null`

## Op Code Reference

|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--|
|[00](#stop)|[01](#add)|[02](#mul)|[03](#sub)|[04](#div)|[05](#sdiv)|[06](#mod)|[07](#smod)|[08](#addmod)|[09](#mulmod)|[0a](#exp)|[0b](#signextend)|  |  |  |  |
|10|11|12|13|14|15|16|17|18|19|1a|1b|1c|1d|  |  |
|20|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|30|31|32|33|34|35|36|37|38|[39](#codecopy)|3a|3b|3c|3d|3e|3f|
|40|41|42|43|44|45|46|47|48|  |  |  |  |  |  |  |
|50|51|52|53|54|55|56|57|58|59|5a|5b|  |  |  |  |
|60|61|62|63|64|65|66|67|68|69|6a|6b|6c|6d|6e|6f|
|70|71|72|73|74|75|76|77|78|79|7a|7b|7c|7d|7e|7f|
|80|81|82|83|84|85|86|87|88|89|8a|8b|8c|8d|8e|8f|
|90|91|92|93|94|95|96|97|98|99|9a|9b|9c|9d|9e|9f|
|a0|a1|a2|a3|a4|  |  |  |  |  |  |  |  |  |  |  |
|b0|b1|b2|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|f0|f1|f2|f3|f4|f5|  |  |  |  |fa|  |  |fd|  |ff|

-----
### STOP
**0x00**

() => ()

halts execution

-----
### ADD
**0x01**

(a, b) => (c)

c = a + b

-----
### MUL
**0x02**

(a, b) => (c)

c = a * b

### SUB
**0x03**

(a, b) => (c)

c = a - b

### DIV
**0x04**

(a, b) => (c)

c = a / b

### SDIV
**0x05**

(a: int256, b: int256) => (c: int256)

c = a / b

### MOD
**0x06**

(a, b) => (c)

c = a % b

### SMOD
**0x07**

(a: int256, b: int256) => (c: int256)

c = a % b

### ADDMOD
**0x08**

(a, b, m) => (c)

c = (a + b) % m

### MULMOD
**0x09**

(a, b, m) => (c)

c = (a * b) % m

### EXT
**0x0a**

(a, b, m) => (c)

c = (a * b) % m

### SIGNEXTEND
**0x0b**

(b, x) => (y)

y = signextend(x, b)

sign extends x from (b + 1) * 8 bits to 256 bits.

### CODECOPY
**0x39**

(memOst, codeOst, len) => ()

memory[memOst:memOst+len] = address(this).code[codeOst:codeOst+len]

CODECOPY: copy bytecode from execution environment (I.b) to memory (μ.m)
 - stack: [0] location in memory to start copying to, [1] location in bytecode to start copying from, [2] number of bytes to copy
