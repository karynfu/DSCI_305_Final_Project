# ClearConsent Documentation

**A web tool that translates clinical trial consent forms into plain English, flags often-overlooked risks, and surfaces the questions patients should actually be asking their doctor.**

Built as a student project at Rice University for DSCI 305: Data, Ethics, & Society.

---

## The Problem

Informed consent is supposed to be the ethical foundation of clinical research, but consent forms are often long, confusing, and filled with medical jargon that make it difficult for patients to understand exactly what they are consenting to.

- A majority of American adults read at **below a 6th-grade level** [[1]](#ref1), and the NIH / NCI recommend consent forms be written at a **6th–8th-grade level** [[2]](#ref2).
- In practice, most clinical trial consent forms are written at a **10th-11th grade level** [[3]](#ref3), with median lengths well over 10 pages of dense legal and medical language [[4]](#ref4).
- Studies have repeatedly shown that a large share of trial participants cannot accurately describe key elements of the study they agreed to, including the risks, that they might receive a placebo, or that their data may be reused [[5]](#ref5).

The result: patients sign documents they do not fully understand, consent to risks they did not notice, and miss the chance to ask their doctor the questions that matter most to them. This is a real ethical gap between the *legal* requirement of informed consent and the *practical* reality of whether consent is truly informed.

## The Solution

ClearConsent is a single-page web application that takes a consent form, pasted text or an uploaded PDF, and produces four things:

1. **Study at a Glance** — a structured extraction of the core facts (condition, drug, phase, duration, number of visits, compensation, location, principal investigator) so the patient can easily understand the key details of the study.
2. **Plain-English Summary** — a 3–5 sentence explanation of the trial written at roughly a 6th-grade reading level.
3. **Red Flags** — specific but important points patients commonly miss: data-reuse clauses, limited injury compensation, unknown long-term risks, invasive procedures, washout periods, randomization to placebo.
4. **Questions to Ask Your Doctor** — 4–5 conversational, actionable questions tailored to the specific form, giving the patient a concrete starting point for a conversation with their care team.

The goal is not to replace the consent process or give medical advice; it's to empower patients with the knowledge to self-advocate when speaking to physicians and considering clinical trial participation.

## Application Functionality

- **Works end-to-end on real consent forms.** Users can paste arbitrary text or upload a PDF of a real clinical trial consent form (the repo includes three real consent forms as examples, and an AI-generated sample consent form built into the website) and get a structured, accurate analysis back.
- **Handles both input modalities.** Text and PDF go through the same analysis pipeline, with the PDF sent as `inline_data` bytes so the model reads the document directly rather than guessing from a filename.
- **The output is structured, not free-form prose.** The model is constrained to return a strict JSON schema (`info`, `summary`, `flags`, `questions`) with `responseMimeType: "application/json"`, so the UI can render it reliably into four dedicated result cards.
- **It is hardened against the common LLM failure modes.** Temperature is lowered, Gemini 2.5 Flash's thinking budget is disabled to reserve output tokens for the JSON, an explicit "use *Not specified* rather than inventing details" instruction is included to reduce hallucination, and errors from the API or malformed JSON are surfaced directly to the user instead of being swallowed.

## How It Works

```
 ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
 │  Paste text OR   │     │  Client-side JS  │     │   Gemini 2.5     │
 │   upload PDF     │ ──▶ │  builds request  │ ──▶ │   Flash API      │
 └──────────────────┘     │  with inline_data│     └──────────────────┘
                          └──────────────────┘              │
                                                            ▼
 ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
 │  4 result cards  │ ◀── │  JSON parsed &   │ ◀── │  Structured JSON │
 │   rendered in UI │     │   validated      │     │     response     │
 └──────────────────┘     └──────────────────┘     └──────────────────┘
```

## Tech Stack

- **Frontend:** Vanilla HTML, CSS, and JavaScript — no build step, no framework, one file.
- **LLM:** Google Gemini 2.5 Flash via the Generative Language REST API. Chosen because its free tier makes the tool accessible for a student project, it supports multimodal input (PDF bytes via `inline_data`), and it supports forced JSON output.
- **PDF handling:** PDFs are read with the browser's `FileReader`, base64-encoded, and sent to Gemini as `inline_data` so the model reads the document directly.
- **Hosting:** Runs as a static HTML file — no server required.

## Running Locally

1. Open [`index.html`](index.html) in any modern browser. That's it.
2. The Gemini API key is embedded at the top of the `<script>` block. Swap in your own key from [Google AI Studio](https://aistudio.google.com/app/apikey) to use the application. Note: for DSCI 305, a zip file containing the web application, with my personal API key, was submitted.

## Using the Tool

See the in-app **[User Guide](guide.html)** page for a step-by-step walkthrough of how to use ClearConsent.

## Limitations and Ethical Considerations

- **Built with ethics as the primary consideration.** ClearConsent was built in compliance with the ACM Code of Ethics and Professional Conduct, and designed to promote true informed consent.
- **Not medical or legal advice.** ClearConsent is an educational tool. It does not replace the clinical investigator, the IRB, or a patient's healthcare team.
- **LLM error is possible.** Even with low temperature, strict prompting, and forced JSON output, a language model can still misread a form. The red flags and summary should be treated as a starting point for conversation, not a definitive analysis.
- **Privacy.** Uploaded consent forms are sent to Google's Gemini API and are subject to Google's data handling policies. Users should not upload forms containing identifying patient information.
- **API key exposure.** Because the app runs entirely in the browser, the API key is visible to anyone who views the page source. This is would need to be moved to a server-side proxy before any public deployment.
- **Language.** Only English-language consent forms are currently supported.

## Future Work

- Server-side proxy so the API key is not exposed to the browser.
- Support for multilingual consent forms.
- Highlighting the specific spans of the original form that generated each red flag, for transparency.
- Allowing users to ask specific questions about their consent form.

## Files

| File | Purpose |
| --- | --- |
| [`index.html`](index.html) | The full application — UI, styles, and Gemini integration. |
| [`guide.html`](guide.html) | User-facing walkthrough of the tool. |
| [`Example Consent Forms`](Example%20Consent%20Forms) | Folder with three sample clinical trial consent forms (`Example 1.pdf`, `Example 2.pdf`, `Example 3.pdf`) for testing the tool. |

## Disclaimer

ClearConsent is a student project and does not constitute medical or legal advice. Always consult with your healthcare provider before making decisions about clinical trial participation.

## References

<a id="ref1"></a>[1] 2024-2025 Literacy Statistics. National Literacy Institute. Accessed April 15, 2026. https://www.thenationalliteracyinstitute.com/2024-2025-literacy-statistics

<a id="ref2"></a>[2] National Institutes of Health. *How To Write An Effective Consent Form: a Workshop for Investigators, Protocol Navigators and Research Staff.* NIH, National Institutes of Health. https://irbo.nih.gov/documents/303/Resource_and_Tools-v._3.14.2024.pdf

<a id="ref3"></a>[3] Paasche-Orlow MK, Taylor HA, Brancati FL. Readability Standards for Informed-Consent Forms as Compared with Actual Readability. N Engl J Med. 2003;348(8):721-726. doi:10.1056/NEJMsa021212

<a id="ref4"></a>[4] Beardsley E, Jefford M, Mileshkin L. Longer consent forms for clinical trials compromise patient understanding: so why are they lengthening? *Journal of Clinical Oncology.* 2007;25(9):e13–e14. https://doi.org/10.1200/JCO.2006.10.3341

<a id="ref5"></a>[5] Joffe S, Cook EF, Cleary PD, Clark JW, Weeks JC. Quality of informed consent in cancer clinical trials: a cross-sectional survey. *The Lancet.* 2001;358(9295):1772–1777. https://doi.org/10.1016/S0140-6736(01)06805-2
