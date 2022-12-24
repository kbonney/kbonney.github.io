# Substring Game: A lesson in efficiency
Hackerrank offers a fun and engaging way to learn coding through challenges. Recenly, I solved a challenge relating to substrings and learned about the important difference between a theoretically adequate script, and one that is practically implementable.
The challenge was this:
> Kevin and Stuart want to play the 'The Substring Game'.  
> **Game Rules:**
> - Both players are given the same string,
> - Both players have to make substrings using the letters of the string
> - Stuart has to make words starting with _consonants_.  
> - Kevin has to make words starting with _vowels_.  
> - The game ends when both players have made all possible substrings.  
> 
> **Scoring:**\
> A player gets `+1` point for each occurrence of the substring in the string
> 
> **For Example:**\
> String= _BANANA_
> Kevin's vowel beginning word = _ANA_ 
> Here, _ANA_ occurs twice in _BANANA_. Hence, Kevin will get `+2` Points.

This isn't really much of a game, since the scoring is completely determined by the choice of string. However, determining the winner of the game and knowing their score is an interesting problem. Let's take a look at my naive first approach.

This problem is about substrings, so my first impulse was to get my hands on all of the substrings of the given string. Here's a function that accomplishes this:
```
def substrings(string):
    subs = []
    for i in range(len(string)):
        for j in range(i+1, len(string)+1):
            subs.append(string[i:j]
    return subs
```
Here we first iterate through all characters of `string` using `i`, and then we look at all substrings starting at `string[i]` using `j`.
Adapting this to the task at hand, I came up with the following:
```
def minion_game(string):
    stuart_score = 0
    kevin_score = 0
    for i in range(len(string)):
        for j in range(i+1, len(string)+1):
            if string[i] in 'AEIOU':
                kevin_score +=1
            else:
                stuart_score +=1
        if kevin_score > stuart_score:
            print("Kevin "+str(kevin_score))
        elif kevin_score < stuart_score:
            print("Stuart "+str(stuart_score))
        else:
            print("Draw")
```
There are some particulars here that come from the output demands of the hackerrank problem, but the basic idea is that I iterate through each possible substring and distribute scores as the game rules dictate. Herein lies my mistake. Trained as a mathematician, it is easy for me to see the above code as a perfectly valid solution for the task. In some sense it is, given arbitrarily large access to computation time/memory this script will return the correct output. When writing proofs, arguably similar to writing an algorithm, this is all that matters. 

However, this script actually has to run, and when we note that the number of substrings for a string of length n is **O(n^2)**, it becomes clear that this code won't cut it. For example, if `string` has a length of 1,000, the script will carry out approximately 1,000,000 elementary operations - this is not tenable.

To fix this terrible inefficiency, I realized that the scoring system doesn't actually care about the entire content of a substring - only the first character. In this way, I can evaluate the scoring for all substrings starting at `string[i]` without actually iterating through the substrings explicitly. For each character `string[i]`, there will be `len(string)-i` substrings which start at this character. Using this idea, we adapt the script to the following:
```
def minion_game(string):
    stuart_score = 0
    kevin_score = 0
    for i in range(len(string)):
        if string[i] in {'A','E','I','O','U'}:
            kevin_score += len(string) - i
        else:
            stuart_score += len(string) - i
    if kevin_score > stuart_score:
        print("Kevin "+str(kevin_score))
    elif kevin_score < stuart_score:
        print("Stuart "+str(stuart_score))
    else:
        print("Draw")
```
By avoiding the `j` iteration, we can downscale our growth to **O(n)** - much better.

The punchline is that sometimes the most straightforward way to approach a coding problem is very inefficient, and that a little bit of critical thinking can go a long way in improving an algorithm.
