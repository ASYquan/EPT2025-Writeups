**Category:** Crypto
## Overview

Three-part crypto challenge centered around elliptic curves. Each part builds on the last and exploits a different classical weakness. I did everything on [SageMathCell](https://sagecell.sagemath.org/) – no local install needed (except part 2 where compute was an issue, more on that).
## Part 1

Pretty straightforward once you understand what's happening. We're given two points QQ Q and RR R on a standard NIST P-256 curve, where R=P+QR = P + Q R=P+Q. So P=R−QP = R - Q P=R−Q, and the flag is encoded in the x-coordinate of PP P.

python

````python
P = R - Q
x_coord = int(P[0])
flag_bytes = x_coord.to_bytes((x_coord.bit_length() + 7) // 8, 'big')
print(flag_bytes.decode('ascii'))
```

**Flag (partial):** `EPT{n0W_y0u_Kn0w_4_f3w_`

---

## Part 2

This time we have $P = G \cdot k$ where $k$ is the flag, and we need to recover $k$ – the discrete log problem. Normally this is infeasible, but the challenge hints that the parameters are "strange". Checking the curve order reveals it's **smooth** – composed entirely of small prime factors:
```
2^2 * 3 * 18479537^2 * 785027357 * 2045936509 * 2067106871 * ...
````

Smooth order means **Pohlig-Hellman** applies, which decomposes the DLP into smaller subgroup problems. SageMath handles this automatically with `discrete_log()` or the faster `P.log(G)`.

I tried running this on SageMathCell but it timed out – the computation is just heavy enough to hit the limit. For this one I had to spin up a local SageMath install. Annoying, but it ran fine locally.

python

```python
k = P.log(G)
flag_bytes = k.to_bytes((k.bit_length() + 7) // 8, 'big')
print(flag_bytes.decode('ascii'))
```

**Flag (partial):** `7h1ng5_4b0UT_el1ipTiC_`

## Part 3

The hint is basically spelling it out: **SMART**. This points directly to [Nigel Smart's attack](https://link.springer.com/article/10.1007/s001459900052) on **anomalous curves** – curves where the order equals the field characteristic (#E(Fp)=p\#E(\mathbb{F}_p) = p #E(Fp​)=p). On these curves the DLP can be solved in polynomial time by lifting points to the p-adics.

SageMath's `P.log(G)` internally detects anomalous curves and calls `padic_elliptic_logarithm()`, so it just works out of the box. This one ran fine on SageMathCell too.

python

```python
k = P.log(G)
flag_bytes = k.to_bytes((k.bit_length() + 7) // 8, 'big')
print(flag_bytes.decode('ascii'))
```

**Flag (partial):** `cUrVes_af137dc886badc}`

---

# (Full) Flag

`EPT{n0W_y0u_Kn0w_4_f3w_7h1ng5_4b0UT_el1ipTiC_cUrVes_af137dc886badc}`

---

## Takeaway

Three classic ECC attacks back to back – point arithmetic, Pohlig-Hellman on smooth-order curves, and Smart's attack on anomalous curves. Part 2 was the only one that needed a local install due to compute limits on SageMathCell. Good intro challenge if you've never looked at ECC weaknesses before.