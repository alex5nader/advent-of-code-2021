[Day 1](https://adventofcode.com/2021/day/1)!

## Part 1
Part 1's input is a simple list of numbers. The goal is to count how many times adjacent numbers increase.

### Solution
This problem is extremely simple in APL. First, to see where all the increases are, we can use a 2-reduction with the <
operator: `2</`. This runs the > function on every set of 2 adjacent elements.

Because booleans in APL are the integer
value 0 or 1, to find the number of increases, we can simply add up the resulting array using another reduction: `+/`.

This is enough to get the complete solution: `+/2</`!

## Part 2
Part 2 is extremely similar to part 1. Instead of checking where individual numbers increase, we check when sum of
the three adjacent elements increases.

### Solution
This is barely more complex than part 1 in APL. To get the sums of three adjacent elements, we can use a 3-reduction
with the + function. This is similar to the 2-reduction from Part 1: `3+/`. Once we have the sums, we can just use
the solution to part 1 on this data!

Here is the final solution: `+/2</3+/`. Wonderfully simple!
