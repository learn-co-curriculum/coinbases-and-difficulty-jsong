# Coinbases and Difficulty Targets

```python
# import everything and define a test runner function
from importlib import reload
from helper import run_test

import block
import ecc
import helper
import script
import tx
```

### Exercise 1

#### 1.1. Make [this test](/edit/session6/tx.py) pass
```
tx.py:TxTest:test_is_coinbase
```


```python
# Exercise 1.1

reload(tx)
run_test(tx.TxTest('test_is_coinbase'))
```

### Exercise 2

#### 2.1. Make [this test](/edit/session6/tx.py) pass
```
tx.py:TxTest:test_coinbase_height
```


```python
# Exercise 2.1

reload(tx)
run_test(tx.TxTest('test_coinbase_height'))
```

### Exercise 3

#### 3.1. Find the output address corresponding to this ScriptPubKey
```
76a914338c84849423992471bffb1a54a8d9b1d69dc28a88ac
```

Remember the structure of pay-to-pubkey-hash (p2pkh) which has `OP_DUP OP_HASH160 <hash> OP_EQUALVERIFY OP_CHECKSIG`.

You need to grab the hash160 and turn that into an address.

#### 3.2. Make [this test](/edit/session6/script.py) pass
```
script.py:ScriptTest:test_address
```


```python
# Exercise 3.1

from helper import h160_to_p2pkh_address
from script import Script

hex_script_pubkey = '76a914338c84849423992471bffb1a54a8d9b1d69dc28a88ac'

# bytes.fromhex to get binary
# parse with Script
# get the 3rd element, which should be the hash160
# convert h160 to p2pkh address
```


```python
# Exercise 3.2

reload(script)
run_test(script.ScriptTest('test_address'))
```

### Exercise 4

#### 4.1. What is the double_sha256 of this block? Notice anything?
```
020000208ec39428b17323fa0ddec8e887b4a7c53b8c0a0a220cfd0000000000000000005b0750fce0a889502d40508d39576821155e9c9e3f5c3157f961db38fd8b25be1e77a759e93c0118a4ffd71d
```

#### 4.2. Make [these tests](/edit/session6/block.py) pass.
```
block.py:BlockTest:test_parse
block.py:BlockTest:test_serialize
block.py:BlockTest:test_hash
```


```python
# Exercise 4.1

from helper import double_sha256

hex_block = '020000208ec39428b17323fa0ddec8e887b4a7c53b8c0a0a220cfd0000000000000000005b0750fce0a889502d40508d39576821155e9c9e3f5c3157f961db38fd8b25be1e77a759e93c0118a4ffd71d'

# bytes.fromhex to get the binary
# double_sha256 the result
# hex() to see what it looks like
```


```python
# Exercise 4.2

reload(block)
run_test(block.BlockTest('test_parse'))
run_test(block.BlockTest('test_serialize'))
run_test(block.BlockTest('test_hash'))
```


```python
# Version Signaling Example

from io import BytesIO
from block import Block

hex_block = '020000208ec39428b17323fa0ddec8e887b4a7c53b8c0a0a220cfd0000000000000000005b0750fce0a889502d40508d39576821155e9c9e3f5c3157f961db38fd8b25be1e77a759e93c0118a4ffd71d'

# bytes.fromhex to get the binary block
# create a stream using BytesIO
# parse the block
# get the version
# rightshift 29 (version >> 29) and see if it's equal to 0b001 for BIP9
# see if bit 4 (version >> 4) from the right is set ( & 1) for BIP91
# see if bit 1 (version >> 1) from the right is set ( & 1) for BIP141
```

### Exercise 5

#### 5.1. Make [these tests](/edit/session6/block.py) pass.
```
block.py:BlockTest:test_bip9
block.py:BlockTest:test_bip91
block.py:BlockTest:test_bip141
```


```python
# Exercise 5.1

reload(block)
run_test(block.BlockTest('test_bip9'))
run_test(block.BlockTest('test_bip91'))
run_test(block.BlockTest('test_bip141'))
```


```python
# Calculating Target from Bits Example

from helper import little_endian_to_int

bits = bytes.fromhex('e93c0118')
exponent = bits[-1]
coefficient = little_endian_to_int(bits[:-1])
target = coefficient*2**(8*(exponent-3))
print('{:x}'.format(target).zfill(64))
```


```python
# Calculating Difficulty from Target Example

from helper import little_endian_to_int

bits = bytes.fromhex('e93c0118')
exponent = bits[-1]
coefficient = little_endian_to_int(bits[:-1])
target = coefficient*2**(8*(exponent-3))

bits_min = bytes.fromhex('ffff001d')
exponent_min = bits_min[-1]
coefficient_min = little_endian_to_int(bits_min[:-1])
min = coefficient_min*2**(8*(exponent_min-3))

difficulty = min // target
print(difficulty)
```

### Exercise 6

#### 6.1. Calculate the target and difficulty for these bits:
```
f2881718
```

Bits to target formula is

\\(coefficient\cdot2^{8\cdot(exponent-3)}\\)

where coefficient is the first three bytes in little endian and exponent is the last byte.

Target to Difficulty formula is

\\(difficulty = min / target\\)

where \\(min = 0xffff\cdot2^{8\cdot(0x1d-3)}\\)

#### 6.2. Make [this test](/edit/session6/block.py) pass.
```
block.py:BlockTest:test_target
```


```python
# Exercise 6.1

from helper import little_endian_to_int

hex_bits = 'f2881718'

# bytes.fromhex to get the bits
# last byte is exponent
# first three bytes are the coefficient in little endian
# plug into formula coefficient * 2^(8*(exponent-3)) to get the target
# print target using print('{:x}'.format(target).zfill(64))

hex_min_bits = 'ffff001d'
# do the same for the min_bits for min target

# difficulty is min target / target
```


```python
# Exercise 6.2

reload(block)
run_test(block.BlockTest('test_target'))
```

### Exercise 7

#### 7.1.Validate the proof-of-work for this block
```
04000000fbedbbf0cfdaf278c094f187f2eb987c86a199da22bbb20400000000000000007b7697b29129648fa08b4bcd13c9d5e60abb973a1efac9c8d573c71c807c56c3d6213557faa80518c3737ec1
```

Check that the proof-of-work (double-sha256 interpreted as a little-endian number) is lower than the target.

#### 7.2. Make [this test](/edit/session6/block.py) pass.
```
block.py:BlockTest:test_check_pow
```


```python
# Exercise 7.1

from io import BytesIO
from helper import double_sha256, little_endian_to_int
from block import Block

hex_block = '04000000fbedbbf0cfdaf278c094f187f2eb987c86a199da22bbb20400000000000000007b7697b29129648fa08b4bcd13c9d5e60abb973a1efac9c8d573c71c807c56c3d6213557faa80518c3737ec1'

# bytes.fromhex to get the binary block
# double_sha256 the block
# interpret the result as a number in little endian
# make a stream using BytesIO
# parse the Block
# get the target
# check proof of work < target
```


```python
# Exercise 7.2

reload(block)
run_test(block.BlockTest('test_check_pow'))
```
