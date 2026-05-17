# Ed25519 Key Derivation Security Analysis

---

## 1. Cryptographic Security and Hard Problems

To prove a security improvement, we rely on the **reduction** of the scheme's security to a well-known hard mathematical problem. In the context of Ed25519 (Edwards-curve Digital Signature Algorithm), the foundational problem is the **Discrete Logarithm Problem (DLP)**.

### The Discrete Logarithm Problem

Given a point $A$ on an elliptic curve and a generator point $G$, find the scalar $k$ such that:

$$A = k \cdot G$$

In a secure curve like Curve25519, this is computationally infeasible for attackers. Your scheme relies on deriving new keys, but as long as the base secret $k$ is protected by the DLP, the scheme remains robust.

---

## 2. Mathematical Construction of Key Derivation

Your proposed scheme involves deriving a sequence of public keys $A_n$ using a series of multipliers (scalars) $s_n$ derived from a pre-agreed series or hash.

### Key Derivation Formulas

- **Master Public Key:** $A_{base} = k \cdot G$
- **Derived Public Key ($A_n$):**
  $$A_n = s_n \cdot A_{base} = s_n \cdot (k \cdot G)$$
- **Derived Private Key ($k_n$):** $k_n = k \cdot s_n \pmod{L}$

### Mathematical Consistency

Because scalar multiplication is associative, the signer can use $k_n$ to sign a message, and the verifier can use $A_n$ to check it:

$$k_n \cdot G = (k \cdot s_n) \cdot G = s_n \cdot (k \cdot G) = A_n$$

This allows for "offline" derivation where the signer and verifier stay in sync without communication.

---

## 3. Threat Model and Side-Channel Attacks (SCA)

### Verifier-Side SCA

If an attacker performs an SCA on the **Verifier hardware**:

- The Verifier only processes **public information**: $A_n, s_n, M, (R, s)$.
- The master secret $k$ never enters the Verifier's memory.
- **Conclusion:** An SCA on the verifier cannot leak the master private key $k$.

### Protection Against Replay Attacks

By rolling the public key $A_n$ for every session, the system prevents replay attacks. A signature $(R, s)$ valid for index $n$ will fail the verification equation at index $n+1$ because $A_n \neq A_{n+1}$.

---

## 4. Multi-Signature Analysis and Nonce Security

An attacker observing multiple signatures $(R_1, s_1), (R_2, s_2), \ldots$ cannot solve for $k$ because each signature introduces a new secret "nonce" $r_n$.

### The Signing Equation

For each signature $n$:

$$s_n = r_n + H(R_n, A_n, M_n) \cdot (k \cdot s_n) \pmod{L}$$

### Critical Security Requirement: Nonce Uniqueness

You must never reuse a nonce $r$. If $r_1 = r_2$ for two different signatures, the attacker can perform the following calculation to extract the master key $k$:

$$s_1 - s_2 = (h_1 \cdot s_{multiplier1} - h_2 \cdot s_{multiplier2}) \cdot k$$

$$k = \frac{s_1 - s_2}{h_1 \cdot s_{m1} - h_2 \cdot s_{m2}} \pmod{L}$$

To prevent this, nonces should be generated deterministically: $r_n = \text{Hash}(k, A_n, M)$.
