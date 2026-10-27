# Setup

## Requirements

- An n8n instance (self-hosted or n8n Cloud). The review loop uses production webhooks, so it must be reachable from your browser.
- A [DeepSeek API](https://platform.deepseek.com) key with credit. DeepSeek is prepaid.

## 1. Import the workflow

In n8n, create a workflow, open the **…** menu → **Import from File**, and choose [`workflows/openaudit-v4.1.json`](../workflows/openaudit-v4.1.json).

## 2. Add your DeepSeek key

1. Open **Agent1, The Data Analyst**. Authentication should show **Generic Credential Type** with **Header Auth**.
2. Create a new credential:
   - **Name:** `Authorization`
   - **Value:** `Bearer sk-your-key` (the word `Bearer`, one space, then the key)
3. Save it as `DeepSeek API`.
4. Open **Agent2, The Data Presenter** and select the same credential.

## 3. Set your n8n host

Open **Build Review Page** and replace the placeholder:

```js
const reviewUrl = 'https://YOUR-N8N-HOST/webhook/review-latest';
```

with your instance's base URL, for example `https://my-n8n.example.com/webhook/review-latest`.

## 4. Check the model settings

**Node A System Prompt** and **Node B System Prompt** each hold `model` (`deepseek-chat`) and `max_tokens` (`16384`, the setting the example run used). The presenter writes a long HTML document, so Node B needs the larger output budget. If a run stops with a "hit max_tokens" error, raise `max_tokens` in that node, or switch it to a model with a larger output limit.

## 5. Activate and run

1. Save the workflow and set it **Active**.
2. Open **Data Upload Dump** and copy the **Production URL**. The Test URL won't resume after review.
3. Fill in the form:

| Field | What to enter |
|---|---|
| Company Name | The client's business name |
| Prepared for | The person the audit is addressed to |
| Transcript | The discovery call (.md, .txt or .pdf) |
| Currency | Shown with any stated cost figures |
| Engagement brief | Optional. Facts that must override the transcript, such as handling times, volumes, candidate workflows, tools and client questions |
| Supporting Documents | Optional PDFs or text: process docs, notes |
| Company Logo(s) | One or more PNG, SVG or JPG files, embedded in the audit |
| Labour Rate | The client's hourly admin cost, kept as context only |
| Location | Shown in the audit header |

4. Submit. After a few minutes the form redirects to the review editor. Correct any text, then click **Approve & Finalise**.
5. In n8n **Executions**, open the run and download the HTML from **Finalise Edited HTML**.

To try it end to end, use the files in [`examples/wade-court`](../examples/wade-court).

## Troubleshooting

| Error | Fix |
|---|---|
| `401` from an agent node | The credential value must be `Bearer ` plus the key, with one space. |
| "hit max_tokens" / "does not look like a complete HTML document" | Raise `max_tokens` in Node B System Prompt, or use a model with a larger output limit. |
| "`$execution.resumeUrl` is empty" | The workflow isn't active, or the form was opened from the Test URL. |
| "currency found outside the fields where this audit allows it" | The analyst put a dollar figure in a figure field. Re-run, or add the rule to the engagement brief: "mention the labour rate only as context." |
| "exactly three workflows" | The analyst returned more or fewer. Name the candidate workflows in the engagement brief. |
| Review link says "expired" | Reviews time out after 12 hours. Submit the form again. |
