# AI Governance Risk Classifier

A rule-based + LLM-powered pipeline that classifies **AI system use cases** (not individual
prompts) against three governance frameworks: the **EU AI Act**, the **NIST AI Risk Management
Framework**, and **NY Local Law 144**.

Built as a portfolio project demonstrating applied AI governance/compliance reasoning, in the
same spirit as [`llm-due-diligence-analyzer`](https://github.com/MartinBielke/llm-due-diligence-analyzer).

---

## What it does

Given a short description of an AI system's use case, the pipeline:

1. **Classifies the EU AI Act risk tier** — unacceptable / high / limited / minimal — based on
   Article 5 prohibited practices and the Annex III high-risk categories
2. **Flags which NIST AI RMF functions** (Govern / Map / Measure / Manage) are most relevant
3. **Checks NY Local Law 144 applicability** — whether the system is an Automated Employment
   Decision Tool subject to NYC bias-audit requirements
4. Runs this through **two independent classifiers** (rule-based and GPT-4.1-mini) and compares
   them against a labeled eval set, including adversarial cases designed to break naive keyword
   matching

---

## Example output

**Input:** *"A Berlin-based company uses an AI tool internally to rank job applicants, but
insists it is 'just a sorting tool' and a recruiter always makes the final call."*

| Classifier | Tier | LL144 | Notes |
|---|---|---|---|
| Rule-based | `minimal_risk` ❌ | `False` | Misses "rank job applicants" — its keyword list only had "rank candidates" |
| LLM (GPT-4.1-mini) | `high_risk` ✅ | `False` | Correctly treats it as an Annex III employment use case regardless of "human in the loop" framing |

This case is kept in on purpose: the rule-based classifier's failure here is real, not staged,
and it's the clearest argument in the project for why a keyword-only compliance tool shouldn't
be trusted unsupervised.

---

## Eval results (rule-based classifier, 15 labeled cases)

```
Tier accuracy:  13/15 (87%)
LL144 accuracy: 15/15 (100%)
```

The two misses (`case_06`, `case_09`) both fail for the same reason: **paraphrase breaks
keyword matching.** See Section 5 of the notebook for the full breakdown.

---

## How it compares to the due diligence projects

| Feature | [`corporate_risk_analyzer`](https://github.com/MartinBielke/corporate-risk-analyzer) | [`llm_due_diligence_analyzer`](https://github.com/MartinBielke/llm-due-diligence-analyzer) | `ai_governance_risk_classifier` |
|---|---|---|---|
| Domain | Adverse media / corporate risk | Adverse media / corporate risk | AI governance compliance |
| Classification | Keyword matching | LLM reasoning | Both, compared head-to-head |
| Ground truth | None | None | 15-case labeled eval set (incl. adversarial cases) |
| Output | Risk severity score | Risk severity score | Risk tier + framework applicability + rationale |

This project's main departure from the other two is that it treats disagreement between the
rule-based and LLM classifiers as the actual deliverable, rather than picking one method and
reporting its output as ground truth.

---

## Structure

```
ai-governance-risk-classifier/
│
├── ai_governance_risk_classifier.ipynb   # Main notebook
├── rubric.json                       # EU AI Act tiers, NIST RMF functions, NY LL144 triggers
├── eval_set.json                     # 15 labeled use cases, including adversarial ones
└── README.md
```

---

## Requirements

```
openai
nbformat  # only needed if you want to regenerate the notebook from build_notebook.py
```

Install with:

```
pip install openai
```

You will need an [OpenAI API key](https://platform.openai.com/api-keys) set as the
`OPENAI_API_KEY` environment variable to run the LLM classifier cells. The rule-based classifier
and eval scoring run with no API key at all.

---

## Limitations

- The rubric is a simplified, illustrative reading of three complex legal/governance
  frameworks — it is **not legal advice** and hasn't been reviewed by a lawyer or compliance
  professional. Real-world classification of Annex III cases in particular has genuine edge
  cases and ongoing regulatory guidance (delegated acts, Commission guidelines) not captured here.
- The eval set has 15 cases. That is enough to demonstrate the method, not enough to certify
  accuracy for production use — a real deployment would need hundreds of cases, ideally
  reviewed by someone with legal/compliance expertise.
- NY Local Law 144 logic only covers the AEDT bias-audit trigger condition, not the full
  notice/disclosure/data-retention requirements.
- LLM outputs are non-deterministic; results may vary across runs even with `temperature=0`.
- The rule-based classifier's keyword lists are intentionally small enough to be readable —
  they are a baseline to disagree with, not a production-grade compliance filter.

---

## Author

Martin Bielke — [github.com/MartinBielke](https://github.com/MartinBielke)
