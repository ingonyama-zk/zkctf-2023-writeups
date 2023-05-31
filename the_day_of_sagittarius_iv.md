# The Day of Sagittarius IV

## Challenge
> We are excited to announce the release of the latest version of the retro terminal game "The Day of Sagittarius IV". This new version is designed to be played peer to peer and does not require any servers to play. Instead, the game data is stored on the players' computers, creating a truly decentralized gaming experience.
> 
> To ensure that players adhere to the rules of the game, the new version utilizes a zero-knowledge virtual machine. This feature allows players to verify that their opponents are playing fairly without revealing any confidential information.
> 
> We are currently in the beta-testing phase, and we invite all gaming enthusiasts to try out the new version of "The Day of Sagittarius IV".
> 
> As part of the beta-launch, we are also excited to announce a special promotional event. Players who win a bot in the game on 52.7.211.188:6000 will be eligible for a prize.
> 
> We believe that the new peer-to-peer version of "The Day of Sagittarius IV" will revolutionize the world of retro gaming, and we are thrilled to share it with you. Join us in this exciting new adventure, and let's explore the universe together!
> 
> https://github.com/ingonyama-zk/ctf-day-of-sagittarius

## Solution 
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)

This was one of the most intriguing challenges: a modified [battleship game](https://en.wikipedia.org/wiki/Battleship_(game)) implemented using Zero-Knowledge (ZK) proofs in RISC-Zero. Each round, you could either: 

1. Fire a single shot in the cell
2. Send scouts to reveal enemy spaceships
3. Fire a claster charge, to hit from 2 to 4 random cells in an area

The game was played on an 8x8 board like the one below

```
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃          Player Board            ┃
┣━━┳━━━┳━━━┳━━━┳━━━┳━━━┳━━━┳━━━┳━━━┫
┃  ┃ a ┃ b ┃ c ┃ d ┃ e ┃ f ┃ g ┃ h ┃
┣━━╋───┼───┼───┼───┼───┼───┼───┼───┨
┃ 1┃ A │ A │ A │ A │   │   │   │   ┃
┣━━╋───┼───┼───┼───┼───┼───┼───┼───┨
┃ 2┃   │   │ D │ D │ B │ B │ B │   ┃
┣━━╋───┼───┼───┼───┼───┼───┼───┼───┨
┃ 3┃   │   │   │   │ C │   │   │   ┃
┣━━╋───┼───┼───┼───┼───┼───┼───┼───┨
┃ 4┃   │   │   │   │ C │   │   │   ┃
┣━━╋───┼───┼───┼───┼───┼───┼───┼───┨
┃ 5┃   │   │   │   │ C │   │   │   ┃
┣━━╋───┼───┼───┼───┼───┼───┼───┼───┨
┃ 6┃   │   │   │   │   │   │   │   ┃
┣━━╋───┼───┼───┼───┼───┼───┼───┼───┨
┃ 7┃   │   │   │   │   │   │   │   ┃
┣━━╋───┼───┼───┼───┼───┼───┼───┼───┨
┃ 8┃   │   │   │   │   │   │   │   ┃
┗━━┷━━━┷━━━┷━━━┷━━━┷━━━┷━━━┷━━━┷━━━┛
```

After playing (and losing) the game a few times against the bot, it became clear that the bot had some unfair advantage. It was hitting the target at each shot. Given that the game was based on a ZK design and  supposed to keep the state private, it wasn't immediately clear to me how the bot was cheating. I felt that either a backdoor was present, or the proofs were leaking information.

I started digging into the code to figure out where the bug was. In RISC Zero, [a proof](https://www.risczero.com/docs/explainers/proof-system/) comprises of a zk-SNARK and a journal entry that includes the public outputs of the computation. My hunch was that the journals were leaking state information so I started looking into the journals of all the actions. The cluster bomb one was interesting since it contained many fields:

```rust
pub struct ClusterCommit {
    pub old_state_digest: Digest,
    pub new_state_digest: Digest,
    pub config: ClusterBombParams,
    pub shots: alloc::vec::Vec<Position>,
    pub hits: alloc::vec::Vec<HitType>,
}
```

Everything seemed fine except for the config variable. Checking the definition of `ClusterBombParams`, we see that it contained the `GameState`, which should have been private. Voila!

```rust
pub struct ClusterBombParams {
    pub state: GameState,
    pub upper_left_coordinates: Position,
    pub down_right_coordinates: Position,
    pub seed: u8,
}
```
After finding this, I wrote a small piece of code that took this game state, rendered it and dumped it into a file. Now, all I had to do was start the game with a cluster charge, get the opponent's board state and use it to plan optimal moves. 

Both us and the bot play optimally now. But since we start first and the bot wastes a turn sending a scout, there we'd almost surely win. After winning, we are awarded with the the flag.

*Shoutout to another team who skipped all of this and instead tackled this challenge in true hacking spirit by writing a script to make the bot play itself and win.*
