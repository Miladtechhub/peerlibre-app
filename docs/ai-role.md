# The role of AI in PeerLibre

Prepared 7 October 2026.

## The principle

AI does the first read and the checking. People make every decision. Each AI output is advisory, is logged with a fingerprint, and sits next to the human decision it informed. Over time, that record shows how often editors agree with the AI and why they overrule it. This is the "trustworthy AI in peer review" evidence the AIT grant proposal promises.

## Where AI appears in the app today

| Moment | AI role | What the person sees | Who decides |
|---|---|---|---|
| 1. Submission | **AI Editor** (first editorial read) | Score on 7 criteria against the chosen journal's bar, estimated tier, recommendation (send to review / revise first / desk reject), strengths, concerns, questions for the author, reviewer expertise needed, integrity notes | Human editor. Agreeing or overriding is logged. An override needs a reason. |
| 2. Author feedback | Same report, shown to the author | Clear "what to fix" list before reviewers spend time on it | Author chooses to revise or continue |
| 3. Reviewer matching | Expertise match | Requests sorted by fit with the reviewer's tags | Reviewer accepts or declines; conflict check blocks own papers |
| 4. Writing a review | **Review Quality Assistant** | Completeness and tone check, missing sections, a suggested quality score Q | Reviewer edits; the assistant never sees the recommendation, so it cannot steer the verdict |
| 5. Validation | Integrity flags | Very fast reports, text overlap, self-validation | Editor sets Q and confirms C before the reward is released |

## How the AI Editor would work in production

A server-side agent (a Supabase Edge Function in Bolt, or a small backend), calling the Claude API. The browser never holds the API key or the manuscript text for longer than the upload.

**Step 1. Prepare.** Extract the text from PDF or DOCX. Strip the names, emails, affiliations and acknowledgements so the AI reads a blinded copy. Record the manuscript fingerprint.

**Step 2. Scope check.** Does the paper fit the journal's aims? If it plainly doesn't, suggest a better-fitting journal in the network instead of a flat rejection.

**Step 3. Rubric scoring.** Score the 7 criteria (originality, significance, method, evidence, clarity, ethics and reporting, scope fit) against that journal's written standard and threshold. Each score must quote the passage it relies on, so the editor can check it in seconds.

**Step 4. Checks with tools.** The agent can call:
- Crossref and OpenAlex: do the references exist, are any retracted, and what are the closest recent papers (a novelty check)?
- Reporting checklists: CONSORT, PRISMA, STROBE and similar, depending on study type.
- Statements: ethics approval, consent, data availability, funding, conflicts of interest, AI-use disclosure.
- Optional: a commercial similarity service such as Crossref Similarity Check.

**Step 5. Report.** Structured JSON (the same shape the prototype already uses), turned into the report card. It covers strengths, concerns, questions for the author, the reviewer expertise needed, and a confidence level. Low confidence routes the paper straight to a human with no recommendation.

**Step 6. Record.** Store the full report off chain. Put only its hash on chain, together with the model name, the prompt version and the journal standard version, so anyone can later prove which AI read which version and under which rules. The editor's decision is a separate event.

## Safeguards

- **The editor decides.** The AI can't reject a paper. A desk rejection always needs a human click, and the author's 60-credit reviewer reserve is refunded.
- **The author can appeal.** The author sees the AI report and can respond or appeal to a second editor.
- **Manuscripts stay confidential.** Use the API without training on the data, keep encrypted storage, and put nothing confidential on chain.
- **Bias is checked.** Agreement and override rates are tracked by field, country and career stage. If the AI is consistently harsher on one group, it gets recalibrated.
- **Calibration.** Every quarter, compare the AI scores with the final editorial outcomes and adjust the journal thresholds.
- **Disclosure.** Authors, reviewers and readers are told plainly where AI was used.

## Later ideas (after the pilot)

- **AI pre-submission check:** an author pays nothing, gets a private readiness report, and fixes issues before spending credits.
- **Reviewer report synthesis:** a summary for the editor of where the reviewers agree and disagree.
- **Revision check:** confirm that each reviewer point was addressed in the revised version.
- **Plain-language summary:** generated for each published paper, and approved by the author.
