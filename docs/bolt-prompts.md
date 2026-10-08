# Bolt prompts for the new PeerLibre

Paste these into your PeerLibre project in Bolt one at a time. Wait for each to finish and check the preview before pasting the next. If Bolt asks to connect Supabase, say yes.

Reference: the finished design is at https://claude.ai/artifact/4Fr7Sn2urq2gzAdkrpsmqT (you can screenshot screens from it and attach them in Bolt for extra accuracy).

---

## Prompt 1: brand, layout and sign-in

Redesign PeerLibre with a classic, professional university-press identity. Keep any existing pages and content that still fit, but apply this design everywhere.

**Logo.** Use this exact SVG as the logo mark (save it as src/assets/logo-mark.svg and also as the favicon):

```
<svg viewBox="0 0 64 64" xmlns="http://www.w3.org/2000/svg"><circle cx="32" cy="32" r="31" fill="#13294B"/><circle cx="32" cy="32" r="27.2" fill="none" stroke="#C9A55A" stroke-width="1.1"/><path d="M13.5 25.5Q22.5 21.5 30.6 25.6V45.2Q22.5 41.4 13.5 45.2Z" fill="#FBF8F1"/><path d="M33.4 25.6Q41.5 21.5 50.5 25.5V45.2Q41.5 41.4 33.4 45.2Z" fill="#FBF8F1"/><path d="M17 30.2Q22.5 28 27.6 30.2M17 34.2Q22.5 32 27.6 34.2M17 38.2Q22.5 36 27.6 38.2M36.4 30.2Q41.5 28 47 30.2M36.4 34.2Q41.5 32 47 34.2" fill="none" stroke="#13294B" stroke-width="1.1" stroke-linecap="round" opacity=".55"/><path d="M42 37.2l2.2 2.2 4-4.4" fill="none" stroke="#C9A55A" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/><path d="M32 10.5l4.2 5.4-4.2 5.4-4.2-5.4z" fill="#C9A55A"/></svg>
```

Next to it, the wordmark "Peer" in bold and "Libre" in italic gold, with a small letter-spaced caption "SCHOLARLY PUBLISHING" underneath.

**Colours** (define as CSS variables / Tailwind theme): navy #13294B (primary, sidebar, hero), gold #C9A55A (highlights, credits, seals), darker gold text #8F6E2A, page background #F5F6F7, cards #FFFFFF, text #14213A, muted text #5B6578, borders #E0E3E8, AI accent teal #2B6A6E, success #1E6B47, warning #9A5512, error #A3281E. Support dark mode: background #0C1220, cards #131B2B, text #E7EAF0, and gold #D7B66E as the primary button colour.

**Type** (Google Fonts): "Libre Caslon Text" for headings (regular weight, italic for emphasis), "Libre Franklin" for all interface text, "IBM Plex Mono" only for hashes and IDs. Small uppercase labels with wide letter spacing. Corners are crisp (4–6px radius), with thin borders and no heavy shadows.

**Sign-in page.** Split screen.
- Left side: navy panel with a slowly animating engraved "guilloche" line pattern (like a banknote or certificate) drawn on a canvas in faint gold and white lines. On top of it: the logo, a gold label "PROOF-OF-CONCEPT · PILOT 2026", and the headline "Publish transparently." with "Review fairly." on the next line in gold italic. Below that is a short paragraph: "An editorial office where an AI Editor gives every manuscript a careful first reading, editors keep the final word, reviewers are rewarded once their reports are validated, and each step carries a verifiable proof."
- Under that, a "Certificate of provenance" card with a double thin gold border. It shows an example manuscript ID, a title, and five progress steps (Submitted, AI screened, Reviewed, Validated, Published) that fill in one by one in a loop. A gold rosette seal with a check mark stamps onto the corner when "Validated" is reached, and a changing hash is shown in mono text.
- Right side: "Sign in" with two groups. "Academic account": Continue with Google, Continue with ORCID, and an email field with an "Email me a code" button. Then "or use a wallet": Browser wallet (MetaMask, sign a message, no gas) and Create a test wallet. Add a note that Google, ORCID and email accounts get a wallet created for them automatically.
- Respect prefers-reduced-motion.

**Authentication (real).** Use Supabase Auth.
- Email: one-time code by email.
- Google: Supabase's built-in Google provider.
- ORCID: OAuth (OpenID Connect) against https://orcid.org/oauth/authorize with scope "openid", handled by a Supabase Edge Function that exchanges the code, reads the ORCID iD and name, and signs the user in.
- Wallet: Sign-In with Ethereum, where the user signs a message containing a nonce and the server verifies it.
- Keep the client IDs in environment variables.

After first sign-in, show an onboarding step: full name, optional ORCID iD, roles (Author, Reviewer, Editor, any combination, switchable later), expertise keywords, and a checkbox to agree to declare conflicts and keep reviews confidential. Give each new user 250 test credits (tPLIB, no monetary value).

**App shell.** Navy left sidebar with the logo and the items Home, Submit manuscript, My manuscripts, Review pool, Editor desk, then a divider, then Provenance ledger, Academic services, Credits & wallet. The active item has a gold bar on its left. The user's avatar and name sit at the bottom with Sign out. A top bar holds a "Working as" role switch (Author / Reviewer / Editor) and a gold credits balance pill. On phones the sidebar becomes a horizontal top bar.

**Home.** A navy welcome banner (with the same guilloche pattern) whose message and button depend on the role:
- Author: "Your research, on the record."
- Reviewer: "Your expertise, recognised."
- Editor: "Your judgement, supported."

Below the banner: four stat cards (credits, manuscripts submitted, reviews validated, reviewer pool balance), a role-specific list, a "How it works" card showing where the 100-credit fee goes, and a "Latest on the ledger" timeline.

---

## Prompt 2: submission, credits and the AI Editor

Add the manuscript submission flow, credits and an AI Editor.

**Database (Supabase).**
- Tables: journals, manuscripts, manuscript_versions, reviews, ledger_events, credit_transactions, pools.
- Seed three journals, each with a name, tier, scope, quality bar and AI threshold:
  - Journal of Open Research & Integrity: Q1 target, threshold 72.
  - Digital Innovation & Society Letters: Q2 target, threshold 62.
  - Applied Computing Reviews: Q2 target, threshold 65.

**Submit manuscript: a 3-step wizard.**
1. Details: journal (show its scope and quality bar beside the form), article type, title, abstract, keywords, and manuscript upload (PDF, DOCX or TXT, extracting the text in the browser). Add a "Fill with an example" button.
2. Fee and proof: the fee is 100 credits. Show a segmented bar of where it goes: 60 reviewer reward pool, 15 editorial and integrity, 10 technical and storage, 10 researcher waivers, 5 community. Three declarations must be ticked: original work; consent to an advisory AI first assessment; all authors approved with conflicts declared. On pay, compute a salted SHA-256 fingerprint of the file in the browser. Store the file privately and record a "Submission proof" ledger event containing only the fingerprint, version and time, never the content. If the editor desk-rejects the paper, the 60 reviewer credits are refunded to the author.
3. AI Editor: show an animated checklist while it works: reading the title and abstract, checking scope fit, assessing method and evidence, looking for ethics and integrity signals, writing the brief.

**AI Editor.** A Supabase Edge Function that calls the Anthropic Claude API (latest Claude Sonnet model, API key stored as a secret). It sends the journal's scope, quality bar and threshold plus the manuscript text, and returns JSON:
- scores from 0 to 10 for scope fit, originality, methodological rigour, evidence and data, literature, clarity, and ethics and transparency;
- an overall score from 0 to 100;
- an estimated tier (Q1–Q4 or "below indexing standard");
- a recommendation (send to review / revise before review / desk reject);
- confidence, a 2–3 sentence summary, strengths, concerns, questions for the authors, suggested reviewer expertise, and integrity notes.

Tell the model to be calibrated and never invent facts. Display it as a card with a circular score ring in teal, a coloured verdict box, horizontal bars per criterion, and the four lists. Add the note: "The AI Editor advises. A human editor makes the decision, and both are logged as separate verifiable events." Log an "AI screening logged" ledger event holding the score, the recommendation and a hash of the report.

**Manuscript page.**
- Header: ID, status pill, title and journal.
- A 6-stage progress bar.
- The AI report and the abstract.
- A side column with the editor's decision, the reviewer reward escrow (unreserved, reserved and released amounts), versions with fingerprints, and a provenance timeline of every ledger event for that paper.

---

## Prompt 3: review pool, editor desk, ledger, services and wallet

Add the remaining workflow.

**Editor desk** (Editor role).
- Queues: screening decisions, reviews awaiting validation, under review, and revised papers.
- On a paper awaiting screening, the editor picks Send to review, Revise before review, or Desk reject. Show whether the choice agrees with or overrides the AI Editor. Overriding requires a written reason.
- Log every decision with an agrees_with_ai flag. Show a "Human–AI agreement" percentage card and an "Override log" table.
- Block editors from handling their own papers.

**Review pool** (Reviewer role).
- Open requests (two reviewer slots per paper) with the author hidden, sorted by an expertise match score against the reviewer's keywords.
- Each request shows the reward range, 30 to 39.6 credits.
- "Accept and reserve 30 tPLIB" moves 30 credits from the paper's escrow into reserved.
- A reviewer can't review their own paper.

**Review form.**
- Structured sections: contribution, methodological quality, evidence and results, clarity, ethical issues, required revisions, recommendation (accept / minor / major / reject), and a confidential note to the editor.
- Show the AI Editor's questions as optional prompts.
- Add a "Check my report" button that calls a Claude Edge Function, the Review Quality Assistant. It returns completeness (0–100), a rating per section (strong / adequate / thin / missing), specificity, tone flags, suggestions, and a suggested quality factor Q between 0.80 and 1.20. It must never receive or consider the recommendation.

**Validation** (Editor).
- For each submitted report: a Q slider from 0.80 to 1.20, pre-filled from the assistant.
- T is computed from the time used, 0.90 to 1.10.
- A C checkbox for "no confidentiality or conflict breach" (C is 1 or 0).
- Show the live formula R = 30 × Q × T × C.
- "Validate and release" pays R to the reviewer and records a non-transferable reputation attestation. "Return for improvement" sends the report back.
- Integrity flags inform, never punish: report submitted within 20 minutes, high text overlap with another report, self-validation.
- Final decision: accept (publish, with an IPFS reference), minor revision, major revision, or reject. Authors upload revisions, which get a new fingerprint.

**Provenance ledger.**
- A table of all events: block number, event type pill, manuscript link, details, short transaction hash and time, with a filter by event type.
- A "Verify a document" tool: upload a file plus the author's salt to check it against a recorded version.
- A panel explaining what is and isn't on chain. Simulate the chain for now (Sepolia-style block numbers) and leave a clean interface for real contracts later.

**Academic services.**
- Cards with prices:
  - conference registration discount (60)
  - systematic review course (45)
  - reference manager (25)
  - e-book credit (35)
  - language editing (50)
  - donate to the waiver pool (20)
- Redeeming logs a ledger event.

**Credits & wallet.**
- Balance, sign-in method and wallet address, and "Link my own wallet".
- Sponsor codes (for example AIT-PILOT gives 200 once), a waiver request from the waiver pool, and the platform pool balances.
- Reputation attestations and a transaction history.

Seed a few example manuscripts from other authors, labelled "Example record", so the review pool and editor desk aren't empty.

---

## After Bolt finishes

- **Google:** in Google Cloud console, create an OAuth client (Web). Add Supabase's callback URL (shown in Supabase > Authentication > Providers > Google) and `https://peerlibre.com` as an authorised origin. Paste the client ID and secret into Supabase.
- **ORCID:** register a client at orcid.org (Developer tools). Set the redirect URI to the URL of the ORCID Edge Function. Put the client ID and secret in Supabase secrets.
- **Claude:** create an API key at console.anthropic.com and add it as the `ANTHROPIC_API_KEY` secret in Supabase.
- Redeploy from Bolt to Netlify.
