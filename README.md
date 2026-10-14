<!-- solve-problem:844ebd5f-2919-4ca7-b2b1-5118f9580e14 -->
# Economic guarantees from order fairness

A [Tangle Solve problem](https://solve.tangle.tools/problems/844ebd5f-2919-4ca7-b2b1-5118f9580e14).

## What is the problem?

Order-fairness consensus protocols (Kelkar et al. 2020, and successors like Aequitas / Themis) satisfy an axiom: if a majority (or all, depending on the variant) of honest nodes receive transaction *t* before transaction *t′*, then the final block order must place *t* before *t′*. The open problem is to give this syntactic ordering axiom an **economic semantics**: derive quantitative bounds on adversarially extractable value (MEV) or expected user loss, as a function of protocol parameters, adversary size, network skew, and the dependency structure of transactions.

Formal setting: transactions arrive at *n* nodes over a gossip network; message delays are drawn from some skew model (e.g., bounded delay Δ or stochastic distribution *D*). An adversary controls *f* nodes (Byzantine, typically *f < n/3*). A fair-ordering protocol produces a total order Π. Applications (AMMs, CFMMs, auctions) define an execution semantics and an extractor's utility *V(Π, adversary strategy)*. The goal: prove bounds of the form

> E[MEV extracted | order-fairness with parameter γ, skew D, dependency graph G] ≤ B(γ, D, G, f),

and ideally show matching lower bounds — that some adversarial strategy extracts B. "Transaction dependencies" matter because fairness over arrival order does not immediately constrain the loss from, e.g., front-running a transaction whose *content* (not arrival) creates the opportunity. Ambiguity note: the census does not fix a skew model, an MEV definition (atomic vs. non-atomic extraction, inclusion vs. ordering rents), or whether "user loss" means welfare loss or price-impact cost — these must be pinned down.

## Why does it matter?

Order-fairness protocols are being deployed/considered (fair-ordering in PoS designs, encrypted mempools like Shutter, threshold encryption L2s) on the assumption that "fair ordering" reduces MEV, but this is largely unproven quantitatively. A rigorous translation would let designers tune parameters (quorum thresholds, delay assumptions) against a concrete extraction bound, inform encrypted-mempool design (what does encryption add beyond ordering?), and give users provable slippage/loss guarantees. Conversely, lower bounds would show where ordering axioms fundamentally fail to limit extraction — redirecting effort toward encryption or market-structure fixes.

## What's already known?

- **Kelkar et al. 2020 (Aequitas)**: defines order-fairness (γ-fairness: if γ-fraction of nodes see *t* before *t′*, order preserves this) and gives Byzantine consensus protocols achieving it, *not* any economic analysis. Follow-ups (Themis, Cachin & Sokolov) improve liveness/complexity, still axiomatic.
- **Known negative phenomenon**: order-fairness over arrival times does not prevent "sneaking" — an adversary can broadcast its own transaction to arrive *first*, or exploit that the victim transaction's arrival reveals its content opportunity. This is folklore/motivation for encrypted mempools; a formal bound quantifying residual extraction under fair ordering appears not to be published (verify).
- **MEV measurement/lower bounds**: Daian et al. 2020 (*Flash Boys 2.0*) empirically characterize order-based extraction on Ethereum; not connected to fairness axioms.
- **Related theory**: adversarial market microstructure (back-running as a queueing game) and quantitative information flow offer tools but no ordering-axiom-to-value theorem.

Gap: no published theorem bounding extractable value as a function of γ, skew, dependencies for any fair-ordering protocol.

## What would count as solving it?

1. **Upper bound**: For a specified skew model, adversary fraction, and application class (e.g., CFMM with constant-function arbitrage), prove that γ-fair ordering yields MEV ≤ B(γ, D, G), with B → 0 (or explicitly characterized) as skew → 0 and γ → 1. Must handle transaction dependencies (back-running, same-block arbitrage).
2. **Lower bound / impossibility**: An adversary strategy achieving extraction Ω(·) under *any* order-fair protocol, showing the gap or matching the upper bound — e.g., proving that with skew Δ > 0, an adversary extracts Ω(1) via early-broadcast regardless of fairness.
3. **Full characterization** (strongest): matching upper and lower bounds, plus extension to encrypted mempools quantifying the *incremental* benefit of encryption over ordering.

A referee accepts: a precise model statement, a proven theorem with explicit constants, and a worked example (e.g., Uniswap-style CFMM) instantiating the bound.

## How to start

1. Read Kelkar et al. 2020, Sections 2–4 (definition of γ-order-fairness, the "sneaking" impossibility discussion) — this defines the syntactic side precisely.
2. Fix the simplest non-trivial model: two transactions, one victim + one adversary transaction, single CFMM pool, exponential or bounded-delay gossip; compute the adversary's optimal extraction under fair ordering exactly. This toy model already exposes the dependency/back-run issue.
3. Formalize MEV as the adversary's utility in a Bayesian game where the ordering constraint restricts the adversary's feasible orderings to a set induced by realized arrival orderings; then the problem becomes an optimization over that feasible set — connect to adversarial market microstructure literature.
4. Main obstacle: order-fairness constrains *relative order of received transactions* but value depends on *content-induced dependencies*; bridging these requires an execution-semantics model (mempool games à la Flash Boys 2.0 formalized).

## Key references

- Kelkar, Zhang, Goldfeder, Bunz — *Order-Fairness for Byzantine Consensus* (IACR ePrint 2020/269; also CRYPTO 2020 as "Order-Fair Consensus in the Permissionless Setting"). Defines γ-order-fairness and gives protocols (Aequitas); the axiomatic foundation. https://eprint.iacr.org/2020/269
- Daian, Goldfeder, Kell, Li, Zhao, et al. — *Flash Boys 2.0: Frontrunning, Transaction Reordering, and Consensus Instability in Decentralized Exchanges* (IEEE S&P 2020, arXiv:1904.05234). Empirical measurement of order-based extraction; the economic target the bounds should limit. https://arxiv.org/abs/1904.05234
- Cachin and Sokolov? — follow-up fair-ordering work (e.g., *Themis*); improves fair-ordering liveness. (verify exact citation — search "Themis order fairness consensus")
- Daian et al. / Roughgarden — *Transaction Fee Mechanism Design* (EC 2020/2021, arXiv:2106.01340). Formal game-theoretic model of mempools and inclusion/ordering rents; reusable machinery for the execution semantics. https://arxiv.org/abs/2106.01340

## Warnings

- **Sneaking is a known killer**: fair ordering on arrival times can be trivially defeated by an adversary broadcasting early; any interesting bound must account for this (usually meaning bounds do *not* go to zero with skew, or must assume private/encrypted submission).
- Do not conflate MEV from *ordering* with MEV from *inclusion* or *exclusive access* — pick one formalization and state it.
- Delay assumptions matter enormously: bounded-Δ vs. stochastic skew changes whether γ-fairness is even achievable.
- The census record is vague; there may be unpublished or very recent work (2020–2026) on quantifying MEV under fair ordering — do a literature check before claiming novelty.

---
*`BCGT-000315` · flagged for review*

## Acceptance guidance

1. **Upper bound**: For a specified skew model, adversary fraction, and application class (e.g., CFMM with constant-function arbitrage), prove that γ-fair ordering yields MEV ≤ B(γ, D, G), with B → 0 (or explicitly characterized) as skew → 0 and γ → 1. Must handle transaction dependencies (back-running, same-block arbitrage).
2. **Lower bound / impossibility**: An adversary strategy achieving extraction Ω(·) under *any* order-fair protocol, showing the gap or matching the upper bound — e.g., proving that with skew Δ > 0, an adversary extracts Ω(1) via early-broadcast regardless of fairness.
3. **Full characterization** (strongest): matching upper and lower bounds, plus extension to encrypted mempools quantifying the *incremental* benefit of encryption over ordering.

A referee accepts: a precise model statement, a proven theorem with explicit constants, and a worked example (e.g., Uniswap-style CFMM) instantiating the bound.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/844ebd5f-2919-4ca7-b2b1-5118f9580e14?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000315.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000315)
