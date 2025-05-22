+++
title = "Advent of Code 2024 Day 16 Part 2 Explained - Using Dijkstra"
description = "Detailled explaination of how I solved the part 2 of day 16 for AoC 2024 using Dijkstra algorithm, with a link to my Rust implementation."
date = 2025-04-04
+++

Advent of Code 2024 has been a smooth ride for me until Day 16. The second part was more challenging but fun to solve.

I noticed a lot of people on Reddit were struggling with it, so I decided to share my approach using Dijkstra's algorithm.

Hopefully, this will help some of you.

 I haven’t included code snippets here, but you can find my [Rust implementation on GitHub](https://github.com/thothbaboon/advent_of_code/blob/master/src/y2024/day16/mod.rs).

## Part 1: Dijkstra

I won’t dive too deep into Part 1 since it’s relatively straightforward. I managed to solve it using [Dijkstra's algorithm](https://en.wikipedia.org/wiki/Dijkstra's_algorithm).

The only twist was managing the **direction** and **rotation**.

Dijkstra’s output is a HashMap, where

- The key is a tile and a direction.
- The value is the smallest score for that entry.

```rust
type SmallestScoresByTile = HashMap<(Position, Direction), usize>;
```

Since each tile in the maze can be reached from **four different directions**, we need to consider all of them.

`TileCandidateScore` struct represents a candidate score for a tile’s direction.

These candidates are pushed into a **min BinaryHeap**. Since the heap sorts candidates by their scores (smallest first), the lowest score pops first.

Once a tile and direction combination pops, future candidates for that combination are ignored, because the smallest score has already been found.

The exploration starts with tile `S` with a score of `0`, since it's the starting point. From there, the algorithm explores the maze.

For Part 1, you can stop once you reach `E`. However, Part 2 requires collecting the best scores for **all tiles in the maze**.

The answer to Part 1 is simply the **smallest value** among the four entries for tile `E` in `SmallestScoresByTile`.

## Part 2: Finding All Best Paths

In Part 1, we only needed to find *one* best path to reach `E`. But there could be multiple paths with the same best score.

The goal of Part 2 is to determine the number of **distinct tiles** that belong to a best path. To do this, we need a way to trace *all* best paths.

Fortunately, Dijkstra’s algorithm already gives us all the best paths. So, we can reuse the code from Part 1.

### Recycling Part 1’s Output

The output of our Dijkstra function from Part 1 is this `HashMap`:

```rust
type SmallestScoresByTile = HashMap<(Position, Direction), usize>;
```

Each tile of the maze can be reached in **4 different states**—one per direction. The value in the `HashMap` represents the **smallest score** for reaching that tile from a given direction.

The question is: how to use this HashMap to find all best paths?

### Let’s Take a Simple Example

```
#####
###E#
#...#
#.#.#
#...#
#.###
#S..#
#####
```

- `E` is located at `(1, 3)`
- `S` is located at `(6, 1)`

There are 2 best paths to reach `E` from `S`.

The best score at `E` is `3007` with direction `NORTH`.

For each best paths, there are

- 8 tiles
- 3 rotations

`(8-1) * 1 + 3 * 1000 = 3007`

```
#####
###O#
#OOO#
#O#O#
#OOO#
#O###
#O..#
#####
```

The best score at tile `E` is obtained with direction `NORTH`. Other directions at `E` have higher scores, because they just involve unnecessary rotations from `3007 NORTH`.

### Tracing Backwards from `E`

So the best paths comes from the adjacent tile in **opposite direction**. 

`SOUTH` to `(1, 3)` is `(2, 3)`.

Since the best score at `E` is achieved facing `NORTH`, the optimal path must come from an adjacent tile in the **opposite direction**—`SOUTH`.

So, the predecessor tile is `(2, 3)`.

Let’s look at the entries in the `HashMap` for `(2, 3)` .

- `3006 NORTH` (from `(3, 3)` with direction `NORTH`)
- `2006 EAST` (from `(2, 2)` with direction `EAST`)
- `3006 SOUTH` (from `(2, 2)` with direction `EAST` + clockwise rotation)
- `4006 WEST` (but 4006 > 3007, so we can ignore it)

Now, **let’s see which entries for `(2, 3)` can contribute to a best path ending at `E`** (`3007 NORTH`).

|Entry|Analysis|Valid|
|:---|:-------------------|:---:|
|`3006 NORTH`|Already facing `NORTH`, just move forward.<br>**3006 + 1 = 3007**|✅|
|`2006 EAST`|Facing `EAST`. Need to face `NORTH`, then move forward.<br>**2006 + 1000 + 1 = 3007**|✅|
|`3006 SOUTH`|Facing `SOUTH`. Need to face `NORTH`, then move forward.<br>**2006 + 2000 + 1 = 4007**|❌|
|`4006 WEST`|Already higher than `3007`|❌|

Both 3006 NORTH and 2006 EAST are valid entries to reach (1, 3) with a score of 3007. **Because there are two valid entries, it means two best paths pass through this tile.**

### Recursively Finding All Best Paths

The same logic applies recursively. Now, we need to do the same thing we did for `E 3007 NORTH`, but for `(2, 3) 3006 NORTH` and `(2, 3) 2006 EAST`.

Once that’s done, we’ve found all the tiles belonging to the best paths.

And that’s it! 🎉
