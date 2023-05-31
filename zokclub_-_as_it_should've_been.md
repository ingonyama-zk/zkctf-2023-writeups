# ZokClub - As it should've been

# Challenge
> After our last infiltration ZokClub layed low and moved to another website: http://52.7.211.188:4000/
> 
> Crypto Police have discovered another former meeting place of a secret society. The police once again found nothing except for a single note.
> 
> **This time the note said: `S: ukEaZLLyQhfoKiKD, N: OCjw4gAbjXtTiJWJ`**
> 
> We looked through their website briefly, the only thing changed was that weird archive at GPI.zip
> 
> We again need your help to understand what are they up to. Try to infiltrate their group and gather the secret information.

## Solution 
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)

This challenge was an extension to the previous ZokClub one. It introduced a bug fix, now checking that the boolean output is `true` within the NFT smart contract. This time, we're also given a valid secret and nullifier. However, the nullifier had already been used to mint an NFT and trying to use it again would be rejected by the NFT contract.

So I went back to the Zokrates code:

```rust
def main(u32[8] root, private MerkleProof merkleProof, field nullifierHash, private field nullifier, private field secret) -> bool {
    // Check that note hash is in the merkle tree
    assert(checkMerkleProof(root, merkleProof));

    // Check that the nullifier hash match with public one
    field trueNullifierHash = calculateNullifierHash(nullifier);
    assert(nullifierHash == trueNullifierHash);

    // Construct note from secret and nullifier
    u8[16] nullifierBytes = cast::<128, 16>(nullifier);
    u8[16] secretBytes = cast::<128, 16>(secret);
    u8[32] constraintPreimageBytes = [...nullifierBytes, ...secretBytes];
    u32[8] trueConstraint = sha256padded::<32>(constraintPreimageBytes);
    
    return trueConstraint == merkleProof.leaf;
}
```

Inspecting further, specifically at the casting function, I found that higher order bytes were ignored during the casting of the nullifier:

```rust
def cast<N, P>(field input) -> u8[P] {
    bool[FIELD_SIZE_IN_BITS] bits = unpack(input);
    bool[N] bits_input = bits[FIELD_SIZE_IN_BITS-N..];
    assert(N == 8 * P);
    u8[P] mut r = [0; P];
    for u32 i in 0..P {
        r[i] = u8_from_bits(bits_input[i * 8..(i + 1) * 8]);
    }
    return r;
}
```

Knowing this, I realized it was possible to take the given nullifier (which was known to be in the merkle tree), prepend some bytes to it, compute the nullifier hash, and generate a valid proof. Since the new nullifier differed from the old one, it wasn't rejected by the smart contract.

So we again generate a fake proof, mint the NFT, and obtain the secret.
