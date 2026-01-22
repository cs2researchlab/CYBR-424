# Hands-On Lab: Demonstrating ECB Mode Weakness

## Objective
Understand why AES-ECB is insecure by observing pattern leakage through hands-on command-line practice.

---

## Stage 1: Create Patterned Image

```bash
mkdir -p ecb_demo && cd ecb_demo
convert -size 512x512 pattern:checkerboard checker.png
```

### Observation
The generated image contains strong repetition and high visual contrast. Any transformation that preserves structure will still show visible patterns, making weaknesses easier to detect.

### Question
**Why is a checkerboard ideal for ECB testing?**

A. Because it increases encryption speed by reducing entropy  
B. Because its repeating blocks make structural leakage easy to observe  
C. Because ECB is designed specifically for image encryption  
D. Because checkerboards prevent compression artifacts  

---

## Stage 2: Convert to RAW

```bash
convert checker.png -depth 8 gray:checker.raw
```

### Observation
The RAW format removes compression and headers, preserving a direct byte-to-pixel mapping. Encryption therefore operates on predictable data blocks.

### Question
**Why must compression be removed?**

A. Compression weakens AES encryption  
B. Compression increases file size unpredictably  
C. Compression alters data patterns before encryption  
D. Compression prevents encryption from working correctly  

---

## Stage 3: AES-ECB Encryption

```bash
KEY=00112233445566778899aabbccddeeff
openssl enc -aes-128-ecb -K $KEY -nosalt -nopad -in checker.raw -out checker_ecb.raw
```

### Observation
Identical plaintext blocks are encrypted independently without randomness or chaining. Repeated structures remain correlated in the ciphertext.

### Question
**What ECB property causes pattern leakage?**

A. ECB uses weak cryptographic keys  
B. ECB encrypts blocks independently without chaining  
C. ECB uses an initialization vector incorrectly  
D. ECB performs lossy encryption  

---

## Stage 4: AES-CBC Encryption

```bash
IV=0102030405060708090a0b0c0d0e0f10
openssl enc -aes-128-cbc -K $KEY -iv $IV -nosalt -nopad -in checker.raw -out checker_cbc.raw
```

### Observation
Each ciphertext block depends on the previous one, and the IV randomizes the first block. Even identical plaintext blocks encrypt differently.

### Question
**How does CBC prevent leakage?**

A. By encrypting each block independently  
B. By compressing data before encryption  
C. By chaining blocks and introducing randomness via an IV  
D. By increasing key length automatically  

---

## Stage 5: Convert Back to Images

```bash
convert -size 512x512 -depth 8 gray:checker_ecb.raw checker_ecb.png
convert -size 512x512 -depth 8 gray:checker_cbc.raw checker_cbc.png
```

### Observation
The ECB-encrypted image still resembles the original pattern, while the CBC-encrypted image appears random and pattern-free.

### Question
**What does the visual difference between ECB and CBC outputs indicate?**

A. ECB is faster but less compatible with images  
B. CBC encrypts images incorrectly  
C. ECB leaks structural information while CBC hides it  
D. CBC changes the image resolution  

---

## Stage 6: Observe Results

```bash
xdg-open checker.png checker_ecb.png checker_cbc.png
```

### Observation
Pattern leakage is visible in the ECB output without decrypting the data. Human visual inspection alone reveals the weakness.

### Question
**Why is this considered a serious confidentiality failure?**

A. Because attackers can recover the encryption key  
B. Because attackers can infer sensitive structure without decryption  
C. Because ECB corrupts encrypted files  
D. Because visual attacks require special hardware  

---

## Final Question (Assessment)

**Why is ECB discouraged even when using AES?**

A. AES is outdated and insecure  
B. ECB leaks structural information and fails semantic security  
C. ECB cannot be implemented efficiently  
D. ECB only works on text data  

---