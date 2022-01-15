---
title: 'What is a Bitcoin address?'
date: 2022-01-14 15:54:31
tags: bitcoin, non-technical, layman
---

An address is a piece of text that encodes some data. This data has a very interesting
property: **you can prove that you own it**.

For that to work, you can't just choose an address. You need to calculate it in a
particular way. First, create a pair of _keys_:

* A **private key** is 32 bytes of (basically) random data. You need to keep it secret.  
  32 bytes, or 256 bits, is a lot. Enough that, if you generated your key randomly, it
  is virtually impossible for someone else to stumble upon the exact same data, ever.
  The key is the only copy of these particular 32 bytes in the world.
* A **public key** is 33 bytes of data, which is calculated from the private key. It is
  impossible to figure out the private key from the public key, so you can show the
  public key to anyone. This will be important soon.

This _key pair_ allows you to sign data. You can use the private key to generate a
**signature** over some data, say, a transaction. Anyone who knows the public key can
take the signature and verify that it was really created by you, the owner of the
private key.

From the public key, you calculate **your address**. There is no registration step. As
soon as you know your address, you can give it out to people and they can send Bitcoins
to it.

The only way to create that particular address is to calculate it from your public key
-- and the only way to create that particular public key is to calculate it from your
private key. Because you have the only copy of the private key, it must have been you
who created the address.

When you want to spend your Bitcoins, you create a transaction: "send X Bitcoins from
address `MyAddress` (public key `MyPubKey`) to address `SomeOtherAddress`". Then you
sign this transaction using your private key.

The network checks that the signature matches the public key in the transaction, and
that the public key matches the address from which it is spending. This **proves
ownership**: the person who has the private key to this address really did authorize
this transaction.
