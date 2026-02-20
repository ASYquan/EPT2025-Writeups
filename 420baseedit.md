**Category:** Crypto
## Overview

The challenge name is the entire hint. **420** = the sum of the base encodings used, **baseedit** = you're chaining base decoders. No crypto theory, just CyberChef and some arithmetic.

## The Solve

Looking at the available base encodings – Base32, Base45, Base58, Base62, Base64, Base85, Base92 – the question is which subset sums to 420:

```
92 + 85 + 85 + 64 + 62 + 32 = 420 ✓
```

Throw the ciphertext into CyberChef and chain the decoders in that order:

```
From Base92
From Base85
From Base85
From Base64
From Base62  (alphabet: 0-9A-Za-z)
From Base32
```

Output:

```
EPT{I5_th3_Ch3f_1n_yet?}
```

