# The Power of iNTEgers

## Challenges 
> 59-213-402-213-964-402-213-149-310-534
> 
> Format: XXX XXX XXX XXX XXX XXX XXX XXX XXX XXX (not specifically 3 letters, English)
> 
> **Hint**
> Greek: equal pebble

## Solution 
### By [Ayush Shukla](https://hackmd.io/@shuklaayush)
From the hint, it was clear that this challenge was based on Isopsephy ("equal pebble"), the Greek practice of adding up the numerical values of the letters in a word to form a total. I constructed an Isopsephy table for English letters to use:

| Key | Value |      | Key | Value |      | Key | Value |      | Key | Value |
| --- | ---   | ---  | --- | ---   | ---  | --- | ---   | ---  | --- | ---   |
| a   | 1     |      | k   | 20    |      | t   | 200   |      | y   | 700   |
| b   | 2     |      | l   | 30    |      | u   | 300   |      | z   | 800   |
| c   | 3     |      | m   | 40    |      | v   | 400   |      |     |       |
| d   | 4     |      | n   | 50    |      | w   | 500   |      |     |       |
| e   | 5     |      | o   | 60    |      | x   | 600   |      |     |       |
| f   | 6     |      | p   | 70    |      |     |       |      |     |       |
| g   | 7     |      | q   | 80    |      |     |       |      |     |       |
| h   | 8     |      | r   | 90    |      |     |       |      |     |       |
| i   | 9     |      | s   | 100   |      |     |       |      |     |       |
| j   | 10    |      |     |       |      |     |       |      |     |       |

The task was to identify meaningful words for each provided number that together also form a meaningful sentence/phrase. I used [NLTK](https://www.nltk.org/) library's Brown word list as my reference dictionary and wrote a small python script to find candidate words for each number. Here are the words that I found (ordered by decreasing frequency):

```bash
['in', 'end', 'aimed', 'media', 'cane']
['the', 'desire', 'homes', 'gate', 'helps', 'acted', 'orange', 'ribbon']
['guard', 'bus', 'urge', 'faster', 'jungle', 'crude']
['the', 'desire', 'homes', 'gate', 'helps', 'acted', 'orange', 'ribbon']
['mighty']
['guard', 'bus', 'urge', 'faster', 'jungle', 'crude']
['the', 'desire', 'homes', 'gate', 'helps', 'acted', 'orange', 'ribbon']
['fiscal', 'flesh', 'pink', 'ladies', 'ideals', 'lion', 'limp', 'shelf']
['not', 'father', 'facts', 'tragic', 'dates', 'sons', 'ton', 'pepper']
['liver', 'drums']
[59, 213, 402, 213, 964, 402, 213, 149, 310, 534]
```

Analyzing the potential word combinations and keeping in mind the CTF's lion-themed context, it was easy to figure out that "[in the jungle, the mighty jungle, the lion sleeps tonight](https://www.youtube.com/watch?v=OQlByoPdG6c)" was the solution. 

