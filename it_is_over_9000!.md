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

