# Lost funds

## Challenge

> So last night I was going to transfer some crypto to a friend of mine. But when I copy-pasted his address - it was replaced by the different one, which i didn't notice till it was too late! Seems like I had some kind of malware installed on my computer... Anyway, can you track down where my funds went? I heard it's possible in blockchains. https://goerli.etherscan.io/tx/0xf681ca8c2999a11dc41983657af5afd164c56322925227b88f5693f1de8130f7 is the transaction link (whatever that is)

## Solution 
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)

Following the trail of transactions on Goerli etherscan, we see that the culprit bridged the funds to the zkSync Era testnet. They then proceeded to transfer them repeatedly from one wallet to another on ZkSync. In order to trace the funds, I wrote a small Python script. 

```python
# Part 1
block = 0
address = "0xa2041c55902585ca3295f034f18d9000ad07738d"
while True:
    print(address)
    txs = get_txs(address)

    for tx in txs:
        if int(tx["blockNumber"]) < block:
            continue
        if "transfer" not in tx:
            continue
        if (
            tx["transfer"]["tokenInfo"]["address"]
            != "0x0000000000000000000000000000000000000000"
        ):
            continue
        if tx["transfer"]["from"] != address:
            continue
        if tx["transfer"]["to"] == address:
            continue

        block = int(tx["blockNumber"])
        address = tx["transfer"]["to"]
        break
    else:
        break
print(address)
# 0xd7120a6b038ad942a2966c2769451d21d608006f
```

The funds lead to [this address](https://goerli.explorer.zksync.io/address/0xD7120a6B038aD942A2966c2769451d21d608006f) and are then transferred back to Goerli. After a couple of hops, they are sent back to the zkSync Era testnet and end up at this [final address](https://goerli.explorer.zksync.io/address/0x475e3f18be51a970a3079dff0774b96da9d22dbe), where they are used up on gas across multiple transactions. Weird.

I wrote a script to decode all these transactions. Cross-referencing the function signatures with similar functions on a contract database, I found out that these belong to contracts for an on-chain casino hosted at [zkasino.io](zkasino.io).

```python
# Part 2
address = "0x475E3f18Be51A970a3079Dff0774B96Da9d22dbE"
txs = get_txs(address)

for tx in txs:
    result = subprocess.run(
        ["cast", "4byte-decode", tx["data"]["calldata"]], capture_output=True, text=True
    )
    print(result.stdout)
```

![transactions](https://github.com/shuklaayush/ingonyama-ctf-solutions/assets/27727946/a589a52e-4a95-4143-acd5-be240fecc2dd)

I found the [profile](https://play.zkasino.io/profile?u=0x475e3f18be51a970a3079dff0774b96da9d22dbe) corresponding to the exploiter's address on zKasino. It had the this badge: 

![crypt0cr1m1nal zkasino](https://github.com/shuklaayush/ingonyama-ctf-solutions/assets/27727946/c44191d3-7139-4f00-830f-36266fe63ed9)

The username piqued my curiosity but at this point, I felt as though I'd hit a dead-end. After a while, I went back to this challenge again and searched for the username on various search engines. This led me to a [Twitter profile](https://twitter.com/crypt0cr1m1nal) with this tweet.

![crypt0cr1m1nal twitter](https://github.com/shuklaayush/ingonyama-ctf-solutions/assets/27727946/ac808f3c-9162-4ad6-a763-fad2f4d61853)

Hurray, we found the flag!
