
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
  a: address(this) address of the account which owns the executing code
  o: tx.origin original sender of the tx that initialized this execution
  p: tx.gasPrice price of gas
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
|[10](#lt)|[11](#gt)|[12](#slt)|[13](#sgt)|[14](#eq)|[15](#iszero)|[16](#and)|[17](#or)|[18](#xor)|[19](#not)|[1a](#byte)|[1b](#shl)|[1c](#shr)|[1d](#sar)|  |  |
|[20](#sha3)|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|[30](#address)|[31](#balance)|[32](#origin)|[33](#caller)|[34](#callvalue)|[35](#calldataload)|[36](#calldatasize)|[37](#calldatacopy)|[38](#codesize)|[39](#codecopy)|[3a](#gasprice)|[3b](#extcodesize)|[3c](#extcodecopy)|[3d](#returndatasize)|[3e](#returndatacopy)|[3f](#extcodehash)|
|[40](#blockhash)|[41](#coinbase)|[42](#timestamp)|[43](#number)|[44](#difficulty)|[45](#gaslimit)|[46](#chainid)|[47](#selfbalance)|[48](#basefee)|  |  |  |  |  |  |  |
|[50](#pop)|[51](#mload)|[52](#mstore)|[53](#mstore8)|[54](#sload)|[55](#sstore)|[56](#jump)|[57](#jumpi)|[58](#pc)|[59](#msize)|[5a](#gas)|[5b](#jumpdest)|  |  |  |  |
|[60](#push1)|[61](#push2)|[62](#push3)|[63](#push4)|[64](#push5)|[65](#push6)|[66](#push7)|[67](#push8)|[68](#push9)|[69](#push10)|[6a](#push11)|[6b](#push12)|[6c](#push13)|[6d](#push14)|[6e](#push15)|[6f](#push16)|
|[70](#push17)|[71](#push18)|[72](#push19)|[73](#push20)|[74](#push21)|[75](#push22)|[76](#push23)|[77](#push24)|[78](#push25)|[79](#push26)|[7a](#push27)|[7b](#push28)|[7c](#push29)|[7d](#push30)|[7e](#push31)|[7f](#push32)|
|[80](#dup1)|[81](#dup2)|[82](#dup3)|[83](#dup4)|[84](#dup5)|[85](#dup6)|[86](#dup7)|[87](#dup8)|[88](#dup9)|[89](#dup10)|[8a](#dup11)|[8b](#dup12)|[8c](#dup13)|[8d](#dup14)|[8e](#dup15)|[8f](#dup16)|
|[90](#swap1)|[91](#swap2)|[92](#swap3)|[93](#swap4)|[94](#swap5)|[95](#swap6)|[96](#swap7)|[97](#swap8)|[98](#swap9)|[99](#swap10)|[9a](#swap11)|[9b](#swap12)|[9c](#swap13)|[9d](#swap14)|[9e](#swap15)|[9f](#swap16)|
|[a0](#log0)|[a1](#log1)|[a2](#log2)|[a3](#log3)|[a4](#log4)|  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|[f0](#create)|[f1](#call)|[f2](#callcode)|[f3](#return)|[f4](#delegatecall)|[f5](#create2)|  |  |  |  |[fa](#staticcall)|  |  |[fd](#revert)|  |[ff](#selfdestruct)|

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

### LT
**0x10**

### GT
**0x11**

### SLT
**0x12**

### SGT
**0x13**

### EQ
**0x14**

### ISZERO
**0x15**

### AND
**0x16**

### OR
**0x17**

### XOR
**0x18**

### NOT
**0x19**

### BYTE
**0x1a**

### SHL
**0x1b**

### SHR
**0x1c**

### SAR
**0x1d**

### SHA3
**0x20**

### ADDRESS
**0x30**

### BALANCE
**0x31**

### ORIGIN
**0x32**

### CALLER
**0x33**

### CALLVALUE
**0x34**

### CALLDATALOAD
**0x35**

### CALLDATASIZE
**0x36**

### CALLDATACOPY
**0x37**

### CODESIZE
**0x38**

### CODECOPY
**0x39**

(memOst, codeOst, len) => ()

memory[memOst:memOst+len] = address(this).code[codeOst:codeOst+len]

### GASPRICE
**0x3a**

### EXTCODESIZE
**0x3b**

### EXTCODECOPY
**0x3c**

### RETURNDATASIZE
**0x3d**

### RETURNDATACOPY
**0x3e**

### EXTCODEHASH
**0x3f**

### BLOCKHASH
**0x40**

### COINBASE
**0x41**

### TIMESTAMP
**0x42**

### NUMBER
**0x43**

### DIFFICULTY
**0x44**

### GASLIMIT
**0x45**

### CHAINID
**0x46**

### SELFBALANCE
**0x47**

### BASEFEE
**0x48**

### POP
**0x50**

### MLOAD
**0x51**

### MSTORE
**0x52**

### MSTORE8
**0x53**

### SLOAD
**0x54**

### SSTORE
**0x55**

### JUMP
**0x56**

### JUMPI
**0x57**

### PC
**0x58**

### MSIZE
**0x59**

### GAS
**0x5a**

### JUMPDEST
**0x5b**

### PUSH1
**0x60**

### PUSH2
**0x61**

### PUSH3
**0x62**

### PUSH4
**0x63**

### PUSH5
**0x64**

### PUSH6
**0x65**

### PUSH7
**0x66**

### PUSH8
**0x67**

### PUSH9
**0x68**

### PUSH10
**0x69**

### PUSH11
**0x6a**

### PUSH12
**0x6b**

### PUSH13
**0x6c**

### PUSH14
**0x6d**

### PUSH15
**0x6e**

### PUSH16
**0x6f**

### PUSH17
**0x70**

### PUSH18
**0x71**

### PUSH19
**0x72**

### PUSH20
**0x73**

### PUSH21
**0x74**

### PUSH22
**0x75**

### PUSH23
**0x76**

### PUSH24
**0x77**

### PUSH25
**0x78**

### PUSH26
**0x79**

### PUSH27
**0x7a**

### PUSH28
**0x7b**

### PUSH29
**0x7c**

### PUSH30
**0x7d**

### PUSH31
**0x7e**

### PUSH32
**0x7f**

### DUP1
**0x80**

### DUP2
**0x81**

### DUP3
**0x82**

### DUP4
**0x83**

### DUP5
**0x84**

### DUP6
**0x85**

### DUP7
**0x86**

### DUP8
**0x87**

### DUP9
**0x88**

### DUP10
**0x89**

### DUP11
**0x8a**

### DUP12
**0x8b**

### DUP13
**0x8c**

### DUP14
**0x8d**

### DUP15
**0x8e**

### DUP16
**0x8f**

### SWAP1
**0x90**

### SWAP2
**0x91**

### SWAP3
**0x92**

### SWAP4
**0x93**

### SWAP5
**0x94**

### SWAP6
**0x95**

### SWAP7
**0x96**

### SWAP8
**0x97**

### SWAP9
**0x98**

### SWAP10
**0x99**

### SWAP11
**0x9a**

### SWAP12
**0x9b**

### SWAP13
**0x9c**

### SWAP14
**0x9d**

### SWAP15
**0x9e**

### SWAP16
**0x9f**

### LOG0
**0xa0**

### LOG1
**0xa1**

### LOG2
**0xa2**

### LOG3
**0xa3**

### LOG4
**0xa4**

### CREATE
**0xf0**

### CALL
**0xf1**

### CALLCODE
**0xf2**

### RETURN
**0xf3**

### DELEGATECALL
**0xf4**

### CREATE2
**0xf5**

### STATICCALL
**0xfa**

### REVERT
**0xfd**

### SELFDESTRUCT
**0xff**

