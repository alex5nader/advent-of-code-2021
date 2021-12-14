
[Day 3](https://adventofcode.com/2021/day/3)!

## Part 1
Day 3's input is a list of binary numbers. For Part 1, the goal is to find the most and least common bits
in each digit position.

### Solution
The hardest part of this problem in APL is parsing the input. It's given as a list of strings, so we have
to convert those to integers. The simplest way to do this is the system function `⎕UCS`. This gives the
unicode value of a character, so then all that's needed is to convert that to its actual numeric value. To
do that, we can simply subtract the unicode value of `0`: `(⎕UCS'0')-⍨`. Combining these gives the final
parsing function: `parse ← (⎕UCS'0') -⍨ ⎕UCS`.

Now for the easy part: arrange the input into a 2d array and just sum it up! Here's what that looks like:
`counts ← +⌿↑`. Really, that simple! This gives us the number of 1s in each digit position.

To check if each digit has a majority of 1s, simply check if it's greater than half the input length.
Finding this is simple: `halfLength ← .5×⍴`.

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

This leaves our final solution: `×/ 2∘⊥¨ ↓ (~,[0.5]⊢) (counts > halfLength) parse¨`

## Part 2
Part 2 asks us to only consider input entries that contain the most or least common value in future processing. This makes the problem significantly harder, as now the second value is *not* just the inversion of the first value (the inversion could be missing from the input, so another value must be the answer).

### Solution
First, our counting function should now only count a single digit. I'll define this as a diadic function where the left argument is the digit index to count and the right argument is the input: `countAt ← {+/ ⍺⊃ ↓⍉↑⍵}`.

Next, we have to figure out the most or least common digit here. Again, we can use `halfLength`. However, Part 2 introduces the fact that there may be a tie, in which exactly half the numbers have a 0 and 1. To handle this, if a tie happens, when finding the most common, 1 is used, and when finding the least common, 0 is used. This means our digit selection function needs three array inputs: the index of digit to check, the winner if a tie happens, and the list of binary numbers. Additionally, whether we want most or least common is also an input. This can be implemented as an operator:

```apl
extremaAt ← {
	k ← 1⊃⍺ ⋄
	tieWinner ← 2⊃⍺ ⋄
	input ← ⍵ ⋄
	oneCount ← k countAt input ⋄
	threshold ← halfLength input ⋄

	oneCount>threshold: ⍺⍺ 1 ⋄
	oneCount<threshold: ⍺⍺ 0 ⋄
	tieWinner
}
```

Here, `⍺⍺` is how this operator differentiates between most and least common. For most common, we pass `⊢` to leave the 1 or 0 unchanged. For least common, we pass `~` to flip it. Finally, if it was neither greater or less than, it must have been equal, so we use the tiebreaker value.

Next, we have to actually use this function. This gives us the target digit for each place, but now we have to figure out what inputs to pass to it. The inputs we give it must have matching digits in all the previous places. We can get these values by filtering the input. Whenever an item has a matching digit, we want to keep it. Otherwise, we want to throw it away. To do this, we need a filter function. Sadly, APL does not provide this, so here's a definition: `filter ← {(⍺⍺¨⍵)/⍵}`. This maps each item of its input using a function, and then takes only the items where that function produced a 1. We can use it as such to find our target values, assuming the most or least common is stored as `extrema`: `{extrema = k⊃⍵} filter`. If the kth digit in an item is equal to `extrema`, the item will be included. Otherwise, it's removed. This is the final piece in our puzzle.

To combine these functions, we will need a recursive operator. Each level of recursion handles a given digit position. Here's the final operator:

```apl
find ← {
	findRecur ← {
		1=⍴⍵:⍵ ⋄
		k←⍺ ⋄
		extrema ← k ⍺⍺ ⍵ ⋄
		(k+1) ∇ ({extrema = k⊃⍵} filter ⍵)
	} ⋄
	⊃ 1 ⍺⍺ findRecur	⍵
}       
```

The function this operator expects differentiates between most and least common. Here are its two definitions:

```apl
{(⍺ 1) ⊢ extremaAt ⍵}
{(⍺ 0) ~ extremaAt ⍵}
```

As you can see, these integrate the two differences between most and least common stated above (1 vs 0 for tie breaking and `⊢` vs `~` for flipping the result). Applying find with both of these and the input gives us our two numbers. To get both at once, we once again use `,[0.5]`: `({(⍺ 0) ~ extremaAt ⍵} find) ,[0.5] {(⍺ 1) ⊢ extremaAt ⍵} find)`. From here, the steps are the same as in Part 1.

This (finally) gives us our final solution!

```apl
×/ 2∘⊥¨ ↓ (({(⍺ 0) ~ extremaAt ⍵} find) ,[0.5] ({(⍺ 1) ⊢ extremaAt ⍵} find)) parse¨
```
