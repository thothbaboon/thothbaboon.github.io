+++
title = "Advent of Code 2024 Day 21 Explained - Keypad Conundrum"
description = "Detailled explaination of how I solved the day 21 for AoC 2024, Keypad Conundrum, with a link to my Rust implementation."
date = 2025-04-21
+++

You can find my [Rust implementation on Github](https://github.com/thothbaboon/advent_of_code_2024/blob/master/src/day21/mod.rs).

For this puzzle, we have multiple layers of keypads.

The first layer is a **numerical keypad**, on which we have to type a code. But it can’t be accessed directly.

Instead, we use a robot to type the code on the numerical keypad. But this robot is not accessible either — we control it using another robot, and **each robot types on the directional keypad of the robot below**.

The goal is to find the shortest sequence of keys to type on the **top-level directional keypad** that makes the **first robot** type the target code on the numerical keypad.

Because robot *n+1* needs to hit multiple keys on its keypad to make robot *n* type a **single key**, the sequence grows **exponentially** as more layers are added.

Brute force works for Part 1 — but Part 2 requires some thinking.

## Part 1 – Understanding the Problem and Brute Forcing It

### Let’s Look at the Given Example

- The **numerical keypad** is in a *depressurized area*. We can’t write the code directly.
- We send **Robot 1 (R1)**. But because of radiation, we can’t type on R1’s keypad (**DK1**).
- We send **Robot 2 (R2)** to type on **DK1**. But it’s -40°C, so we can’t access **DK2** either.
- So we send **Robot 3 (R3)** to type on **DK2** — finally, we can type on **DK3**.

So we end up with a chain 

```
DK3 → R3 → DK2 → R2 → DK1 → R1 → numerical keypad
```

Our task: find the **shortest sequence** to type on **DK3** that makes **R1** type the desired code.

Let’s say we want **R1** to type `029A`.

- Typing `<A^A>^^AvvvA` on **DK1** makes R1 write `029A`.
- Typing `v<<A>>^A<A>AvA<^AA>A<vAAA>^A` on **DK2** makes DK1 write the above sequence.
- Typing `<vA<AA>>^AvAA<^A>A<v<A>>^AvA^A<vA>^A<v<A>^A>AAvA^A<v<A>A>^AAAvA<^A>A` on **DK3** makes DK1 write `029A`.

This last sequence is one of the shortest possible for DK3: **68 keys long**.

### Dijkstra Forever

We’re asked to find the **shortest sequence length** to type on **DK3**.

To make **R1** type `029A`, we break it down into shortest paths between:

- `A → 0` (the robot arm is over A by default)
- `0 → 2`
- `2 → 9`
- `9 → A`

So on **DK1**, we just need to concatenate the shortest paths between each of these key pairs — for example, `<A^A>^^AvvvA`.

Then, we move up a level:

- Find how to type this full sequence on **DK2** to simulate **DK1**.
- Then repeat again for **DK3**.

Since each keypad is a grid, we can represent it as a graph and use **Dijkstra** to find the shortest path between any two keys.

I’d already used Dijkstra to solve 3 AoC 2024 problems — here’s one more!

## I Got a Wrong Answer

One valid **DK3** sequence for making **R1** type `029A` is:

```
<vA<AA>>^AvAA<^A>A<v<A>>^AvA^A<vA>^A<v<A>^A>AAvA^A<v<A>A>^AAAvA<^A>A
```

Length: **68**

But my algorithm first returned:

```
<v<A>A<A>>^AvAA<^A>A<v<A>>^AvA^A<v<A>>^AA<vA>A^A<A>A<v<A>A>^AAAvA<^A>A
```

Length: **70**

At first, I thought my code was wrong — but the sequence *does* work and produces `029A`.

So what was wrong?

Turns out, **a sequence can be one of the shortest of layer `n` but does not generate a shortest sequence in layer `n+1`**.

Let’s say we want to find the shortest **DK1** sequences from `2 → 9`.

There are 3 options:

- `>^^A`
- `^^>A`
- `^>^A`

But the cost of producing these sequences on **DK2** can be different.

For example:

- `>^^A` → `vA<^AA>A` → length 8
- `^^>A` → `<AAv>A^A` → length 8
- `^>^A` → `<Av>A<^A>A` → **length 10!**

Even though `^>^A` is a valid shortest sequence on **DK1**, it’s not one of the shortest possible sequence on **DK2**.

Instead of returning the first shortest sequence in Dijkstra, I had to return all shortest sequences.

My solution for part 1 was:

- Generating **all** shortest sequences for all layers.
- Return the length of one shortest sequence in the last layer.

I won’t give too much detail here, because this approach **doesn’t scale**. Let’s talk about the sexier one.

# Part 2 - Forget the Strings, Go Recursive

In Part 1, we:

- Generated all shortest sequences on layer `n`
- Then simulated those on layer `n+1`
- Repeat until the top

With 25 layers, this completely blows up. The number of strings is insane.

### Key Insights

We **don’t need the actual sequence** — just its **length**.

As we saw before, to get the valid shortest sequences on layer `n`, we need to know the shortest sequences on layer `n+1` - the keys involved in one shortest sequence of layer `n` will have an impact of the length of the sequences created from this sequence on the layer `n+1`. 

So instead of generating strings, we just **ask the layer above**:

> "Hey, what’s the shortest sequence to go from X to Y?"
> 

### Recursive Solution

In the example, there is no layer on top of **DK3**. Which means any shortest sequence from key X to key Y in **DK3** is actually one of the shortest sequence.

We can give this shortest sequence length to **DK2**.

We sum everything up to the numerical keypad, and get the answer without any strings! 🎉