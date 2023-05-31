# A Tale of Two Keys
## Challenge
> Alice deployed a Groth16 based system, and to convince everyone her system is secure and the secrets used in the setup are not exposed, she thought of a clever way - she would publish a few different circuits that share the same secrets, that don't have a valid solution. In this way, the only way malicious prover could create proofs would be by using the exposed secret. she put a large bounty in one of these to incentivize hackers to look at it.
> 
> See more details on Github: https://github.com/ingonyama-zk/TaleOfTwoKeys
## Comments 
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)

(This challenge was updated during the CTF. I only solved the initial unmodified challenge.)

In the initial version, the problem posed was to find the square root of 15 and 17. Given that 15 is a quadratic residue in the BLS12-377 scalar field, calculating the square root was trivial. Thus, by simply generating a valid proof for the square root of 15 and submitting it, I was able to get the flag.

The revised challenge tweaked the parameters. The value 15 was replaced with 11, which is a non-residue. This turned the challenge into a seemingly more complex problem that I didn't have time to look into.
