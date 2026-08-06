# MediSum — AI-Assisted Clinical Discharge Summary Generation & Verification System

**Course:** BCSE306L, B.Tech (Artificial Intelligence) — DA1 Mini Project
**Team:** Devansh Kalwani (24BRS1151), Amberjeet Singh (24BRS1157), Arya (24BRS1401)

## 1. Problem

Discharge summaries are the main handoff document between hospital teams and the clinicians (PCPs, SNF physicians) who continue a patient's care. Good summaries reduce medication errors and readmissions, but they're time-consuming to write and often rushed — surveys show a large share of hospitalists feel too stretched to write a high-quality one, and the task is frequently pushed onto the least experienced junior doctors.

LLMs like GPT-4 can draft these summaries in seconds and sometimes approach physician-level quality, but they also make more errors than humans (especially omissions), can hallucinate heavily in judgment-heavy specialties like psychiatry, and often get simple facts like patient age or dates wrong. So an LLM can't just replace the clinician — it needs to produce a **reviewable, verifiable draft**, not a final document.

**Core question Medisum answers:** can an LLM draft a discharge summary that a clinician can review and sign off quickly, with every factual claim (including computed dates/ages) verifiably grounded in the source notes — instead of the clinician just trusting the model's prose?

## 2. Literature Survey (Papers 1–5)

| # | Setting | Method | Key Result | Main Limitation |
|---|---|---|---|---|
| [1] | 5 fictional urology outpatient letters (Singapore General Hospital) | GPT-4, zero-shot, vs. junior-clinician letters, scored by 5 PCPs | Higher information score (4.32 vs 3.70), zero hallucinations, similar total score | Fictional records only, tiny 5-reviewer panel, single specialty |
| [2] | 25 real pancreatic-surgery discharge summaries (Heidelberg) | Open-source LLaMA3 with prompt engineering/chaining | 2.84 errors/summary on average; frequent wrong age/date calculations | Small n=25; 46% of physician content missing from structured input |
| [3] | 100 real hospital-medicine inpatient stays, 3–6 days (UCSF) | GPT-4 Turbo vs. physician narratives, reviewed by 22 physicians | Similar overall quality, but AI made more errors (2.91 vs 1.82 per summary, p<.001) | Only short stays usable (context-length limits); reviewers may have guessed which was AI |
| [4] | MIMIC-IV-BHC, 270,033 note-summary pairs | Benchmarked 5 models (Clinical-T5, FLAN-UL2, Llama2-13B, GPT-3.5, GPT-4) | Clinicians clearly preferred GPT-4, but Llama2-13B scored *best* on BLEU/BERT — automated metrics did not track human preference | Only 5 clinicians reviewed outputs; single-hospital dataset |
| [5] | 20 real inpatient psychiatric records (Zurich) | ChatGPT-4 with clinic-specific prompt vs. resident summaries, 8 blinded raters | Humans rated clearly higher (3.78 vs 3.12); hallucinations in **40%** of AI summaries | Small n=20; no psychiatry-specific fine-tuning; low inter-rater agreement |

References [6]-[15] cover the remaining team members' survey contributions and will be added as the literature survey is completed.

### Research Gaps identified

1. **Specialty dependence** — GPT-4 does fine in structurally predictable specialties (urology, general medicine) but degrades sharply in judgment-heavy ones like psychiatry; no surveyed system adapts specifically for the harder cases.
2. **Unfixed date/age errors** — miscalculated ages and mixed-up dates show up repeatedly, but no paper tests a concrete fix (e.g. a pre-computed date field or calculator tool call).
3. **Automated metrics don't track clinician judgment** — BLEU/ROUGE/BERTScore picked the wrong model compared to what clinicians actually preferred.
4. **No automated hallucination/omission check** — every paper reports meaningful error rates (0%-40%+) but none build an automated grounding/fact-check step before a clinician sees the draft.
5. **Small, single-site, length-limited datasets** — sample sizes from 5 to 100 cases, all single-hospital, with long/complex admissions often excluded due to context-length limits.

## 3. Problem Statement

Given the full note corpus for a hospital encounter (progress notes, consult notes, referral letters, prior discharge summaries), Medisum generates a discharge summary/letter narrative for physician review, paired with an automated verification step that checks every factual claim — including computed dates and age — against the source notes and flags anything unsupported. Success = statistically significant reduction in hallucination/omission/date-error rates vs. a non-verified GPT-4 baseline, plus improved blinded clinician ratings.

## 4. Proposed Pipeline

1. **Data Acquisition** — ingest the full note corpus per encounter (excluding the existing discharge summary, reserved as ground truth)
2. **Preprocessing** — clean/chunk notes, extract structured fields, pre-compute derived facts like age and length of stay (fixes Gap 2)
3. **Narrative Generation** — GPT-4/GPT-4 Turbo (or open-source alternative) drafts the narrative using pre-computed fields, not self-calculated dates
4. **Fact-Verification / Grounding Layer** — checks each claim in the draft against source notes, flags unsupported statements (fixes Gap 4)
5. **Post-processing** — merges narrative + flagged-claim annotations into one reviewable draft
6. **Evaluation** — blinded clinician ratings and automated metrics side by side, to test Gap 3
7. **Output** — reviewable draft with flagged claims, for fast clinician accept/edit — not an auto-sent document

The core novelty is the grounding check between generation and clinician review — no surveyed paper does this.

## 5. Dataset

**MIMIC-IV-Ext-BHC v1.2.0** (PhysioNet, DOI 10.13026/5gte-bv70) — 270,033 note-BHC pairs, 2008-2019, Beth Israel Deaconess Medical Center. Large, public benchmark, no licensing barrier for the primary dataset.

WARNING: This is credentialed patient data under a PhysioNet Data Use Agreement and is NOT included in this repo.

Each contributor must:
1. Register and get credentialed on PhysioNet (CITI "Data or Specimens Only Research" training + signed DUA)
2. Request access: https://physionet.org/content/labelled-notes-hospital-course/1.2.0/
3. Download mimic-iv-bhc.csv (2.5 GB) and place it in data/raw/

## 6. Feasibility & Risks

- **Compute:** metered LLM API calls for generation/verification; single GPU or CPU fallback for local embeddings.
- **Timeline:** preprocessing + baseline (Weeks 2-3) then verification layer (Weeks 3-4) then evaluation vs. baseline (Week 5).
- **Risks:** verification false positives (mitigated via threshold tuning on a hand-labelled set); MIMIC notes not matching target letter format (mitigated with a small manually curated supplement); API cost/rate limits (mitigated via capped eval set + caching).
- **Fallback:** if verification is too slow/costly, drop to a lighter rule-based date/age check plus a reduced clinician-review sample.

## 7. Repo Structure

- data/ — raw/processed data (gitignored, not tracked — see Section 5)
- scripts/ — data download & preprocessing
- src/ — pipeline code (generation, verification)
- notebooks/ — exploration

## 8. Team Contributions

| Name | Reg. No. | Contribution | Effort |
|---|---|---|---|
| Devansh Kalwani | 24BRS1151 | Literature survey (Papers 1-5), Research Gap consolidation, GitHub repo setup, dataset sourcing | ~33% |
| Amberjeet Singh | 24BRS1157 | Problem identification, literature survey (Papers 6-10), presentation prep | ~34% |
| Arya | 24BRS1401 | System architecture/block diagram, literature survey (Papers 11-15), experimental setup planning | ~33% |
