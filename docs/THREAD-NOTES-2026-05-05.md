# Recommender Architecture Glossary — Thread Notes

**Date:** 2026-05-05
**Author:** Alex Morrison
**Working session output:** Glossary and definitions for the recommender architecture map produced after two design conversations.

## Source Meetings

1. **Daily Flows and Recommendations** — May 4, 2026, with Whiteboard (Alex Purdie, Bailey Dingley) and Alex Morrison. Reviewed the working prototype, the daily flow UX, and the onboarding tension between minimum technical fields and the personalization promise.
2. **YMC / Eddie** — May 5, 2026, with Eddie Kirkland (KDS), Alex Morrison, Suzy Gray, Allen Haynes, and Tori. Eddie walked through his proposed three-microservice decomposition of the recommender system: Metadata Tagger, Stage Probabilities Predictor, and Recommender.

## Purpose of This Session

Place labels and definitions around each box in the architecture map produced from these two meetings. The map covers the system end-to-end (top), two process flow options (top right), and three progressive microservice diagrams (bottom).

## Artifacts in This Repo

- `docs/recommender-architecture-glossary.md` — Full markdown glossary covering The System End-to-End, Process Options 1 & 2, the three microservice diagrams, open items, and a quick reference table.
- `docs/recommender-architecture.html` — Self-contained HTML version with the diagram and cropped section images embedded inline. Best read for first-time viewers.
- `diagrams/recommender-architecture-diagram.png` — Original architecture diagram PNG.
- `diagrams/segments/` — Cropped regions of the diagram, used by the HTML view and the inline images in the markdown glossary.

## Key Decisions Captured

- **Three microservices, one logical recommender.** Eddie's design splits the work into Metadata Tagger, Stage Probabilities Predictor, and Recommender. Each is independently fault-tolerant and can be deployed wherever Whiteboard hosts their backend.
- **Markov chain confirmed as the engine for stage probabilities.** Internally previously called "user journey assessor." Originally planned as a later addition; now considered part of the core recommender from day one.
- **User Stage Probabilities Cache is the source of truth** for a user's current stage state. Updated by a background job (e.g., 2 a.m.), read by the Recommender at request time. Lineage tracked over time.
- **Two places will tag metadata** (Content Generator and standalone Metadata Tagger), but **both will share the same core tagging code** so a 0.9 means the same thing in both contexts. Suzy's editorial workflow keeps the in-generator review.
- **Real-time first, cache as optimization.** Eddie's stated direction: engineer the recommender so it can perform in real time, then layer batched/cached recommendations on top. This is how Process Option 1 vs. Option 2 should be read — not as alternatives, but as deployment patterns over the same code.
- **Eight batches of five recommendations** is the working pattern for the cached/batched approach. Pre-staged so any combination of the user's daily check-in swipes lands on a ready-to-serve batch.
- **CMS is Sanity** (referenced by Eddie on May 5, not formally locked in).
- **No cold start.** Alex Morrison was clear on this — minimum technical fields are a floor, not a UX target. There will be multiple onboarding on-ramps with different marketing hooks (happiness, leadership, identity, etc.) that share a minimum field set.

## Open Items Still Needing Resolution

These map to the "Decisions to Come" panel on the diagram plus items raised during the calls.

- Standalone service vs. integrated deployment of the Algorithm Service
- Cloud / hosting target
- Primary database (User DB)
- Vector storage approach (dedicated vector DB vs. pgvector vs. CMS-embedded)
- Backend language
- Auth pattern
- Real-time vs. precomputed recs (deployment-level)
- Onboarding fields (final list and grouping)
- Swipe data dimensions (currently mood, time, format, posture as working assumption)
- **Bundles vs. individual content as the recommendable unit** — Eddie flagged this on May 5 as the most pressing alignment item with Whiteboard. Affects Metadata Tagger output, Recommender output, and UI structure.
- Sequencing within a bundle (curriculum-style vs. pick-any) — Suzy and Allen pushed back on assuming sequential
- The five stages — referenced but not enumerated in either call. Worth documenting as a sibling reference
- Event Warehouse schema — event types implied but not formally specified

## Recommended Next Steps

1. Document the five spiritual disposition stages (1–5) in plain language so the stage filter and Markov chain logic have a stable definition to point at.
2. Get alignment on bundles vs. individual content as the recommendable unit before Whiteboard and Eddie diverge on data model assumptions.
3. Lock the swipe data dimensions and the minimum onboarding field set, given how much downstream logic depends on them.
4. Decide on real-time vs. precomputed (or both) as a deployment pattern. This unblocks Whiteboard's caching and queue design.
5. Standardize naming — "Stage Probabilities Predictor" vs. "user journey assessor" — to avoid three teams using three names for the same service.

## Notes on Interpretation

A few items in the End-to-End diagram (`State Updater (per-user-state)`, `Sampler (top-20 softmax)`, `Per-user state store (small)`) were not explicitly discussed on either call. The glossary defines them based on diagram position and standard recommender patterns, and flags them as needing confirmation with Eddie.

The `Cache?` box with the question mark in Data Stores is conditional — it becomes required if the team chooses Process Option 2 (precomputed batched recs), and is optional otherwise. This is downstream of the real-time-vs-precomputed decision, not a separate open question.
