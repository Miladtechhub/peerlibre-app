# PeerLibre web app

Version 1, a clickable prototype of PeerLibre: transparent, credit-based peer review with an AI Editor and a verifiable provenance ledger.

![PeerLibre](brand/peerlibre-logo-preview.png)

## Try it

- Open `index.html` in any browser. There is nothing to install.
- Or turn on GitHub Pages: Settings > Pages > Deploy from a branch > `main` / root. The app is then live at `https://<your-user>.github.io/<repo>/`. Use that https address as the Google and ORCID redirect when you switch on real sign-in (see below).

## Repository layout

| Path | What it is |
|---|---|
| `index.html` | The whole app in one file (HTML, CSS and JavaScript) |
| `favicon.svg`, `brand/` | Logo, seal mark and preview |
| `docs/ai-role.md` | The role of AI and how the production AI Editor works |
| `docs/credits-and-payments.md` | Credits, token and payment model |
| `docs/bolt-prompts.md` | Prompts used to rebuild the site in Bolt |


`index.html` is the whole app in one file. Open it in any browser; no install or server needed. The live preview is published as a claude.ai artifact, where the AI Editor is backed by Claude.

## What's in v1

| Area | What works |
|---|---|
| Sign in | Real Google and ORCID sign-in (switch on with client IDs, see below), email with 6-digit code, browser wallet (MetaMask etc., sign-in message, no gas), test wallet. Non-wallet sign-ins get an embedded wallet automatically. |
| Onboarding | Name, ORCID, roles (author / reviewer / editor, switchable any time), expertise tags, confidentiality and conflict declaration. 250 test credits on joining. |
| Submit | Journal choice with scope and quality bar, PDF/DOCX/TXT upload with text extraction, 100-credit fee, live split into the five pools (60/15/10/10/5), salted SHA-256 fingerprint recorded. |
| AI Editor | Reads the manuscript against the journal's quality bar: 7 criteria, overall score, estimated tier, recommendation (send to review / revise first / desk reject), strengths, concerns, questions for authors, suggested reviewer expertise, integrity notes. Uses Claude in the artifact; falls back to a rule-based screen elsewhere. |
| Editor desk | Human decision on every paper. Agreeing or overriding the AI is logged; overriding needs a reason. Human–AI agreement rate and override log. Desk rejection returns the 60-credit reviewer reserve to the author. |
| Review pool | Blinded requests sorted by expertise match. Accepting reserves 30 credits in escrow. Conflict check blocks reviewing your own paper. |
| Review form | Structured report (contribution, method, evidence, clarity, ethics, revisions, recommendation, confidential note). AI Review Quality Assistant checks completeness and tone and never sees the recommendation. |
| Validation | Editor sets Q (0.80–1.20, pre-filled from the AI check), T is computed from time used, C is an integrity checkbox; R = B × Q × T × C is released. Integrity flags (very fast reports, text overlap, self-validation) inform the editor. |
| Revisions & publication | Revised versions get their own fingerprint; acceptance creates a publication record with an IPFS reference. |
| Provenance ledger | Every event with block, tx hash and time. "Verify a document" checks a file against a recorded version using the author's private salt. |
| Academic services | Spend credits on conference, training, software, books, editing, or donate to the waiver pool. |
| Get credits | One panel, opened from the fee step when short or from Credits & wallet: buy by card (packs of 100/300/1,000, example price 1 credit = USD 1, Stripe-style demo checkout; card 4000 0000 0000 0002 shows a decline), sponsor code, waiver request, pay with USDC (wallet), institution invoice request. Payments are simulated. |
| Credits & wallet | Balance, platform pools, non-transferable reputation attestations, transaction history. |

## What is simulated

- The chain: events are hashed and numbered in the browser, not sent to Sepolia or Polygon Amoy.
- Email codes are shown on screen instead of being emailed.
- Google and ORCID show a labelled demo form until the client IDs below are filled in and the app is served over https.
- Data is kept in the browser's local storage, per person. "Sign out" clears it.
- Example manuscripts from other authors are marked "Example record".

## Path to a production build

1. **Frontend:** port the screens to Next.js (static rendering fixes the crawlability gap noted in the analysis).
2. **Login:** wagmi + RainbowKit for wallets with Sign-In with Ethereum; Privy, Web3Auth or Dynamic for email/Google with embedded wallets and gas sponsorship; ORCID OAuth for researcher identity.
3. **Back end:** Postgres for the workflow, encrypted object storage with per-manuscript keys, Claude API for the AI Editor and Review Quality Assistant (same prompts as `editorPrompt` and `coachPrompt` in `index.html`).
4. **Contracts (Sepolia / Amoy):** Submission Registry, Reward Escrow, tPLIB ERC-20, non-transferable Reputation Attestation. Batch reward payouts so reviewers can't be linked to a manuscript.
5. **Integrations:** Crossref DOIs and IPFS pinning for published versions.

## Brand

`brand/peerlibre-logo.svg` (full logo) and `brand/peerlibre-mark.svg` (seal only, for favicons and avatars). Colours: Oxford navy `#13294B`, seal gold `#C9A55A`, paper `#FBF8F1`. Type: Libre Caslon Text for headings, Libre Franklin for the interface, IBM Plex Mono for proofs (all free Google Fonts).

## Turning on real Google and ORCID sign-in

Both are free. The app must be served from a real https address (for example GitHub Pages, Netlify or Vercel, or `peerlibre.com`).

**Google**
1. Go to console.cloud.google.com, create a project, then APIs & Services > OAuth consent screen. Choose External, add the app name, support email and logo.
2. APIs & Services > Credentials > Create credentials > OAuth client ID > Web application.
3. Under "Authorised JavaScript origins" add your site, e.g. `https://peerlibre.com`.
4. Copy the client ID into `AUTH_CONFIG.googleClientId` in `index.html`.

**ORCID**
1. Sign in at orcid.org, open Developer tools, and register a Public API client (free). Use sandbox.orcid.org first if you want to test with fake records.
2. Set the redirect URI to the exact page address, e.g. `https://peerlibre.com/` (or `/index.html` if that is how it is served).
3. Copy the client ID (looks like `APP-XXXXXXXXXXXXXXXX`) into `AUTH_CONFIG.orcidClientId`. While testing on sandbox, also set `orcidBase` to `https://sandbox.orcid.org`.

The buttons then switch from "Demo" to "Live". Google returns the person's name and email; ORCID returns their verified ORCID iD and name. In the production build these tokens should also be checked on the server before an account is created.
