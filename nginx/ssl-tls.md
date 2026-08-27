Perfect. **This is the point where most tutorials become confusing.** They say "client and server exchange keys" without explaining **what exactly is being sent, why it's sent, and how both end up with the same secret.**

I'll explain it like you're watching the packets travel over the network.

---

# First, one important misconception

Many people think:

> Browser encrypts everything using the server's public key.
> 

**This is NOT how modern TLS (1.2 with ECDHE and TLS 1.3) works.**

Actually,

- The **certificate contains the server's public key**, but
- That public key is **mainly used to prove the server's identity (authentication)**.
- The actual encryption key is created using a **key exchange algorithm** like **ECDHE**.

This is why modern TLS provides **Perfect Forward Secrecy**.

Let's first understand the old method because it's easier.

---

# Old TLS (RSA Key Exchange)

Imagine:

Server has:

```
Public Key:
PUB123

Private Key:
PRIV123
```

Server sends the certificate:

```
Certificate

Domain:
example.com

Public Key:
PUB123

Signed by:
Let's Encrypt
```

Client receives it.

Browser verifies:

```
✓ Certificate valid

✓ Not expired

✓ Signed by trusted CA

✓ Domain matches
```

Now browser creates a random secret.

Suppose:

```
Secret = APPLE123
```

Browser encrypts it using server's public key.

```
Encrypt(APPLE123, PUB123)

Result:

X9@#AA7812$%S
```

Browser sends:

```
X9@#AA7812$%S
```

Anyone intercepting it sees only:

```
X9@#AA7812$%S
```

Server decrypts it using its private key.

```
Decrypt(
X9@#AA7812$%S,
PRIV123
)

↓

APPLE123
```

Now both know:

```
APPLE123
```

This becomes the session key.

---

## Why was this abandoned?

Imagine an attacker records your encrypted traffic today.

Five years later, the attacker somehow steals the server's private key.

Now he can decrypt:

```
X9@#AA7812$%S

↓

APPLE123
```

Now he knows the session key.

He can decrypt your old HTTPS traffic.

Bad.

This is why RSA key exchange is no longer used.

---

# Modern TLS (ECDHE)

Now comes the interesting part.

**The server's public key is NOT used to encrypt the session secret anymore.**

Instead, client and server **both independently calculate the same secret**.

This feels like magic at first.

Let's understand with a simple analogy.

---

# Paint analogy

Suppose:

Server chooses

```
Blue
```

Client chooses

```
Yellow
```

Both agree on a public color

```
White
```

Everyone can see

```
White
```

Now

Server mixes

```
White + Blue

↓

Light Blue
```

Client mixes

```
White + Yellow

↓

Light Yellow
```

They exchange these mixed colors.

Attacker sees

```
White

Light Blue

Light Yellow
```

But he never sees:

```
Blue

or

Yellow
```

Now

Server mixes

```
Light Yellow + Blue

↓

Green
```

Client mixes

```
Light Blue + Yellow

↓

Green
```

Both independently get

```
Green
```

Attacker cannot.

ECDHE works similarly but uses mathematics instead of paint.

---

# Real TLS 1.3 Handshake

Let's say you open

```
https://amazon.com
```

---

## Step 1

Browser →

Client Hello

It sends:

```
Hi!

Supported TLS versions:

TLS 1.3
TLS 1.2

Supported Cipher Suites

AES-GCM

ChaCha20

Random Number

Client Random

ECDHE Public Key

Extensions

SNI

Supported Groups

Signature Algorithms
```

Notice:

The browser already sends **its ECDHE public key**.

---

Example:

```
Client Private Key

13
```

From this,

browser mathematically creates

```
Client Public Key

834729
```

Browser sends

```
834729
```

It never sends

```
13
```

---

# Step 2

Server replies:

```
Server Hello
```

It contains:

```
TLS Version

TLS 1.3

Chosen Cipher

AES-GCM

Server Random

Server ECDHE Public Key

Certificate

Certificate Verify

Finished
```

Suppose

Server Private

```
91
```

Server Public

```
581932
```

Server sends only

```
581932
```

---

# Wait...

Now both exchanged only PUBLIC KEYS.

Nobody has exchanged the secret.

How do they get the same secret?

This is where Diffie-Hellman mathematics comes in.

---

Client has

```
Private

13
```

Server Public

```
581932
```

Client computes

```
Secret =
Math(
13,
581932
)

↓

984761234
```

---

Server has

```
Private

91
```

Client Public

```
834729
```

Server computes

```
Math(
91,
834729
)

↓

984761234
```

Both get

```
984761234
```

Exactly the same.

Neither side sent

```
984761234
```

over the network.

It was **derived**, not transmitted.

---

# Attacker sees

```
Client Public

834729

Server Public

581932
```

But does NOT know

```
13

or

91
```

Without those private values, the attacker cannot derive the shared secret.

---

# Then what?

Now both have

```
Shared Secret

984761234
```

TLS does **not** use this directly.

It runs it through a Key Derivation Function (HKDF).

Suppose:

```
Shared Secret

984761234

↓

HKDF

↓

Encryption Key

ABC111

↓

MAC Key

XYZ555

↓

IV

PQR222
```

Now actual encryption begins.

---

# Why does the server still send its certificate?

Because otherwise an attacker could pretend to be Amazon.

Imagine:

You connect.

Attacker intercepts.

He sends his own ECDHE public key.

You generate a shared secret with him.

Everything works.

But you're talking to the attacker.

To prevent this, the server sends its certificate.

The certificate contains the server's long-term public key and is signed by a trusted CA.

The server then **digitally signs parts of the handshake** (including its ephemeral ECDHE key) using its **private key corresponding to the certificate**. The browser verifies that signature using the **public key in the certificate**.

This proves:

- "I really own the private key for `amazon.com`'s certificate."
- "This ECDHE public key really came from the legitimate server."

Without this signature, a man-in-the-middle could substitute their own ECDHE public key.

---

# What exactly is "Finished"?

After both derive the shared secret, they each send a **Finished** message.

This message is encrypted using the newly derived keys and contains a cryptographic check of the entire handshake.

If the checks match, both sides know:

- We derived the same secret.
- Nobody modified the handshake.
- We can safely start exchanging application data.

Only after this point does your browser send the encrypted HTTP request, such as:

```
GET / HTTP/1.1
Host: amazon.com
```

---

# Complete TLS 1.3 Flow

```
Browser                              Server
   |                                    |
   |------ ClientHello ---------------->|
   |   TLS versions                     |
   |   Cipher suites                    |
   |   Client Random                    |
   |   Client ECDHE Public Key          |
   |                                    |
   |<----- ServerHello -----------------|
   |      Chosen cipher                 |
   |      Server Random                 |
   |      Server ECDHE Public Key       |
   |      Certificate                   |
   |      CertificateVerify             |
   |      Finished                      |
   |                                    |
   | Derive Shared Secret               |
   | Verify Certificate                 |
   | Verify Signature                   |
   |                                    |
   |------ Finished ------------------->|
   |                                    |
   |==== Encrypted HTTP Starts ========>|
```

Once this flow makes sense, you're already ahead of what many networking courses cover.

A natural next topic is **the mathematics behind Diffie-Hellman and Elliptic Curve Diffie-Hellman (ECDHE)**—specifically *how two computers can compute the same shared secret without ever transmitting it*. That's the core cryptographic idea behind TLS 1.3, and understanding it makes the entire handshake click into place.
