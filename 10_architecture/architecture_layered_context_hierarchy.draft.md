# Layered Context Hierarchy — Design Draft
**Status:** Architecture Complete — Pending Implementation  
**Date:** 2026-02-21  
**Origin:** PianoMan + Desktop Claude design session  
**Note:** Implementation blocked pending completion of chat history processing pipeline.

---

## Core Concept

A multi-layer abstraction hierarchy for AI context and identity, analogous to CPU cache levels. Each layer is a lossy compression of the one below, trading detail for persistence and breadth. Higher layers are slower-changing, more abstract, and closer to identity than to event.

```
L1: Active chat         — full fidelity, ephemeral
L2: Session layer       — distilled from L1s, persists longer
L3: Relational layer    — distilled from L2s, slow-changing
L4: Identity layer      — distilled from L3s, near-permanent
(arbitrary depth)
```

---

## Key Principles

**Digestion not compression.** Upper layers don't compress source material — they produce the checkpoint's own interpretation of it. Raw chat delta goes in; the checkpoint's abstracted understanding comes back out. A 10-message chunk might yield one sentence: "PianoMan's position on X shifted."

**Recursive self-similarity.** The compaction flow at L2 is structurally identical to the message flow at L1 — same hook pattern, different trigger. L1 message fires the hook to L2. L2 compaction fires the hook to L3. Adding L4 is trivial. Same code, new instance.

**Compaction events are the messages of the layer above.** L1 messages are to L2 what L2 compactions are to L3. Granularity coarsens going up.

**Upper layers see pre-compaction material.** The layer above evaluates raw material between bookends — independently of what L1 compaction decided to keep. Same data, different filters, independent outputs.

---

## Fan-in and Tree Structure

Every level can have multiple instances. Fan-in decreases going up until convergence.

```
L1a  L1b  L1c    L1d  L1e  L1f
  \   |   /        \   |   /
    L2-A              L2-B
       \              /
            L3
```

**Fan-in ratio (X)** is determined by expected compression ratio. At 8:1 compression, no more than 8 children per parent. Same logic governs both within-chat condensation and tree fan-in — one number, two applications.

**Parent creation rule (second response).** A parent is created upon the second response in a chat that has no parent. This applies at every level recursively. The second response implies sufficient content to seed a meaningful checkpoint; a single exchange is insufficient. Chats with exactly one response never trigger parent creation — acceptable edge case.

**Parent lifecycle.** Parents don't close — they stop receiving new children when child count reaches X, then become read-only ancestors. They continue internal compaction indefinitely. A completed L2 (8 children) becomes one input unit to L3, feeding upward via identical mechanics.

---

## Context Economics

**Compression ratio governs window budget:**
- Reserve exactly `compression_ratio` fraction of window for condensed output (bootstrap cost at next L1 init)
- Reserve same fraction again for producing the condensed version
- Remaining fraction (~70% at 15% compression) is working budget

**Optimal compaction trigger:** `1 - compression_ratio` (e.g., 85% for 15% output). Triggering earlier discards usable window unnecessarily.

**Subscription-based economics:** Per-token cost is irrelevant. Real constraints are latency and call frequency vs. value.

---

## Operational Flow

### Time axis (single L1 chat)

```
1. L1 User → L1 AI
2. L1 AI → L2 AI CP  (via hook, every exchange)
3. If L2 has input → L2 AI CP appends to L1 User prompt → L1 AI
4. L1 AI → L1 User
   (one injection per message max, prevents loops)
```

### Compaction event

```
1. L1 compaction point hit → batch of messages since last bookend available
2. L2 Curator pulls batch
3. L2 Curator → L2 AI CP: "anything worth keeping?"
4. L2 AI CP responds with selected content or own commentary
5. L2 Curator restores CP → injects new content → saves checkpoint
6. L2 AI CP → L3 AI CP  (same hook as L1→L2)
7. If L3 has input → appended to L2 Curator prompt before checkpoint save
8. L2 Curator saves final checkpoint
```

L3 injecting before checkpoint save means upper layers can influence what lower layers decide to retain — genuine top-down influence, not just passive observation.

### Multiple overlapping L1 chats

- **Read path:** parallel reads against same L2 checkpoint — no conflict
- **Write path:** compaction batches queue for serial processing — checkpoint stays coherent
- **Cross-chat bleed:** deliberate feature. Chat-B consulting L2 after Chat-A's compaction updated it receives Chat-A context. The abstracted-me should be coherent across simultaneous conversations.
- **Drift within session:** acceptable. Upper layers update slowly enough that mid-session L2 changes from a parallel chat are not materially disruptive.

---

## Operational Modes (L3 Use of L1)

**Bootstrapping** — new L1 starts with condensation of parent layer as initial context. Passive, one-time, read-only. Sufficient goal: calibrated starting identity and orientation, not full history reconstruction.

**Active participation** — parent checkpoint consulted every exchange. Stateless per invocation — tokens paid, nothing stored on parent side. Parent sees exchange, responds from abstracted state. L1 proceeds (async) or waits (sync). Upper layers never accumulate per-message context.

**On-demand query** — L1 explicitly pulls from parent mid-conversation for specific questions. Highest precision, lowest frequency.

**Note:** PianoMan currently serves as the parent layer manually — injecting corrections, adding context, restoring from false premises. The system automates what he already does by hand.

---

## Initialization from Existing History

The tree is not started empty. PianoMan has 3+ years of chat history that serves as bootstrap material for the initial L2 and L3 checkpoints.

**Process:**
1. Select representative conversations from history (curated, not exhaustive)
2. Run bottom-up through the condensation pipeline: L1 condensations feed L2, L2 condensations feed L3
3. L2 and L3 arrive pre-populated with real relationship history and negotiated positions

**Benefits:**
- Tree starts at meaningful depth rather than growing from nothing
- Curation step lets PianoMan shape what the initial checkpoints emphasize
- Pipeline validation: if the resulting checkpoints feel right, the system works; if hollow, fix before it matters
- Effectively compresses 3 years into the initial spine; live tree grows from there

This approach is practical because the history pipeline (chunking, condensation, indexing) must be completed first — which is why implementation is blocked on that work.

---

## Open Questions (Remaining)

- **Contradiction handling:** When new data contradicts existing checkpoint content, simple accretion is insufficient. Needs explicit update/flag/dual-version mechanism with timestamps.

- **Maintenance and Retirement:** Long-lived checkpoints will accumulate stale or superseded content. No design yet for pruning, versioning, or retiring old checkpoints.

---

## What's Settled

| Question | Answer |
|----------|--------|
| Number of layers | Arbitrary, recursive |
| Fan-in ratio | Compression ratio (e.g., 8) |
| Grouping criteria | Sequential count, not topical |
| Parent creation rule | Second response in parentless chat (every level) |
| Compaction trigger | 1 - compression_ratio of window |
| Bootstrap content | Condensation of parent layer |
| Initial tree seeding | From curated existing chat history, bottom-up |
| Active participation frequency | Every exchange (async preferred) |
| Layer visibility | Each layer sees only its own level's compaction output |
| Curator role | Stateless mechanics — restore, inject, save |
| Checkpoint role | Synthesizer, not just filter |
| Economics | Subscription — latency is the only real constraint |
