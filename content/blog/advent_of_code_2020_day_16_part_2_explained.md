+++
title = "Advent of Code 2020 Day 16 Part 2 Explained - Ticket Translation"
description = "Detailled explaination of how I solved the part 2 of day 16 for AoC 2020, Ticket Translation, with a link to my Rust implementation."
date = 2025-05-27
+++

You can find my [Rust implementation on Github](https://github.com/antoineprdhmm/advent_of_code/blob/master/src/y2020/day16/mod.rs).

## Part 1

As usual, this was the easy part.

For each field, a set of valid value ranges is specified. If a value fits at least one field range, it’s considered valid.  
Summing all the invalid values across all nearby tickets gives the answer for part 1.

Next!

## Part 2

Nothing too difficult here either. But it did teach me a lesson, which is why I decided to write a quick blog post about it.

The goal is to determine the order of the fields on the ticket based on the ticket values.

The tricky part is that a single field can match multiple positions.

```txt
class: 0-1 or 4-19
row: 0-5 or 8-19
seat: 0-13 or 16-19

your ticket:
11,12,13

nearby tickets:
3,9,18
15,1,5
5,14,9
```

In this example, `row` could correspond to the first, second, or third values.

So, my initial approach was recursive:
- All tickets have N values.
- Start at value index 0, with all fields available.
- For each value index (from 0 to N-1):
    - Identify all fields that are valid for that position across all valid tickets.
    - For each possible field, make a recursive call to the next index with that field marked as used.
- Stop when each field has been assigned to one position—solution found!

**This works and gives the correct answer. But it’s slow..** - about 110 seconds, due to the number of combinations explored.

So I searched for a faster approach.

That’s when I realized something important when looking more deeply at the input: **there is actually only one possibility for each field**.

Here’s a log of value indexes (i.e. ticket columns) followed by the field indexes that are valid for each:

```
13 -> [12]
10 -> [8, 12]
12 -> [8, 12, 14]
9 -> [8, 9, 12, 14]
18 -> [4, 8, 9, 12, 14]
16 -> [4, 8, 9, 12, 14, 18]
5 -> [4, 8, 9, 12, 14, 15, 18]
1 -> [4, 5, 8, 9, 12, 14, 15, 18]
4 -> [4, 5, 8, 9, 10, 12, 14, 15, 18]
3 -> [4, 5, 8, 9, 10, 12, 14, 15, 18, 19]
2 -> [4, 5, 8, 9, 10, 12, 14, 15, 17, 18, 19]
0 -> [2, 4, 5, 8, 9, 10, 12, 14, 15, 17, 18, 19]
8 -> [2, 4, 5, 7, 8, 9, 10, 12, 14, 15, 17, 18, 19]
7 -> [0, 2, 4, 5, 7, 8, 9, 10, 12, 14, 15, 17, 18, 19]
17 -> [0, 2, 4, 5, 7, 8, 9, 10, 12, 14, 15, 16, 17, 18, 19]
14 -> [0, 2, 4, 5, 6, 7, 8, 9, 10, 12, 14, 15, 16, 17, 18, 19]
11 -> [0, 2, 4, 5, 6, 7, 8, 9, 10, 12, 13, 14, 15, 16, 17, 18, 19]
19 -> [0, 2, 3, 4, 5, 6, 7, 8, 9, 10, 12, 13, 14, 15, 16, 17, 18, 19]
6 -> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 12, 13, 14, 15, 16, 17, 18, 19]
15 -> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
```

For the value at index 13, only the field 12 is valid.

For the value at index 10, fields 8 and 12 are valid—but since field 12 is already assigned to index 13 (it's the only valid option), only field 8 can be assigned here.

And so on.

This creates a chain of constraints that leads to a unique mapping. Once I realized that, the solution became straightforward.

In theory, there could have been multiple mapping giving the same answer, as we only care about multiplying the values for the fields starting with `departure`. That's what led me to a recursive approach in the first place.

**But I should have spent more time analyzing the input before jumping into
writing code. 😅**

I even saw later on Reddit that this unique solution made it possible to solve part 2 by hand.

Still, I got the answer pretty quickly with my first solution.
