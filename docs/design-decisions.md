# Design decisions

OpenAudit started as a two-day sprint to turn a sales call into a client deliverable. These are the choices that shaped it, and why.

## 1. The LLM judges, code does the maths

Language models are good at reading a rambling conversation and deciding what matters. They are unreliable at arithmetic, and they will happily state the same figure two different ways in one document.

So the analyst agent never produces a total. It supplies only raw inputs: hours per occurrence before and after, and occurrences per year. The **Harden Stats Parsed** code node computes every annual, weekly, percentage and working-day figure from those inputs and rejects impossible values, such as more hours after automation than before. The presenter is told to display numbers exactly as given and never recalculate them.

The result: every figure on the dashboard traces to one field, computed one way.

## 2. Two agents with a contract between them

One prompt that both analyses and designs a dashboard does both jobs badly. Splitting them gives:

- **Agent A (analyst):** low temperature, JSON mode, a strict schema. Its only job is to get the facts right.
- **Agent B (presenter):** builds the HTML from hardened data and has no access to the raw transcript, so it can't reintroduce unverified claims.

The hardened JSON is the only interface between them. The presenter can be restyled or swapped without touching the analysis, and a failed presentation can be retried without paying for the analysis again.

## 3. Hours first, not dollars

Automation audits often lead with savings figures built on assumed rates. They impress at first and fall apart as soon as a client asks how they were worked out. OpenAudit leads with **hours freed**, which the client can check against their own week.

A dollar figure may appear only when the client actually stated one, and only as narrative context. A code check rejects currency anywhere in totals, KPIs or workflow figures.

## 4. Exactly three workflows

A discovery call surfaces a dozen ideas. An audit that quantifies all of them reads like a wish list. OpenAudit quantifies exactly **three**, the strongest and best-evidenced. Everything else becomes a **supporting flow** linked to one of the three, with a description and no figures. Code enforces the count and that supporting flows stay unquantified.

## 5. Every claim carries its confidence

Each figure is marked `verified` (stated in the call) or `estimate` (inferred or approximated), and the dashboard shows it. The analyst is told to under-claim when unsure. A consultant can then defend every number in the room.

## 6. The engagement brief

Transcripts are messy: speaker labels are wrong, figures get corrected mid-call, and the client's real priority is often implied rather than said. The optional **engagement brief** lets the consultant pin down the facts that must win, such as fixed handling times, volumes, candidate workflows and existing tools, without editing prompts. The same workflow then serves a GP clinic or a food business unchanged.

## 7. A human approves before anything ships

The generated audit opens in a browser editor, and nothing is final until a person clicks **Approve & Finalise**. The approved file is re-validated before download. The editor runs inside an iframe so its controls and styles never leak into the client's file.

## 8. Single-file output

The audit is one HTML file with inline styles and scripts and the logo embedded as a data URI. It can be emailed, opened offline or printed, with nothing to host and no broken images. The logo is passed to the presenter as a placeholder and swapped in afterwards, so image data never enters a model prompt.

## 9. Model-agnostic

The original client run used Claude Sonnet. The public version uses DeepSeek through its OpenAI-compatible API, at a fraction of the cost. The response parsers accept both formats, and the model name and token limit sit in one Set node per agent.
