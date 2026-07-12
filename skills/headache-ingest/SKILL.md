---
name: headache-ingest
description: Use when someone wants to add sources to the headache wiki — fetches ONLY whitelisted peer-reviewed papers, guidelines, and reference texts, drafts entries into pending/, and moves them to wiki/ only after the maintainer explicitly approves. Logs rejections and keeps the index, citations, and answer-log current.
---

# Headache Ingest

Fetch candidate sources for the headache wiki from the approved whitelist only, draft them into `pending/`, and **never** promote to `wiki/` without the maintainer's explicit approval.

**Where the wiki lives:** by default, `headache-wiki/` in the current project.

## Source gate (apply before drafting anything)

**ACCEPT** (draft into `pending/`):
- **Tier 1 — clinical guidelines & classification:** ICHD-3, American Headache Society (AHS), American Academy of Neurology (AAN), European Headache Federation (EHF), European Academy of Neurology (EAN), NICE.
- **Tier 2 — peer-reviewed literature:** Cochrane systematic reviews; PubMed/MEDLINE peer-reviewed articles; trusted field journals (Cephalalgia; Headache; The Journal of Headache and Pain; Neurology; Lancet Neurology; JAMA Neurology; Brain; Nature Reviews Disease Primers / Neurology).
- **Tier 3 — reference texts & registry:** neurology textbooks; UpToDate (manual entry); ClinicalTrials.gov for trial *existence/status* only, never as evidence of conclusions.

**REJECT** (never draft; if pasted in, quarantine and label):
- **bioRxiv / medRxiv preprints** → label "PREPRINT — not evidence".
- General web, blogs, news, Wikipedia, forums, social media, supplement/clinic marketing sites.
- Anything generated from the model's training instead of a fetched, cited source.
- **Topically-adjacent-but-wrong-type/topic** sources — impressive-looking but not clinical headache evidence, or about a different disorder than the topic at hand.

## Procedure

1. Take a topic/question. Query **whitelisted sources only** (e.g., PubMed), preferring guidelines and systematic reviews.
2. Apply the source gate. Drop anything that fails it, say what you dropped and why, and append it to `pending/REJECTED.md` (date · source · reason · category).
3. For each accepted source, draft a file into `pending/` using `_TEMPLATE.md`, filling all frontmatter EXCEPT `approved_by` (leave blank until approval), and every body section from the source itself. Verify each source's DOI/citation.
4. Present a summary table of the pending candidates (title, year, tier, source_type, one-line finding). **STOP. Do not touch `wiki/`.**
5. On the maintainer's explicit approval of specific candidates: set `approved_by:` and `added: <today>`, move those files from `pending/` to `wiki/`, append their rows to `INDEX.md`, and append their **formatted citations to `CITATIONS.md`** under the relevant topic heading (create the heading if the topic is new). Leave un-approved candidates in `pending/`.
6. **Re-validate the answer log.** After the approved files are in `wiki/`, re-run every question in `ANSWER-LOG.md` against the updated wiki. If any answer changes — different sources selected, or recency-wins-within-tier now favors a newly-added source — flag it prominently with the old answer, the new answer, and the source that caused the change, and update that entry's "last re-checked" line. If unchanged, just update the date.

## Rules

- **One source in = one file the maintainer approved.** No source, no claim.
- **Whitelist at the door.** Preprints and off-whitelist sources never enter, even if pasted in — quarantine and label them.
- **Recency-wins-within-tier**, and state when it applied.
- Keep `INDEX.md` and `CITATIONS.md` in sync with `wiki/` on every approval.
