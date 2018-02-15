---
layout: post
title: "Starting to figure out SSL"
---

# Introduction

I've been slowly working towards a personal finance app, but before I can actually put it on the open web, I want to be reasonably confident that I am not opening myself up to being hacked.

One of the pieces to that puzzle is having secure connections in the browser for accessing my site.  That involves HTTPS.


## Progress

Here's what I've got so far.  There's symmetric and asymmetric encryption.  Symmetric is when both parties share the same secret. They use the same secret to encode and decode messages.  In Caesar's cipher, you shift all the letters in your message by some number N, so the secret would be N.  When somebody knows N, they can encode your message, and they can eaisly re-encode another message.

Asymmetric encryption involves 2 keys: public, and private.  The public key is for encryption and the private key is for decryption.  The public key is not sensitive and can be shared.   The private key is kept secret because it is the only thing that can decrypt something that has been encrypted with the public key.  This means posting your public key somewhere is like saying here is a secure way to send me information at any point in time.  It also provides a way for you to prove your identity.  For instance, if you get a public key from a source you trust, you can verify that someone holds the private key by encrypting something, sending it to them, and having them send your message back to you..

## Remaining questions

* How does signing of a certificate work? Like you create your public key, and then a CA signs it, but only your private key can still decrypt it?

( Maybe there's a portion of it that indicates that it's signed, and then there's your public key in there.)

* How does the math work in symmetric and asymmetric encryption?

* What is X.509?

* How do I use a public key to verify something without consulting the private key? I want to verify the CA signed this thing, but that doesn't fit in my pub-encrypt, priv-decrypt model. 


To be continued!
