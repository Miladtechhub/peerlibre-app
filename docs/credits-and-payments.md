# Credits, token and payments: recommendation for PeerLibre

Prepared 7 October 2026. This is product advice, not legal advice. Get a lawyer to check the token parts before anything is sold.

## The short version

Sell **credits, not tokens**. Authors and institutions buy PeerLibre Credits with ordinary money, by card or invoice. Credits are prepaid service credits held in your own database, like a gift card or an APC voucher. Keep the PLIB token as a testnet tPLIB with no value during the pilot. Use the blockchain for proofs and reward receipts, not for payments.

Why:
- **Most academics have no crypto and won't get any.** Universities pay by invoice or purchase order, never by wallet. Requiring a token purchase would stop most of your users at the first step.
- **Selling a token is regulated.** In Thailand, offering digital tokens to the public falls under the Emergency Decree on Digital Asset Businesses (2018) and the Thai SEC. In the UK, promoting cryptoassets is restricted by the FCA's financial promotions rules. The white paper already says no public token before legal review. Prepaid service credits avoid this, provided they can't be traded or cashed out.
- **Funders are more comfortable.** A grant panel or university finance office can approve "publication credits". A token sale is far harder to get approved.

## The model

There are three kinds of value. Keep them separate.

| | What it is | Where it lives | Can be bought? | Can be cashed out? |
|---|---|---|---|---|
| **Credits** | Pays the 100-credit submission fee and services | Your database (Supabase) | Yes, by card or invoice | No |
| **Earned credits** | What reviewers and editors receive after validation | Same database, flagged as earned | No | Not at first (see below) |
| **tPLIB / PLIB** | Test token mirroring rewards on chain | Sepolia / Polygon Amoy testnet | No | No |

Earned credits can be spent on the Academic services marketplace, used to pay the reviewer's own submission fees, or donated to the waiver pool. That gives reviewers real value without paying them cash, which avoids payroll and tax questions in the pilot. Cash payouts can come later through Stripe Connect, which handles identity checks and tax forms.

## How a user pays

**Author, at step 2 of submission, when their balance is under 100 credits:** a "Get credits" panel opens with four options.
1. **Sponsor code**: their university or funder has prepaid credits (the code AIT-PILOT already works in the app).
2. **Waiver**: request credits from the waiver pool. An editor approves.
3. **Buy by card**: credit packs (for example 100, 300 or 1,000) through Stripe Checkout. Stripe returns them to the submission, the balance updates, and they pay the fee in one click. Stripe sends the receipt.
4. **Pay with USDC** (optional, wallet users only): through a crypto payment processor. They still receive ordinary credits, not a token.

Behind the scenes:
1. The server creates a Stripe Checkout session.
2. Stripe calls your webhook when the payment succeeds.
3. Only the webhook adds credits to the account; the browser never does.
4. Each credit change is written to the credit transactions table, and its hash is added to the provenance ledger.

**Institutions:** they buy a bundle by invoice. Credits go into an institutional account. Staff then claim them with a sponsor code, or automatically by signing in with an email on the university's domain. This is also how a funder can sponsor a reviewer reward pool.

**Reviewers:** they don't pay. They earn R = B × Q × T × C after validation and spend it in the app.

## Pricing

The white paper doesn't set a price, and a funder will ask. Suggested approach:
- Set 1 credit = a fixed amount in one currency. Then 100 credits is the cost of one reviewed submission. Compare that with typical APCs and with what two reviewer honoraria would cost.
- Publish the 60/15/10/10/5 split next to the price, so authors can see where the money goes.
- Give the waiver pool a fixed share. It is already 10% of each fee.

## What to build next

1. In the prototype: a "Get credits" panel with the four options above (card and USDC in demo mode). I can add this.
2. In Bolt (Supabase): Stripe Checkout plus a webhook Edge Function, a credit_transactions table, sponsor codes and institutional accounts.
3. Later, after legal advice: decide whether PLIB ever becomes a real token. If it does, the safest first step is a non-transferable reward receipt, not a tradable token.
