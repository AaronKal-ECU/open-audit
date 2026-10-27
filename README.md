# OpenAudit

**Turn a discovery-call transcript into a client-ready, interactive automation audit.**

OpenAudit is an n8n pipeline I designed and built solo during my AI & Automation internship at Eloize. You upload a sales or discovery call transcript, and it produces a self-contained HTML audit: where the client's staff time goes, it picks the best automations worth doing first, and how many hours each one frees. Then you can edit the final HTML before it's finished.

**[View the live example audit →](https://aaronkal-ecu.github.io/open-audit/examples/wade-court/output.html)**

![Generated audit dashboard](docs/images/dashboard-hero.png)

## Used on a real client

OpenAudit produced the audit Eloize pitched to a Perth food-delivery business. After seeing it, the client hired Eloize to automate parts of the business's systems.

That audit covered:

| | |
|---|---|
| Workflows quantified | 3, plus 5 supporting flows and 1 next-phase item |
| Pain points identified | 4 |
| Manual handling today | ~67 hrs/yr |
| After automation | ~19 hrs/yr |
| Time freed | **~48 hrs/yr, a 72% reduction** |

Client details are withheld. [Full breakdown →](docs/using-openaudit.md#3-real-client-perth-food-delivery-business)

## What it does

```mermaid
flowchart LR
    A[Intake form<br/>transcript, brief, logo] --> B[Extract and<br/>build context]
    B --> C[Agent A<br/>analyst]
    C --> D[Harden Stats<br/>code verifies figures]
    D --> E[Agent B<br/>presenter]
    E --> F[Human review<br/>edit and approve]
    F --> G[Final HTML audit]
```

1. **Intake.** An n8n form takes the transcript, any supporting documents, the client logo, the labour rate and an optional engagement brief.
2. **Extraction.** PDFs and text files are extracted and assembled into one context block, with the transcript first as the primary evidence.
3. **Agent A, the analyst.** It reads the material and returns strict JSON: the three strongest workflows, pain points, supporting flows, next-phase items and the raw inputs for every figure. Each claim is marked `verified` or `estimate`.
4. **Harden Stats.** A code node recomputes every hours figure, total and percentage from those raw inputs, rejects impossible values and resolves the headline KPIs. The model never does the arithmetic that reaches the client.
5. **Agent B, the presenter.** It builds a single-file interactive dashboard from the hardened data, with charts, expandable workflow cards and before/after process maps.
6. **Human review.** The submitter is redirected to an in-browser editor. Nothing is final until a person approves it, and the approved file is re-validated before download.

## Design principles

- **The LLM judges, code does the maths, a human approves.** Language models are good at reading a messy conversation and deciding what matters, and unreliable at sums. Each job goes to the part that does it well. [More in design decisions](docs/design-decisions.md).
- **Hours first.** The audit leads with time freed, not invented dollar savings.
- **Honest confidence.** Every figure carries a `VERIFIED` or `EST.` marker traced back to the transcript.
- **Grounded, not padded.** Exactly three quantified workflows. Smaller ideas become supporting flows with no figures attached.

## Quick start

1. Import [`workflows/openaudit-v4.1.json`](workflows/openaudit-v4.1.json) into n8n.
2. Add a DeepSeek API key as a Header Auth credential on both agent nodes.
3. Set your n8n host in the **Build Review Page** node, then activate the workflow.
4. Open the form's production URL and submit the files in [`examples/wade-court`](examples/wade-court).

Full instructions are in [docs/setup.md](docs/setup.md).

## Example run

[`examples/wade-court`](examples/wade-court) contains a complete synthetic engagement: a discovery call with a GP clinic owner whose admin staff are stuck on repetitive paperwork, the engagement brief, and the [generated audit](examples/wade-court/output.html). From that one call, OpenAudit found **~464 hrs/yr** of admin time to free across three workflows, a 72% reduction.

| Intake form | Review step |
|---|---|
| ![Intake form](docs/images/form.png) | ![Review editor](docs/images/review-page.png) |

| Hours per workflow | Next-phase item |
|---|---|
| ![Hours chart](docs/images/dashboard-charts.png) | ![Next-phase item](docs/images/dashboard-next-phase.png) |

![Expanded workflow card](docs/images/dashboard-workflow.png)

More screenshots are in the [usage guide](docs/using-openaudit.md#2-example-wade-court-medical-centre).

![n8n workflow](docs/images/canvas.png)

## Repository layout

```
workflows/
  openaudit-v4.1.json   current version (DeepSeek, engagement brief, review loop)
  openaudit-v1.json     original August 2026 version, kept for history
examples/wade-court/    synthetic transcript, brief, logo and generated audit
docs/                   architecture, design decisions, setup, usage guide
```

## Docs

- [Architecture](docs/architecture.md): every stage and node in order
- [Design decisions](docs/design-decisions.md): why it's built this way
- [Setup](docs/setup.md): import, credentials, running your first audit
- [Using OpenAudit](docs/using-openaudit.md): engagement brief template, the Wade Court example, the real client audit

## Built with

n8n · DeepSeek API (the original client run used Claude Sonnet) · JavaScript code nodes · Chart.js

## Author

**Aaron Mark Kalaji**, Software Engineering student at Edith Cowan University, Perth. [GitHub](https://github.com/AaronKal-ECU)

Released under the [MIT License](LICENSE).
