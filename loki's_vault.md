# Loki's Vault

## Challenge 
> After years of careful investigation, you have reached the gate to Loki's vault in the icy mountains of Norway, where it is said that many great treasures and powerful weapons are hidden. The gate seems unbreakable, but you spot some ancient machinery with inscriptions in old runes.
> 
> Read more: https://github.com/ingonyama-zk/breaking_into_vault_of_loki

## Solution 
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)

This challenge was one of the toughest and needed a good understanding of how KZG commitments work. We are provided with the following polynomial

$$
\begin{aligned}
p(x) &= 69 +78x + 32x^2 + 65x^3 + 82x^4 + 71x^5 + 69x^6 + 78x^7 + 84x^8 + 73x^9 \newline &+78x^{10} + 65x^{11} + 32x^{12} + 78x^{13} + 65x^{14}+ 67x^{15} + 73x^{16} + 32x^{17} \newline
&+ 84x^{18} + 73x^{19} + 69x^{20} + 82x^{21} + 82x^{22} + 65 x^{23} 
\end{aligned}
$$

Our objective is to generate a KZG proof that the polynomial equals $3$ at $x = 1$ within the BLS12-381 scalar field.

Given that the polynomial's value is clearly not $3$ at $x = 1$, it is clear that forging a proof is our only option. So I started looking into KZG commitments in detail. [Dankrad's article](https://dankradfeist.de/ethereum/2020/06/16/kate-polynomial-commitments.html) was a great resource in understanding how KZG commitments work. Essentially, a person can create a fraudulent proof if they know the secret "toxic waste" $s$ from the trusted setup. I looked into how trusted setups work by reading [Vitalik's article](https://vitalik.ca/general/2022/03/14/trustedsetup.html). The SRS string derived from the trusted setup has the following form:

$$
[G_1, G_1 * s, G_1 * s^2 ... G_1 * s^{n_1-1}]\\
[G_2, G_2 * s, G_2 * s^2 ... G_2 * s^{n_2-1}]\\
$$

The given SRS had $n_1 = 1024$ and $n_2 = 2$ with $G_1$ and $G_2$ representing the generators of the main and secondary group of the elliptic curve.

Analyzing the powers in the main group, I realized that they were repeating with a frequency of 64 i.e. $s^64 = 1\mod p$. This implied that $s$ is a 64th root of unity. To determine $s$, I wrote a simple Sage script to calculate the 64 roots of unity and find the correct one:

```python
from sage.all import *

p = 0x1a0111ea397fe69a4b1ba7b6434bacd764774b84f38512bf6730d2a0f6b0f6241eabfffeb153ffffb9feffffffffaaab
K = GF(p)
a = K(0x00)
b = K(0x04)
E = EllipticCurve(K, (a, b))
G = E(0x17F1D3A73197D7942695638C4FA9AC0FC3688C4F9774B905A14E3A3F171BAC586C55E83FF97A1AEFFB3AF00ADB22C6BB, 0x08B3F481E3AAA0F1A09E30ED741D8AE4FCF5E095D5D00AF600DB18CB2C04B3EDD03CC744A2888AE40CAA232946C5E7E1)
E.set_order(0x73EDA753299D7D483339D80809A1D80553BDA402FFFE5BFEFFFFFFFF00000001 * 0x396C8C005555E1568C00AAAB0000AAAB)

sG = E(0xb45e08705bc9f96ddef642f24f7e6d326c5e450aefb21363fd8c6788591afca990680a8f862e8d43609430f54aca45f, 0x63e8c7cd26cee9463932fd15ddaac016f42d598bd1abedfdfc37bfeb9f326cd80e36ab003b5b4a79bb25c5695e291b)

q = G.order()
L = GF(q)
g = L.multiplicative_generator()

n = 64
s = 0
for i in range(n):
    s = pow(g, i * (q - 1) // n)
    if s*G == sG:
        print(f"Found s: {s}")
        break
```

Once $s$ is known, we can construct a fake proof using the formula:

$$
\pi_{fake} = \frac{1}{s - y} (C - yG) 
$$

Here, $s$ is the secret, $C$ is the commitment, and $y$ is the intended fake value for which the proof is to be generated.

Below is another Sage script to compute this:

```python
x = 1
ynew = 3

k = pow(L(s - x), -1)
C = E(0x1167e707d11074bef0ee02040d38c06e32d829341246af1ba03572a61a3d2052d687d5ebb5de356ff089006e6318bb8b, 0x1537d55fffdd6d31d1b831bb8ac2e24142084f122c830cddc117d31505d49becd3854df61ce8b7f7ca14aa8f3a0eb0c)

proof = k*(C - ynew*G)
print([hex(z) for z in proof])
```

As stated in the problem description, the $x$ coordinate of the fake proof is the solution.
