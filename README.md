# How Group Theory Secures Online Banking and E-Commerce: A Study of RSA Cryptography

A study of RSA public-key cryptography through the lens of group theory. The project shows how the multiplicative group of integers modulo *n* and Euler's theorem make RSA work, walks through a complete numerical example, analyses its security, and looks at real-world use in banking, e-commerce and TLS.

**Department of Mathematics, Mahindra University**
**Submitted to:** Professor Pradeep Kumar
**Date:** November 2025

## Team

- Geetika Pachauri
- Anusha Anindita
- Harshil Pansala
- Shirsha Pattanaik
- Pearl Mendapara

## Overview

RSA operates inside the finite abelian group `(Z*ₙ, ×)`, where `Z*ₙ = { a ∈ Zₙ : gcd(a, n) = 1 }`. For `n = pq` with distinct primes `p` and `q`, the group has order `φ(n) = (p − 1)(q − 1)`. Euler's theorem (a consequence of Lagrange's theorem) says `a^φ(n) ≡ 1 (mod n)`, and this "return to the identity" is why decryption recovers the message.

### Algorithm

| Step | Operation |
|------|-----------|
| 1 | Choose two distinct large primes `p`, `q` |
| 2 | Compute `n = p·q` |
| 3 | Compute `φ(n) = (p − 1)(q − 1)`, the group order |
| 4 | Choose `e` with `1 < e < φ(n)` and `gcd(e, φ(n)) = 1` |
| 5 | Compute `d` with `e·d ≡ 1 (mod φ(n))` (Extended Euclidean Algorithm) |
| 6 | Public key `(e, n)`, private key `(d, n)` |

- **Encrypt:** `c ≡ mᵉ (mod n)`
- **Decrypt:** `m ≡ cᵈ (mod n)`

### Correctness

Since `ed ≡ 1 (mod φ(n))`, write `ed = 1 + kφ(n)`. Then for `m ∈ Z*ₙ`:

```
m^(ed) = m^(1 + kφ(n)) = m · (m^φ(n))^k ≡ m · 1^k ≡ m  (mod n)
```

## Worked Example

Toy parameters only. Real keys use primes of 1024 bits or more (2048/4096-bit moduli).

| Quantity | Value |
|----------|-------|
| `p`, `q` | 61, 53 |
| `n = p·q` | 3233 |
| `φ(n)` | 3120 |
| `e` | 17 (`gcd(17, 3120) = 1`) |
| `d` | 2753 (`17 × 2753 = 15 × 3120 + 1`) |
| Public key | (17, 3233) |
| Private key | (2753, 3233) |
| Message `m` | 65 |
| Ciphertext `c = 65¹⁷ mod 3233` | **2790** |
| Decrypted `2790²⁷⁵³ mod 3233` | **65** |

Square-and-multiply for encryption: `65² ≡ 992`, `65⁴ ≡ 1232`, `65⁸ ≡ 1547`, `65¹⁶ ≡ 789`, so `65¹⁷ ≡ 789 × 65 ≡ 2790 (mod 3233)`.

## Security

RSA's security rests on the hidden group order `φ(n)`:

- Computing `d` from `(e, n)` requires `φ(n)`, which requires factoring `n = pq`.
- Breaking RSA is also equivalent to extracting an `e`-th root modulo `n`, which is infeasible without `d`.
- For a 2048-bit modulus the group is far too large for brute force.
- Prime quality matters: primes must be large and random (the report cites Cloudflare's lava-lamp entropy wall as an example).

## Applications

- **Online banking:** login and OTP protection, transaction security (RSA key exchange plus AES for data), ATM certificates, compliance (PCI DSS, RBI Cybersecurity Framework, ISO 27001)
- **E-commerce:** payment processing, card-data protection, payment gateways (PayPal, Stripe, Razorpay), PCI DSS
- **HTTPS/TLS:** RSA certificates, key exchange and hybrid encryption (RSA for the session key, AES for bulk data, HMAC for integrity), certificate authorities
- **Digital signatures:** hash the message, sign the hash with the private key, verify with the public key. Used for software signing, S/MIME email, legal documents and blockchain validation

## RSA vs ECC

| Feature | RSA | ECC |
|---------|-----|-----|
| Key size | 2048–4096 bits | 256–521 bits |
| Speed | Slower | Faster |
| Computation cost | High | Low |
| Mobile/IoT | Less efficient | Highly efficient |
| Typical use | Banking certificates, legacy systems | Modern TLS, mobile apps |

Report recommendations: RSA-2048 is still secure and compatible, ECC (e.g. P-256) for new systems, and hybrid deployments where multi-device compatibility matters.

## Quantum Computing and the Future

Shor's algorithm can factor large integers efficiently on a sufficiently large quantum computer, which would break RSA. NIST is standardising post-quantum schemes (CRYSTALS-Kyber, CRYSTALS-Dilithium). The report recommends RSA-3072/4096 for long-term needs, ECC for constrained devices, migration planning towards post-quantum cryptography, and crypto-agility.

## Python Demo

Save as `rsa_demo.py` and run with `python rsa_demo.py` (standard library only, educational use only):

```python
# rsa_demo.py - RSA demonstration (educational only)

def egcd(a, b):
    """Extended Euclidean Algorithm"""
    if b == 0:
        return (1, 0, a)
    x, y, g = egcd(b, a % b)
    return (y, x - (a // b) * y, g)

def modinv(a, m):
    """Compute modular inverse"""
    x, y, g = egcd(a, m)
    if g != 1:
        raise Exception('No inverse')
    return x % m

def validate_message(m, n):
    """Validate message in Z*_n"""
    if not (1 <= m < n):
        raise ValueError("Invalid range")
    if egcd(m, n)[2] != 1:
        raise ValueError("gcd must be 1")

# Example parameters (SMALL - demo only!)
p = 61
q = 53
n = p * q
phi = (p - 1) * (q - 1)
e = 17
d = modinv(e, phi)

print("RSA KEY GENERATION")
print(f"Prime p: {p}")
print(f"Prime q: {q}")
print(f"Modulus n: {n}")
print(f"phi(n): {phi}")
print(f"Public exponent e: {e}")
print(f"Private exponent d: {d}")
print(f"Public Key: ({e}, {n})")
print(f"Private Key: ({d}, {n})")

# ENCRYPTION
m = 65
validate_message(m, n)
c = pow(m, e, n)
print(f"\nEncrypting m={m}")
print(f"Ciphertext c: {c}")

# DECRYPTION
m_dec = pow(c, d, n)
print(f"\nDecrypting c={c}")
print(f"Recovered m: {m_dec}")

# Verification
if m == m_dec:
    print("SUCCESS: Decryption works!")
else:
    print("FAILURE")
```

Expected output includes `d = 2753`, ciphertext `2790`, recovered message `65`, and `SUCCESS`.

## Repository Contents

```
.
├── README.md
├── report.pdf                                                   # Full report
├── How-Group-Theory-Secures-Online-Banking-and-E-Commerce.pdf   # Companion document
└── rsa_demo.py                                                  # Python demo (above)
```

## References

1. Rivest, R. L., Shamir, A., & Adleman, L. (1978). A method for obtaining digital signatures and public-key cryptosystems. *Communications of the ACM*, 21(2), 120–126.
2. Katz, J., & Lindell, Y. (2014). *Introduction to Modern Cryptography* (2nd ed.). CRC Press.
3. Dummit, D. S., & Foote, R. M. (2004). *Abstract Algebra* (3rd ed.). John Wiley & Sons.
4. NIST. Post-Quantum Cryptography Standardization. https://csrc.nist.gov/projects/post-quantum-cryptography
5. Cloudflare (2017). Randomness 101: LavaRand in Production. https://blog.cloudflare.com/randomness-101-lavarand-in-production/
