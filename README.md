# Claude Certifications — Study Guide

Study material for Anthropic's official Claude Certification Program, grounded in the verbatim official exam guides (saved locally in `source_pdfs/`, fetched 2026-09-28 from Anthropic's own S3-hosted instructor content). All four guides were read in full and cross-checked for a consistent source (same instructor ID across all four PDFs).

## The four certifications

| # | Certification | Code | Fee | Format | Length | Passing score |
|---|---|---|---|---|---|---|
| 01 | [Claude Certified Associate — Foundations](01_Claude_Certified_Associate_Foundations/) | CCAO-F | $99 | 60 scored items, MC/MR | 120 min | 720/1000 scaled |
| 02 | [Claude Certified Developer — Foundations](02_Claude_Certified_Developer_Foundations/) | CCDV-F | $125 | 53 scored items, MC/MR | 120 min | 720/1000 scaled |
| 03 | [Claude Certified Architect — Foundations](03_Claude_Certified_Architect_Foundations/) | CCAR-F | $125 | 60 scored items, scenario-anchored | 120 min | 720/1000 scaled |
| 04 | [Claude Certified Architect — Professional](04_Claude_Certified_Architect_Professional/) | CCAR-P | $175 | 63 scored items, scenario-anchored | 120 min | 720/1000 scaled |

All four: delivered via Pearson VUE (online proctored or test center), 12-month validity with free on-time renewal, up to 4 attempts per 12 months (14/30/90-day waiting periods between attempts).

## Suggested order

1. **Associate Foundations (01)** — Claude's capabilities, prompting basics, safety/responsible-AI framing. No coding assumed. Do this first regardless of your role.
2. **Developer Foundations (02)** — building with the Claude API: messages, tool use, prompt engineering for applications, evaluation. Do this if you write code that calls Claude.
3. **Architect Foundations (03)** — designing multi-component Claude-based systems: the exam itself is built around 6 realistic scenarios (4 drawn per sitting), each spanning several domains. Do this after 02, since it assumes API/tool-use fluency.
4. **Architect Professional (04)** — the advanced tier: production-scale architecture decisions (cost, latency, multi-agent orchestration, governance, evaluation at scale). Do this last; it assumes everything in 03.

Each numbered folder follows the same five-file pattern used elsewhere in `AI-ML-Tutorials`:

- `tutorial.md` — domain-by-domain concept teaching, grounded in the guide's own task statements/knowledge/skill bullets
- `official_samples.md` — the exam guide's own published sample questions, reproduced verbatim with their official rationale (these are public study material Anthropic distributes for exactly this purpose, not live exam content)
- `practice_questions.md` — additional original practice questions in the same MC/MR style, covering every domain, with answer key + rationale
- `scenario_qa.md` — extended situation → reasoning Q&A (for 03/04 this maps directly onto the exam's own named scenarios)
- `references.md` — the official exam guide, Anthropic docs, and other primary sources, all link-checked

## A note on exam-content confidentiality

The exam guides' NDA clause protects the *live, proctored item bank* (the actual questions you'll see in the real exam) — candidates agree not to reproduce or share those. The guides themselves, including their published sample questions and task-statement breakdowns, are Anthropic's own public pre-exam study material, meant to be read and studied before registering. Everything in `official_samples.md` is reproduced from that public document. Everything in `practice_questions.md` is original material written for this study guide, not reproductions of real exam items — nobody involved in building this has seen the live item bank.

## Source

- Official exam guide PDFs, fetched 2026-09-28 from `everpath-course-content.s3-accelerate.amazonaws.com` (Anthropic's certification platform), saved locally in `source_pdfs/` for reference.
- [anthropic.com/learn/certification](https://www.anthropic.com/learn/certification) — program overview, registration, Pearson VUE scheduling.
