# 🧩 CTF Challenge: Breaking the Cipher Mode

## Category
🔐 Cryptography

## Difficulty
🟡 Easy–Medium (Conceptual–Hands-on)

---

## 📜 Challenge Story

You intercepted two encrypted files from a compromised system.  
Both files were encrypted using **AES**, but **different modes** were used.

One of the encryption modes is **cryptographically weak** and leaks patterns.

Your mission is to identify the weak encryption mode and explain **why it is insecure**.

---

## 📁 Provided Files

- `cipher1.bin`
- `cipher2.bin`

> Both files were encrypted using the **same plaintext and key**, but different AES modes.

---

## 🎯 Objectives

1. Identify **which file uses AES-ECB**
2. Explain **why AES-ECB is insecure**
3. Capture the **CTF flag**

---

## 🛠 Allowed Tools

- `hexdump`
- `xxd`
- `openssl`
- `strings`
- `diff`

---

## 🧪 Tasks

### Task 1: Inspect the Ciphertexts

```bash
hexdump -C cipher1.bin | head -20
hexdump -C cipher2.bin | head -20
```

📌 **Question:**  
Which ciphertext shows visible structure or repeated blocks?

---

### Task 2: Identify the Encryption Mode

Determine:
- Which file was encrypted using **AES-ECB**
- Which file was encrypted using **AES-CBC**

🧠 **Hint:**  
Identical plaintext blocks encrypted with ECB produce identical ciphertext blocks.

---

### Task 3: Security Analysis

Answer briefly:

- Why is AES-ECB insecure?
- How does AES-CBC prevent this weakness?

---

## 🚩 Flag Format

```
FLAG{ECB_LEAKS_PATTERNS}
```

---

## 🏁 Submission Format

```
ECB file: <filename>
FLAG{ECB_LEAKS_PATTERNS}
```

---

## 🔥 Bonus Challenge (Optional)

Encrypt an image using AES-ECB and AES-CBC and visually compare the results.

---

## ✅ Instructor Solution (Hidden)

- ECB file: `cipher1.bin`
- CBC file: `cipher2.bin`

**Reason:**  
ECB encrypts blocks independently, leaking patterns.  
CBC uses chaining and an IV to hide patterns.

---

## 📌 Key Takeaway

AES-ECB should never be used for sensitive data. AES-CBC (or AEAD modes like GCM) should be preferred.
