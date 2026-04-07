**Category:** Crypto  
**Challenge:** _"Now that you're an expert in elliptic curves, meet their hyperactive cousin!"_

## Overview

This one is about hyperelliptic curves – a generalization of elliptic curves where the genus $g>1$. Specifically, we're working on a genus-3 curve and its Jacobian group. The tricky part is figuring out the group order.

The curve is defined over $\mathbb{F}_p$ as:

$$
H: v^2 + v = \alpha x^7
$$

where $p = \frac{a^7 - 1}{a - 1} = \Phi_7(a)$ – the 7th cyclotomic polynomial evaluated at $a$. This choice of $p$ is the key to everything.

The flag is encoded as the $x$-coordinate of a point $P \in H(\mathbb{F}_p)$, and we're given $C = e \cdot J(P)$ in the Jacobian, along with the $(u,v)$ Mumford representation of $C$.

## Getting the Jacobian Order

Since ciphertext is $C = e \cdot J(P)$, recovering $J(P)$ is just computing $e^{-1} \pmod{\lvert J(\mathbb{F}_p) \rvert}$ ... if we know the order. And SageMath's `.order()` won't cut it here for genus-3.

This is where the cyclotomic structure of $p$ pays off. Because $p = \Phi_7(a)$, we can work in the cyclotomic field $K = \mathbb{Q}(\zeta_7)$ and use Stickelberger's theorem to compute the Jacobi sum $J(\chi, \chi)$.

The idea is that $p$ factors in $K$ as a product of primes $\sigma_k(\pi)$ where $\pi = a - \zeta$ and $\sigma_k : \zeta \mapsto \zeta^k$ are Galois automorphisms. Stickelberger tells us which of those primes appear in the Jacobi sum:

$$
J(\chi, \chi) = \prod_{i=1}^{6} \sigma_i(\pi)^{e_i}, \quad e_i = \left\lfloor \frac{2i}{7} \right\rfloor - 2\left\lfloor \frac{i}{7} \right\rfloor
$$

Plugging in, only $e_4 = e_5 = e_6 = 1$ are nonzero, so:

$$
J_1 = \sigma_4(\pi) \cdot \sigma_5(\pi) \cdot \sigma_6(\pi)
$$

Then by a result from Koblitz, the Jacobian order is:

$$
\lvert J(\mathbb{F}_p) \rvert = \left| \text{Norm}_{K/\mathbb{Q}}(1 + J(\chi, \chi)) \right|
$$

Since $J(\chi,\chi)$ is only determined up to a 7th root of unity and sign, we sweep all 14 candidates $\pm \zeta^k \cdot J_1$ and check which one annihilates $C$ (i.e., $N \cdot C = \mathcal{O}$).

---

## Solving

Once we have the correct $N = \lvert J(\mathbb{F}_p) \rvert$, it's straightforward:

```python
JP = pow(e, -1, N) * C
for root, _ in JP[0].roots():
    print(long_to_bytes(ZZ(root)).decode())
```
