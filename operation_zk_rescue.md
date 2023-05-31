# Operation ZK rescue

## Challenge
> Message from HQ: High priority
> 
> **Agent Zulu has gone M.I.A.**
> 
> We have received reports that he has been kidnapped by the notorious Woe Jinx. Direct intervention without evidence is not an option. Our friendly enemy The Concierge of crime: Red, has managed to get one of his associates (The forger) infiltrate into Jinx's organization, who will be your point contact.
> 
> Your task is to send us a confirmation message that indeed Zulu is inside Jinx's base so we can rescue our man quietly.
> 
> Go to the Github Repository to get the full details on this challenge.
> 
> https://github.com/ingonyama-zk/operation_zk_rescue
> 

After executing the challenge using `cargo run`, you get an interactive story-telling prompt:

```
Agent Zulu has gone M.I.A.

We have received reports that he has been kidnapped by the notorious Woe Jinx.
Direct intervention without evidence is not an option.
Our friendly enemy The Concierge of crime: Red, has managed to get one of his associates
(The forger) infiltrate into Jinx's organization, who will be your point contact.

Your task is to send us a confirmation message that indeed Zulu is inside Jinx's base so we
can rescue our man quietly.

Jinx uses a sumcheck protocol that validates the sender's identity in the base,
when the sumcheck evaluates to zero.

The Forger has forged an identity for you in order to faciliate a one time message.
However, we ran some tests and found that it may not pass the validation.
We have no idea what game Red and the forger are playing here.

We do know that Woe Jinx protects his men from HQ by anonymizing the validation process, this basically
adds a random polynomial to the claimed polynomial. This is usually a real pain in the butt.
But, perhaps the anonymization can be used to your advantage this time. Just watch out that Jinx double checks the anonymization,
so if you use a constant polynomial for anonymization, you will get caught!

Once you have cleared the validation, we will use a security lapse window to activate recieving a one time message from you.
We have been told by Red that you will have to eventually find some of the information you need on your own.
U have got Big intELLIGENCE, be YOURSELF! We are expecting your message in 8 in the futURE.

Note that if Jinx learns the message during the validation, the probability you will live is pretty low.

One more thing, Red and the Forger cannot be trusted, there is always more to what meets the eye!
Watch out!  Good luck!! - HQ
```

## Solution 
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)

I wasted a lot of time analyzing the weird capitalization in one of the sentences. This turned out to be a red-herring and actually, none of the prompt is important.

If you convert the polynomial evaluations in the code into ASCII, you get the following list:
```
INGONYAMA_THEö
              #@Åç"
PINOCCHIO
TinyRAM
GROTH16
BULLETPROOFS
STARK
SONIC
PLONK
TURBOPLONK
SPARTAN
HALO
AURORA
MARLIN
FRACTAL
LUNAR
VIRGO
SUPERSONIC
PLOOKUP
BRAKEDOWN
NOVA
PLONKY2
HALO2
GEMINI
CAULK
CAULK+
ORION
FLOOKUP
HYPERPLONK
BALOO
CQ
SUPERNOVA
CQlin
```

Apart from the first one, the others are all names of modern ZK proving systems. The first entry looks like a corrupted flag and gives away a hint that the flag is related to the evaluations somehow.

Looking into the sum-check code, I realized that this is exactly the [ZK-hack puzzle](https://gist.github.com/shuklaayush/b92e6b53b0ff8571c0e73d42b504f7e3) that I did a few months ago. It's a sum-check protocol where the masking is improper. We can select any masking polynomial whose sum over the domain is 0, and the check will pass. If we calculate this sum over domain and turn that number into ASCII, we get the flag `INGONYAMA_THE_LION_INSIDE`. Upon entering this flag when the code prompts, the code calculates its hash and confirms that this is indeed the solution.
