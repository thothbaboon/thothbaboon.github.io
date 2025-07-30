+++
title = "Mnemonic Seeds"
description = "Mnemonic seeds are a way to represent a private key in a form that is easier for humans to remember."
date = 2022-12-18
+++

A blockchain account is just a **private key**, from which
**public keys** (addresses) are derived.

Suppose we are the owner of an account, and a NFT is owned by one
public address of this account. Then we are the owner of this NFT.

As we say, **not your key, not your coins**. So we better not
forgot our private key. Else, we’ll loose everything we own !

This is why **mnemonic seeds** are useful.

## Mnemonic Seed

A mnemonic seed is a way of representing a large, randomly generated
number as a sequence of words.

This sequence is usually 12, 15, 18, 21 or 24 words long. The words
are chosen from a predefined list, and the specific word chosen at
each position in the sequence is determined by the value of the
private key at that position.

To compute the mnemonic seed

- The private key is first converted into a binary representation
- The binary representation is then divided into groups of 11 bits
- Each group is used to index into the list of 2048 words to select
  a specific word
- The selected words are then concatenated to form the mnemonic
  seed.

See BIP-39 for more infos about mnemonic standard.

Here is the official english word list [https://github.com/bitcoin/bips/blob/master/bip-0039/english.txt](https://github.com/bitcoin/bips/blob/master/bip-0039/english.txt)

## Concrete Example

Say the private key is **0x4B6150645367566B5970337336763979**

The binary representation of this private key is

```
01001011011 00001010100 00011001000 10100110110 01110101011
00110101101 01100101110 00000110011 01110011001 10110011101
10001110010 1111001
```

The list contains 2048 words. Each word is encoded on 11 bits. The
private key is encoded on 128 bits. 128/11 is not even, so we can’t
represent the private key like this into words.

We need to add 4 more bits, because 132/11 is even (12). These 4 bits
are the 4 leftmost bits of the SHA 256 checksum of the private key.

```
SHA256(4B6150645367566B5970337336763979) = 81339e3d40865082004a0eccf5aba7d15a05d5d99a74547be4718b78f6da1617
```

1 digit in hexa is 4 digits in binary. The leftmost hexa char is 8,
which is 1000 in binary

The 132 bits are

```
01001011011 00001010100 00011001000 10100110110 01110101011
00110101101 01100101110 00000110011 01110011001 10110011101
10001110010 11110011000
```

In decimal it's

```
603, 84, 200, 1334, 939, 429, 814, 51, 921, 1437, 1138, 1944
```

The indexes for the 2048 words are 0 to 2047. So 603 means we look for
the 604th word in the list, which is **enter**.

Repeat for each, and we get the mnemonic seed !

```
enter appear boil plug install cup grape all industry recipe mixture vessel
```

## Conclusion

Private keys are long strings of random characters that are difficult
to remember, read, and write for humans. It’s easy to make a mistake
by copying the private key, and therefore, loose access the your
account.

Mnemonic seeds are a way to represent a private key in a form that is
easier for humans to remember, because it’s made up of words.

With a deterministic wallet, the original seed is enough to recover
all private and public keys. A few words sequence is the only thing to
remember.
