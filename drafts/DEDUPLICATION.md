# Cross-workshop deduplication policy

This file governs how release-driven workshop drafts reuse concepts across dates. It is an instruction for draft generation and Learning Coach planning; it is not permanent teaching material.

## Required inputs before drafting or resuming

1. Read `learning/README.md` and the relevant finalized topic and review-card files. These remain authoritative for persisted knowledge.
2. Read `drafts/CONCEPT-REGISTRY.md` and compare each candidate with earlier dated workshop drafts. The registry is a fast index, not a replacement for checking relevant prior drafts when a match is plausible.
3. Read the applicable dated `workshop-progress/` records. They are the source of truth for `Outstanding`, `Persisted`, `Skipped`, and `Deferred` workshop status. The registry must not duplicate or override those statuses. If a progress file is absent, follow the Learning Coach's established first-session rule for that workshop.
4. When resuming after a gap, reconcile candidates across all relevant outstanding/deferred workshop dates before selecting the next session; do not assume the latest draft is the only backlog.

## Normalize and classify each candidate

Compare the **learning objective and underlying mechanism**, not just product names, release versions, or wording. Record one classification for every candidate:

- **New** — a reusable concept not already represented in finalized material or earlier draft proposals.
- **Extension** — adds a materially distinct mechanism, prerequisite, misconception, decision rule, or application to a canonical concept. Name the concept it extends and state what is genuinely new.
- **Review** — deliberate retrieval practice for an existing concept. Use only when review is intended; do not relabel it as new.
- **Duplicate** — substantially the same learning objective and mechanism as an earlier candidate. Merge the new release signal into the canonical concept's evidence/examples and do not create a second full lesson or review card.

A new release, library, API version, or vendor feature is **not** a new learning concept by itself. Do not classify an item as New merely because its destination topic file is empty. A current release can update the evidence for an existing concept without repeating its full explanation.

## Avoid over-merging

A shared umbrella such as security, validation, observability, or AI evaluation is not enough to make two concepts duplicates. Keep concepts separate when they teach different mechanisms or require different decisions. For example, SQL parameterization, TLS verification, credential rotation, parser resource limits, and prompt-injection-resistant tool permissions are related security topics, but are not interchangeable lessons.

When only part of a broader object overlaps, preserve the broader distinct objective and map the overlapping sub-concept to the existing canonical entry. Do not erase useful nuance or silently rewrite historical drafts.

## Draft and registry requirements

For each proposed learning object, include:

- its canonical concept name or registry ID;
- classification: New, Extension, Review, or Duplicate;
- for Extension, Review, or Duplicate, the canonical concept and prior draft/topic reference;
- the distinct learner outcome that justifies keeping it as a separate object;
- new release facts as evidence/examples, clearly separated from the reusable concept.

Update `drafts/CONCEPT-REGISTRY.md` in the same change as each new workshop draft. The registry indexes candidate concepts and cross-date relationships; it does not decide what the learner has mastered. Keep learning-status changes exclusively in the dated workshop-progress files and permanent finalized content in `learning/` and `review-cards/`.

## Session-planning requirements

Before presenting a workshop agenda, merge Duplicate candidates into their canonical concept and show only distinct New/Extension objects, plus any explicitly intended Review objects. If multiple releases support one concept, retain their source links as examples under that one object. Preserve each workshop's source provenance and progress tracking even when its candidate concept is merged into a cross-workshop queue.

## Initial registry seed

The initial registry records a conservative audit of the seven dated drafts from 2026-09-11 through 2026-11-09. It identifies four high-confidence repeated placements and several broader recurring families. These are routing aids for future comparison, not automatic permission to discard historical text or to claim that a concept was taught.
