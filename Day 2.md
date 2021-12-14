[Day 2](https://adventofcode.com/2021/day/2)!

## Part 1
Day 2's input is a list of movement commands. Part 1's goal is to figure out the final position from these commands.
To model this, we'll make a nested array where each element is a pair of direction and magnitude, like this:

```apl
('forward' 1) ('up' 2) ('down' 3)
```

### Solution
Because APL only has arrays, we'll model a position using an array of length 2. The first item will be horizontal
position and the second item will be depth.

The first step is to translate the verbal commands into changes in position. `forward X` translates to the array `X 0`,
as when this is added to a position, the result is moved by X horizontally. `up X` translates to the array `0 ¯X` and
`down X` translates to `0 X`. These might seem flipped, but remember that depth increases as you go *down*, not up. The
actual function that does this is fairly complex:

```apl
translate ← {
  ('forward'≡1⊃⍵): (2⊃⍵) 0 ⋄
  ('up'≡1⊃⍵): 0 (-2⊃⍵) ⋄
  0 (2⊃⍵)
}
```

To apply this to the whole input, we can use map (`¨`).

After this, we can just apply all these changes in position. This is exactly what a + reduction (`+/`) is for!

Here's the final solution, compacted a bit: `+/translate¨`.

(Technically the actual answer requires us to multiply the resulting two coordinates.This can be done with a
× reduction: `×/`. *Also*, the result is boxed, so it must be unboxed with `⊃` This means the *real* final
solution is `×/⊃+/translate¨`.)

## Part 2
Part 2 modifies how the commands affect our position. Now, up and down don't directly move, but change "aim". This
is basically a third coordinate, so we can model the new position as an array of length 3, where the first two items
are the same as before, and the third item now represents the aim. `forward X` now affects horizontal position by `X`,
but modifies depth by the product of `X` and "aim".

### Solution
Unfortunately, since `forward` now requires the current position to apply its operation, we can't map the list and sum
afterwards. However, we can do these operations at the same time using a reduction! Unfortunately again, APL's
reduction operator runs from right-to-left, which means we have to define our own modified version:

```apl
foldl←{⊃⍺⍺⍨/(⌽⍵),⊂⍺}
```

This performs a left-to-right reduction (also called a fold in other languages). We can use this, along with a modified
translation function, to find the new ending position:

```apl
translate ← {
  ('forward'≡1⊃⍵): ⍺+ (2⊃⍵) ((2⊃⍵)×(3⊃⍺)) 0 ⋄
  ('up'≡1⊃⍵): ⍺+ 0 0 (-2⊃⍵) ⋄
  ⍺+ 0 0 (2⊃⍵)
}
```

This is a dyadic function, where `⍺` represents the current position, and `⍵` represents the command that was issued.

To apply this to our input, we use foldl and an initial posiiton of `0 0 0`: `0 0 0 ∘ (translate foldl)`. `∘` is needed to
partially apply foldl, which normally would expect the initial value *and* the input to be specified.

(Again, the actual answer wants the product of depth and horizontal position, which is now more annoying to get. I
just decided to take the first two items (`2↑`) of the result and multiply reduce (`×/`) them again. This makes the final
solution `×/2↑ 0 0 0 ∘ (translate foldl)`)
