# The Lost Relic

## Challenge
> During their quest to find the greatest treasure in the world, the One Piece, Luffy and his friends are wandering inside a subterranean maze. After many hours, they arrive at the door hiding an old relic, which can be instrumental to achieving their goal. The big problem is that it is made of sea stone and Luffy is unable to use his strength to break it. There are some inscriptions on the walls, which Nico Robin is able to translate.
> 
> It says: "If you can find the secret hidden among these texts, the door will open."
> 
> There are many input plaintexts and their corresponding ciphertexts, all of them encrypted using a custom MiMC algorithm under the same key. There are also many skeletons around, of all the people who have so far failed this test. Luckily, Usopp brought his computing device and will try to break the secret. What can he do to recover the secret?
> 
> https://github.com/ingonyama-zk/the_lost_relic

## Comments
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)
We're given a bunch of plaintext-ciphertext pairs. The block cipher function, a modified MiMC hash, for each round is given as:

$$
x_{n+1} = (x_n + k)^2
$$

Our goal is to find the secret key, $k$.

I spent a lot of time trying to devise an algebraic attack on the cipher, given its simplicity. However, I couldn't figure out a way. After the CTF was over, I was given the hint that one of the pairs was a slide pair and we had to do a [slide attack](https://en.wikipedia.org/wiki/Slide_attack). Below is the implementation of this attack in Sage:

```python
from sage.all import *
from itertools import permutations, product

# Stark252 prime field
p = 0x800000000000011000000000000000000000000000000000000000000000001
Fp = GF(p)

g = Fp.multiplicative_generator()
print(g.order())
print(factor(p - 1))

# Plaintext-ciphertext pairs
data = [
    (0x28b0063846b89588ca3685b89a541f0d928fe5a9cbe96d5abb7f7abb629d3e2, 0xce80a279e03ffd68f6394329f7d10991cc93950cddb7506485d8ff6ed7b8e2),
    (0xbf1ce8008359702c83d1bac70b7612c928687a3eeb1f753968d2e194868429, 0x45230772a45ccf3dd959fe19551b8e8a2d477bb9cc34ddada45f8b5e1df7c60),
    (0x7e2b171d99abe9032dfea6b45a926f5ee48f8e12b0ff78adcdf584c4c639bd1, 0x305f75862c31d4b39d39d153bbcfdc057ad3c1be8ea7ff4caf98c19b902063a),
    (0x2e15cb9cc16936d3ae45a48908ae64188ad3e54b6461d026de2e9119a0fb92c, 0x6a03ffde37a63f1d6f8457c32319095fdd66f0aaa2bcd11aae2c392c2b2a5bd),
    (0x5302462b15a1278251a78d55d73808c6baea5f97239698ec01ffe8a9ae08b9d, 0x6f6242d42fff75a04ff5616c1e8462885c9ae45823f66313f99f2e746df9add),
    (0x136d355d557ae436cb02f0b10b846771cfc84b9179043c846508844b14027f7, 0x4f052e8a02fc19db85484ab772b86e873d225077fc62050e42611d104283cae),
    (0x122fead16809905bdeca0792e650b0c14d217a2a6e1228621f51361594b4f06, 0x3f7755174d8812e217affe00a53cf48ecf507d0ba0f7ffd0a52eda877ae67d2),
    (0x77c1a014b96c0de113bd4594a98d5e15241969b88629ad1a18ca4a324af353b, 0x6f87bb1349583cf1d382f0ddf9987d21cbbdb54afe279e87df057ee8516318b),
    (0x77365982f9d84204a371ee8301edc2eff4a907094b2c81992727738758db99c, 0xc40395a16b69960867177ccad31c172dfc205fa4f305eaf7f4f44e08629da9),
    (0x5d46134ee8397c82949da3f15831730dae280e40d0ee4af4f9be056cd1ce05a, 0x3690fd96de5feadedb4a588eb8a27c4b279d384c3529a0b9eabd5546d79d331),
]
data = [(Fp(x), Fp(y)) for x, y in data]

# Find slide pair by looking at all permutations and calculate the secret key
for (x1, y1), (x2, y2) in permutations(data, 2):
    if pow(y2, (p-1)//2) == 1 and pow(x2, (p-1)//2) == 1:
        x2roots = [x2.sqrt(), -x2.sqrt()]
        y2roots = [y2.sqrt(), -y2.sqrt()]

        for (x2root, y2root) in product(x2roots, y2roots): 
            if y2root - x2root == y1 - x1:
                k = x2root - x1
                print(f"k: {k}")
                print(f"k: {hex(k)}")
                break
```

