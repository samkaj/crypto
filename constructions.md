OTP
------
#otp 
- one time pad is a perfectly secure cryptographic scheme.
- $E_k(m)=k\oplus m \rightarrow c$
- $D_k(c) = k \oplus c = k \oplus \underbrace{k \oplus m}_{c}=m$ 
- #perfect-secrecy: $Pr[E(k,m_0) = c] = Pr[E(k,m_1) = c]$

OTP is perfectly secure, but it cannot reuse the key, and the key len has to be the same as the message length, which is bad and annoying to work with.
____
Block ciphers
----------------------
#### ECB
#ecb, electronic code book
Works in parallel for both encryption and decryption. The message is divided into blocks of fixed length, then each block is encrypted with the key, using AES typically. The ciphertext is a concatenation of all the encrypted blocks. Decryption is done via AES typically. 

It is an insecure mode of operation, since it contains no randomness and consequently, the same message and key will always produce the same ciphertext. Furthermore, since each bit is encrypted individually, patterns will leak through. If we take an image, all black pixels will produce the same cipher and it will be easy to see what the original image might look like.
#### CBC
#cbc , cipher block chaining
Encryption works by XORing the current block with the previous. The first block is XORed with an IV. This ensures that each ciphertext depends on the preceding blocks. The IV is used for ensuring that identical messages don't produce the same ciphertext. Encryption is serializable, decryption parallelizable.

**Encryption:**
- $C_0 = IV$
- $C_i = E_k(P_i \oplus C_{i-1})$
**Decryption:**
- $C_0 = IV$
- $P_i=D_k(C_i) \oplus C_{i-1}$
#### CTR
#ctr #stream-cipher
Fully parallelizable. Uses a nonce (same as IV) which is then incremented for each block, that is why it is called a counter. Encryption works by sending the nonce and key into the encryption (AES), the result is then XORed with the ciphertext.

Decryption works by using the corresponding counter with the key, using the **encryption** algorithm again, the ciphertext is then XORed with the result to get the plaintext back.

This operation uses a counter block which contains the nonce and counter concatenated when the nonce is not random! When it is random, you can use any inversible actions, such as addition, concatenation or XOR. Since concatenating works for both, I'd stick to that..
____
RSA
-------
#rsa 
Works in four steps:
### 1. Key generation
This is the most complicated part.
1. Choose two large primes $p,q$.
2. Compute $n=pq$.
	- $n$ is used as the modulus for the public and private keys. It is the key length.
	- $n$ is released as part of the public key, and is therefore public.
3. Compute $\lambda(n)$, where $\lambda$ is Carmichael's totient function. $\lambda(n)=lcm(p-1, q-1)$. The lcm can be calculated as follows: $lcm(a,b)= \frac{|ab|}{gcd(a,b)}$. $\lambda(n)$ must be kept secret.
4. Choose an integer $e$ s.t. $2 < e < \lambda(n)$ and $e, \lambda(n)$ are coprime. $e$ is usually chosen to be $2^{16}+1=65537$. $e$ is public information and sent with the public key.
5. Determine $d = e^{-1} \mod \lambda(n)$. This is the *private key exponent*.
**Public information:** $n,e,d$
**Private information:** $p,q,\lambda(n)$
#### 2. Key distribution
Given that Bob wants to send a message to Alice. Alice sends her public key $(n,e)$ to Bob and keeps her secret to herself.
#### 3. Encryption
With the public key from Alice, bob can encrypt the message M.
1. Bob turns $M$ into an integer $m$ (the padded plaintext), such that $0\leq m \lt n$, by using a padding scheme. 
2. He computes $c=m^e (\mod n)$ and sends it to Alice.
#### 4. Decryption
Alice receives $c$ and uses $c^d=(m^e)^d = m\ (\mod n)$. 
#### Signing
It is possible to encrypt a message which may be decrypted by anyone, but only encrypted by one person, this is a digital signature.
1. Alice wants to send a signed message to Bob
2. She produces a hash of the message, $h=hash(m)$
3. She computes $h^d \mod n$, and attaches is to the message.
4. Bob receives the signed message ($h^d$), and uses the same hash algo in conjunction with Alice's pk
5. Bob raises the signature to the power of e mod n, computes $(h^e)^d = h^{ed} = h^{de} = (h^d)^e = h \mod n$. If $h$ matches the one Alice sent, it is correctly signed and accepted.

#### Blind signatures
Alice chooses a secret random value $r$ and computes $(r^ec)^d \mod n$. Using Euler theorem, the following is equiv. $(r^ec)^d = rc^d \mod n$, so the effect of $r$ can be removed by multiplying by its inverse.
- $E_k(m)=(mr)^e \mod n = c,\ gcd(r,n)=1$
- $D_k(c) = (cr)^{ed} = cr \mod n \implies c = crr^{-1} \mod n$
In this way, since it is a blind signature, the signer can digitally sign the message without learning the contents.

_______
Diffie-Hellman key exchange
-----------------------------------------------
### Setup
Let $p$ be a large prime. Find a generator $g$ of a subgroup of prime order $q$ in $\mathbb{Z}_p^*$. Let $p,q,g$ be public.
### Key exchange
Alice and bob wants to exchange keys. 
Both parties know $p,q,g$
1. Alice picks $a \leftarrow \$\mathbb{Z}_q^*$
2. Alice computes $A=g^a\mod p$
3. Alice sends $A$ to Bob
4. Bob picks $b \leftarrow \$\mathbb{Z}_q^*$
5. Bob computes $B=g^b\mod p$
6. Bob sends $B$ to Alice
7. Bob computes the shared secret $K=A^b$
8. Alice computes the shared secret $K=B^a$
___
ElGamal
-------------
Uses Diffie-Hellman key exchange to establish a shared secret, then using OTP for encrypting the message.
### Keygen
Alice generates a key pair.
- Generate a cyclic croup $G$ of order $q$ with a generator $g$.
- Choose $x$, a random element from $\{1,\dots,q-1\}$
- Compute $h=g^x$
- Alice keeps $x$ as her private key. The public key consists of $(G,q,g,h)$
### Encryption
Bob uses Alice's public key to encrypt the message $M$:
- Map the message $M$ to an element $m$ of $G$ using a reversible mapping function
- Choose $y$, a random element from $\{1,\dots,q-1\}$
- Compute $s=h^y$, this is the **shared secret**
- Compute $c_1=g^y$
- Compute $c_2=m\cdot s$
- Bob sends $(c_1,c_2)$ to Alice
### Decryption
Alice now decrypts the pair $(c_1,c_2)$ with her private key:
- Compute $s=c_1^x$ this is because $c_1=g^y \implies c_1^x=g^{xy} = h^y$
- Compute the inverse of $s$ in $G$ using the extended Euclidean algorithm for example
- Compute $m=c_2\cdot s^{-1}$. This produces the original message since $c_2=m\cdot s\implies c_2\cdot s^{-1}=(m\cdot s)\cdot s^{-1} = m\cdot e = m$
- Map $m$ back to the plaintext $M$
___
Additive secret sharing
--------------------------------------
Secret sharing scheme that involves breaking a secret into multiple shares/fragments that prevents a single shareholder from having complete knowledge of the original secret.

Additive secret sharing is done by splitting up the secret such that if everyone adds their share it will build the original secret.

There are two phases, share and reconstruct.
- Let $N$ be a large integer and $s \in \mathbb{Z}_N$ be the secret yo be shared.
- $Share(s): s_1,\cdots,\ s_{n-1} \leftarrow \$\mathbb{Z}_N, s_n = s - (s_1+\cdots+s_{n-1}) \mod N$, return $\{s_1,\cdots, s_n\} \subset \mathbb{Z}_n$.
- $Reconstruct(s_1,\cdots,s_n)$: compute $s = s_1 +\cdots+s_n \mod N$
Some properties:
- Algorithms are efficiently computable
- $t$-correctness: any set of $t$ parties can reconstruct $s$ together
- $(t-1)$-security: any subset of less than the whole set of target cannot compute the secret
___
Shamir secret sharing
------------------------------------
This scheme uses polynomials to construct a secret. By having a specific degree polynomial, set the shares as coefficients to it which together solve the polynomial, which is the secret.
### Share
$Share(s)$, given the value $s \in \mathbb{Z}_p$ where $p$ is a prime or a power of a prime, do:
- Sample $t-1$ random values, where $t$ is the number of parties, $a_1,\cdots, a_{t-1}\leftarrow \$\mathbb{Z}_p$
- Construct the polynomial $f(x) = s+ a_1x + a_2x^2+\cdots+a_{t-1}x^{t-1}\in \mathbb{Z}_p[x]$
- Compute the $n$ shares by evaluating $f(x)$ on $n$ *distinct* points: $s_1=f(i)\ for\ i \in \{1,\cdots,n\}$ 
- Send the share to $s_i$ to party $P_i$ via a secure channel
### Reconstruct
$Reconstruct(s_{i_1},s_{i_2},\cdots,s_{i_t})$: let $I=\{i_1,\cdots i_t\}$ be the set of distinct share indices, do:
- Compute the Lagrange coefficients for the set $I$: $\mathscr{l}_i=\prod_{j\in I \setminus \{i\}}\frac{j}{j-i} \mod p$
- Recover the secret using the sum and coefficients: $s=\sum_{i\in I}(s_i\cdot \mathscr{l}_i(0))\mod p$. 
	- Clarification: $\mathscr{l}_i(0) = \frac{-s_1}{i_1-i} + \cdots + \frac{-s_t}{i_t-i} \mod p$, where each term $\frac{-s_j}{i_j-i}$ is evaluated modulo $p$.
___
Pedersen Commitment Scheme
----------------------------------------------------
Let $\mathbb{G}=\langle g \rangle$ be a cyclic group of primer order $q$, and $h$ be a random element in $\mathbb{G}\setminus g$. Let $q,g,h$ be public. The commitment function is as follows: 
- $Commit(m,r) = g^mh^r \mod p = c, r \leftarrow \$\mathbb{Z}_q, p=2q+1$
- It is important that $p$ is prime, since it will guarantee that $\mathbb{G}$ is a subgroup of the larger $\mathbb{Z}_p^*$.
___
Schnorr
-------------
### Identification
Given two large primes, $p, q=2p+1$, $g, \langle g \rangle = \mathbb{G}$, where the subgroup of $\mathbb{Z}_p^*$ is of prime order $q$, we also have the secret key $x$. The private key is the witness identifying the prover. The public key is $X=g^x \mod p$. The identification works as follows:
1. The prover chooses some random number $v$ and sends the commitment, $V=g^v\mod p$ to the verifier.
2. The verifier chooses some random number $c$, known as the challenge, and sends it to the prover.
3. The prover computes the response and returns it to the verifier, $b=v+cx \mod q$.
4. The verifier accepts iff $g^b=V\cdot X^c$

### $\Sigma$-protocol
Given a relation $R$, a prover $V$, and a verifier $V$, a $\Sigma$-protocol satisfies the following properties:
1. **Completeness**: if $P$ and $V$ follow the protocol on a given input, $x$ and private input $w$ to $P$ where $(x,w)\in R$, then $V$ always accepts
2. **Special soundness**: there exists a PPT algorithm $\mathscr{E}$, the extractor, that given any $x$ and any pair of accepting transcripts $(a,e,z)$ and $(a,e',z')$ for $x$ with $e\neq e'$, outputs $w$ s.t. $(x,w)\in R$.
3. **SHVZK**: for every $x,w \in R$ and every $e \in \{0,1\}^t$ there exists a PPT algorithm $Sim$, the simulator, which given input $(x,e)$ outputs transcripts $(a,e,z)$ that are distributed like real conversations: $\{Sim(x,e)\}=\{\langle P(x,w), V(x,e)\rangle\}$
	- The simulator exists to show that the verifier doesn't learn anything from the interaction with the prover, except that they know the secret.
	- If the simulator can generate a transcript that is indistinguishable from a real one, then it means that the verifier cannot be learning anything about the secret, since the simulator generates the transcript without knowing the secret!
