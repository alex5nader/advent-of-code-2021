[Day 3](https://adventofcode.com/2021/day/3)!

## Part 1
Day 3's input is a list of binary numbers. For Part 1, the goal is to find the most and least common bits
in each digit position.

### Solution
The hardest part of this problem in APL is parsing the input. It's given as a list of strings, so we have
to convert those to integers. The simplest way to do this is the system function `⎕UCS`. This gives the
unicode value of a character, so then all that's needed is to convert that to its actual numeric value. To
do that, we can simply subtract the unicode value of `0`: `(⎕UCS'0')-⍨`. Combining these gives the final
parsing function: `parse←(⎕UCS'0')-⍨⎕UCS`.

Now for the easy part: arrange the input into a 2d array and just sum it up! Here's what that looks like:
`counts←+⌿↑`. Really, that simple! This gives us the number of 1s in each digit position.

To check if each digit has a majority of 1s, simply check if it's greater than half the input length.
Finding this is simple: `halfLength←.5×⍴`.

Now we have two functions that both need the input. To combine them, we can use a fork. This is a special
operation that takes two functions, applies them both using the same input, then combines their output.
We want to see if the counts are greater than half the length, so function one is `counts`, function two
is `halfLength`, and the combining function is `>`. To create a fork, just put the three functions next
to each other: `counts > halfLength`. It's really that simple!

To find the least common, we can just invert each bit with `~`. To get both, we can use another fork,
this time with `,[0.5]` (the .5 indicates we're making a new axis, and the 0 indicates it should go after
the first axis): `~,[0.5]⊢`. Now we have both binary numbers stacked on top of each other.

The only step left is to turn these back into decimal and multiply them. This takes a few simple steps:

1. Convert to nested array: `↓`
2. Convert each number to decimal: `2∘⊥¨`
3. Multiply: `×/`

This leaves our final solution: `×/2∘⊥¨↓(~,[0.5]⊢) (counts > halfLength) parse¨`

## Part 2
TODO
