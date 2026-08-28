# How balanced is *Watergate*?

*Watergate* is an asymmetric two-player game: Nixon is trying to survive, while the Editor is trying to build an evidence network around him.

I wanted to know whether one side has a real advantage under strong play. So I built a simulator, trained a self-play agent, checked that it actually got stronger, and then let the same model play both sides.

## Current result

**Nixon won 60.7% of 1,000 role-balanced games.**

| Role | Wins | Win rate |
|---|---:|---:|
| **Nixon** | **607** | **60.7%** |
| Editor | 393 | 39.3% |

Approximate 95% interval for Nixon: **57.7%–63.7%**.

This is a clear Nixon advantage at the current level of play. It is **not** a claim that perfect play is 61/39. The main question now is whether the estimate stays in roughly the same place when the agent becomes materially stronger.

> **Status:** preliminary result from checkpoint 54, using 25 information-set MCTS simulations per decision.

---

## Why not just simulate random games?

I tried that first.

Over 10,000 random-policy games, Nixon won **97.39%**.

That is not a useful balance estimate. The Editor's win condition requires building coherent evidence connections, and random play is particularly bad at doing that.

It was still a useful failure. It made the real problem obvious: before estimating balance, I needed players that were at least competent.

---

## What I built

The project has four main parts:

1. **A rules engine** for the 2019 base game, including the evidence graph, cards, reactions, initiative, momentum, round flow and victory conditions.
2. **Information-set MCTS**, so search only uses information available to the player whose turn it is.
3. **A shared policy/value neural network** trained from self-play.
4. **Separate checkpoint evaluation**, with roles swapped, so lower training loss is not confused with stronger play.

The network itself is small by current standards: about **1.43 million parameters**. Search is doing a lot of the work.

For the technical details, see [Methodology](methodology.md).

---

## Did the agent actually get stronger?

The early checkpoint ladder was fairly clean:

| Candidate | Opponent | Result |
|---:|---:|---:|
| 5 | 0 | 51–49 |
| 10 | 5 | 67–33 |
| 15 | 10 | 66–34 |
| 21 | 15 | 63–37 |
| 21 | 0 | 75–25 |

Later progress became noisier:

| Candidate | Opponent | Result |
|---:|---:|---:|
| 54 | 25 | **62–38** |
| 54 | 40 | **59–41** |
| 50 | 25 | **64–36** |
| 40 | 25 | 53–47 |

The later ladder is not monotonic, and checkpoint 54 is not assumed to be close to optimal. The useful conclusion is narrower: the agent has clearly learned something substantial.

Full ladder data: [data/checkpoint_ladder.csv](data/checkpoint_ladder.csv)

---

## The 1,000-game balance run

Checkpoint 54 played itself for 1,000 games:

- 500 games in each candidate-role assignment;
- 25 search simulations per decision;
- deterministic evaluation policy;
- no training exploration noise;
- 0 failures or deadlocks.

The evaluator reported:

- candidate wins as Editor: 184/500;
- candidate wins as Nixon: 291/500;
- evidence-connection victories: 393;
- momentum victories: 495;
- momentum-supply victories: 112.

Because both players were the same checkpoint, the candidate/opponent split is irrelevant. Reconstructing by actual role gives:

- **Nixon: 607 wins**
- **Editor: 393 wins**

There is a useful internal check here: the 393 evidence-connection wins are exactly the 393 Editor wins, while the two Nixon victory types sum to 607.

Balance data: [data/balance_checkpoint54.csv](data/balance_checkpoint54.csv)

---

## What does 60.7% mean?

At this point, sampling error is not the main problem. With 1,000 games, a result this far from 50/50 is not plausibly just noise.

The harder uncertainty is **player strength**.

A self-play model can have systematic blind spots, and an asymmetric game can change character as play improves. So another 9,000 games with checkpoint 54 would mostly make the same provisional answer more precise.

The more useful next experiment is to train a clearly stronger model and ask the same question again.

If stronger checkpoints keep landing near 60/40, the result becomes much more persuasive. If the estimate moves steadily toward 50/50, then checkpoint 54 was still strategically immature.

---

## What happens next?

The plan is:

1. train to a materially later checkpoint, roughly around 300;
2. run a sparse checkpoint ladder to verify that the newer model is actually stronger;
3. repeat the 1,000-game role-balanced self-play estimate;
4. check whether a somewhat larger search budget changes the result;
5. if the answer is stable, consider a 10,000-game final run for tighter sampling precision.

There is also a local browser interface for playing against a trained checkpoint. That is useful as a qualitative check: a human player can often spot nonsense that a checkpoint ladder will not.

---

## Current answer

**Under checkpoint-54 strong-AI play, *Watergate* looks substantially Nixon-favoured: about 61/39.**

I will update this repository after the stronger-agent run.

---

### Notes

This is an unofficial hobby project and is not affiliated with the game's designer or publisher. No commercial game artwork is included here.
