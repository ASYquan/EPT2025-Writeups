
**Category:** Crypto
## Overview
We were given `encryptor.py`, which makes the encryption scheme fully transparent. For each character in the key, the entire plaintext gets ROT-shifted by that character's ASCII value, then base64-encoded. Repeat for every key character.

---

## Understanding the Encryptor

The provided script does the following:

python

```python
def rot(char, shift):
    return chr(33 + (ord(char) - 33 + shift) % 94)

def rot_string(text, shift):
    return ''.join(rot(char, shift) for char in text)

def encrypt(plaintext, key):
    ciphertext = plaintext
    for key_char in key:
        ciphertext = rot_string(ciphertext, shift=int(ord(key_char)))
        encoded_bytes = base64.b64encode(ciphertext.encode('ascii'))
        ciphertext = encoded_bytes.decode('ascii')
    return ciphertext
```

Each encryption layer is: ROT-shift → base64. Since both operations are fully reversible, decryption is just: base64-decode → un-ROT-shift, applied in reverse order over the key.

---

## The Attack
Since we have the full encryption code, reversing it is mechanical. The only unknown is the key itself. I modified the provided script to work backwards through the layers using BFS:

1. Base64 decode the current ciphertext
2. Try all 94 printable ASCII characters as the ROT un-shift
3. Keep only results where ≥95% of characters are printable ASCII
4. Score surviving candidates by frequency of common English letters and `{}` characters (flag-shaped content floats to the top)
5. Push the best candidates back into the queue and repeat
6. Stop when `EPT{` appears in the text

The BFS naturally discovers the key length – you don't need to guess it upfront. Each level of the tree corresponds to one key character peeled back.
## The Script

python

```python
#!/usr/bin/env python3
import base64
import re
from collections import deque

def rot(char, shift):
    """Apply ROT transformation to a single character"""
    return chr(33 + (ord(char) - 33 + shift) % 94)

def rot_string(text, shift):
    """Apply ROT transformation to entire string"""
    return ''.join(rot(char, shift) for char in text)

def encrypt(plaintext, key):
    """Encrypt plaintext with given key"""
    ciphertext = plaintext
    for key_char in key:
        ciphertext = rot_string(ciphertext, shift=ord(key_char))
        ciphertext = base64.b64encode(ciphertext.encode('ascii')).decode('ascii')
    return ciphertext

def decrypt(ciphertext, key):
    """Decrypt ciphertext with given key"""
    plaintext = ciphertext
    for key_char in reversed(key):
        plaintext = base64.b64decode(plaintext.encode('ascii')).decode('ascii')
        plaintext = rot_string(plaintext, shift=-ord(key_char))
    return plaintext

def find_key_backwards(cipher_text, max_depth=100, max_candidates=1000):
    queue = deque([(cipher_text, "", 0)])
    explored_count = 0
    
    while queue:
        current_text, key_so_far, depth = queue.popleft()
        explored_count += 1
        
        if explored_count % 50 == 0:
            print(f"Explored {explored_count} states | Depth: {depth} | Current key: '{key_so_far[::-1]}'", end='\r')
        
        if depth > max_depth:
            continue
        
        if "EPT{" in current_text:
            actual_key = key_so_far[::-1]
            flags = re.findall(r'EPT\{[^}]+\}', current_text)
            print(f"\nKey: '{actual_key}'")
            for flag in flags:
                print(f"Flag: {flag}")
            return actual_key, current_text
        
        try:
            decoded_bytes = base64.b64decode(current_text)
            decoded_text = decoded_bytes.decode('ascii')
            candidates = []
            
            for key_char_code in range(33, 127):
                key_char = chr(key_char_code)
                unrotted = rot_string(decoded_text, shift=-key_char_code)
                
                if len(unrotted) > 0:
                    printable_count = sum(33 <= ord(c) <= 126 for c in unrotted)
                    printable_ratio = printable_count / len(unrotted)
                    
                    if printable_ratio >= 0.95:
                        score = sum(1 for c in unrotted[:200] if c in 'etaoinshrdlu ETAOINSHRDLU{}')
                        candidates.append((score, key_char, unrotted))
            
            candidates.sort(reverse=True)
            candidates = candidates[:max_candidates]
            
            for _, key_char, next_text in candidates:
                new_key = key_so_far + key_char
                queue.append((next_text, new_key, depth + 1))
        
        except Exception:
            continue
    
    return None, None

def main():
    with open("original_cipher.txt", "r") as f:
        cipher = f.read().strip()
    find_key_backwards(cipher, max_depth=100, max_candidates=1000)

if __name__ == "__main__":
    main()
```

The key is accumulated in reverse as we peel back layers, so it gets flipped at the end with key_so_far[: : -1]. The `encrypt()` and `rot_string()` functions are taken straight from the provided challenge script.

# The flag:
```EPT{a42d061e-5798-459e-a1a7-9d04ee236dab}```

