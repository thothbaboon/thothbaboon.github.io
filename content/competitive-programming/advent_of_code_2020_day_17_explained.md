+++
title = "Advent of Code 2020 Day 17 Explained - Conway Cubes"
description = "Detailled explaination of how I solved day 17 for AoC 2020, Conway Cubes, with a link to my Rust implementation."
date = 2025-06-05
+++

You can find my [Rust implementation on Github](https://github.com/antoineprdhmm/advent_of_code/blob/master/src/y2020/day17/mod.rs).


As we get closer to the end, problems usually become harder. For this one, the difficulty wasn’t solving the problem, but understanding it.  

The solution is actually super simple, and now that I’ve completed it, I feel like this blog post is kind of dumb. But still, it might not be useless for some.

## Some Clarity About the Problem

We are in an infinite grid, where only a few cubes in a compact area are active. Even though the grid is 3D in part 1 and 4D in part 2, it can be represented in 2D at first because it’s flat: all active cubes are on the same `z` (and also `w` for part 2) layer.

```
.#.
..#
###
```

It’s represented in 2D, but it’s actually 3×3×1 for part 1 (3D), and 3×3×1×1 for part 2 (4D).

The rules are as follows:

- If a cube is *active* and *exactly `2` or `3`* of its neighbors are also active, the cube remains *active*. Otherwise, the cube becomes *inactive*.
- If a cube is *inactive* but *exactly `3`* of its neighbors are active, the cube becomes *active*. Otherwise, the cube remains *inactive*.

This means that at each cycle, inactive neighbors of active cubes might become active.

That’s why after the first cycle, the area containing active cubes is no longer flat: there are inactive neighbors at `z = -1` and `z = 1` because they are adjacent to cubes at `z = 0`. So the rules must be applied to those as well—and some of them become active.

That’s also why after cycle 2, the active area isn’t 3×3 anymore but expands to 5×5, because some neighbors around the original 3×3 area in the `x` and `y` dimensions have become active.

```
After 1 cycle:

z=-1
#..
..#
.#.

z=0
#.#
.##
.#.

z=1
#..
..#
.#.

After 2 cycles:

z=-2
.....
.....
..#..
.....
.....

z=-1
..#..
.#..#
....#
.#...
.....

z=0
##...
##...
#....
....#
.###.

z=1
..#..
.#..#
....#
.#...
.....

z=2
.....
.....
..#..
.....
.....
```

## Implementation

We might first think of representing the grid as an array, but that’s not a good idea because:

- We actually only care about the active cells (see the rules), so we'd be storing a lot of useless data.
- The frame of view expands over time.

The only information we need to compute the next cycle is the set of active cubes from the previous cycle.

Then, the only cubes that can be affected are the active cubes and their inactive neighbors. All we have to do is count the number of active neighbors for each of these cubes and, based on the rules, determine if they’re active in the next cycle.

## Then I released…

I found out this is essentially the Game of Life problem ([https://en.wikipedia.org/wiki/Conway's_Game_of_Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life)).
