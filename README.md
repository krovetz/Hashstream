# Hashstream

---

Hashstream is a simple, efficient, and flexible keyed PRF. Conceptually it takes a (data, nonce) pair as input, where "data" can be any length and "nonce" is fixed length, and produces as many pseudorandom bytes as desired. The data is hashed and then the hash result and nonce are supplied to a stream cipher to produce the pseudorandom bytes.

Since the data is hashed, it is conceivable that keeping the nonce steady and changing only the data could result in a hash collision and produce the same output both times, but this is unlikely unless a very large number hashes are done with the same nonce. It is therefore recommended to ensure the nonce is different each time Hashstream is invoked if more than 2^32 invocations are anticipated. Below this number it is acceptable to use a constant for the nonce if it is more convenient.

This repository implements the Hashstream versions discussed in the paper *blah blah blah*.

HS1 combines the hash HS1-Hash with Chacha
One version combines the Poly1305 hash with the Chacha20 stream cipher, while another version combines the GHASH hash (aka, the hash used in GCM) with AES-CTR as the stream cipher. The implementations leverage the OpenSSL cryptographic library and so are relatively short.

---

###Important files in this repository:

**hashstream.h**

Defines and documents the API for Hashstream. In summary:

`hs_ctx_new_and_init(key, keybytes)` allocates, initializes and returns an `hs_ctx` structure.  
`hs_ctx_zero_and_free(ctx)` erases sensitive data and frees an `hs_ctx`.   
`hs_hash(ctx, in, inbytes)` hashes an input, stores resulting value in the `hs_ctx`.  
`hs_stream(ctx, nonce, out, outbytes)` produces pseudorandom bytes based on the nonce and last hash.  
`hs_hashstream(ctx, nonce, in, inbytes, out, outytes)` all-in-one hash and then stream.

Passing NULL for nonce will auto-increment the previously used nonce. A pointer to the nonce used is returned by `hs_stream` and `hs_hashstream`. A pointer to the hash generated is returned by `hs_hash`. The memory to which these pointers refer should be considered read-only.

**hspc.c**

A Hashstream implementation using Poly1305 for hashing and Chacha20 for streaming. Preferred key length is 16 bytes. Nonce length is 12 bytes. Hash length is 16 bytes.

**hsga.c**

A Hashstream implementation using GHASH for hashing and AES-CTR for streaming. Preferred key length is 16 bytes. Nonce length is 16 bytes. Hash length is 16 bytes.

---

###To build

1. Download OpenSSL anywhere on your system: https://www.openssl.org/source/. This project was developed using version 1.1.1.

2. Build OpenSSL: Untar the tarball, cd into the new directory, `./config -march=native -mtune=native`, `make`. Depending on your architecture you may need to change the `-march=native -mtune=native` to whatever is right for your machine. Adding `CC=clang` appears to work on clang-based installations.

3. Compile Hashstream with the resulting `libcrypto.a` file: `gcc -march=native -mtune=native -O3 -Wall hs_timer.c hspc.c ~/openssl/openssl-1.1.1-pre6/libcrypto.a`.

4. The Hashstream code uses some internal All of the function prototypes needed are embedded in the Hashstream code, so no headers need to be included. This is convenient, but means that any changes in OpenSSL's API