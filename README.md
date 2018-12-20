# emp-tool [![Build Status](https://travis-ci.org/emp-toolkit/emp-tool.svg?branch=stable)](https://travis-ci.org/emp-toolkit/emp-tool)
<img src="https://raw.githubusercontent.com/emp-toolkit/emp-readme/master/art/logo-full.jpg" width=300px/>

# Installation

1. Install prerequisites using instructions [here](https://github.com/emp-toolkit/emp-readme#detailed-installation).
2. `git clone https://github.com/emp-toolkit/emp-tool.git`
3. `cd emp-tool && cmake . && sudo make install`
    1. Alternatively, you can also `cd emp-ot && mkdir -p build && cd build && cmake .. && sudo make install` if out-of-source build is preferred.
    2. By default it will build for Release. `-DCMAKE_BUILD_TYPE=[Release|Debug]` option is also available.
    3. No sudo? change [`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/v2.8.8/cmake.html#variable%3aCMAKE_INSTALL_PREFIX)

# Performance

# Usage

## Basic Primitives
### Pseudorandom generator
PRG is implemented as AES-NI in the CTR mode. Usage is presented as the following code sample.

```cpp
#include<emp-tool/emp-tool.h>
using namespace emp;
PRG prg;//using a secure random seed

int rand_int, rand_ints[100];
block rand_block[3];
mpz_t integ;mpz_init(integ);

prg.random_data(&rand_int, sizeof(rand_int)); //fill rand_int with 32 random bits
prg.random_block(rand_block, 3);	      //fill rand_block with 128*3 random bits

prg.reseed(&rand_block[1]);                   //reset the seed and counter in prg
prg.random_data_unaligned(rand_ints+2, sizeof(int)*98);  //when the array is not 128-bit-aligned
prg.random_mpz(integ, 1024);                  //random number with 1024 bits.
```

### Pseudorandom permutation
PRP is implemented based on AES-NI. Code sample:
```cpp
block key;
PRG().random_block(&key,1)
PRP prp(&key); //if no parameter is provided, a public, fixed key will be used

block rand_block[3], b3[3];
int rand_ints[100];

prp.permute_block(rand_block, 3);//applying pi on each block of data
prp.permute_data(rand_ints, sizeof(int)*100);

block b2 = prp.H(rand_block[1], 1); //b2 = pi(r)\xor r, where r = doubling(random_block)\xor 1

prp.H<3>(b3, rand_block, 0);// apply the above H on three blocks using counter 0, 1, and 2 resp.
```

### Hash function
Essentially a wrapper of ```<openssl/sha.h>```
```cpp
Hash hash;

char * data = new char[1024*1024], dig[Hash::DIGEST_SIZE];

hash.put(data, 1024*1024);
hash.digest(dig);
```

### Commitment
```cpp
Commitment c;

char * data = new char[1024*1024];
Com com;
Decom decom;

c.commit(decom, com, data, 1024*1024); // commit, will fill values decom and com
assert(c.open(decom, com, data, 1024*1024)); // open, return if the decommitment is valid or not
```

## On-the-fly Circuit Compiler
There is no official circuit compiler offered by emp-toolkit, but Juzix Corp.Ltd () has made one with Frutta which is a C-like script language, detail of it can be found here [Frutta](https://www.juzix.net/en/JUGOEN/index.jhtml)

## [Acknowledgement, Reference, and Questions](https://github.com/emp-toolkit/emp-readme/blob/master/README.md#citation)

