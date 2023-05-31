# ZokClub

## Challenge
> In the last night's operation, the Crypto Police have discovered the former meeting place of a notorious anonymous crypto criminal secret society known as the "ZokClub." The police raided the location after receiving a tip-off from an anonymous source and found nothing except for a single note.
> 
> Unfortunately, the note said nothing... The meaning behind the note remains a mystery, but it is suspected to be a code that only members of the ZokClub would understand.
> 
> The ZokClub has been known to operate in the shadows of the dark web, engaging in illegal activities such as money laundering and hacking. The group's anonymity and sophisticated use of blockchain technology have made it difficult for law enforcement agencies to track them down, despite the fact they have their own public web page: https://zokclub.ctf.ingonyama.com/
> 
> We need your help to understand what are they up to. Try to infiltrate their group and gather the secret information.

## Solution 
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)
The website made it clear that only holders of the ZokClub NFT could access the secret information. The site featured a detailed diagram outlining the club's operational blueprint: each club member possesses a secret and a nullifier that are used to create a Merkle tree. Using the secret-nullifier combo and a given Zokrates program, they generate a zero-knowledge proof proving their club membership. By submitting this proof on another page, they could mint a club NFT. Once the NFT is in the wallet, the secret can be revealed.

As we're not give a secret or a nullifier, I guessed that the objective was to forge a proof using  arbitrary values. The Zokrates code, which generates the ZK proof, is as follows:

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

On closer inspection, one can spot that there's no assert statement in the last line. Instead, the function merely returns a boolean indicating whether or not the nullifier+secret combo is contained in the tree. Even if the boolean is false, a proof can still be generated. So the exploit is to use an arbitrary nullifier, its hash, and the correct Merkle root and Merkle proof to generate a counterfeit proof.

Upon submitting the forged proof and minting the club NFT, we get access to the secret flag, thereby accomplishing our infiltration into the ZokClub.
