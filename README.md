# Cryptus

[![NPM version](https://img.shields.io/npm/v/cryptus.svg?style=flat-square)](https://www.npmjs.com/package/cryptus)
[![NPM downloads](https://img.shields.io/npm/dm/cryptus.svg?style=flat-square)](https://www.npmjs.com/package/cryptus)
[![Node.js CI](https://github.com/acuminous/cryptus/workflows/Node.js%20CI/badge.svg)](https://github.com/acuminous/cryptus/actions?query=workflow%3A%22Node.js+CI%22)
[![Code Climate](https://codeclimate.com/github/acuminous/cryptus/badges/gpa.svg)](https://codeclimate.com/github/acuminous/cryptus)
[![Test Coverage](https://codeclimate.com/github/acuminous/cryptus/badges/coverage.svg)](https://codeclimate.com/github/acuminous/cryptus/coverage)
[![Dependency Status](https://david-dm.org/acuminous/cryptus.svg)](https://david-dm.org/acuminous/cryptus)
[![devDependencies Status](https://david-dm.org/acuminous/cryptus/dev-status.svg)](https://david-dm.org/acuminous/cryptus?type=dev)

Cryptus is an ultra thin crypto wrapper for encrypting and decrypting utf-8 strings. It is based purely on Node's crypto library and has no additional production dependencies. The main reason to use cryptus is because of its reasonably secure default behaviour, whereas if you use the Node.js crypto module directly you may make a mistake. In short, what cryptus brings to the table is:

* Reasonably secure defaults
* Initialisation vector is combined with encrypted secret for easy storage
* Choice of synchronous, callback, and promise based APIs

## TL;DR
```
const { promiseApi: initCryptus } = require('..');

const cryptus = initCryptus();
const key = await cryptus.createKey('super secret password');

const encrypted = await cryptus.encrypt(key, 'text to be encrypted');
console.log('Encrypted', encrypted);

const decrypted = await cryptus.decrypt(key, encrypted);
console.log('Decrypted', decrypted);
```
### Output:
```
Encrypted v1:8f910e22140e4ea1c4640b19c7ad2eff:1ba80a0b72660ed3e0b6b18781e6e7ca66cef8b8e2ca73f0aff06f223c9a5ad2
Decrypted text to be encrypted
```

## Command Line Scripts

```
npm install --production cryptus

# Create a key
./node_modules/.bin/create-key 'super secret password'
f81db52a3b2c717fe65d9a3b7dd04d2a08793e1a28e3083db3ea08db56e7c315

# Encrypt some text
./node_modules/.bin/encrypt f81db52a3b2c717fe65d9a3b7dd04d2a08793e1a28e3083db3ea08db56e7c315 'super secret text'
v1:0cb7e69b6ad9db09039e941f4bdd4e20:355098d39feb5ba580dfdf2193434116f73875cc8f89fddae2f099affb5684f7

# Decrypt the encrypted text
./node_modules/.bin/decrypt f81db52a3b2c717fe65d9a3b7dd04d2a08793e1a28e3083db3ea08db56e7c315 v1:0cb7e69b6ad9db09039e941f4bdd4e20:355098d39feb5ba580dfdf2193434116f73875cc8f89fddae2f099affb5684f7
super secret text
```

## Configuration
```
const { promisify } = require('util');
const { promiseApi: initCryptus } = require('..');

// These are cryptus's default options. Change as required
const cryptus = initCryptus({
  promisify: promisify,
  algorithm: 'aes-256-cbc',
  iterations: 100000,
  keyLength: 32,
  ivLength: 16,
  saltLength: 32,
  digest: 'sha512',
});

// As before...
```
See the [examples](https://github.com/acuminous/cryptus/tree/master/examples) for more details.


## Cryptography
Warning: if you lose your creds, you ain't gonna get your stuff back.

Please be aware that this is a difficult area to get right. It would be unwise to use this module for something that was extremely sensitive. If you are handling something that needs very good secrecy, it's imperative that the system is designed from the ground up to incorporate that need. You would be well advised to get specialist advice.

### Limitations of this module
This module can only encrypt utf-8 encoded strings with a block cipher. I.e. you provide a string, you get an encrypted string back, assuming you don't lose your key/password, you will then be able to decrypt the string to get the original string back.

This module does not authenticate. You may need authentication.

This module does not provide one way hash functionality.

If you are encrypting many many messages, you will need to cycle your key. How many messages? I don't know, please consult the literature.

### Details
At the time of writing, this module default to aes-256-cbc as the block cipher, it uses a random initialisation vector (IV) for the first block. It uses pbkdf2() (password based key derivation function 2) to create keys (both from random bytes if no password is provided and also from a password you provide).

This module wraps NodeJS's crypto module primarily to make it easier to use. Any defects in that module will leak into this one.

AES-256-CBC + pbkdf2 was chosen because it is a conservative choice in the spirit of: "You don't get fired for choosing IBM". There are newer, and potentially better, ciphers out there but cryptography is not based on mathematical proof, so battle hardened algorithms with a long track record are desirable.

### Why use this
Why use this module over just a simple aes192 and password combination?

1. It is advised to aim for security of 128 bits. Key length != security. 256 bit key should hopefully give you 128 bits security.
1. The standard aes192 doesn't use an IV, this means you're leaking information. For example if we have the same password and you encrypt the same thing with both of our keys, you will get the same ciphertext. The attacker now knows that they can attack your security to break mine.
1. aes192 just encrypts block by block, so similarly to the above point, it leaks information about the key. Worse, if you have repeating text in your plaintext, it may be possible to statistically analyse the ciphertext to figure out your plaintext, especially poignant in situations where the plaintext can only be one of a finite number of plaintexts.

Want to learn more? [Practical Cryptography](https://www.schneier.com/books/practical_cryptography/) is a good place to start.

## Credits
The inspiration, much of the code and almost all of this readme is thanks to the hard work of [Jake Howard](https://github.com/jakehoward). I shamelessly (but with permission) ripped off one of his modules and open sourced it.
