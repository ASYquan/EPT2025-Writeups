**Category:** Crypto
**Challenge:** _"Now that you're an expert in elliptic curves, meet their hyperactive cousin!"_
## Overview

This one is about hyperelliptic curves – a generalization of elliptic curves where the genus g>1g > 1 g>1. Specifically, we're working on a genus-3 curve and its Jacobian group. The tricky part is figuring out the group order.

The curve is defined over Fp\mathbb{F}_p Fp​ as:

H:v2+v=αx7H: v^2 + v = \alpha x^7H:v2+v=αx7

where p=a7−1a−1=Φ7(a)p = \frac{a^7 - 1}{a - 1} = \Phi_7(a) p=a−1a7−1​=Φ7​(a) – the 7th cyclotomic polynomial evaluated at aa a. This choice of pp p is the key to everything.

The flag is encoded as the x-coordinate of a point P∈H(Fp)P \in H(\mathbb{F}_p) P∈H(Fp​), and we're given C=e⋅J(P)C = e \cdot J(P) C=e⋅J(P) in the Jacobian, along with the (u,v)(u, v) (u,v) Mumford representation of CC C.

## Getting the Jacobian Order

Since ciphertext is C=e⋅J(P)C = e \cdot J(P) C=e⋅J(P), recovering J(P)J(P) J(P) is just computing e−1(mod#J(Fp))e^{-1} \pmod{\#J(\mathbb{F}_p)} e−1(mod#J(Fp​))... if we know the order. And SageMath's `.order()` won't cut it here for genus-3.

This is where the cyclotomic structure of pp p pays off. Because p=Φ7(a)p = \Phi_7(a) p=Φ7​(a), we can work in the cyclotomic field K=Q(ζ7)K = \mathbb{Q}(\zeta_7) K=Q(ζ7​) and use Stickelberger's theorem to compute the Jacobi sum J(χ,χ)J(\chi, \chi) J(χ,χ).

The idea is that pp p factors in KK K as a product of primes σk(π)\sigma_k(\pi) σk​(π) where π=a−ζ\pi = a - \zeta π=a−ζ and σk:ζ↦ζk\sigma_k : \zeta \mapsto \zeta^k σk​:ζ↦ζk are Galois automorphisms. Stickelberger tells us which of those primes appear in the Jacobi sum:

J(χ,χ)=∏i=16σi(π)ei,ei=⌊2i7⌋−2⌊i7⌋J(\chi, \chi) = \prod_{i=1}^{6} \sigma_i(\pi)^{e_i}, \quad e_i = \left\lfloor \frac{2i}{7} \right\rfloor - 2\left\lfloor \frac{i}{7} \right\rfloorJ(χ,χ)=i=1∏6​σi​(π)ei​,ei​=⌊72i​⌋−2⌊7i​⌋

Plugging in, only e4=e5=e6=1e_4 = e_5 = e_6 = 1 e4​=e5​=e6​=1 are nonzero, so:

J1=σ4(π)⋅σ5(π)⋅σ6(π)J_1 = \sigma_4(\pi) \cdot \sigma_5(\pi) \cdot \sigma_6(\pi)J1​=σ4​(π)⋅σ5​(π)⋅σ6​(π)

Then by a result from Koblitz, the Jacobian order is:

#J(Fp)=∣NormK/Q(1+J(χ,χ))∣\#J(\mathbb{F}_p) = \left| \text{Norm}_{K/\mathbb{Q}}(1 + J(\chi, \chi)) \right|#J(Fp​)=​NormK/Q​(1+J(χ,χ))​

Since J(χ,χ)J(\chi,\chi) J(χ,χ) is only determined up to a 7th root of unity and sign, we sweep all 14 candidates ±ζk⋅J1\pm \zeta^k \cdot J_1 ±ζk⋅J1​ and check which one annihilates CC C (i.e., N⋅C=ON \cdot C = \mathcal{O} N⋅C=O).

---

## Solving

Once we have the correct N=#J(Fp)N = \#J(\mathbb{F}_p) N=#J(Fp​), it's straightforward:

python

```python
JP = pow(e, -1, N) * C
for root, _ in JP[0].roots():
    print(long_to_bytes(ZZ(root)).decode())
```

The decrypted u(x)u(x) u(x) polynomial has the original x-coordinate as a root, and converting it back to bytes gives the flag.

I solved this entirely in [SageMath Online](https://sagecell.sagemath.org/) – no local install needed. Full solve script:

python

```python
from Crypto.Util.number import long_to_bytes

a = 18446744073709551862
alpha = 115792089237316201600239092629181903022853076627057745643635536179347243697936
n = 7
e = 0x1337deadbeef1337
p = (a^n - 1)//(a - 1)

F = GF(p)
R.<x> = F[]
H = HyperellipticCurve(alpha*x^7, 1)
J = H.jacobian()(F)

v = R([21531145860198597476878989237755710606843064008828229889869373339271596611957323351613721272489730980549489312170394, 8261157019291686096998910330610725289103621846918848921362473394138486389948240100075107913867104846969233604431830, 12307880594487906245908563779709403748007966885981814022547160387211433226745858660170917861206356995209959276604903])
u = R([37591645601766195349914241190244587541126648045894370496224605285205051789795607044058331351759777461098164398507872, 31825584496706095821807938099201614630698246114642375148953595216939791581619471005478949323508227490535910661480600, 38070154357732543786398829210104835275871944922030673096683631338658551155399049387419283060209447466278551582717763, 1])
C = J([u, v])

K.<zeta> = CyclotomicField(7)
pi = K(a) - zeta

def galois_embedding(k, x):
    return K.hom([zeta^k])(x)

J1 = prod([galois_embedding(i, pi) for i in [4, 5, 6]])

for sgn in (+1, -1):
    for k in range(7):
        N = ZZ(norm(1 + sgn*(zeta^k)*J1))
        if N*C != J(0):
            continue
        JP = pow(e, -1, N) * C
        for root, _ in JP[0].roots():
            print(long_to_bytes(ZZ(root)).decode())
```

# Flag: 
`EPT{Now_you_can_count_cyclotomically_f045e0f0cd}`