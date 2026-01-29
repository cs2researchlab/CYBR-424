# RSA & Diffie–Hellman (DH) Hands‑On Lab

- **RSA**: key generation, encryption/decryption, signing/verification  
- **Diffie–Hellman (DH)**: shared secret derivation, session key creation, symmetric encryption (AES)

**Real-world story:** Alice and Bob establish a **shared session key** using DH, then use **RSA** to protect secrets and authenticate messages—similar to how modern secure systems combine asymmetric and symmetric cryptography.

---

## Lab 0 — Setup (Linux + Python)

### 0.1 Verify tools
```bash
openssl version
python3 --version
```

### 0.2 Install Python crypto library (for “real RSA” in Python)
```bash
python3 -m pip install --user cryptography
```

### 0.3 Create a workspace
```bash
mkdir -p Public_crypto/{rsa,dh,msg}
cd Public_crypto
```

---

# Part A — RSA (CLI): Key Generation + Encrypt/Decrypt + Sign/Verify

## A1) Generate RSA Keys (Private + Public)

### Step 1: Move into RSA working directory
```bash
cd rsa
```

**What this does**
- Changes the current directory to `rsa` where all RSA-related files are stored.

---

### Step 2: Generate a 2048-bit RSA Private Key
```bash
openssl genpkey -algorithm RSA -pkeyopt rsa_keygen_bits:2048 -out alice_rsa_priv.pem
```

**What this does**
- Creates Aliceâ€™s **RSA private key**, used for **decryption and digital signing**.

#### Command Explanation
- **`openssl`**  
  Invokes the OpenSSL cryptographic toolkit.
- **`genpkey`**  
  Generates a new **private key**.
- **`-algorithm RSA`**  
  Specifies **RSA** as the cryptographic algorithm.
- **`-pkeyopt rsa_keygen_bits:2048`**  
  Sets the key size to **2048 bits** (current security standard).
- **`-out alice_rsa_priv.pem`**  
  Saves the private key in PEM format to `alice_rsa_priv.pem`.

---

### Step 3: Extract the RSA Public Key
```bash
openssl pkey -in alice_rsa_priv.pem -pubout -out alice_rsa_pub.pem
```

**What this does**
- Derives Aliceâ€™s **public key**, which is shared with others for **encryption and signature verification**.

#### Command Explanation
- **`openssl`**  
  Runs the OpenSSL tool.
- **`pkey`**  
  Processes public/private key files.
- **`-in alice_rsa_priv.pem`**  
  Reads the existing RSA private key.
- **`-pubout`**  
  Extracts the **public key** from the private key.
- **`-out alice_rsa_pub.pem`**  
  Writes the public key to `alice_rsa_pub.pem`.

---

### Step 4: Inspect RSA Key Parameters
```bash
openssl pkey -in alice_rsa_priv.pem -text -noout | head -n 25
```

**What this does**
- Displays internal RSA parameters (modulus size, exponent) for inspection.

#### Command Explanation
- **`openssl pkey`**  
  Reads and displays key information.
- **`-in alice_rsa_priv.pem`**  
  Specifies the RSA private key file.
- **`-text`**  
  Outputs key details in human-readable form.
- **`-noout`**  
  Suppresses raw key output.
- **`| head -n 25`**  
  Shows only the first 25 lines to keep output readable.

---

### Key Concept (Summary)
- **Public key** â†’ encrypt data, verify signatures  
- **Private key** â†’ decrypt data, create signatures  

This asymmetric separation enables **confidentiality, integrity, and authentication**.

---


## A2) Encrypt a short secret message with the public key (OAEP) and decrypt with private key

### Create a message
```bash
echo "Meet at 10:30PM. Room 204." > secret.txt
```

### Encrypt using public key (OAEP padding — recommended)
```bash
openssl pkeyutl -encrypt -pubin -inkey alice_rsa_pub.pem \
  -in secret.txt -out secret.enc \
  -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256
```

### Decrypt using private key
```bash
openssl pkeyutl -decrypt -inkey alice_rsa_priv.pem \
  -in secret.enc -out secret.dec \
  -pkeyopt rsa_padding_mode:oaep -pkeyopt rsa_oaep_md:sha256

cat secret.dec
```

**What students learn:** Anyone can encrypt to Alice using her **public key**, but only Alice can decrypt using her **private key**.

---

## A3) Sign a file (integrity + authenticity) and verify with public key

### Create a “contract” file
```bash
echo "Approve payment: $5000 to Vendor Z on Jan-31." > contract.txt
```

### Sign (private key)
```bash
openssl dgst -sha256 -sign alice_rsa_priv.pem -out contract.sig contract.txt
```

### Verify (public key)
```bash
openssl dgst -sha256 -verify alice_rsa_pub.pem -signature contract.sig contract.txt && echo "✅ Signature valid"
```

### Try tampering
```bash
echo "Approve payment: $9000 to Vendor Z on Jan-31." > contract.txt
openssl dgst -sha256 -verify alice_rsa_pub.pem -signature contract.sig contract.txt || echo "❌ Signature failed (tampered)"
```

**What students learn:** Signatures detect tampering and prove who signed.

---

# Part B — Diffie–Hellman (CLI): Derive a Shared Session Key

## B1) Create DH parameters (group)

```bash
cd ~/crypto_lab/dh

# DH group parameters (2048-bit; may take a bit on slow machines)
openssl dhparam -out dhparams.pem 2048
```

**What students learn:** DH needs agreed group parameters (prime/modulus + generator).

---

## B2) Alice and Bob generate DH keypairs

```bash
# Alice keypair
openssl genpkey -paramfile dhparams.pem -out alice_dh_priv.pem
openssl pkey -in alice_dh_priv.pem -pubout -out alice_dh_pub.pem

# Bob keypair
openssl genpkey -paramfile dhparams.pem -out bob_dh_priv.pem
openssl pkey -in bob_dh_priv.pem -pubout -out bob_dh_pub.pem
```

---

## B3) Derive shared secrets (both sides) and compare

### Alice derives using (Alice private + Bob public)
```bash
openssl pkeyutl -derive -inkey alice_dh_priv.pem -peerkey bob_dh_pub.pem -out alice_shared.bin
```

### Bob derives using (Bob private + Alice public)
```bash
openssl pkeyutl -derive -inkey bob_dh_priv.pem -peerkey alice_dh_pub.pem -out bob_shared.bin
```

### Compare hashes (should match)
```bash
openssl dgst -sha256 alice_shared.bin
openssl dgst -sha256 bob_shared.bin
cmp -s alice_shared.bin bob_shared.bin && echo "✅ Shared secret matches"
```

**What students learn:** DH lets Alice and Bob compute the **same secret** without sending it directly.

---

## B4) Turn DH shared secret into a symmetric key and encrypt a message

### Make a 256-bit key from the shared secret (hash-based key derivation)
```bash
KEYHEX=$(openssl dgst -sha256 -binary alice_shared.bin | xxd -p -c 256)
echo "$KEYHEX" > session_key.hex
cat session_key.hex
```

### Encrypt a file using AES-256-CBC (demo)

```bash
echo "Session message: launch analysis at 02:00." > session_msg.txt

# random IV
openssl rand -hex 16 > iv.hex

# encrypt
openssl enc -aes-256-cbc -K "$(cat session_key.hex)" -iv "$(cat iv.hex)" \
  -in session_msg.txt -out session_msg.enc

# decrypt (Bob uses same derived key)
openssl enc -d -aes-256-cbc -K "$(cat session_key.hex)" -iv "$(cat iv.hex)" \
  -in session_msg.enc -out session_msg.dec

cat session_msg.dec
```

**Real-world meaning:** DH is typically used to create a **session key**, then **fast symmetric crypto (AES)** encrypts the actual data (similar to TLS/HTTPS).

---

## (Optional) Python Extensions (if you want students to see the math + real RSA)

### Toy DH math demo (not production secure)
Create `dh_toy.py`:
```python
# dh_toy.py (TOY DEMO - not production secure)
import secrets
import hashlib

p = 0xFFFFFFFB  # toy prime-like value (NOT secure)
g = 5

a = secrets.randbelow(p - 2) + 1  # Alice private
b = secrets.randbelow(p - 2) + 1  # Bob private

A = pow(g, a, p)  # Alice public
B = pow(g, b, p)  # Bob public

s_alice = pow(B, a, p)
s_bob   = pow(A, b, p)

print("Shared equal?", s_alice == s_bob)
print("Derived key (sha256 hex):", hashlib.sha256(str(s_alice).encode()).hexdigest())
```

Run:
```bash
python3 dh_toy.py
```

### Real RSA in Python using cryptography
Create `rsa_real.py`:
```python
from cryptography.hazmat.primitives.asymmetric import rsa, padding
from cryptography.hazmat.primitives import hashes
from cryptography.exceptions import InvalidSignature

private_key = rsa.generate_private_key(public_exponent=65537, key_size=2048)
public_key = private_key.public_key()

message = b"Payroll checksum: 9f2a... (demo)"

ciphertext = public_key.encrypt(
    message,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None,
    ),
)

plaintext = private_key.decrypt(
    ciphertext,
    padding.OAEP(
        mgf=padding.MGF1(algorithm=hashes.SHA256()),
        algorithm=hashes.SHA256(),
        label=None,
    ),
)

print("Decrypted OK?", plaintext == message)

signature = private_key.sign(
    message,
    padding.PSS(
        mgf=padding.MGF1(hashes.SHA256()),
        salt_length=padding.PSS.MAX_LENGTH,
    ),
    hashes.SHA256(),
)

try:
    public_key.verify(
        signature,
        message,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH,
        ),
        hashes.SHA256(),
    )
    print("Signature valid ✅")
except InvalidSignature:
    print("Signature invalid ❌")
```

Run:
```bash
python3 rsa_real.py
```

---

## Quick Cleanup (optional)
```bash
# Remove generated secrets after grading
rm -f secret.enc secret.dec contract.sig alice_shared.bin bob_shared.bin session_msg.enc session_msg.dec session_key.hex iv.hex
```

---

## Notes for GitHub
Suggested repo structure:
```
crypto_lab/
  RSA_and_DH.md
  rsa/
  dh/
  msg/
```

---

**End of lab.**
