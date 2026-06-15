# Slide 0: Diversity

Before we get into the results, let's talk about diversity — what it actually means in a genetic algorithm and why it matters. Diversity is just a measure of how different the individuals in your population are from each other. We measure it as the mean pairwise Hamming distance, normalized between zero and one. Zero means everyone's identical, one means they're maximally spread out.

Why should you care? Because diversity is the tension at the heart of every evolutionary search. If you lose it too fast, your population collapses onto a local optimum and you're stuck — that's premature convergence. If you have too much, selection can't do its job and you're basically doing random search. Every practitioner knows this trade-off. But here's the question we asked: what actually controls it? Most people reach for mutation rates, selection pressure, population size. We found something surprising — it's the topology. How your islands are wired together dominates everything else.

# Slide 1: Multi-Domain Topology Ordering

This is the main graph that I want you to take home. Topology determines diversity. Across six completely unrelated domains — from trivial bit-counting to NP-hard constraint satisfaction to co-evolutionary board games — the same ordering holds: isolated islands preserve the most diversity, fully connected islands preserve the least. The bars always step down in the same order: None, Ring, Star, Random, Fully Connected. Kendall's W = 1.0 — perfect concordance. Not a single domain breaks the pattern. The thing you set once and forget — how your islands are wired together — matters more than what you're actually evolving.

# Slide 2: The Domains

1. **OneMax** — Maximize the number of 1s in a binary string. Trivial, unimodal landscape — useful as a baseline.

2. **Maze** — Navigate a grid maze from start to goal. Genomes encode sequences of moves. Deceptive because the shortest Hamming-distance path to the solution isn't necessarily the best route through the maze.

3. **Graph Coloring** — Assign colors to graph vertices so no two adjacent vertices share a color. NP-hard constraint satisfaction — rugged landscape with many plateaus.

4. **Knapsack** — Pick items with given weights and values to maximize total value without exceeding a weight limit. NP-hard, but with a smoother landscape than graph coloring.

5. **No Thanks!** — A card game with strategic decision-making. Co-evolutionary: fitness depends on opponents' strategies, so the landscape shifts as the population evolves.

6. **Checkers** — Two-player board game with evolved evaluation functions. Co-evolutionary, real-valued genomes evaluating board positions. The most complex domain in the set.

The domains span three axes: **difficulty** (trivial → NP-hard), **landscape type** (fixed → co-evolutionary), and **representation** (binary → real-valued).

# Slide 3: Island Models & Topology

So what do we mean by topology? In an island-model GA, instead of one big population, you split it into separate sub-populations — islands — and occasionally migrate individuals between them. The topology is the graph that determines who talks to whom. On the left, no edges — completely isolated islands, no migration at all. Each island evolves independently. Then a ring — each island only shares with its two neighbors. Star — one hub connected to everyone. Random — an Erdős–Rényi graph, roughly half the possible edges. And fully connected — everyone shares with everyone.

The key number under each topology is lambda two — the algebraic connectivity, the second-smallest eigenvalue of the graph Laplacian. It controls how fast information diffuses across the network. Lambda two equals zero means the graph is disconnected — information never spreads. Lambda two equals five means mixing is instantaneous — every island sees every other island's best solutions right away. And that's exactly what kills diversity. The arrow at the bottom tells the whole story: more connected, faster mixing, lower diversity.

# Slide 4: Multi-Domain Topology Ordering (reprise)

Now let's come back to this graph with fresh eyes. You know the six domains — you know they have nothing in common. Different representations, different fitness landscapes, different difficulty. And you know the five topologies — you know what lambda two means and why connectivity controls mixing speed. So look at the bars again. In every single domain, the ordering is the same: None, Ring, Star, Random, Fully Connected. The bars step down left to right without exception. That's what Kendall's W = 1.0 means — perfect agreement across all six domains. And the effect isn't subtle: topology explains 23.9 times more variance in diversity than domain choice. The problem you're solving barely matters. How you wire your islands is what determines whether diversity lives or dies.

# Slide 5: Statistics

Let me walk you through the statistics, because they're unusually clean and I want you to understand exactly what each number means.

Start with Kendall's W. This is a concordance test — it asks: do the six domains agree on the ranking of the five topologies? W ranges from zero to one. Zero means the domains completely disagree — each one ranks the topologies in a different order. One means perfect agreement — every domain produces the exact same ranking. We got W = 1.0. Not 0.98, not 0.95 — exactly 1.0. Every single domain ranks the topologies in the same order: None preserves the most diversity, then Ring, then Star, then Random, then Fully Connected at the bottom. No exceptions. The chi-squared value is 24.0 with a p-value of 0.00008. That p-value means: if topology had no effect on diversity, the probability of seeing this level of agreement by chance is less than one in ten thousand.

To double-check, we computed all fifteen pairwise Spearman rank correlations — that's every pair of domains compared head to head. Every single one came back rho = 1.0. OneMax agrees with Checkers. Maze agrees with Knapsack. No Thanks agrees with Graph Coloring. Fifteen out of fifteen perfect correlations.

Now the two-way ANOVA. This separates the variance in final diversity into two sources: topology and domain. The F-statistic for topology is 47.8 with p less than ten to the minus six — massively significant. The F-statistic for domain is 0.13 with p = 0.945 — essentially zero. Domain explains nothing. To put it in perspective: topology explains 23.9 times more variance than domain. That means if you're trying to predict how diverse your population will be at the end of a run, knowing the topology gives you almost all the information you need. Knowing whether you're solving OneMax or Checkers tells you almost nothing.

# Slide 6: The End

[No speaking notes — transition slide.]

# Slide 7: Kleisli Arrows

So how did we actually discover this? We started with a working GA written in Rust — a maze generator called mazegen-rs. About 75 lines of imperative code in a for loop. It worked fine. But when we translated it into Haskell, something interesting happened — the structure became visible.

In Rust, selection returns an index. An integer. The connection between selection and crossover is implicit — you use that index to look up an individual from an array. In Haskell, selection is a morphism: it takes a population and returns a population. Same type in, same type out. Every operator has the same signature: Pop a arrow M Pop a. Population in, population out, with all the side effects — randomness, configuration, logging — bundled into this monad M.

That monad is the key. In Rust, you thread the random number generator as a mutable reference through every function call. You pass the config separately. You handle logging separately. In Haskell, EvoM unifies all three into one stack: ReaderT for config, WriterT for logging, State for the RNG. Adding a new effect changes the stack once, not every function signature.

And because every operator has the same type, you can compose them with Kleisli composition — that's the fish operator, >=>. Evaluate, then select, then crossover, then mutate. Six lines instead of seventy-five. But here's the real payoff: that pipeline is now a first-class value. It's not code inside a loop — it's a thing you can pass around, lift into an island functor, compose with other strategies. The composition is explicit and type-checked.

That's what made the experiment possible. Once the island model was a functor parameterized by a migration graph, swapping topologies became trivial — change the graph, keep everything else identical. And swapping domains was just as easy — change the genome type and the fitness function, the pipeline shape stays the same. In the Rust version, either change would have meant rewriting the loop. In Haskell, they're both parameters. That's how we ran 900 experiments across six domains and five topologies with the same code. The categorical structure was always there in the Rust code. Haskell just made it visible.

# Slide 8: Why?

So far this is an experimental result. We showed that topology determines diversity — but we haven't explained why. Why should lambda two predict the ordering so perfectly? Why should the result be domain-independent? It is a subject of ongoing research to find a categorical explanation — to prove that the topology ordering follows from the structure of the island functor itself, not just from the data. If you have ideas, we would love to hear them.

This is the core finding. It's not that topology matters — most people would guess that. It's the magnitude: topology doesn't just matter more than domain, it dominates by a factor of twenty-four. The thing most practitioners set once and never think about again is the single most important structural decision in their algorithm.
