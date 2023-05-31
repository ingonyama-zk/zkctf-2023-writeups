# It is over 9000!

## Challenge

> The Saiyans have landed on planet earth. Our great defenders Krillin, Piccolo, Tien and Gohan have to hold on till Goku arrives on the scene.
> 
> Vegeta and Nappa have scouters that indicate our heroes power levels and sadly we are not doing too well.
> 
> Somehow, Gohan has raised his power level to `p_4(X) = 9000`, but it is not good enough. Piccolo `p_3(X)` can help but he is still regenerating, and Krillin `p_2(X)` and Tien `p_1(X)` are in bad shape. The total power of the team is computed as
> ```
> P = p_1(X) * 0 + p_2(X) * 0 + p_3(X) * 0 + p_4(X)
> ```
> At the current moment, the X is equal to `42`.
> 
> Suddenly Gohan, and Piccolo recieve a message from Bulma that the scouters verify the sensed power level of individual enemies using KZG and for multiple enemies with batched KZG method. Vegeta knows for sure that the power level of Gohan is `p_4(X) = 9000`, so he will know if we change that. If only the team had a way to trick their opponents to believe that their total power level is `P > 9000` - then the enemies will surely flee.
> 
> To run
> ```bash
> cargo run --release
> ```
> Verification
> Vegeta is running the scouters from `52.7.211.188:8000`.
> 
> https://github.com/ingonyama-zk/ctf-over-9000

## Solution 
* By [Vladimir](https://github.com/ingonyama-zk/zkctf-2023-writeups/blob/main/loki's_vault.md#by-vladimir)
* By [Ayush Shukla](https://github.com/ingonyama-zk/zkctf-2023-writeups/blob/main/it_is_over_9000!.md#by-ayush-shukla)

---

### By [Ayush Shukla](https://hackmd.io/@shuklaayush)

The key to solving this challenge is understanding how batching works in KZG commitments. Looking into the code reveals two important functions, `open_batch` and `verify_batch`. The crucial insight comes from realizing that the `upsilon` parameter has to be random. Otherwise, it can be used to make the verification ignore some of the polynomials.

```rust
fn open_batch(
    &self,
    x: &FieldElement<F>,
    ys: &[FieldElement<F>],
    polynomials: &[Polynomial<FieldElement<F>>],
    upsilon: &FieldElement<F>,
) -> Self::Commitment {
    let acc_polynomial = polynomials
        .iter()
        .rev()
        .fold(Polynomial::zero(), |acc, polynomial| {
            acc * upsilon.to_owned() + polynomial
        });

    let acc_y = ys
        .iter()
        .rev()
        .fold(FieldElement::zero(), |acc, y| acc * upsilon.to_owned() + y);

    self.open(x, &acc_y, &acc_polynomial)
}

fn verify_batch(
    &self,
    x: &FieldElement<F>,
    ys: &[FieldElement<F>],
    p_commitments: &[Self::Commitment],
    proof: &Self::Commitment,
    upsilon: &FieldElement<F>,
) -> bool {
    let acc_commitment =
        p_commitments
            .iter()
            .rev()
            .fold(P::G1Point::neutral_element(), |acc, point| {
                acc.operate_with_self(upsilon.to_owned().representative())
                    .operate_with(point)
            });

    let acc_y = ys
        .iter()
        .rev()
        .fold(FieldElement::zero(), |acc, y| acc * upsilon.to_owned() + y);
    self.verify(x, &acc_y, &acc_commitment, proof)
}
```

In the code, `u` (`upsilon`) can be selected by the prover. By hardcoding it to `0` instead of a random value, only `p1` is checked. Now since `p4` isn't checked, you can keep the polynomial the same so that Vegeta can verify the commitment to it, and confirm Gohan's power level. But you can change the `y4` value, which would never be verified in the `batch_verify` function. As a result, the sum of power levels appears to be over 9000, fooling the villains.

---
### By [Vladimir](https://twitter.com/zkBob_)

#### Notation
* $G_1$, $G_2$ - multiplicative group generators on BLS12-381
* $[x]_1$ is a point $x*G_1$ as opposed to  $[x]_2$ which is $x*G_2$. It can be thought as encryption of $x$ that is additively homomorphic: $[x]_1$ + $[y]_1$ = $[x+y]_1$, but you can't get $x$ from $[x]_1$ or $[x]_2$, and you can't get $[x]_1$ from $[x]_2$ either

#### Batched KZG protocol 
The prover is trying to assure the Verifier that a set of polynomials have certain evaluations at some point. Below we look into more details how batching is performed and how it can be broken.

An individual proof for KZG10 is calculated as following:

$$\pi_i = [\frac {p_i(\tau) - y_i}{\tau - X}]_1$$

where 
* $p_i$ is an individual polynomial in the set
* $\tau$ is a secret point from $SRS$
* $X$ is a point where the evaluation is being proven
* $y_i$ is an individual evaluation being proven

The fragile part of this protocol lies of course in the batching mechanism. Batch proof is just a proof for a new polynomial that is composed from individual polynomials:

$$P(x) = \sum_{i=0}^n \upsilon^{i-1} *p_i(x)$$
The evaluation is calculated correspondingly:
$$Y = \sum_{i=0}^n \upsilon^{i-1}*y_i(X)\tag 1$$
and then the batch proof is
$$\pi_{batch} = [\frac {P(\tau) - Y}{\tau - X}]_1$$

So the Prover sends the following:
* batch proof $\pi_{batch}$
* individual commitments $p(\tau)$
* individual evaluations $y_i$
* challenge $\upsilon$

And the verifier checks that 
$$ e(\pi_{batch}, [\tau-X]_2) = e([P(\tau)-Y]_1,[1]_2)\tag {2}$$

The challenge requires us to change an individual component $y_3$ with $y_3'=y_3+\delta$ where $\delta$ is some positive number and still make batch proof pass the check $(2)$

The suggested protocol could be fine if only it is interactive whereas $\upsilon$ is provided by the verifier or there is a some random oracle to provide it. It could also be fixed by implementing Fiat-Shamir heuristic in a proper way, where all the public inputs are included in the hash calculation.
For more details on the topic you can see [zk hack challenge](https://zkhack.dev/events/puzzle5.html) and [plonk vulnerability description](https://blog.trailofbits.com/2022/04/18/the-frozen-heart-vulnerability-in-plonk/)

Since in this particular case $\upsilon$ is chosen arbitrarily by the Prover, an easy way to cheat is to change both $y_3$ to $y_3'$ and $\upsilon$ to $\upsilon'$ for the same proof before sending both of them to the Verifier.
In order to preserve consistency we have to make $Y$ the same:

$$Y = \upsilon^3*y_3+\upsilon^2*y_2 + \upsilon*y_1 + y_0 = \upsilon'^3*(y_3+1)+\upsilon'^2*y_2 + \upsilon'*y_1 + y_0$$
$$\upsilon^3*y_3+\upsilon^2*y_2 + \upsilon*y_1 = \upsilon'^3*(y_3+1)+\upsilon'^2*y_2 + \upsilon'*y_1$$

In general this is a cubic equation with respect to $\upsilon'$, but since we can set $\upsilon$ to whatever we want, we can just use $0$ for both of them and that would make the equation hold. So the first way is to find such a $\upsilon$ for any desired $y'$ so that the proof is the same

Alternatively we could leave $\upsilon$ as it is and mess with individual $y_i$ so that the sum is the same, but $y_3$ is changed to $y_3+1$. 

$$Y' = \upsilon^3*(y_3+1)+\upsilon^2*y_2 + \upsilon*y_1 + y_0 = \upsilon^3*y_3+\upsilon^3+\upsilon^2*y_2 + \upsilon*y_1 + y_0 = Y + \upsilon^3$$

So the only thing that we need to do is to make one of other polynomials contribution to evaluation to be equal exactly $\upsilon^3$ and than exclude it from the list of evaluatiions by zeroing it.
It doesn't matter which other polynomial we use, for example we could take $y_2$ and set it's only coefficient to be $\upsilon$. 
$\upsilon^3*y_3+\upsilon^2*y_2 = \upsilon^3*y_3+\upsilon^2*\upsilon = \upsilon^3*(y_3+1) + 0 = \upsilon^3*(y_3+1) + y_2'$

So that gives us exactly what we wanter if we pass a fake $y_2$ evaluation $y_2'=0$
We could do the same with $y_1$ or $y_0$ changing a constant coefficient to $\upsilon^2$ and $\upsilon^3$ respectively
#### Code 

The solution for $\upsilon$ trivial , we just set it to zero 
```rust
let u = FieldElement::zero();
```
and now we can create a new set of evaluations, that includes a fake value
```rust
let y4_fake = y4.clone() + FieldElement::one();
let ys_fake = [y1.clone(), y2.clone(), y3.clone(), y4_fake.clone()];
let total_power = u32::from_str_radix(&ys_fake[3].to_string()[2..], 16).unwrap();
println!("total power: {}", total_power);
assert!(total_power > 9000);
assert!(kzg.verify_batch(&x, &ys_fake, &ps_c, &proof, &FieldElement::zero()));
```
Now we have Gohan power 9001 that still statisfies batch proof check

For alternative solution we change coefficient for an individual polynomial:
```rust!
// Sample random u
let u = FieldElement::from(rand::random::<u64>());
let p1_coeffs = [u.pow(3 as u32)];
```
This is the case for $y_0$ (indexes in code start from 1 for some reason, so it corresponds to `y1` variable and `ys[0]` element in evaluations array). Of course for $y_1$ and $y_2$ everything is the same
After that you remove $y_1$ contribution to the batch, so that $y_3$ would take all the credit
```rust
let ys_fake = [FieldElement::zero(), y2, y3, y4 + FieldElement::one()];
```
And you're good to go!

```rust
let total_power = u32::from_str_radix(&ys_fake[3].to_string()[2..], 16).unwrap();
println!("total power: {}", total_power);
assert!(total_power > 9000);
assert!(kzg.verify_batch(&x, &ys_fake, &ps_c, &proof, &u));
```
You will see that $y_3$ (`y4` variable, `y[3]` element in evaluations ) is greater than 9000 and the batch proof is again successfully checked.

#### Overview

This challenge was comparatively easy if you get familiar with KZG. From the very first sight batch proof seems suspicious. Furthermore vulnerabilities that are caused by bad Fiat-Shamir are very common, so this should be one of the first things to check.
To fix this protocol we would have to either check individual proofs, which defeats the purpose of batching, or we need to make sure, that the Prover can't mess with the challenge, because otherwise everything can be just solved backwards for any specific result the Prover chooses.

