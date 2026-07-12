# Headache Helper — Design & Data Contract

**Purpose:** A pair of Claude skills that answer headache/migraine questions **grounded exclusively in a curated local wiki** of peer-reviewed papers, clinical guidelines, and neurology reference texts — never from the model's own training knowledge.

This document is the source of truth for how the wiki is structured and how the two skills behave. If you change the data format, update this file.

---

## 1. Why grounding, not the model

General-purpose LLM answers are unacceptable for clinical questions for one documented reason: **residual hallucination persists even in clinically-tuned models.** (A published example: an RLHF-tuned diagnostic model still produced incorrect outputs ~2% of the time, down from 12% — grounding, not model quality, is what makes an answer trustworthy.) That is *why* the core rule below is non-negotiable.

**Hard rules (bind every part of the system):**

1. **No source, no claim.** Every factual statement in an answer must trace to a source file in the wiki. If the wiki doesn't cover it, the skill says so — it never fills the gap from training knowledge presented as fact.
2. **Whitelist at the door.** Only sources from the approved list (§4) can enter the wiki. Everything else is rejected or quarantined and labeled.
3. **Recency-wins-within-tier.** When sources conflict, the higher evidence tier wins first; among sources of the same tier, the newer publication wins. The skill states out loud when it applied this rule.
4. **Human approval gate.** Nothing becomes answerable-from until the maintainer approves it. Fetched candidates land in `pending/` and move to `wiki/` only on explicit approval.
5. **Not medical advice.** Every answer carries a standing footer routing decisions to a qualified clinician.
6. **No hedged speculation.** The skill must not soften an ungrounded statement into a guess ("might", "possibly", "some sources suggest"). If it isn't in the wiki, it names the gap.

---

## 2. Layout

```
headache-wiki/
├── DESIGN.md          ← this file
├── _TEMPLATE.md       ← the entry template (one source = one file)
├── INDEX.md           ← auto-maintained scan table: title · year · tier · type · topics · file
├── CITATIONS.md       ← running bibliography: formatted, DOI-linked citations grouped by topic
├── ANSWER-LOG.md      ← questions answered + sources relied on; re-checked on each ingest
├── wiki/              ← canonical, answerable sources (one file per source)
└── pending/
    └── REJECTED.md    ← audit trail of sources the ingest gate refused
```

Two skills, because asking and adding are different jobs with different risk. **Asking** (`/headache-helper`) is read-only and safe. **Ingesting** (`/headache-ingest`) reaches the internet and writes files, so it lives entirely behind the approval gate.

---

## 3. Wiki entry format (one file per source)

One source = one file = one thing the maintainer approved. Each file opens with a metadata header the skill compares mechanically, so recency-wins-within-tier is a lookup, not a judgment call. See `_TEMPLATE.md`.

```yaml
---
title:
authors:
year:
tier:            # 1=guideline/classification, 2=peer-reviewed lit, 3=textbook/registry
source_type:     # RCT | systematic-review | narrative-review | guideline | textbook | registry
journal:
doi:
topics: []
added:
approved_by:
---
```

Body sections (fixed order): **Question addressed · Key findings · Dosing / evidence · Limitations · Direct quotes** (≤15 words each, attributed; never reconstruct the source from quotes).

---

## 4. Source whitelist (the trust boundary)

**Tier 1 — clinical guidelines & classification (highest authority):** ICHD-3; American Headache Society (AHS); American Academy of Neurology (AAN); European Headache Federation (EHF); European Academy of Neurology (EAN); NICE.

**Tier 2 — peer-reviewed literature:** Cochrane systematic reviews; PubMed/MEDLINE peer-reviewed articles; trusted field journals — Cephalalgia; Headache; The Journal of Headache and Pain; Neurology; Lancet Neurology; JAMA Neurology; Brain; Nature Reviews Disease Primers / Neurology.

**Tier 3 — reference texts & registry:** neurology textbooks; UpToDate (manual entry); ClinicalTrials.gov for trial *existence/status* only, never as a source of conclusions.

**Rejection audit trail:** every rejection is appended to `pending/REJECTED.md` (date · source · reason · category).

**Blocklist (never enters, even if pasted in):**
- bioRxiv / medRxiv preprints — not peer-reviewed. Hard-excluded; if referenced, quarantined and labeled "PREPRINT — not evidence."
- General web, blogs, news, Wikipedia, forums, social media, supplement/clinic marketing sites.
- Anything generated from the model's training instead of a fetched, cited source.
- **Topically-adjacent-but-wrong-type/topic sources** — impressive-looking but not clinical headache evidence, or about a different disorder than the topic.

---

## 5. `/headache-helper` — the Q&A flow

1. Search `wiki/` (via `INDEX.md` topic tags + content) for entries bearing on the question.
2. Resolve conflicts with recency-wins-within-tier; name the override in the answer.
3. Answer only what the wiki supports, each claim citing its source file. Name any uncovered part and offer `/headache-ingest`.
4. If nothing applies, say so plainly. Never fill gaps from general knowledge; never hedge/speculate about ungrounded content.
5. Flag pregnancy/preconception-relevant drugs (e.g., topiramate, valproate).
6. Log the exchange to `ANSWER-LOG.md`.
7. End with the standing footer: *"Wiki-sourced information, not medical advice. Treatment decisions belong with a qualified clinician."*

---

## 6. `/headache-ingest` — the fetch-and-propose flow

1. Query whitelisted sources only (§4).
2. Reject anything off-whitelist; log rejections to `pending/REJECTED.md`.
3. Draft entry files into `pending/` with full metadata + verified citation.
4. Present a summary; **nothing moves to `wiki/` without explicit approval.**
5. On approval: move files to `wiki/`, stamp `approved_by` / `added`, update `INDEX.md`, and append the formatted citation to `CITATIONS.md` under its topic heading.
6. **Re-validate the answer log:** re-run every logged question against the updated wiki; flag any answer that changed (old vs new + the causing source).

---

## 7. The answer log & re-validation

`ANSWER-LOG.md` records what the maintainer has been told: the question, the answer summary, the exact source files relied on, and a "last re-checked" line. Its purpose is the re-validation loop — when `/headache-ingest` approves new material, step 6 re-runs every logged question and flags any answer that changed (e.g., a newer same-tier or higher-tier source overtook prior guidance). Because answers are grounded and cite specific sources, a change is meaningful.

---

## 8. Success criteria

- Every answer cites its sources or plainly states the wiki doesn't cover the question.
- No source enters `wiki/` without passing the whitelist and the maintainer's approval.
- When two sources disagree, the skill picks correctly by tier-then-recency and says why.
- The standing footer appears on every answer; the skill never hedges/speculates outside the wiki.
- After an ingest that adds a newer source on a logged topic, re-validation flags the affected answer.
