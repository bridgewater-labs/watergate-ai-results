# Methodology

This is the technical appendix to the main [README](README.md).

## Research question

The quantity of interest is the win-rate advantage of Nixon versus the Editor **under strong AI play**.

This is not a solved-game or perfect-play result. A self-play model estimates the role advantage induced by its own policy class and search procedure. The main robustness question is whether the estimate is stable as those players get stronger.

## Simulator

The implementation covers the 2019 base game, including:

- the evidence connection graph;
- role-specific card decks;
- card actions and reactions;
- initiative and momentum;
- evidence movement, revealing and pinning;
- round transitions;
- asymmetric victory conditions.

During development, simulator diagnostics exposed several rule and timing errors. Random-policy games were useful for this purpose, but not for estimating balance.

## Hidden information and search

The search agent uses root-player Information-Set Monte Carlo Tree Search.

The authoritative game engine contains the full state, but player-facing search operates through scoped information states. Hidden cards and hidden evidence are sampled consistently for search rather than exposed directly.

Search uses a policy/value neural evaluator in place of random rollouts.

## Neural network

One shared model is used for both roles.

Current architecture:

- 1,168 base state features;
- 1,402-action vocabulary;
- 2,570 total model input;
- 256 hidden units;
- three residual MLP blocks;
- policy head over legal/search actions;
- scalar value head;
- approximately 1.43 million parameters.

Training data are generated from self-play. Each decision contributes an information state, a search-derived policy target, and the eventual game outcome from the current player's perspective.

## Training

Production iterations currently use:

- 96 self-play games;
- 25 search simulations per decision;
- process-based parallel workers;
- a 100,000-example replay window;
- AdamW;
- learning rate 0.001;
- weight decay 0.0001;
- batch size 256;
- three epochs.

Evaluation is kept separate from production training.

## Strength evaluation

Later checkpoints are tested against earlier frozen checkpoints with both roles swapped.

This matters because a falling training loss does not establish stronger game play. The strongest cumulative evidence so far is that checkpoint 54 beat checkpoint 25 by 62–38 and checkpoint 40 by 59–41 over 100-game matchups.

The checkpoint ladder is noisy and not perfectly monotonic, so "latest" is not assumed to mean "best".

## Balance evaluation

For the current estimate, checkpoint 54 played checkpoint 54 for 1,000 games:

- 500 candidate-as-Editor games;
- 500 candidate-as-Nixon games;
- 25 search simulations per decision;
- deterministic evaluation policy;
- no training exploration noise.

Reconstructing the games by actual role produced:

- Nixon 607 wins;
- Editor 393 wins.

The approximate 95% binomial interval for Nixon is 57.7%–63.7%.

## Main limitation

The estimate is precise enough statistically to reject a 50/50 interpretation **for checkpoint-54 play**. That is not the same thing as establishing the game's perfect-play value.

The next test is to train a materially stronger model and repeat the same balance experiment. Stability across strong checkpoints matters more than squeezing the present sampling interval from roughly ±3 percentage points to ±1.

## Practical stopping rule

For this project, a "good enough" result requires:

1. clear evidence that training improved playing strength;
2. diminishing strength gains at later checkpoints;
3. no obvious incompetence in human qualitative play;
4. a stable Nixon/Editor estimate across several strong checkpoints;
5. no large balance shift from a modest increase in search budget.

At that point the result should be described as **balance under strong AI play**, not as a proof of optimal play.
