---
title: 'Pay to public key'
date: 2022-01-15 17:08:00
tags: bitcoin
---

Previously, [we covered] what is a Bitcoin address and what properties it has.

[we covered]: /2022/01/14/what-is-an-address/

Today, we will start looking into the insanity that is technical details.

To recap, the address is calculated from your public key. We need the address to be
uniquely tied to the public key -- two public keys must not have the same address. This
way we can verify that only the owner of the private key, and nobody else, has rights to
the address.

### But how do we do that?

**First idea:** let's just [base58]-encode the public key and be done with it!

[base58]: https://en.bitcoin.it/wiki/Base58Check_encoding

Nice and simple. And it actually exists, kind of. It is called **P2PK**, or **Pay to
Public Key**. But it was never really used for text addresses, because...

**Problem:** the public key is 33 bytes long. Its encoding is 50 characters. That's not
great.

**Solution:** hash the public key with RIPEMD-160, so the result is 20 bytes, and the
address is 33 characters.

In addition to being significantly shorter, it also hides the public key. While we don't
technically _need_ to hide the public key (it is public after all), it is nice to have.

So yeah, let's just...

**Problem:** what if someone breaks RIPEMD-160?

...what?

**Solution:** hash the public key with SHA-256, then hash the result with RIPEMD-160, so
the address is still 33 characters.

...what??

### And we are done

This is called **P2PKH**, or **Pay to Public Key Hash**. When verifying the transaction,
the payer provides the signature and their public key. The network checks that the
signature is valid, and that the public key, hashed with SHA-256 and then RIPEMD-160,
matches the address.

The choice to chain two hash functions is ... not bad or wrong or anything. It is just
weird.

### Are we done though?

Not by a long shot.

You might have noticed that there are already two different "address formats": P2PK and
P2PKH. We need to somehow distinguish between the two when validating the transaction.

As it turns out, those two are not the only ones. But that is for next time.
