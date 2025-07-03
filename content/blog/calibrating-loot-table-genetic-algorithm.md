+++
title = "Calibrating a Loot Table Using a Genetic Algorithm"
description = "Finding the right distribution of rewards and probabilities for a lotery game, using a genetic algorithm."
date = 2025-07-03
+++

# Introduction to the Project

👉 [Github Repository](https://github.com/antoineprdhmm/loot-table-tuning)

I was building a lottery game on the blockchain when I ran into this problem. I don’t need to go into much detail about the project itself — let’s keep it simple.

Say you buy a pack for $10. This pack contains a value between $3 and $50.

That means when you open it, you might lose a few dollars or win up to 5 times your initial investment.

There are 4 tiers that determine the value range of each pack. The tier is randomly selected when you buy the pack. Then, when you open it, a value is randomly generated within that tier’s range.

Example of tiers:

- **Common** – $3 to $8 – 30% probability
- **Uncommon** – $8 to $12 – 45% probability
- **Rare** – $12 to $20 – 15% probability
- **Epic** – $20 to $50 – 10% probability

```python
(8 - 3) * 30% + (12 - 8) * 45% + (20 - 12) * 15% + (50 - 20) * 10%
```

This example looks nice, but the values have been set arbitrarily — which is a bad idea when money and probabilities are involved. It's worth doing some actual calculations.

To make the project viable, I need to make sure:

- I make a profit **over time** (this isn’t a charity),
- I keep users engaged and excited to buy more packs.

This means I need to carefully define the **ranges** and **probabilities** for each tier.

# Expected Value

The **expected value** (EV) is the average value a pack returns over time.

A pack costs $10, and I need to make a profit. So the EV should be **less than $10**.

According to ChatGPT, an EV between 80–90% of the pack price offers a good balance — users feel like they get value, and I still make a profit.

I chose 85%, meaning the EV should be **$8.50** — so I make an average profit of $1.50 per pack.

The EV is calculated by multiplying the **average value of each tier** by its **probability**.

```python
EV = (Avg T1) * P1 + (Avg T2) * P2 + (Avg T3) * P3 + (Avg T4) * P4
```

With tier ranges defined as `(min, max)`, the average value per tier is `(min + max) / 2`.

So, to get an EV of $8.50, I need to find values for:

- T1_min, T1_max, P1
- T2_min, T2_max, P2
- T3_min, T3_max, P3
- T4_min, T4_max, P4

Let’s go back to our example:

```python
EV = (8 - 3) * 30% + (12 - 8) * 45% + (20 - 12) * 15% + (50 - 20) * 10%
EV = 6.35
```

This means I’d be making **$3.65 profit per pack on average**.

That’s great for me — but not for the user. And if it’s not good for the user, it’s not good for me in the long run.

So yeah, it was worth doing the math.

**The Big Question:** 

**EV = 8.5** has *many* possible solutions. Which one should I pick?

# Finding a Good Fit

Some loot table combinations are boring. Others are exciting.

Some make you lose often but occasionally win big. Others feel more balanced and linear.

That’s why it’s important to define what a **good fit** actually means.

## Probabilities

I decided to keep the probabilities for each tier between **4% and 65%**.

Obviously, all tier probabilities must sum to 100%.

This range is arbitrary, but it feels right — it allows for rare tiers without making them frustratingly impossible to hit. It also avoids a single dominant tier.

Also, **T4 (epic)** must have the **lowest probability**, since it contains the highest rewards.

## Tier Price Ranges

Each tier’s price range must follow this logic:

- T4’s range must be above T3’s
- T3’s above T2’s
- T2’s above T1’s

Additionally, the **ranges should be wide enough** to keep things surprising. A range like $8–$8.5 isn’t exciting.

## Expected Value (EV)

The **target EV is $8.50**, but it doesn’t need to be exact. A small margin above or below is acceptable.

## Running Simulations

The EV is a theoretical value — it’s only reached if you open an **infinite number** of packs.

But in small runs, the **actual return** can vary a lot from the EV.

That could be a problem when launching the app

- I don’t want to run into a **big deficit** in the app’s treasury.
- I don’t want users to feel like they’ve been **ripped off**.

To reduce that risk, I made small simulations and compared the **actual average outcome** with the theoretical EV.

# Fitness Function

Here’s one possible Python representation of a loot table:

```python
class LootTable:
    def __init__(self, probabilities: List[float], prices: List[float]):
        self.probabilities = probabilities
        self.prices = prices
```

Using the earlier example:

```python
probabilities = [0.3, 0.45, 0.15, 0.1]
prices = [3, 8, 12, 20, 50]
```

From the previously discussed fitness criteria, we can now build a **fitness function** to evaluate a loot table.

The function applies **penalties** based on how far the loot table deviates from our ideal configuration. The lower the score, the more "fit" the loot table is.

```python
def compute_fitness(lt: LootTable, config: Config) -> float:
    score = 0

	# Deviation from theoretical EV
    ev = get_expected_value_absolute_deviation(lt, config)

	# Run multiple short simulations
    sim_evs = [
        get_simulation_results_absolute_deviation(
            run_simulations(lt, config.genetic_algorithm.simulation_runs), config
        )
        for _ in range(20)
    ]

    # Compare simulation mean to EV
    sim_evs_mean = sum(sim_evs) / len(sim_evs)
    sim_dev = abs(abs(ev) - abs(sim_evs_mean))
    score += abs(sim_dev) * 1000

    # Add penalty for high standard deviation across simulations
    sim_std = statistics.stdev(sim_evs)
    score += sim_std * 1000

    # Penalize EV deviation itself
    score += abs(ev) * 1000

    # Structural penalties
    if not get_are_prices_sorted(lt):
        score += 1000
    if not get_are_probabilities_within_range(lt, config):
        score += 1000
    if not get_is_highest_tier_lowest_probability(lt):
        score += 1000
    if not get_are_ranges_wide_enough(lt):
        score += 1000

    return score 
```

So now we can generate loot tables, compute their fitness score, and compare them.

But generating them randomly isn't very efficient — we can do better!

# Using a Genetic Algorithm

Genetic algorithms are inspired by natural selection. The idea is that better-performing candidates are more likely to survive and propagate, leading to gradual improvement over generations.

If you’d like to dive deeper: 👉 [Wikipedia Genetic Algorithm](https://en.wikipedia.org/wiki/Genetic_algorithm)

This approach works well in our case, where we don’t just want *any* loot table that works — we want a **good** one.

Instead of endlessly generating and testing random tables:

- We **generate an initial population** of random loot tables.
- Each loot table is evaluated using the fitness function.
- The best-performing tables (lowest score) are **more likely to survive**.
- The next generation is created via **selection, crossover, and mutation**.
- Over time, the population evolves toward better solutions.

Here’s my configuration:

```yaml
genetic_algorithm:
  population_size: 1000
  mutation_rate: 0.2
  crossover_rate: 0.8
  elite_size: 25
  generations: 50
  simulation_runs: 10
```

- **Elite size**: Number of top candidates directly selected for the next generation.
- **Simulation runs**: Number of draws per short simulation in the fitness function. I keep this small on purpose to stress-test volatility. If you use 100+ draws, you just converge closer to the EV — which hides short-term deviation risks.

The other parameters were chosen through testing. I tried larger population sizes, but it didn't improve results — it just made things slower. Increasing generations improved the outcome, but beyond 50 generations, improvements plateaued.

# Result

The algorithm runs in a few seconds and returns results like this:

```txt
FITNESS DEBUG:
- EV deviation: 0.2332935998983281
- Prices sorted: True
- Probabilities within range: True
- Highest tier lowest probability: True
- Contains duplicate prices: False
- Ranges wide enough: True
SIMULATIONS:
- Simulation results deviation: 1.0653241098043411
- Simulation results deviation: -0.7363306578779483
- Simulation results deviation: -1.5912145090228975
- Simulation results deviation: 0.04945076702478168
- Simulation results deviation: -1.1712967377203753
- Simulation results deviation: -0.0798430959134162
- Simulation results deviation: -3.6611694414769396
- Simulation results deviation: 1.146239115294975
- Simulation results deviation: -2.5398544847667957
- Simulation results deviation: -0.6366327885268142

Probabilities: [0.27970313941528896, 0.6282451231342908, 0.0520276918043768, 0.0400240456460434]
Prices: [3, 5, 15, 17, 50]
```

The fitness criteria are satisfied.

Running 10 simulations with this config gives manageable worst-case deviation — `~2.29`, which is acceptable. Since the results are probabilistic, outliers may still happen — but what matters is that most short simulations remain within a healthy range.

Final tier structure:

- **T1**: $3–$5 → 27.9%
- **T2**: $5–$15 → 62.8%
- **T3**: $15–$17 → 5.2%
- **T4**: $17–$50 → 4.0%

That’s a solid configuration:

- T1 is a loss
- With T2, you can lose or win
- T3 is a win
- T4 is rare but wide and very rewarding — lots of excitement when it hits
