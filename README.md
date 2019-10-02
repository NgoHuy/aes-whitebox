# AES Whitebox

This code implements [Chow et al.](https://www.cs.colorado.edu/~jrblack/class/csci7000/s03/project/oorschot-whitebox.pdf) scheme, following [Muir's](https://eprint.iacr.org/2013/104.pdf) tutorial using protected L × MB bijections and XOR tables, extending the logic to AES-192 and AES-256.

The tables generated by this program follow the _Chow et al._ proposition for AES-128. Here is a summary (all values in bytes):

| Table                | AES-128     | AES-192     | AES-256     |
|----------------------|-------------|-------------|-------------|
| XOR tables           | 221,184     | 270,336     | 319,488     |
| Tyi-boxes            | 147,456     | 180,224     | 212,992     |
| T-boxes (last round) | 4,096       | 4,096       | 4,096       |
| inv(MB) × L          | 147,456     | 180,224     | 212,992     |
| **Total**            | **520,192** | **634,880** | **749,568** |

In order to build a full 'oracle' based in the original paper, this code also implements CFB, OFB and CTR block cipher modes of operation around the inner cipher.

External encodings were not implemented, but these are easy to apply if needed.


## How to use

- `aes.cc`, `aes.h` and `aes_private.h` implement the standard AES-{128,192,256} encryption, used only as a reference and can be easily removed from the final build.
- `aes_whitebox_compiler.cc` contains the AES-{128,192,256} 'compiler' (the tool that generates the WBC tables), that generates the `aes_whitebox_tables.cc` file that contains the cipher tables.
- `aes_whitebox.h` and `aes_whitebox.cc` contain the AES implementation that consumes the `aes_whitebox_tables.cc`.

So for a final build, you must (1) compile the `aes_whitebox_compiler` tool, (2) generate the `aes_whitebox_tables.cc` source file using it and add `aes_whitebox.h` together with `aes_whitebox.cc` to the final build.

The accompanying `Makefile` performs all steps in a single execution, including an unit test at the end.

The final API is just:

```
void aes_whitebox_encrypt_cfb(const uint8_t iv[16], const uint8_t* m, size_t len, uint8_t* c);
void aes_whitebox_decrypt_cfb(const uint8_t iv[16], const uint8_t* c, size_t len, uint8_t* m);

void aes_whitebox_encrypt_ofb(const uint8_t iv[16], const uint8_t* m, size_t len, uint8_t* c);
void aes_whitebox_decrypt_ofb(const uint8_t iv[16], const uint8_t* c, size_t len, uint8_t* m);

void aes_whitebox_encrypt_ctr(const uint8_t nonce[16], const uint8_t* m, size_t len, uint8_t* c);
void aes_whitebox_decrypt_ctr(const uint8_t nonce[16], const uint8_t* c, size_t len, uint8_t* m);
```

Where `iv` is the initialization vector (CTR adopts a `nonce` instead of `iv`), `m` is the message, `c` is the ciphertext and `len` is the number of bytes to encode/decode and can be of any size (does not have to be multiple of 16 bytes as required by other block cipher modes such as EBC and CBC).

Internally the code will manage to use the embedded AES-128, AES-192 or AES-256 key using the obfuscated tables.


## BGE Attack

The time complexity of [Billet et al.’s key extraction attack](https://link.springer.com/chapter/10.1007/978-3-540-30564-4_16) is less than 2^30, while [some further optimizations](https://eprint.iacr.org/2013/450.pdf) suggest 2^22.

So, care should be taken before using this implementation in production.
