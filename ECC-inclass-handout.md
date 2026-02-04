# ECC In-Class Handout
## Elliptic Curve Cryptography: Linux Hands-on Lab (30 minutes)

---

## Lab Scenario

**Scenario:**
Alice and Bob work for a tech company and need to securely exchange sensitive project files. They decide to use Elliptic Curve Cryptography for encryption and ECDH for key exchange.

**Your Tasks:**
1. Help Alice and Bob generate their ECC key pairs
2. Learn to sign and verify messages using ECC digital signatures
3. Perform ECDH key exchange to establish a shared secret
4. Verify that both parties arrive at the same shared secret

**Expected Outcomes:**
- Alice and Bob can sign messages with their private keys
- Signatures can be verified using the corresponding public keys
- Both Alice and Bob compute identical shared secrets through ECDH
- The shared secret should be 32 bytes (256 bits) for prime256v1 curve
- Modified messages fail signature verification

---

## Exercise 1: Generate ECC Private Key (3 minutes)

**Concept:** The private key is a random large integer that must be kept secret. It's the foundation of your cryptographic identity.

**Commands:**
```bash
# Generate a 256-bit ECC private key using secp256k1 curve
openssl ecparam -name secp256k1 -genkey -noout -out ecc_private.pem

# View the private key
openssl ec -in ecc_private.pem -text -noout
```

**Command Breakdown:**
- `ecparam`: Work with elliptic curve parameters
- `-name secp256k1`: Use the secp256k1 curve (256-bit, used in Bitcoin)
- `-genkey`: Generate a new private key
- `-noout`: Don't output the curve parameters (only the key)
- `-out ecc_private.pem`: Save private key to this file

**What you'll see:** Private key value (a large hexadecimal number), public key point (x, y coordinates), and curve information.


### Question 1 
What is the primary advantage of ECC over RSA?

A) ECC is older and more tested  
B) ECC provides equivalent security with shorter key lengths  
C) ECC is easier to implement  
D) ECC doesn't use prime numbers  

---

## Exercise 2: Extract Public Key from Private Key (3 minutes)

**Concept:** The public key is derived from the private key through elliptic curve point multiplication (Public Key = Private Key × Base Point). This is a one-way function.

**Commands:**
```bash
# Extract the public key from the private key
openssl ec -in ecc_private.pem -pubout -out ecc_public.pem

# View the public key
openssl ec -pubin -in ecc_public.pem -text -noout
```

**Command Breakdown:**
- `-pubout`: Extract and output only the public key
- `-pubin`: Input is a public key (not private)

**What you'll see:** Public key coordinates (x, y) on the elliptic curve. These are the coordinates of the point obtained by multiplying the generator point by your private key.

---

## Exercise 3: List Available Elliptic Curves (2 minutes)

**Concept:** Different curves offer different security levels and performance characteristics. Common curves include secp256k1, prime256v1, and secp384r1.

**Commands:**
```bash
# List all available elliptic curves in OpenSSL
openssl ecparam -list_curves | grep -E "prime256v1|secp256k1|secp384r1"
```

**Popular curves:**
- `secp256k1`: 256-bit (used in Bitcoin)
- `prime256v1`: 256-bit (NIST standard, also called secp256r1)
- `secp384r1`: 384-bit (higher security)


### Question 2 (Yes/No)
Can you use the command `openssl ecparam -list_curves` to view all available elliptic curves in OpenSSL?

---

## Exercise 4: Sign and Verify a Message with ECC (4 minutes)

**Concept:** ECC is commonly used for digital signatures. You sign a message with your private key, and others can verify it with your public key.

**Commands:**
```bash
# Create a test message
echo "Secret Project: Quantum Computing Initiative - Confidential" > message.txt

# Sign the message with private key
openssl dgst -sha256 -sign ecc_private.pem -out message.sig message.txt

# View signature (in hexadecimal)
xxd message.sig | head -10

# Verify the signature with public key
openssl dgst -sha256 -verify ecc_public.pem -signature message.sig message.txt
```

**Command Breakdown:**
- `dgst -sha256`: Create SHA-256 hash of the message
- `-sign ecc_private.pem`: Sign the hash with private key
- `-verify ecc_public.pem`: Verify signature with public key
- `xxd`: Creates a hex dump of the file

**What you'll see:** "Verified OK" if the signature is valid, random-looking hexadecimal signature data.


### Question 3 
Which operation is the foundation of ECC cryptography?

A) Modular exponentiation  
B) Point addition and point multiplication  
C) Matrix multiplication  
D) Polynomial division  


---

## Exercise 5: Test Signature Verification (3 minutes)

**Concept:** Digital signatures ensure message integrity and authenticity. Any modification to the message will cause verification to fail.

**Commands:**
```bash
# First verify the original signature works
openssl dgst -sha256 -verify ecc_public.pem -signature message.sig message.txt

# Now modify the message
echo "Modified: Secret Project - CHANGED" > message_modified.txt

# Try to verify modified message with original signature
openssl dgst -sha256 -verify ecc_public.pem -signature message.sig message_modified.txt

# Create new signature for modified message
openssl dgst -sha256 -sign ecc_private.pem -out message_modified.sig message_modified.txt

# Verify new signature
openssl dgst -sha256 -verify ecc_public.pem -signature message_modified.sig message_modified.txt
```

**Command Breakdown:**
- First verification should succeed (Verified OK)
- Second verification should fail (Verification Failure) - modified message doesn't match signature
- Third signature creation makes a new valid signature for the modified message


### Question 4 (True/False)
The discrete logarithm problem in ECC (ECDLP) is easier to solve than the discrete logarithm problem in traditional Diffie-Hellman.


---

## Exercise 6: ECDH Key Exchange - Alice's Side (3 minutes)

**Concept:** In ECDH, Alice generates her private key (a) and computes her public key (A = a × G), where G is the base point.

**Commands:**
```bash
# Alice generates her key pair
openssl ecparam -name prime256v1 -genkey -noout -out alice_private.pem
openssl ec -in alice_private.pem -pubout -out alice_public.pem

# View Alice's public key
echo "=== Alice's Public Key (share this with Bob) ==="
cat alice_public.pem
```

**Command Breakdown:**
- `prime256v1`: NIST standard 256-bit curve (widely compatible)

**What happens:** Alice can share `alice_public.pem` with Bob openly (even over an insecure channel).


---

## Exercise 7: ECDH Key Exchange - Bob's Side (3 minutes)

**Concept:** Bob independently generates his private key (b) and public key (B = b × G).

**Commands:**
```bash
# Bob generates his key pair
openssl ecparam -name prime256v1 -genkey -noout -out bob_private.pem
openssl ec -in bob_private.pem -pubout -out bob_public.pem

# View Bob's public key
echo "=== Bob's Public Key (share this with Alice) ==="
cat bob_public.pem
```

**What happens:** Bob shares `bob_public.pem` with Alice openly.

---

*End of Handout*
