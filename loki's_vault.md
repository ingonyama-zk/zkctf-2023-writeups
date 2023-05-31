# Loki's Vault

## Challenge 
> After years of careful investigation, you have reached the gate to Loki's vault in the icy mountains of Norway, where it is said that many great treasures and powerful weapons are hidden. The gate seems unbreakable, but you spot some ancient machinery with inscriptions in old runes.
> 
> Read more: https://github.com/ingonyama-zk/breaking_into_vault_of_loki

## Solution 
* By [Vladimir](https://github.com/ingonyama-zk/zkctf-2023-writeups/blob/main/loki's_vault.md#by-vladimir)
* By [Ayush Shukla](https://github.com/ingonyama-zk/zkctf-2023-writeups/blob/main/loki's_vault.md#by-ayush-shukla)
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

---
### By [Vladimir](https://twitter.com/zkBob_)


#### Notation
* $G_1$, $G_2$ - multiplicative group generators on BLS12-381
* $[x]_1$ is a point $x*G_1$ as opposed to  $[x]_2$ which is $x*G_2$. It can be thought as encryption of $x$ that is additively homomorphic: $[x]_1$ + $[y]_1$ = $[x+y]_1$, but you can't get $x$ from $[x]_1$ or $[x]_2$, and you can't get $[x]_1$ from $[x]_2$ either
#### KZG recap

The idea behind KZG polynomial commitment scheme is to provide an ability for the prover to assure verifier that a certain operation (which could have been agreed upon beforehand and expressed as a polynomial) has been applied to some input. This is a very useful thing for protocols where computational integrity is important because you can encode pretty much any computational instructions as a polynomial. We can think of it as a hash function for some lambda function that captures how it affects the input parameters and therefore it's result can be effectively checked by the verifier without having to redo everything.

**The statement to be proved:** an $n$-degree polynomial $p(x)=\sum_{i=0}^n c_i*x^i$ evaluates to $y$ at a field element $X$
1. Setup $\tau \leftarrow  random, SRS=  \{[\tau^i]_1\}, [\tau]_2$. It's very important that $\tau$ is selected at random, we'll see later why. . The powers of $\tau$ must be generated using a trusted setup ceremony which is a complicated topic that we skip. The most important is that neither Prover, nor Verifier can find $\tau$
2. Prover commits to $p$ by evaluaing it at $\tau$ using powers of $\tau$ from $SRS$ : $C=[p(\tau)]_1$ = $sum_i^n c_i*[\tau^i]_1$
3. Prover evaluates $p$ at $x$, generates a proof $\pi$ that $p(x)=y$. Let's look into more details for this step, since it's important to understand how we can break it. The statement $p(X)=y$ means that a polynomial $p(x)-y$ must have a root at $X$ so it can be formulated as following: 
$p(x)$ evaluates to $y$ at $X$ if 
* $x-X$ divides $p(x) - y$ without a remainder
* there exists such h(x) so that $p(x)-y=h(x)*(x-X)$

This property of polynomial $p(x) - y$ must hold everywhere, including a secret point $\tau$, so the prover calculates $h(x)$ in polynomial form
$$ h(x) = \frac{p(x)-y}{x-X}$$
and then provides commitment to evaluation at $\tau$ using $SRS$
So the proof is $\pi = [h(\tau)]_1$
It also could have been expressed in a scalar form 
$$\pi = [\frac {p(\tau) - y}{\tau - X}]_1$$
But the prover doesn't have $\tau-X$ so she can't calculate it directly.
The prover passes $\pi$  to the Verifier

4. Verifier calculates  $[\tau-X]_2$ using SRS, checks if $e( -[y]_1 + C,[1]_2) = e(\pi, [\tau-X]_2)$  where $e$ is a pairing on BLS12-381 which can be thought as multiplication of encrypted values. More on this topic here in the layman terms [here](https://hackmd.io/@benjaminion/bls12-381#Pairings). if equality holds, she accepts, rejects otherwise

##### Overview 
As you have seen the protocol itself is pretty elegant and not that complicated, at least if we treat pairings as a black box. KZG is not the only solution for computational integrity, there are also other schemes based on FRI, Inner Product Argument, but KZG has some distinct and very usefull features such as 
1. It is binding, which means that the polynomial at open phase must be the same as previously committed (if the setup is made correctly)
2. It is hiding, which means that verifier doesn't find out anything about  $p$ because Prover gives only $C$ which is a commitment to evaluation of $p$ 
3. Proof size is constant and consists of a single elliptic curve point 
4. Verification time is constant and requires two pairing operations
5. We can effectively batch multiple proofs for a single polynomial


#### Forging a proof
We need to forge a proof for a false statement that for $X=1, p(1)=y'=3$. Since it's not true and in fact $p(1)=1564$, then $p(x)-3$ doesn't have a root at $1$, and therefore $(x - 1)$ doesn't divide $p(x) - 3$. If provided with an honest proof the pairings check would show that $\tau-1$ doesn't divide $p(\tau)-3$ without remainder and the statement is false. 
 
In order to cheat we need to find some other proof $\pi'= h'(\tau)$ such it satisfies the divisibility check just at $\tau$ even if it doesn't hold anywhere else
$e( -[y']_1 + C,[1]_2) = e(\pi', [\tau-X]_2)$
Or with substituted values
$e( [C-3]_1,[1]_2) == e(\pi', [\tau-1]_2)$
That would fool the verifier if the prover have used completely different polynomial to get the fake value $3$. 

We need to find $h'$ for $C - 3 = h'(\tau)(\tau-1)$. But the only thing that we got from honest proof generation is that $C -1564=h(\tau)(\tau-1)$. So we can try to express the difference in the LHS as a multiple of $\tau-1$
$C -1564= C-3-1561 = h(\tau)*(\tau-1)$
Moving $1561$ to the RHS gives:
$C-3 =h(\tau)*(\tau-1)+1561$
If only we could express 1561 as some $R(\tau)$,which is a multiple of $\tau -1$: $R(\tau) = (\tau-1)*r(\tau)$
That would give us that RHS would be divisible by $\tau-1$ in the following:
$$C - 3 = h(\tau)*(\tau-1) + R(\tau) = (h(\tau)+r(\tau))*(\tau-1)\tag{1} = h'(\tau)*(\tau-1)$$
Then the proof then would be $\pi'=[h'(\tau)]_1=[h(\tau)]_1+[r(\tau)]_1 \tag{2}$ 
The proof verification check would be give followng:
$$e( C - [3]_1,[1]_2) == e([h(\tau)+r(\tau)]_1, [\tau-1]_2) =  e(\pi', [\tau-1]_2)$$
which would hold since $(1)$ holds, because it's the same statement but that is checked using pairings.  

How do we find to find $r(\tau)$?

Normaly we would need to somehow extract $\tau$ from $SRS$ for this which is infeasible due to [q-SDH assumption](https://ai.stanford.edu/~xb/eurocrypt04a/bbsigs.pdf). But in this case $\tau$ was chosen from a set of 64th roots of unity of the base field which means that $\tau^{64}=1$ and what is more useful $\tau^{32}=-1$. Substracting $1$ from both sides gives us that $\tau^{32}-1=-2$.

Again if $\tau$ would have been chosen correctly we wouldn't be able to do this, since SRS only gives us monomials in the form of $\{[τ^i]_1\} = \{G_1*\tau^i\}$, so there wouldn't be any way to get rid of $G_1$. 
It is the verifier who performs multiplication in the RHS when verifying proof, and the prover is unable to effectively find any value, that would give LHS when multiplied by $\tau -1$. But in this particular case we're able to express the difference between true and fake evaluation 
$y-y'=1561=-\frac{1561}{2}*(\tau^{32}-1)$
Then we use [factorization of differences of power](https://proofwiki.org/wiki/Difference_of_Two_Powers) to factor out the $\tau-1$:

$(\tau^{32}-1)=(\tau-1) * \sum_{i=0}^{31}\tau^i$

Now we are ready to express the difference between true and fake evaluations as a multiple of $\tau-1$ 
$y-y'=1561=-\frac{1561}{2}*(\tau^{32}-1)=(\tau-1)*\frac{-1561}{2}*(\sum_{i=0}^{31}\tau^i)$

Which gives us that $r(\tau) = \frac{-1561}{2}*(\sum_{i=0}^{31}\tau^i)$

Substituting $r$ to $(1)$ to find the fake proof gives us
$$\pi'=[h'(\tau)]_1=\pi+[\frac{-1561}{2}*(\sum_{i=0}^{31}\tau^i)]_1$$
which can be calculated using $SRS$

#### The code

We're proving evaluation at this point
```rust
let x = FieldElement::one();
```

This is real evaluation
```rust
let y_true = challenge_polynomial().evaluate(&x);
```
This is the fake evaluation that we're trying to prove using the fake proof
```rust
let y_fake = FieldElement::from(3);
```
This is commitment to p equal to evaluation at tau
```rust
let p_commitment: G1Point = kzg.commit(&p);
```
This is prime group generator, we use it just to check that setup is broken
```rust
let g1 = BLS12381Curve::generator();
```
The code below would print the following lines, which proves that ideed $\tau$ is the 64th prime root of unity, which means that every $64*n$ power gives 1, and every $32+64*n$ power gives -1:
```rust
    for (pow, tau_pow) in srs.powers_main_group.clone().into_iter().enumerate()  {
        if tau_pow == g1 {
            println!("tau^{:?}=1", pow);
        } else if tau_pow == g1.neg() {
            println!("tau^{:?}=-1",pow);
        }    
    }
```
```
tau^0=1
tau^32=-1
tau^64=1
tau^96=-1
tau^128=1
tau^160=-1
```


this is scaling factor that allows to express 1561 as multiple of $\tau-1$ using the fact that $\tau^{32}-1 = -2$
```rust
let scaling_factor = ((y_true.clone() + y_fake.clone().neg()) * FieldElement::from(2).inv()).neg();
```
We calculate $[\sum_{i=0}^{31}\tau^i]_1$ using $SRS$:
```rust
    let mut tau_sum: ShortWeierstrassProjectivePoint<BLS12381Curve> =
        srs.powers_main_group.get(0).unwrap().clone();

    for pow in 1..32 {
        let tau_pow: ShortWeierstrassProjectivePoint<BLS12381Curve> =
            srs.powers_main_group.get(pow).unwrap().clone();
        tau_sum = tau_sum.operate_with(&tau_pow)
    }
```
Next we get $[r(\tau)]_1 = \frac{-1561}{2}*(\sum_{i=0}^{31}\tau^i)$:
```rust
let r = tau_sum.operate_with_self(scaling_factor.representative());
```

This is a true proof for value 1564  $\pi = [h(\tau)]_1$
```rust
let pi = kzg.open(&x, &y_true, &p);
```
Finaly this is $\pi' = \pi+[r(\tau)]_1$

```rust
let fake_proof = r.operate_with(&h);
```

