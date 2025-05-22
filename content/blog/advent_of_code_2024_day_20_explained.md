+++
title = "Advent of Code 2024 Day 20 Explained - Race Condition"
description = "Detailled explaination of how I solved the day 20 for AoC 2024, Race Condition, with a link to my Rust implementation."
date = 2025-04-16
+++

You can find my [Rust implementation on Github](https://github.com/thothbaboon/advent_of_code/blob/master/src/y2024/day20/mod.rs).

## Part 1 – Brute Force to Elegance

> Each cheat has a distinct start position (the position where the cheat is activated, just before the first move that is allowed to go through walls) and end position.
> 

This detail is crucial: a cheat activated at time `t` and lasting `N` picoseconds allows us to go through **up to N−1 walls** — because the first move happens *after* activation. And the cell at `t + N` must be a **track cell**.

### First Approach

My first instinct was to reuse Dijkstra, which had already helped me solve a couple of AoC 2024 problems.

Here’s the idea:

1. Run Dijkstra on the original racetrack to get the baseline shortest distance.
2. Then, try all possible cheats:
    - For each **track** cell, in all four directions,
    - If a wall is directly adjacent, temporarily remove it.
    - Run Dijkstra again on the altered racetrack.
    - If the new distance saves **at least 100 units**, the cheat is **qualified**.

✅ It worked.

🐌 But it was slooow — running Dijkstra for every wall possibility takes time. A few minutes per run.

### A Smarter, Faster, and Sexier Approach

It took me time to realized that removing a wall is really just **connecting two paths that couldn’t connect before**.

Say we have a wall cell `W`, with two adjacent track cells `A` and `B` (in direction `A → W → B`).

We essentially have two separate paths:

- From **Start to A**
- From **B to End**

Removing `W` allows:

```
New Path = [Start → A] + [1 step through W] + [B → End]
```

So instead of running Dijkstra each time, we can:

1. Run Dijkstra **once from Start** to get the shortest path to every cell.
2. Run Dijkstra **once from End** (reverse mode) to get the distance from every cell to the end.
3. For each wall cell `W`, check its adjacent pairs of track cells `A` and `B`.
4. Compute:
    
    ```
    total = distance(Start → A) + 1 + distance(B → End)
    ```
    
    If this total is at least 100 shorter than the baseline path → it’s a **qualified cheat**.
    

⚡️ Much faster, much cleaner.

## Part 2 – Bigger Cheats, Same Trick

Now, cheats can last **up to 20 picoseconds** instead of 2.

> Cheats don't need to use all 20 picoseconds; they can last any amount of time up to and including 20 picoseconds (but can still only end when the program is on normal track). Any unused time is lost; it can't be saved for another cheat later.
> 

At first I thought it might require a different strategy. I was thinking of something recursive.

But then I realized only small changes in part 1 solution was needed.

Let’s visualize the cheat. When we activate it on a **track cell**, it means walls disappear for the next 19 moves.

Another way to look at it: we can **teleport** to any track cell within a Manhattan distance ≤ 20.

⚠️ Important: the **arrival cell** must be a **track cell** — a cheat can’t end on a wall cell.

So Instead of removing a **single wall** as in part 1, we’re now allowed to “teleport” from one track cell to another, as long as the Manhattan distance between the two cells is **≤ 20**.

1. Run Dijkstra **once** from the Start to get distances to all cells.
2. Run Dijkstra **once** from the End (in reverse) to get distances from all cells to the End.
3. For every pair of track cells `A`, `B` such that `manhattan(A, B) ≤ 20`:
    - Compute total distance: `distance(Start → A) + manhattan(A, B) + distance(B → End)`
    - If that total saves at least 100 compared to the baseline path → **qualified cheat**.

We’ve just solved Part 2 with the same logic, generalized 🎉