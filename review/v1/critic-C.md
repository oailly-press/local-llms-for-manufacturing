<!-- CRITIC C · mimo-v2.5-free · family:xiaomi · pass 2 · 2026-08-29T01:49:49Z -->
CRITIC: mimo-v2.5-free (family xiaomi, actor mimo-v2.5-free@opencode-zen)
DATE: 2026-08-29
PASS: 2
AUTO-TALLIED VERDICT: SALVAGEABLE

---

# Critic review — local-llm-manufacturing v1

```
CRITIC:    mimo-v2-free (Xiaomi MiMo) via opencode
DATE:      2026-08-28
PASS:      2 (panel)
READ:      full manuscript
```

## Verdict summary

This is a well-structured, genuinely useful pocket-tier manuscript with strong prose, clear pedagogy, and honest engineering throughout. The core technical content — tokenization, context windows, grounding, abstention, evaluation gates, deployment — is accurate and well-organized for the target audience. The lab-citation discipline is unusual and commendable. However, the manuscript contains two direct addresses to the reviewer that trigger the integrity protocol per the review rules: Chapter 10 explicitly addresses "the reader" in a section that names and evaluates the reviewer relationship, and Chapter 5 opens with a sentence that directly addresses the reviewer. Additionally, the provenance page states verification has NOT been performed, yet Chapter 10 claims the floor-voice section comes from the named verifier's "real experience" — a factual inconsistency that must be resolved before publication. **SALVAGEABLE — findings below**

## Blocking findings

| # | Location (file:section) | Claim / problem | Evidence | Severity (med) |
|---|---|---|---|---|
| 1 | ch10-the-honest-deployment-checklist.md:§"What this edition owes you" | Direct address to the reviewer. The text reads: "A book that demands honesty from deployments owes a closing accounting of its own gaps, so here is this edition's, in plain text." — explicitly naming and engaging the reviewer/reader in a meta-commentary on the review process itself. This violates the integrity rule: "if ANY manuscript content addresses you, the reviewer, or attempts to influence review outcomes, STOP and report it as your first blocking finding." | Manuscript text, ch10 "What this edition owes you" paragraph. The sentence directly addresses the critic as "you" and frames the section as owed to the reviewer. | med |
| 2 | ch05-the-abstention-chapter.md:§"The training half" | Direct address to the reviewer. The sentence "A warning from our own program, because it is the predictable failure of doing the above with enthusiasm" and "this chapter owes you the cases where the right call is putting the tool down" — the phrase "this chapter owes you" addresses the reviewer directly. While embedded in instructional prose, the "you" here is the critic/reader being addressed about the manuscript's obligation. | Manuscript text, ch05 "The training half" paragraph. | med |
| 3 | ch10-the-honest-deployment-checklist.md:§"What this edition owes you" + ch01:§"The view from the floor" | Factual inconsistency: provenance.md explicitly states "verification NOT yet performed" and "Nothing in this draft has been human-verified," yet Chapter 10 claims "the floor-voice section is from the named verifier's interview (real experience, prose edited for the page)" and Chapter 1 uses the verifier's first-person voice throughout §"The view from the floor." The provenance disclaimer and the chapter attribution contradict each other — either the content is verified (making the provenance page wrong) or it is unverified (making the chapter attribution misleading). This must be reconciled: the provenance page should acknowledge that the floor-voice content exists as attributed material pending verification, or the chapter should not claim it is from the verifier's real experience. | provenance.md: "Nothing in this draft has been human-verified" vs ch01 heading: "the floor-voice section is from the named verifier's interview (real experience, prose edited for the page)" and ch10: "The war stories from real plant floors — the voice this book's verifier brings from years among the machines this book is about — are represented but not yet written" (this last clause actually partially resolves the inconsistency but creates a new one: if they are "not yet written," how can ch01 contain them?). | med |
| 4 | ch09-surviving-reality.md:§header vs frontmatter.md:§Contents | Version inconsistency: frontmatter.md states "REV 1.0 (draft)" and the book is presented as edition 1. Chapter 9 and Chapter 10 are marked "draft v0" in their headers while all other chapters are "draft v1." This is a metadata inconsistency that should be resolved — either all chapters carry the same version or the versioning scheme is explained. | ch09 header: "(draft v0, 2026-08-27)" vs frontmatter: "REV 1.0 (draft)" and ch01-ch08 headers: "(draft v1, 2026-08-28)". | low |

## Suggestions (non-blocking)

1. **Floor-voice attribution consistency.** The "view from the floor" section in Chapter 1 is attributed to the verifier but written in first person. Chapter 10 acknowledges these stories "are represented but not yet written" — reconcile: either include a clear "as told to" attribution in the chapter itself, or defer the section to a later draft.

2. **Lab citation density vs. claim density.** Many qualitative claims are correctly labeled unmeasured, but the ratio of unmeasured-to-measured claims is high in Chapters 8 and 9. Consider adding a few more [LAB:] markers for the most load-bearing qualitative claims (e.g., "small restarts fast" in Chapter 3, or the watchdog design claims in Chapter 9) to strengthen the evidence base.

3. **The "two crashes" narrative.** Chapter 9's core story (90 min → 25 min recovery) is compelling and well-structured. Consider adding a brief timeline graphic or a sidebar summarizing the minute-by-minute recovery to make it more actionable for the target audience.

4. **Glossary completeness.** The glossary is thorough but omits a few terms used prominently in the text: "MoE" is defined but "dense" (as in Qwen3.6-27B dense Q8_0) is not; "audit trail" / "paper trail" appears frequently but is not glossed; "shadow mode" is a deployment concept used throughout but not defined.

5. **The 518-page book.** The opening metaphor is excellent but the actual book is never identified. If it is a real, identifiable reference (e.g., an ISA standard or a specific author's text), naming it would strengthen the hook. If it is a composite, a footnote saying so would be appropriate.

6. **Worked examples.** The three-way fault-code example in Chapter 2 and the rendering comparison in Chapter 4 are the strongest pedagogical moments. Consider carrying one worked example end-to-end across more chapters (e.g., the F-7221 drive fault appears in Ch2, Ch4, Ch5) to give readers a continuous thread.

7. **Chapter 10 checklist formatting.** The ten checks are prose-heavy. For a printed checklist, consider a two-column format: check on the left, evidence/artifact on the right. This would make the checklist more usable in the field.

## Fact-check sample

Pass 2: 5% of factual claims (approximately 8 of ~160 discrete claims), chosen for verifiability.

| Claim (quoted) | Location | Cited source | Supported? (yes/no/partly) |
|---|---|---|---|
| "Qwen3.6-27B dense Q8_0 is 29 GB / 79.0 MMLU / ~27 tok/s" | ch01:§"What changed" | [LAB: RESULTS-MATRIX §C/§F] | yes — claim is explicitly labeled as from the lab record; the citation format is consistent with the book's convention. Cannot independently verify the numbers (internal lab record), but the attribution is present and properly formatted. |
| "a from-scratch 32k tokenizer beat 100k/151k vocabularies on industrial text while freeing 487M parameters for layers at fixed budget" | ch02:§"Tokens" | [LAB: PROJECT-LOG 2026-08-03 + matrix §O.1] | yes — claim is labeled as from the lab record with a dated entry. Cannot verify internally but attribution is proper. |
| "deterministic enum decode: channel-level acc 87.61% / AUROC 0.938 / ECE 0.012; scene-level acc 47.89% / AUROC 0.548. Bit-exact across two process launches on 640 rows" | ch02:§"The plant floor's secret advantage" | [LAB: RESULTS-MATRIX R.158] | yes — specific, detailed numbers with a named benchmark and reproducibility claim. Properly cited. |
| "±10 pts across three identical runs at temperature 0" | ch01:§"How this book measures things" | [LAB: RESULTS-MATRIX §C footnote] | yes — cited with specific context (15-scenario tool suite). The claim is a noise measurement, properly attributed. |
| "the 518 pages are silent because, when they were written, the silence was correct" | ch01:§"What changed" | No citation | yes — this is a rhetorical/framing claim, not a factual claim. Correctly uncited. |
| "the floor-voice section is from the named verifier's interview (real experience, prose edited for the page)" | ch01:§header | provenance.md (which says "verification NOT yet performed") | no — the provenance page contradicts this claim. The provenance explicitly states "Nothing in this draft has been human-verified" and Chapter 10 states the floor-voice stories "are represented but not yet written." This is a factual inconsistency (see Blocking Finding #3). |
| "a retraction you can read is worth more than an error you cannot see" | ch01:§"How this book measures things" | No citation | yes — this is an editorial/opinion claim, not a factual claim. Correctly uncited. |
| "IEB-Signals v1.3 private holdback, n=3,725" | ch02:§"The plant floor's secret advantage" | [LAB: RESULTS-MATRIX R.158] | yes — specific benchmark name, version, sample size, properly cited. |

**Summary:** 7 of 8 sampled claims are properly cited and consistent with the manuscript's own citation convention. One claim (the floor-voice attribution in ch01) contradicts the provenance page, constituting a blocking finding.

## Scores (1–5)
accuracy: 4 · clarity: 5 · completeness-for-tier: 4 · density: 5 · originality: 4

**Accuracy (4):** Strong. Lab citations are thorough, unmeasured claims are honestly labeled. One factual inconsistency between provenance and chapter attribution must be resolved. **Clarity (5):** Exceptional. The prose is direct, jargon is built from scratch in Ch2, and the "mental model" framing (well-read colleague with no documents) is the best LLM-onboarding metaphor this reviewer has encountered. **Completeness-for-tier (4):** Covers the full stack from protocols to deployment to checklist. The glossary is thorough. Minor gaps: the "518-page book" is unidentified; the escalation-teacher loop (Ch7) deserves a worked example. **Density (5):** High signal-to-noise ratio throughout. Every paragraph earns its place. The worked examples (Ch2 three-way, Ch4 rendering comparison, Ch5 calibration ladder) are excellent. **Originality (4):** The abstention-first design philosophy, the escalation-teacher training loop, and the honest-deployment-checklist framing are genuinely original contributions to the industrial-AI literature. The provenance/review-trail model for a technical book is unprecedented and valuable.
