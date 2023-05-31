# Safe bn254

## Challenge

> doesn’t bn254 look safer now?
> 
> y^2=x^3+2023 if you are not afraid of generating curves and discrete logarithms, you could try looking for a flag in x, where res = x * curve_gen
> 
> The generators are given by (in affine coordinates)
> 
> curve_gen_x=14810849444223915365675197147935386463496555902363368947484943637353816116538 curve_gen_y=742647408428947575362456675910688304313089065515277648070767281175728054553
> 
> The result coordinates are res_x=5547094896230060345977898543873469282119259956812769264843946971664050560756 res_y=14961832535963026880436662513768132861653428490706468784706450723166120307238
> 
> you can use any language for finding the solution and convert the flag into text format
> 
> The prime modulus in GF(p) p=21888242871839275222246405745257275088696311157297823662689037894645226208583

## Solution
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)

We've been provided with a modified BN-254 elliptic curve, and our task is to calculate a discrete logarithm on this curve (ECDLP). 

First, I set up the problem in [Sagemath](https://www.sagemath.org/):



```python
import math
from sage.all import *

# BN-254 prime
p = 21888242871839275222246405745257275088696311157297823662689037894645226208583

# Generator
Gx = 14810849444223915365675197147935386463496555902363368947484943637353816116538
Gy = 742647408428947575362456675910688304313089065515277648070767281175728054553

# P = kG
Px = 5547094896230060345977898543873469282119259956812769264843946971664050560756
Py = 14961832535963026880436662513768132861653428490706468784706450723166120307238

F = GF(p)
E = EllipticCurve(F, [0, 2023])

print(E)
```

```bash
Elliptic Curve defined by y^2 = x^3 + 2023 over Finite Field of size 21888242871839275222246405745257275088696311157297823662689037894645226208583
```

The ECDLP problem, generally regarded as computationally infeasible, hinges on the order of the subgroup of the elliptic curve. The curve should have a large prime order subgroup, as operations are conducted in this group. If the subgroup's order is small, the [Pohlig-Hellman algorithm](https://en.wikipedia.org/wiki/Pohlig%E2%80%93Hellman_algorithm) can be used to fragment the problem into smaller, solvable parts. These parts can then be pieced back together using the Chinese Remainder Theorem (CRT) to arrive at the final solution. 

To check this, I factorized the order of the subgroup containing $G$:

```python
G = E(Gx, Gy)
order = G.order()
factors = factor(order)
print(f"Order of G = {order}")
print(f"           = {factors}")

P = E(Px, Py)
```
```bash
Order of G = 2203960485148121921418603742825762020959382274778627105322
           = 2 * 3^2 * 13 * 19 * 29 * 37 * 613 * 983 * 11003 * 346501 * 6248149 * 405928799 * 79287328374952431757
```

The largest prime in the prime factorization of the order is huge, making it computationally infeasible to solve. Therefore, I ignored the largest prime while calculating the discrete log. This gives us a solution modulo the remaining factors.

```python
dlogs = []
moduli = [p ** e for p, e in factors]
# Ignore largest factor to make problem computationally feasible
for m in moduli[:-1]:
    t = order // m
    dlog = discrete_log(t * P, t * G, operation="+")
    dlogs.append(dlog)

k = crt(dlogs, moduli[:-1])
print(f"Secret k: {k} mod {math.prod(moduli[:-1])}")

flag = bytearray.fromhex(hex(k)[2:]).decode("utf-8")
print(f"flag: {flag}")
```

Decoding the secret reveals that it conforms to the flag format:

```bash
Secret k: 381276930664415168886989783656063357 mod 27797133921898830561267529521791838546
flag: IngoCTF{b67e8a}
```

So, we've successfully found the flag.
