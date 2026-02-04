# ECC In-Class Handout
## Elliptic Curve Cryptography: Linux Hands-on Lab (30 minutes)

---

## Lab Scenario

**Scenario:**
Alice and Bob work for a tech company and need to securely exchange sensitive project files. They decide to use Elliptic Curve Cryptography for encryption and ECDH for key exchange.

**Your Tasks:**
1. Help Alice and Bob generate their ECC key pairs
2. Simulate a secure message exchange between them
3. Perform ECDH key exchange to establish a shared secret
4. Verify that both parties arrive at the same shared secret

**Expected Outcomes:**
- Alice can encrypt a message using Bob's public key
- Bob can decrypt the message using his private key
- Both Alice and Bob compute identical shared secrets through ECDH
- The shared secret should be 32 bytes (256 bits) for prime256v1 curve

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

**Task:** Generate your private key and identify the hexadecimal private key value in the output.

### Question 1 (Multiple Choice)
What is the primary advantage of ECC over RSA?

A) ECC is older and more tested  
B) ECC provides equivalent security with shorter key lengths  
C) ECC is easier to implement  
D) ECC doesn't use prime numbers  

**Answer:** B  
**Solution:** A 256-bit ECC key provides security equivalent to a 3072-bit RSA key. This means smaller keys, faster computations, less bandwidth, and lower power consumption - critical for mobile devices and IoT.

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

**Task:** Note down your public key coordinates. Can you derive the private key from these? (Answer: No, due to ECDLP)

### Question 2 (Fill in the Blank)
In the elliptic curve equation y² = x³ + ax + b, the curve is symmetric about the __________ axis.

**Answer:** x (or x-axis)  
**Solution:** Due to the y² term, if (x, y) is on the curve, then (x, -y) is also on the curve. This creates symmetry across the x-axis. This property is used in point addition operations.

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

**Task:** Find curves with 256, 384, and 521 bits. Why would you choose a larger curve?

### Question 3 (Yes/No)
Can you use the command `openssl ecparam -list_curves` to view all available elliptic curves in OpenSSL?

**Answer:** Yes  
**Solution:** This command lists all elliptic curves supported by your OpenSSL installation, including curve names (e.g., secp256k1, prime256v1), bit sizes, and descriptions. Different curves offer different security/performance tradeoffs.

---

## Exercise 4: Encrypt a Message Using ECC Public Key (4 minutes)

**Concept:** Encryption transforms plaintext into ciphertext using the recipient's public key. Only the holder of the corresponding private key can decrypt it.

**Commands:**
```bash
# Create a test message
echo "Secret Project: Quantum Computing Initiative - Confidential" > message.txt

# Encrypt the message using the public key
openssl pkeyutl -encrypt -pubin -inkey ecc_public.pem -in message.txt -out encrypted.bin

# View encrypted data (in hexadecimal)
xxd encrypted.bin | head -10
```

**Command Breakdown:**
- `pkeyutl`: Public key algorithm utility
- `-encrypt`: Perform encryption operation
- `-pubin`: Input key is a public key
- `-inkey ecc_public.pem`: Use this public key for encryption
- `xxd`: Creates a hex dump of the file

**What you'll see:** Random-looking hexadecimal data (the ciphertext).

**Task:** Try to read the encrypted file with `cat encrypted.bin`. What do you see? (Unreadable binary data)

### Question 4 (Multiple Choice)
Which operation is the foundation of ECC encryption?

A) Modular exponentiation  
B) Point addition and point multiplication  
C) Matrix multiplication  
D) Polynomial division  

**Answer:** B  
**Solution:** ECC uses geometric operations on elliptic curves: point addition (P + Q) and point multiplication (k·P = P + P + ... + P, k times). RSA uses modular exponentiation; ECC doesn't.

---

## Exercise 5: Decrypt the Message Using ECC Private Key (3 minutes)

**Concept:** Decryption reverses the encryption using the private key, recovering the original plaintext.

**Commands:**
```bash
# Decrypt the message using the private key
openssl pkeyutl -decrypt -inkey ecc_private.pem -in encrypted.bin -out decrypted.txt

# View the decrypted message
cat decrypted.txt

# Verify it matches original
diff message.txt decrypted.txt && echo "SUCCESS: Files match!" || echo "ERROR: Files differ!"
```

**Command Breakdown:**
- `-decrypt`: Perform decryption operation
- `-inkey ecc_private.pem`: Use the private key (must match the public key used for encryption)
- `diff`: Compare two files (no output means they're identical)

**Task:** Verify your decrypted message matches the original.

### Question 5 (True/False)
The discrete logarithm problem in ECC (ECDLP) is easier to solve than the discrete logarithm problem in traditional Diffie-Hellman.

**Answer:** False  
**Solution:** ECDLP is actually HARDER to solve, which is why ECC can use shorter keys. The best known algorithms for ECDLP have exponential time complexity O(√n), whereas traditional DLP has sub-exponential algorithms. This is ECC's main advantage.

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

**Task:** Save Alice's public key. This will be shared with Bob.

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

**Task:** Save Bob's public key for sharing with Alice.

### Question 6 (Fill in the Blank)
In ECDH, if Alice's private key is 'a' and Bob's public key is B, Alice computes the shared secret as S = __________.

**Answer:** aB or a·B or a×B  
**Solution:** Alice performs scalar multiplication: S = a·B = a·(b·G) = (a·b)·G. Bob computes S = b·A = b·(a·G) = (b·a)·G. Since scalar multiplication is commutative (a·b = b·a), both get the same point S.

---

## Exercise 8: ECDH - Compute Shared Secret (5 minutes)

**Concept:** 
- Alice computes: S = a × B (her private key × Bob's public key)
- Bob computes: S = b × A (his private key × Alice's public key)
- Due to commutativity: a × B = a × (b × G) = (a × b) × G = (b × a) × G = b × (a × G) = b × A
- **Result: Both get the same shared secret!**

**Commands:**
```bash
# Alice computes shared secret using Bob's public key
openssl pkeyutl -derive -inkey alice_private.pem -peerkey bob_public.pem -out alice_shared.bin

# Bob computes shared secret using Alice's public key
openssl pkeyutl -derive -inkey bob_private.pem -peerkey alice_public.pem -out bob_shared.bin

# Compare the shared secrets
echo "=== Alice's shared secret ==="
xxd alice_shared.bin

echo -e "\n=== Bob's shared secret ==="
xxd bob_shared.bin

echo -e "\n=== Verification ==="
diff alice_shared.bin bob_shared.bin && echo "✓ SUCCESS: Shared secrets match!" || echo "✗ ERROR: Shared secrets differ!"

# Check the size
ls -lh alice_shared.bin bob_shared.bin
```

**Command Breakdown:**
- `-derive`: Derive a shared secret (ECDH operation)
- `-peerkey`: Use the other party's public key
- `xxd`: Display hex dump of binary file

**What you'll see:** Both hex dumps should be identical. Should be 32 bytes (256 bits) for prime256v1 curve.

**Task:** Verify that Alice and Bob computed identical shared secrets. This secret can now be used as a symmetric encryption key (e.g., for AES).

### Question 7 (Multiple Choice)
After completing ECDH key exchange, what should Alice and Bob's computed shared secrets be?

A) Completely different  
B) Similar but not identical  
C) Identical (bit-for-bit)  
D) One should be the inverse of the other  

**Answer:** C  
**Solution:** Due to the commutative property of scalar multiplication on elliptic curves, Alice's computation a·B = a·(b·G) = (a·b)·G equals Bob's computation b·A = b·(a·G) = (b·a)·G. The `diff` command should show no differences. This shared secret becomes the symmetric key for further encrypted communication.

### Question 8 (Yes/No)
In ECDH key exchange, can an attacker who intercepts both Alice's and Bob's public keys compute the shared secret?

**Answer:** No  
**Solution:** The attacker would need to solve the Elliptic Curve Discrete Log Problem (ECDLP): given A = aG, find a. This is computationally infeasible for properly sized keys. The attacker needs at least one private key to compute the shared secret S = ab·G.

---

## Verification Checklist

After completing all exercises, verify:

- [ ] Your ECC keys are properly generated (Exercise 1-2)
- [ ] You can encrypt and decrypt messages successfully (Exercise 4-5)
- [ ] Alice and Bob both generated key pairs (Exercise 6-7)
- [ ] **CRITICAL:** Alice's and Bob's shared secrets are identical (Exercise 8) - use `diff` command
- [ ] Shared secret file size is 32 bytes for prime256v1 curve
- [ ] You understand why an attacker can't compute the shared secret from public keys alone

---

## Expected Scenario Outcomes

✅ **Message Encryption:** Bob successfully decrypts Alice's encrypted message  
✅ **ECDH Key Exchange:** Alice and Bob compute identical 32-byte shared secrets  
✅ **Security:** Public keys can be shared openly without compromising security  
✅ **Efficiency:** ECC keys are much smaller than equivalent RSA keys  

---

## Quick Reference - Essential Commands

```bash
# Key Generation
openssl ecparam -name prime256v1 -genkey -noout -out private.pem
openssl ec -in private.pem -pubout -out public.pem

# View Keys
openssl ec -in private.pem -text -noout
openssl ec -pubin -in public.pem -text -noout

# Encryption/Decryption
openssl pkeyutl -encrypt -pubin -inkey public.pem -in plain.txt -out cipher.bin
openssl pkeyutl -decrypt -inkey private.pem -in cipher.bin -out plain.txt

# ECDH Shared Secret
openssl pkeyutl -derive -inkey my_private.pem -peerkey their_public.pem -out shared.bin

# View/Compare Files
xxd file.bin              # Hex dump
cat file.txt              # View text
diff file1 file2          # Compare files
ls -lh file.bin           # Check file size
```

---

## Answer Key Summary

1. **B** - ECC provides equivalent security with shorter key lengths
2. **x-axis** - Elliptic curves are symmetric about the x-axis
3. **Yes** - `openssl ecparam -list_curves` shows all available curves
4. **B** - Point addition and point multiplication are the foundation
5. **False** - ECDLP is harder to solve than traditional DLP
6. **aB (or a·B)** - Alice multiplies her private key by Bob's public key
7. **C** - Shared secrets must be identical (bit-for-bit)
8. **No** - Attacker cannot compute shared secret without a private key

---

## Additional Notes

**Security Considerations:**
- Always keep private keys secure and never share them
- Public keys can be shared openly
- The security relies on the computational difficulty of ECDLP
- Use standard curves (prime256v1, secp384r1) for compatibility

**Common Errors:**
- Mismatched curve names between Alice and Bob (use same curve!)
- Using wrong keys for decryption
- File permission issues (use `chmod 600` for private keys)

**Real-World Applications:**
- TLS/SSL certificates
- Bitcoin and cryptocurrency wallets
- Secure messaging (Signal, WhatsApp)
- IoT device authentication
- Smart card authentication

---

**Lab Completion Time:** 30 minutes  
**Questions:** 8 integrated questions  
**Difficulty Level:** Intermediate

**Instructor Notes:**
- Ensure OpenSSL is installed: `openssl version`
- Students should work in pairs (one as Alice, one as Bob)
- Emphasize the importance of verifying shared secrets match
- Discuss real-world implications of key size differences

---

*End of Handout*
