+++
title = "Advent of Code 2024 Day 17 Part 2 Explained - Chronospatial Computer"
description = "Detailled explaination of how I solved the part 2 of day 17 for AoC 2024, Chronospatial Computer, with a link to my Rust implementation."
date = 2025-04-08
+++

When on Day 17 you see an input as small as this one, you know it’s not gonna be easy.

Here is a detailed explanation of how to solve it.

You can find my [Rust implementation on Github](https://github.com/thothbaboon/advent_of_code_2024/blob/master/src/day17/mod.rs).

# Part 1: Running the Program

The logic to solve part 1 is actually pretty simple. The only difficulty is the amount of information to process. One can easily miss one bit of information or mix a few things and spend time finding where the issue comes from.

But getting the right answer to part 1 ensure we understood well how the computer is working.

And we are ready for the meat and potatoes.

# Part 2: Program Printing Itself

The goal is to find a value for register A that makes the program print itself.

## Example Input

I started by analysing the example program to understand how the program prints stuff - **when**, and **what value**. If we understand this, then we can probably make the program print what we want.

We’re given:

```
Register A: 2024
Register B: 0
Register C: 0

Program: 0,3,5,4,3,0
```

The program is composed of the following instructions:

- `0,3` → `A = A / 8`
- `5,4` → prints `A % 8`
- `3,0` → jumps at the beginning if `A != 0`

Or written in code:

```rust
while a > 0 {
    a = a / 8;
    println!("{}", A % 8);
}
```

`A` is getting smaller at each iteration, which makes the program stop at some point. The bigger `A`, the most values will be printed.

*I supposed there was some mathematical trick here with the 8. But I’m not a math guy, so I asked a hint to chat GPT about this program.*

*He told me to look at octal decomposition.*

💡  `% 8` gives the least 3 significant bits (from 0 to 7)

- 000 = 0
- 111 = 7

💡 `/ 8` dividing by 8 is making a right shift of **3** bits (2^**3** = 8)


### Octal Representation

The program simply prints the **octal representation** of A: it takes the bits 3 by 3 from A, and prints the values.

It prints the rightmost 3 bits first. So we just need to take the printed values in reverse order, which gives us A in base 8. Then convert from base 8 to base 10, and we get A.

`A` base 8 = 0345300

*Notice that there is an extra 0 at the right. It’s because the program starts with a division, then prints. If we want the program to print 6 values, we need 7 values. We set 0 because it’s the smallest value, which is what we want.*

`A` base 10 = **117440** 🎉

### Another Way to See It

The goal is to make the program print `0, 3, 5, 4, 3, and 0`.

Let’s simply run the program in reverse !

Which A % 8 gives 0 ? **0**

Which A % 8 gives 3 ? **3**

Which A % 8 gives 5 ? **5**

Which A % 8 gives 4 ? **4**

Which A % 8 gives 3 ? **3**

Which A % 8 gives 0 ? **0**

Building `A` backward gives `034530` to which we add a trailing 0 for the first `/ 8` and we get `0345300` base 8, which is 117440 base 10 🎉.

 

```txt
Register A: 117440
Register B: 0
Register C: 0

Program: 0,3,5,4,3,0
```

Running this program with part 1 code gives `0,3,5,4,3,0` 🎉.

This one was easy, now let’s do the real puzzle input.

## Puzzle Input

My input was the following:

```txt
Register A: 30899381
Register B: 0
Register C: 0

Program: 2,4,1,1,7,5,4,0,0,3,1,6,5,5,3,0
```

As for the example input, let’s break it down into instructions:

- `2,4` → `B = A % 8`
- `1,1` → `B = B ^ 1`
- `7,5` → `C = A / pow(2,B)`
- `4,0` → `B = B ^ C`
- `0,3` → `A = A / 8`
- `1,6` → `B = B ^ 6`
- `5,5` → prints `B % 8`
- `3,0` → jumps at the beginning if `A != 0`

Or written in code:

```rust
while a > 0 {
    b = (a % 8) ^ 1;
    c = a.div(2_usize.pow(b as u32));
    b = b ^ c;
    a = a / 8
    b = b ^ 6;
    println!("{}", b % 8);
}
```

We still see the octal structure:
- `% 8` → working with 3-bit chunks
- `/ 8` → right shift

But now there’s a twist:
- We compute `C = A >> B` (`A / 2.pow(B)` is a right shift of B bits)
- We then XOR `B` with `C`

This means that the output (`B % 8`) doesn’t depend *only* on the current 3-bit chunk of `A`.

It also depends on **other bits further left**, based on the value of `B`.

### Reconstructing `A`

In the example input, we could reconstruct `A` digit by digit **from right to left** (starting with the first printed value).

- Output = `v1,v2,v3,v4,v5,v6`
- `A` = `aaa_bbb_ccc_ddd_eee_fff_ggg` (`ggg` being the added 0 for to handle the first division)

We started by trying to find which `fff` gives `v1`, then which `eee` gives `v2`, etc ..

But for the puzzle input program, we can’t do this because of `^ C`.

- Output = `v1,v2,v3,…,v15,v16`
- `A` = `aaa_bbb_..._nnn_ooo_ppp`

To find `ppp` we need to know `ooo` , `nnn` , etc .. because of `C`. 

So the only way is to start from `aaa`, because we know what’s left to `aaa`: nothing ! (0).

Once `aaa` is known, we can find `bbb`. Because to find `bbb` we need to know `aaa` and we will know it. And then we can do the rest. 

 

Let’s find `aaa`:

```rust
fn run_instructions(a: usize) -> usize {
    let b = (a % 8) ^ 1;
    let c = a.div(2_usize.pow(b as u32));
    ((b ^ c) ^ 6) % 8
}

for aaa in 0..8 {
    let output = run_instructions(aaa);
    if output == 0 {
        println!("{} -> {}", aaa, output);
    }
}

// 7 -> 0
```

There is only one possibility for `aaa` which is `7`.

Now let’s find `bbb`, knowing `aaa`.

**Be careful, we are in octal ! Let’s shift `aaa` 3 bits to the left.**

`7 << 3` gives 111000 (`aaabbb` with `bbb` = 0)

Let’s try all possibilities for `bbb` (from 000 to 111).

```rust
for bbb in 0..8 {
    let a = 7 << 3 | bbb; // one possibility for aaabbb
    let output = run_instructions(a);
    if output == 3 {
        println!("{} -> {}", a, output);
    }
}

 // 56 -> 3
```

To get `3,0` printed, `A` should be `56`. Meaning `bbb` is 0, as `56` in decimal is `111000` in binary (`70` in octal).

We repeat this until we reconstruct all digits of `A`.

At some steps, there will be several possibilities for A. This can be handled by doing a BFS.

At the end, we pick the smallest `A` and we get our 2nd star for Day 17 🎉.