# Headache Helper

**A Claude Code plugin for answering headache & migraine questions that refuses to make things up.**

Headache Helper answers your questions *only* from a curated local wiki of peer-reviewed papers, clinical guidelines, and neurology reference texts that **you approve** — with inline citations, source-conflict resolution, and a hard no-fabrication rule. If the wiki doesn't cover something, it says so and offers to research it. It never answers from the model's general knowledge.

It ships two skills:

- **`/headache-helper`** — asks the wiki and answers (read-only, safe).
- **`/headache-ingest`** — fetches candidate sources from a trusted whitelist, drafts them, and adds them to the wiki **only after you approve** (the write path, gated).

> ⚠️ **Not medical advice.** This is a research and knowledge-management tool. Its answers are summaries of the sources *you* curated — not clinical guidance. Treatment decisions belong with a qualified clinician. You are responsible for the sources you ingest and how you use the output.

---

## Why this exists

Large language models are useful but they **hallucinate** — and residual hallucination persists even in clinically-tuned models (a published example: an RLHF-tuned diagnostic model still produced incorrect outputs ~2% of the time). For anything health-related, a plausible-but-wrong answer is worse than no answer.

Headache Helper's premise is simple: **grounding, not model quality, is what makes an answer trustworthy.** So it inverts the usual flow — instead of asking the model what it "knows," it asks a curated corpus of real, cited, maintainer-approved sources and forbids the model from going beyond them.

---

## Key features

- **Grounded-only answers.** Every claim traces to an approved source file, cited inline as `[title, year]`. No source, no claim.
- **Never fabricates or hedges.** If the wiki doesn't cover it, the skill says "the wiki doesn't cover this yet" and offers to ingest it — it won't guess, and it won't soften an ungrounded statement into "might/possibly/some sources suggest."
- **Tiered trust whitelist.** Only guidelines/classification (Tier 1), peer-reviewed literature (Tier 2), and reference texts/registries (Tier 3) can enter. Preprints, blogs, marketing, and social media are hard-blocked.
- **Recency-wins-within-tier conflict resolution.** When sources disagree, the higher evidence tier wins first; among equals, the newer one wins — and the answer says when it applied this.
- **Human approval gate.** Fetched sources land in `pending/` and never enter the answerable wiki without your explicit approval.
- **Re-validation safeguard.** Every answer is logged with the exact sources it used. When you ingest new material, the plugin re-checks past answers and flags any that a newer/higher-tier source has changed — so guidance can't quietly drift.
- **Running bibliography.** A `CITATIONS.md` of every source, DOI-linked and grouped by topic, is kept current automatically.
- **Rejection audit trail.** Every source the gate refuses is logged with a reason, so you have a durable record of what was excluded and why.
- **Standing disclaimer** on every answer.

---

## How it works

```
                 you ask ─────────────► /headache-helper ──► reads headache-wiki/ ──► cited answer
                                                │                                        │
                                                └──── logs Q + sources ──► ANSWER-LOG.md ◄┘

  you request a topic ──► /headache-ingest ──► whitelist gate ──► pending/ ──► YOU APPROVE ──► wiki/
                                                     │                                          │
                                              REJECTED.md ◄─ off-whitelist            INDEX.md + CITATIONS.md
                                                                                        + re-validate ANSWER-LOG
```

The wiki is plain Markdown — **one file per source** with a metadata header (`tier`, `year`, `source_type`, `doi`, `topics`, `approved_by`, …) plus a fixed-format summary body. Because the metadata is structured, conflict resolution is a mechanical lookup rather than a judgment call. See [`knowledge/DESIGN.md`](knowledge/DESIGN.md) for the full data contract.

---

## Installation

Requires [Claude Code](https://claude.com/claude-code).

**Option A — as a marketplace (one command):**

```
/plugin marketplace add GenkiTaco/headache-helper
/plugin install headache-helper
```

**Option B — manual:**

```bash
git clone https://github.com/GenkiTaco/headache-helper.git
```

Then add the cloned path to your Claude Code plugin configuration (or drop `skills/headache-helper` and `skills/headache-ingest` into your `~/.claude/skills/`).

---

## Setup — create your wiki

The plugin ships an empty knowledge-base template in [`knowledge/`](knowledge/). Copy it into the project where you want to use the wiki, named `headache-wiki/`:

```bash
cp -R knowledge/ headache-wiki/
```

That gives you:

```
headache-wiki/
├── DESIGN.md        # the data contract (source of truth)
├── _TEMPLATE.md     # entry template (one source = one file)
├── INDEX.md         # scan table (auto-maintained)
├── CITATIONS.md     # running bibliography (auto-maintained)
├── ANSWER-LOG.md    # answered questions + re-validation
├── wiki/            # approved sources live here (starts with one EXAMPLE to delete)
└── pending/
    └── REJECTED.md  # rejection audit trail
```

Delete `wiki/EXAMPLE-entry.md`, then populate the wiki with `/headache-ingest`.

> The skills default to `headache-wiki/` in your current project. If you keep the wiki elsewhere, tell the skill the path when you invoke it.

---

## Usage

**Add sources to a topic** (nothing enters the wiki without your OK):

```
/headache-ingest preventive treatment of chronic migraine
```

The skill queries whitelisted databases (e.g., PubMed), rejects anything off-whitelist (logging why), drafts candidate entries into `pending/`, and shows you a summary table. You reply with which to approve; approved sources move into `wiki/` and the index, citations, and answer-log update automatically.

**Ask a question:**

```
/headache-helper What does the evidence say about CGRP antibodies for chronic migraine prevention?
```

A grounded answer, every claim cited to a wiki file, with the standing disclaimer — or a plain "the wiki doesn't cover this yet" if it doesn't.

---

## The source whitelist (trust boundary)

| Tier | What's allowed | Examples |
|------|----------------|----------|
| **1** | Clinical guidelines & classification | ICHD-3, AHS, AAN, EHF, EAN, NICE |
| **2** | Peer-reviewed literature | Cochrane, PubMed/MEDLINE; Cephalalgia, Headache, J Headache Pain, Neurology, Lancet Neurology, JAMA Neurology, Brain, Nat Rev Dis Primers/Neurology |
| **3** | Reference texts & registry | Neurology textbooks, UpToDate (manual); ClinicalTrials.gov (status only) |

**Hard-blocked:** bioRxiv/medRxiv preprints, general web/blogs/news/Wikipedia, forums/social media, supplement & clinic marketing, anything generated from the model's training, and topically-adjacent-but-wrong-type sources. Every rejection is logged.

Edit the whitelist in `knowledge/DESIGN.md` (§4) and the ingest skill to match your own trust boundary.

---

## Design principles

1. **No source, no claim.** The model may not exceed the wiki.
2. **Whitelist at the door.** Trust is decided before a source is ever summarized.
3. **Recency-wins-within-tier.** Conflicts resolve mechanically and transparently.
4. **Human approval gate.** You decide what counts as authoritative.
5. **Log and re-validate.** New evidence surfaces any past answer it changes.

Full rationale and the complete data contract: [`knowledge/DESIGN.md`](knowledge/DESIGN.md).

---

## Extending it

- **Other headache types / topics:** just `/headache-ingest` them — the wiki grows one approved source at a time and organizes by `topics`.
- **A different domain entirely:** the pattern (tiered whitelist → approval gate → grounded-only Q&A → re-validation) is domain-agnostic. Swap the whitelist journals/guidelines in `DESIGN.md` and the ingest skill, and you have a grounded assistant for any evidence-based field.

---

## A note on sources & copyright

This repository ships **only the framework** — the skills, the design, and an empty wiki. It does **not** include anyone's curated source corpus. When you ingest sources, you create local summaries with citations for your own reference; respect the copyright and terms of the works you summarize, and cite them properly.

---

## License

MIT — see [LICENSE](LICENSE).

Built with [Claude Code](https://claude.com/claude-code).
