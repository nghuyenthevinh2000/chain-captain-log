# Understanding of BIP340 schnorr signature (Taproot upgrade)

Why understanding of how Schnorr signature works is crucial? 
1. this is to understand how Discreet Lock Contract (DLC) works, the innovation underlying trustless Bitcoin Bridge
2. this is to understand how Lightning network leverage it for their Taproot assets innovation, enabling the first multi - hop transfer, precusor to DEX aggregator
3. this is to understand how RGB++ use it to expand the programmability layer on Bitcoin
4. this is to understand how BitVM use it for their ZK general computation verification
5. it explains the OP_CAT hype, the keyword for Bitcoin L2s hype

Taproot upgrade has started the great restoration of Bitcoin script, let's find out what this mean? Taproot bases on Schnorr signature scheme. It allows verification of multiple scripts through Schnorr key aggregation characteristic, multiple scripts can be verified with just one aggregated Schnorr public key. But how, it can verify multiple scripts?

## I. Schnorr signature scheme
1. Signature scheme: (R, s) - (R: Point, s: scalar)
2. Signature creation overview: sign(M, d, k) -> (R, s)
* M: "Common signature message" - BIP341
* d: private key
* k: nonce
3. Verification overview: $\text{verify}((R,s), M, P) \to \text{bool}$
* (R, s): Schnorr signature scheme
* M: message
* P: public key
### I. a. Signature creation break down
1. Public key derivation: $P = d \cdot G$
* d: private key
* G: Generator point (a pre - defined point on the Elliptic curve secp256k1)
* a special characteristic of secp256k1 is that given public key, and generator point, it is computationally infeasible to derive private key (the exact reason why can be explained by ChatGPT)

2. Public nonce commitment: $R = k \cdot G$ (first item R in (R, s))
* k: nonce
* G: pre - defined generator point on curve secp256k1
* a special characteristic of secp256k1 is that given nonce, and generator point, it is computationally infeasible to derive private key (the exact reason why can be explained by ChatGPT)

3. second item s in (R, s) =  $\text{H}(\text{"BIP0340/challenge"}, R \parallel P \parallel M) \cdot d + k$
* $x \parallel y$ : concatenate x and y
* $\text{H}(\text{tag}, x) = \text{SHA256}(\text{SHA256}(\text{tag}) \parallel \text{SHA256}(\text{tag}) \parallel x)$
* R: Public nonce commitment
* P: Public key derivation
* M: message
* d: private key
* k: nonce

#### I. b. Signature verification breakdown
1. Verification goal: $R = s \cdot G - e \cdot P$
* R: Public nonce commitment
* s: second item s in Schnorr signature (R, s)
* G: pre - defined generator point on curve secp256k1
* e: $\text{H}(\text{"BIP0340/challenge"}, R \parallel P \parallel m)$
* P: Public key

2. Why it works?
* s = $\text{H}(\text{"BIP0340/challenge"}, R \parallel P \parallel M) \cdot d + k$
* $s \cdot G = (e \cdot d + k) \cdot G$
* $s \cdot G = e \cdot d \cdot G + k \cdot G$
* $s \cdot G = e \cdot P + k \cdot G$ (as $P = d \cdot G$)
* $s \cdot G = e \cdot P + R$ (as $R = k \cdot G$)
* $R = s \cdot G - e \cdot P$

#### I. c. Schnorr key aggregation
1. Key aggregation for multiple participants to jointly own a single public key and produce a single signature that proves possession of their corresponding private keys

2. Provided that there are participants $1, 2, 3, \ldots$, the public key aggregation would be: $P_{\text{agg}} = P_1 + P_2 + \ldots + P_n = (d_1 + d_2 + \ldots + d_n) \cdot G$ 
* $d_1, d_2, d_3, \ldots$ private keys of participants 1, 2, 3, ...
* G: pre - defined generator point on curve secp256k1
* $P_1, P_2, P_3, \ldots$ public keys of participants 1, 2, 3, ...

3. Provided that there are participants $1, 2, 3, \ldots$, the public nonce commitment aggregation would be: $R_{\text{agg}} = R_1 + R_2 + \ldots + R_n = (k_1 + k_2 + \ldots + k_n) \cdot G$ 
* $k_1, k_2, k_3, \ldots$ nonce from participants 1, 2, 3, ...
* G: pre - defined generator point on curve secp256k1
* $R_1, R_2, R_3, \ldots$ public nonce commitment aggregation of participants 1, 2, 3, ...

4. Creating signature: $s_\text{agg} = \text{H}(R_\text{agg} \parallel P_\text{agg} \parallel M) \cdot (d_1 + d_2 + \ldots + d_n) + (k_1 + k_2 + \ldots + k_n)$
* $R_\text{agg}$ Aggregated public nonce commitment
* $P_\text{agg}$ Aggregated public key
* $d_{i}$ private key of participant i
* $k_{i}$ nonce from participant i

5. Verification: $R_\text{agg} = s_\text{agg} \cdot G - e_\text{agg} \cdot P_\text{agg}$
* $R_\text{agg}$ Aggregated public nonce commitment
* $P_\text{agg}$ Aggregated public key
* $s_{agg}$ Aggregated signature
* $e_{agg}$ Aggregated $\text{H}(R_\text{agg} \parallel P_\text{agg} \parallel M)$
* G: pre - defined generator point on curve secp256k1
* The moment I look at this, I understand how Schnorr signature is powerful. Given public aggregated R, P, no matter how many participants are involved, only one signature is required on the blockchain. The only size limit is the weight limit of a block which is 4mb. In the context of multiple complex scripts, it can include a bunch of locking scripts. Smart contract on UTXO can be achieved by chaining a bunch of locking scripts together (more details are not covered in here)

## II. Ordinals
Ordinals is actually quite simple, and exploits a bug feature in Bitcoin Core. Ordinals is only possible with the new Taproot upgrade. The element script size limit is 520bytes, but by obfuscating data as program script, inscriptions circumvent this limit. The whole Ordinals trend can be stopped if this bug is fixed. Taproot allows script verification with only the information of aggregated public nonce commitment, and public key thus masking the whole script data.

What is the different between element script and program script? a program script contains multiple operation scripts. An element script is at most 520 bytes: https://github.com/bitcoin/bitcoin/blob/v26.1/src/script/script.h#L26

1. Program script building source code: https://github.com/ordinals/ord/blob/master/src/inscriptions/inscription.rs#L119
2. Transaction example: https://mempool.space/tx/521f8eccffa4c41a3a7728dd012ea5a4a02feed81f41159231251ecf1e5c79da

```
OP_FALSE
OP_IF
	OP_PUSH "ord"
	OP_PUSH 1
	OP_PUSH "text/plain;charset=utf-8"
	OP_PUSH 0
	OP_PUSH "Hello, world!"
OF_ENDIF
```

Program script will then be parsed into a bytes array to be inscribed on Bitcoin and later indexed by Ordinals protocol (more details are not covered in here)

## III. OP_CAT
> “In the context of Bitcoin, the most useful definition of covenant is that it’s when the scriptPubKey of a UTXO restricts the scriptPubKey in the output(s) of a tx spending that UTXO.” —Anthony Towns

Background: script cannot enforce velocity limits, restrict coins to going to certain locations, or anything like that, because the Script execution environment does not have access to transaction data. Ultimately, adding covenants to Bitcoin would mean adding the ability to introspect transactions to Script. This ability is the main core feature of OP_CAT.

You achieve covenants by selectively choosing which tx can spend that UTXOs. How? by inspecting spending transaction hashes which compose of the covenant script and among other things. Transaction hash needs to be **available already** in the unlocking script to perform the evaluation, but **not possible** because the transaction hash can only be **derived after** that transaction is created.

You achieve this by abusing the Taproot scheme, here is how?

$s = \text{H}(\text{"BIP0340/challenge"}, R \parallel P \parallel M) \cdot d + k$

By setting d and k to 1, 
* $R = G$
* $P = G$
* $s = \text{H}(\text{"BIP0340/challenge"}, G \parallel G \parallel M) + 1$

Now, s represents the transaction data M (s is the transaction hash of M, and it can be derived at script evaluation, this is our goal). Reconstructing the transaction hash s with just transaction data M.

Transaction data will be OP_CAT together, and then hashed before going into the message field M. Then do more SHA256, and CAT to derive $\text{H}(\text{"BIP0340/challenge"}, G \parallel G \parallel M)$. But  $(\text{G}, \text{H}(\text{"BIP0340/challenge"}, G \parallel G \parallel M))$ is not verifiable, it needs to be $(\text{G}, \text{H}(\text{"BIP0340/challenge"}, G \parallel G \parallel M) + 1)$. I don't quite get why Andrew Poelstra manipulates the signature like so to include that "+1".

#### III. a. OP_CAT vault
## References
1. https://www.youtube.com/watch?v=U5qcL0hI30k&t=13016s
2. https://www.binance.com/de/square/post/1000337190129
3. https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki
4. https://www.wpsoftware.net/andrew/blog/cat-and-schnorr-tricks-i.html