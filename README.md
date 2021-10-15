
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

-----
### SUB
**0x03**

(a, b) => (c)

c = a - b

-----
### DIV
**0x04**

(a, b) => (c)

c = a / b

-----
### SDIV
**0x05**

(a: int256, b: int256) => (c: int256)

c = a / b

-----
### MOD
**0x06**

(a, b) => (c)

c = a % b

-----
### SMOD
**0x07**

(a: int256, b: int256) => (c: int256)

c = a % b

-----
### ADDMOD
**0x08**

(a, b, m) => (c)

c = (a + b) % m

-----
### MULMOD
**0x09**

(a, b, m) => (c)

c = (a * b) % m

-----
### EXT
**0x0a**

(a, b, m) => (c)

c = (a * b) % m

-----
### SIGNEXTEND
**0x0b**

(b, x) => (y)

y = SIGNEXTEND(x, b)

sign extends x from (b + 1) * 8 bits to 256 bits.

-----
### LT
**0x10**

(a, b) => (c)

c = a < b

all values interpreted as uint256

-----
### GT
**0x11**

(a, b) => (c)

c = a > b

all values interpreted as uint256

-----
### SLT
**0x12**

(a, b) => (c)

c = a < b

all values interpreted as int256

-----
### SGT
**0x13**

(a, b) => (c)

c = a > b

all values interpreted as int256

-----
### EQ
**0x14**

(a, b) => (c)

c = a == b

-----
### ISZERO
**0x15**

(a) => (c)

c = a == 0

-----
### AND
**0x16**

(a, b) => (c)

c = a & b

-----
### OR
**0x17**

(a, b) => (c)

c = a | b

-----
### XOR
**0x18**

(a, b) => (c)

c = a ^ b

-----
### NOT
**0x19**

(a) => (c)

c = ~a

-----
### BYTE
**0x1a**

(i, x) => (y)

y = (x >> (248 - i * 8) & 0xff

-----
### SHL
**0x1b**

(shift, value) => (res)

res = value << shift

-----
### SHR
**0x1c**

(shift, value) => (res)

res = value >> shift

-----
### SAR
**0x1d**

(shift, value) => (res)

res = value >> shift

value: int256

-----
### SHA3
**0x20**

(offset, len) => (hash)

hash = keccak256(memory[offset:offset+len])

-----
### ADDRESS
**0x30**

() => (address(this))

-----
### BALANCE
**0x31**

() => (address(this).balance)

-----
### ORIGIN
**0x32**

() => (tx.origin)

-----
### CALLER
**0x33**

() => (msg.sender)

-----
### CALLVALUE
**0x34**

() => (msg.value)

-----
### CALLDATALOAD
**0x35**

(index) => (msg.data[index:index+32])

-----
### CALLDATASIZE
**0x36**

() => (msg.data.size)

-----
### CALLDATACOPY
**0x37**

(memOffset, offset, length) => ()

memory[memOffset:memOffset+len] = msg.data[offset:offset+len]

-----
### CODESIZE
**0x38**

() => (address(this).code.size)

-----
### CODECOPY
**0x39**

(memOffset, codeOffset, len) => ()

memory[memOffset:memOffset+len] = address(this).code[codeOffset:codeOffset+len]

-----
### GASPRICE
**0x3a**

() => (tx.gasprice)

-----
### EXTCODESIZE
**0x3b**

(addr) => (address(addr).code.size)

-----
### EXTCODECOPY
**0x3c**

(addr, memOffset, offset, length) => ()

memory[memOffset:memOffset+len] = address(addr).code[codeOffset:codeOffset+len]

-----
### RETURNDATASIZE
**0x3d**

() => (size)

size = RETURNDATASIZE()

The number of bytes that were returned from the last ext call

-----
### RETURNDATACOPY
**0x3e**

(memOffset, offset, length) => ()

memory[memOffset:memOffset+len] = RETURNDATA[codeOffset:codeOffset+len]

RETURNDATA is the data returned from the last external call

-----
### EXTCODEHASH
**0x3f**

(addr) => (hash)

hash = address(addr).exists ? keccak256(address(addr).code) : 0

-----
### BLOCKHASH
**0x40**

(number) => (hash)

hash = block.blockHash(number)

-----
### COINBASE
**0x41**

() => (block.coinbase)

-----
### TIMESTAMP
**0x42**

() => (block.timestamp)

-----
### NUMBER
**0x43**

() => (block.number)

-----
### DIFFICULTY
**0x44**

() => (block.difficulty)

-----
### GASLIMIT
**0x45**

() => (block.gaslimit)

-----
### CHAINID
**0x46**

() => (chainid)

where chainid = 1 for mainnet & some other value for other networks

-----
### SELFBALANCE
**0x47**

() => (address(this).balance)

-----
### BASEFEE
**0x48**

() => (block.basefee)

current block's base fee (related to EIP1559)

-----
### POP
**0x50**

(a) => ()

discards the top stack item

-----
### MLOAD
**0x51**

(offset) => (value)

value = memory[offset:offset+32]

-----
### MSTORE
**0x52**

(offset, value) => ()

memory[offset:offset+32] = value

-----
### MSTORE8
**0x53**

(offset, value) => ()

memory[offset:offset+32] = value & 0xff

-----
### SLOAD
**0x54**

(key) => (value)

value = storage[key]

-----
### SSTORE
**0x55**

(key, value) => ()

storage[key] = value

-----
### JUMP
**0x56**

(dest) => ()

pc = dest

-----
### JUMPI
**0x57**

(dest, cond) => ()

pc = cond ? dest : pc + 1

-----
### PC
**0x58**

() => (pc)

-----
### MSIZE
**0x59**

() => (memory.size)

-----
### GAS
**0x5a**

() => (gasRemaining)

not including the gas required for this opcode

-----
### JUMPDEST
**0x5b**

() => ()

noop, marks a valid jump destination

-----
### PUSH1
**0x60**

() => (address(this).code[pc+1:pc+2])

-----
### PUSH2
**0x61**

() => (address(this).code[pc+2:pc+3])

-----
### PUSH3
**0x62**

() => (address(this).code[pc+3:pc+4])

-----
### PUSH4
**0x63**

() => (address(this).code[pc+4:pc+5])

-----
### PUSH5
**0x64**

() => (address(this).code[pc+5:pc+6])

-----
### PUSH6
**0x65**

() => (address(this).code[pc+6:pc+7])

-----
### PUSH7
**0x66**

() => (address(this).code[pc+7:pc+8])

-----
### PUSH8
**0x67**

() => (address(this).code[pc+8:pc+9])

-----
### PUSH9
**0x68**

() => (address(this).code[pc+9:pc+10])

-----
### PUSH10
**0x69**

() => (address(this).code[pc+10:pc+11])

-----
### PUSH11
**0x6a**

() => (address(this).code[pc+11:pc+12])

-----
### PUSH12
**0x6b**

() => (address(this).code[pc+12:pc+13])

-----
### PUSH13
**0x6c**

() => (address(this).code[pc+13:pc+14])

-----
### PUSH14
**0x6d**

() => (address(this).code[pc+14:pc+15])

-----
### PUSH15
**0x6e**

() => (address(this).code[pc+15:pc+16])

-----
### PUSH16
**0x6f**

() => (address(this).code[pc+16:pc+17])

-----
### PUSH17
**0x70**

() => (address(this).code[pc+17:pc+18])

-----
### PUSH18
**0x71**

() => (address(this).code[pc+18:pc+19])

-----
### PUSH19
**0x72**

() => (address(this).code[pc+19:pc+20])

-----
### PUSH20
**0x73**

() => (address(this).code[pc+20:pc+21])

-----
### PUSH21
**0x74**

() => (address(this).code[pc+21:pc+22])

-----
### PUSH22
**0x75**

() => (address(this).code[pc+22:pc+23])

-----
### PUSH23
**0x76**

() => (address(this).code[pc+23:pc+24])

-----
### PUSH24
**0x77**

() => (address(this).code[pc+24:pc+25])

-----
### PUSH25
**0x78**

() => (address(this).code[pc+25:pc+26])

-----
### PUSH26
**0x79**

() => (address(this).code[pc+26:pc+27])

-----
### PUSH27
**0x7a**

() => (address(this).code[pc+27:pc+28])

-----
### PUSH28
**0x7b**

() => (address(this).code[pc+28:pc+29])

-----
### PUSH29
**0x7c**

() => (address(this).code[pc+29:pc+30])

-----
### PUSH30
**0x7d**

() => (address(this).code[pc+30:pc+31])

-----
### PUSH31
**0x7e**

() => (address(this).code[pc+31:pc+32])

-----
### PUSH32
**0x7f**

() => (address(this).code[pc+32:pc+33])

-----
### DUP1
**0x80**

(1, 1) => (1, 1)

-----
### DUP2
**0x81**

(1, 2) => (2, 1, 2)

-----
### DUP3
**0x82**

(1, 2, 3) => (3, 1, 2, 3)

-----
### DUP4
**0x83**

(1, ..., 4) => (4, 1, ..., 4)

-----
### DUP5
**0x84**

(1, ..., 5) => (5, 1, ..., 5)

-----
### DUP6
**0x85**

(1, ..., 6) => (6, 1, ..., 6)

-----
### DUP7
**0x86**

(1, ..., 7) => (7, 1, ..., 7)

-----
### DUP8
**0x87**

(1, ..., 8) => (8, 1, ..., 8)

-----
### DUP9
**0x88**

(1, ..., 9) => (9, 1, ..., 9)

-----
### DUP10
**0x89**

(1, ..., 10) => (10, 1, ..., 10)

-----
### DUP11
**0x8a**

(1, ..., 11) => (11, 1, ..., 11)

-----
### DUP12
**0x8b**

(1, ..., 12) => (12, 1, ..., 12)

-----
### DUP13
**0x8c**

(1, ..., 13) => (13, 1, ..., 13)

-----
### DUP14
**0x8d**

(1, ..., 14) => (14, 1, ..., 14)

-----
### DUP15
**0x8e**

(1, ..., 15) => (15, 1, ..., 15)

-----
### DUP16
**0x8f**

(1, ..., 16) => (16, 1, ..., 16)

-----
### SWAP1
**0x90**

(1, 2) => (2, 1)

-----
### SWAP2
**0x91**

(1, 2, 3) => (3, 2, 1)

-----
### SWAP3
**0x92**

(1, ..., 4) => (4, ..., 1)

-----
### SWAP4
**0x93**

(1, ..., 5) => (5, ..., 1)

-----
### SWAP5
**0x94**

(1, ..., 6) => (6, ..., 1)

-----
### SWAP6
**0x95**

(1, ..., 7) => (7, ..., 1)

-----
### SWAP7
**0x96**

(1, ..., 8) => (8, ..., 1)

-----
### SWAP8
**0x97**

(1, ..., 9) => (9, ..., 1)

-----
### SWAP9
**0x98**

(1, ..., 10) => (10, ..., 1)

-----
### SWAP10
**0x99**

(1, ..., 11) => (11, ..., 1)

-----
### SWAP11
**0x9a**

(1, ..., 12) => (12, ..., 1)

-----
### SWAP12
**0x9b**

(1, ..., 13) => (13, ..., 1)

-----
### SWAP13
**0x9c**

(1, ..., 14) => (14, ..., 1)

-----
### SWAP14
**0x9d**

(1, ..., 15) => (15, ..., 1)

-----
### SWAP15
**0x9e**

(1, ..., 16) => (16, ..., 1)

-----
### SWAP16
**0x9f**

(1, ..., 17) => (17, ..., 1)

-----
### LOG0
**0xa0**

(offset, length) => ()

emit(memory[offset:offset+length])

-----
### LOG1
**0xa1**

(offset, length, topic0) => ()

emit(memory[offset:offset+length], topic0)

-----
### LOG2
**0xa2**

(offset, length, topic0, topic1) => ()

emit(memory[offset:offset+length], topic0, topic1)

-----
### LOG3
**0xa3**

(offset, length, topic0, topic1, topic2) => ()

emit(memory[offset:offset+length], topic0, topic1, topic2)

-----
### LOG4
**0xa4**

(offset, length, topic0, topic1, topic2, topic3) => ()

emit(memory[offset:offset+length], topic0, topic1, topic2, topic3)

-----
### CREATE
**0xf0**

(value, offset, length) => (addr)

addr = keccak256(rlp([address(this), this.nonce]))[12:]
addr.code = exec(memory[offset:offset+length])
addr.balance += value
this.balance -= value
this.nonce += 1

-----
### CALL
**0xf1**

(gas, addr, value, argsOffset, argsLength, retOffset, retLength) => (success)

memory[retOffset:retOffset+retLength] = address(addr).callcode.gas(gas).value(value)(memory[argsOffset:argsOffset+argsLength])
success = true (unless the prev call reverted)

-----
### CALLCODE
**0xf2**

(gas, addr, value, argsOffset, argsLength, retOffset, retLength) => (success)

memory[retOffset:retOffset+retLength] = address(addr).callcode.gas(gas).value(value)(memory[argsOffset:argsOffset+argsLength])
success = true (unless the prev call reverted)

TODO: what's the difference between this & CALL?

-----
### RETURN
**0xf3**

(offset, length) => ()

return memory[offset:offset+length]

-----
### DELEGATECALL
**0xf4**

(gas, addr, argsOffset, argsLength, retOffset, retLength) => (success)

memory[retOffset:retOffset+retLength] = address(addr).delegatecall.gas(gas)(memory[argsOffset:argsOffset+argsLength])
success = true (unless the prev call reverted)

-----
### CREATE2
**0xf5**

(value, offset, length, salt) => (addr)

initCode = memory[offset:offset+length]
addr = keccak256(0xff ++ address(this) ++ salt ++ keccak256(initCode))[12:]
address(addr).code = exec(initCode)


-----
### STATICCALL
**0xfa**

(gas, addr, argsOffset, argsLength, retOffset, retLength) => (success)

memory[retOffset:retOffset+retLength] = address(addr).delegatecall.gas(gas)(memory[argsOffset:argsOffset+argsLength])
success = true (unless the prev call reverted)

TODO: what's the difference between this & DELEGATECALL?

-----
### REVERT
**0xfd**

(offset, length) => ()

revert(memory[offset:offset+length])

-----
### SELFDESTRUCT
**0xff**

(addr) => ()

address(addr).send(address(this).balance)
this.code = 0
