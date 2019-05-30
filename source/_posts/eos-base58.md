---
title: EOS's creative use of base58check
date: 2019-05-30 18:28:12
tags: eos
---

In cryptocurrency world, the **[base58]** encoding is pretty well known. It is also
conceptually very simple: just take a byte array, interpret is as a big-endian number,
and convert that number to base 58.

[base58]: https://en.bitcoin.it/wiki/Base58Check_encoding

Why 58 in particular? Let's look at the character set: it's all upper-case and
lower-case latin letters, plus decimal digits. That is 26 lower-case, 26 upper-case, 10
digits, for a total of 64. Now remove `l` and `I` and `O` and `0` because they get
confused easily, and you are down to 58.

By the standards of this blog, this is an extremely smart, reasonable and well thought
out idea. It does have the curious property that zero is encoded as `1` (because we
removed `0`), but that's nothing we actually care about.

It's also somewhat impractical: given that 256 and 58 are not divisible by each other,
you can't just cut-and-paste parts of the base58 representation as if you were cutting
and pasting bytes. `base58(x + y) != base58(x) + base58(y)` for any `x` and `y`. But,
again, we are not doing that, so we don't mind.

With that, enter **base58check**: take the byte array you want to encode, calculate
SHA-256 of it, take first four bytes as a checksum, append to data, encode. Like this:
```python
def base58check_encode(data):
    sha = hashlib.sha256(data).digest()
    return base58_encode(data + sha[:4])
```
Now if a user types the thing wrong, the checksum will fail and you can tell them. Nice,
simple, very reasonable.

That's why folks at EOS decided to shake it up.

Firstly, SHA-256 is not good enough for EOS. Instead we'll use RIPEMD-160. Now, I have
nothing against RIPEMD-160, except there is _no reason whatsoever_ to not use SHA-256,
but, OK, you do you. What comes next is worse.

EOS signature looks like this: `SIG_K1_KirzetKmDKt1Rbf4k(...)`. The `SIG` means
signature, `K1` is one of two supported curves, and the alphabet soup is
base58check-encoded data.

So you take the `K1`, and add it to the checksum calculation.

Not to the data. No. To the checksum calculation only.

So we can't use the function given above, because the checksum is taken _from something
other than the encoded data_!!

I'm not even talking about how you take the _prefix_ and add it _at end_. Like this:
```python
def base58check_eos_encode(data, curve_type):
    checksum = ripemd160(data + curve_type).digest()
    return base58_encode(data + checksum[:4])
```

I am ... not sure what the goal was here. Tie the curve type to the _base58
represenation_ of signature data? So that the user can't change `SIG_K1_` to `SIG_R1_`
just like that? But then **whyyyyy** would you not add the curve name at start of the
data? Like, `base58check_encode(b"K1" + signature)`? Then you could even leave it out of
the readable prefix and just send `SIG_lkasjdfasdoi`, right?

For a bit of more fun, there are two formats of public keys: `EOSe3LkLKEJSf(...)`, which
is normal base58check, and `PUB_K1_e3LkLKEJSf(...)`, where the `K1` goes into the
checksum. That's the only difference, so the base58 string will be mostly the same,
except for the checksum at end, so you can't simply overwrite `EOS` with `PUB_K1_`.
Because $deity forbid we allow users to simply overwrite the prefix and get, well, the
exact same thing in a newer format.

And this is what this blog is about. There are dozens of ways you can go with this
combination of formats. You could even choose a fixed prefix for the data so that the
Base58 representation starts with ascii string `K1` - Bitcoin-like coins do this to get
distinct addresses. You could keep base58check as-is and it would be fine.

But no. Out of the countless reasonable options, you had to go and do ... well ... this.
