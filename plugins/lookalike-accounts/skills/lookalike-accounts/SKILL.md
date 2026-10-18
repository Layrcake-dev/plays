---
name: lookalike-accounts
description: Start from one to three customer domains the user names and find companies similar to them, saying on what basis each one matches and how closely, dropping poor fits, and optionally confirming the trait that made those customers buy on each company's own website, then hand back a ranked account list ready for finding contacts. Use this whenever someone names a customer and says "find more companies like X", "lookalikes of our best customers", "similar companies", "who else is like this account", wants to expand a target list from customers who already bought, or asks for accounts that resemble a closed-won deal, even if they don't say "lookalike".
---

# Lookalike accounts

The user's best customers are the clearest picture of who buys. This skill takes one to three of them, finds companies that do the same thing for the same kind of buyer, explains why each one matches, and hands back a short ranked list worth prospecting. You pick the seeds, name the trait that matters and judge the matches; Layrcake finds the similar companies with a reason and a score for each, and can check each company's own website for the trait.

## Live data: check once, up front

Call Layrcake's `get_me`. If the Layrcake tools aren't available, still help: ask what the best customers have in common and why they bought (step 1), and say what the search will cost. Then give the connect steps under "Connecting Layrcake" at the end of this file. Never list lookalikes from memory: a company named without a source might not exist, might have pivoted, or might already be a customer.

## 1. Get the seeds and the reason they bought

- **One to three best customers**, as domains. Best means the user would gladly sign ten more of them: they bought fast, stayed, or pay the most. A famous logo that was a poor fit is a bad seed.
- **Why they bought**, in a sentence: the trait that made them need the product ("they take online bookings from patients", "they run a field team"). This is what you confirm in step 4.
- **Companies to leave out**: current customers, open deals, the user's competitors. Ask for the list, or at least the names.

## 2. Find similar companies

Call `find_companies` with `similar_to` set to one seed domain. It takes **one seed per call**; with three seeds, run three calls and split the budget between them.

Two ways to run it, priced per company returned (no results is free):

- **Unfiltered, 13 credits a company.** Each comes back with `company_domain`, `company_name` and a `matched_signals` entry of signal `similar`, with a one-line `summary` of what it shares with the seed and a `score`.
- **Filtered by place or size, 33 credits a company kept.** Add `country_codes` (ISO alpha-2, for example `["US"]`) and/or `company_headcount` (`{"min": 50, "max": 2000}`). Each lookalike then also comes back with headcount, industry and country, companies that don't fit are dropped for free and counted in `result.filtered_out`, and you pay only for the ones kept. Use it when the user sells only in some countries or only to a size band; otherwise unfiltered is cheaper and you can filter by eye.

Set `limit` and pass `budget_credits` as the most the user wants to spend on the search: the call then lowers `limit` to fit, and `result.budget_applied` says so. If not even one company fits the budget (33 credits for one filtered match), the call is refused before any charge. Send `dry_run: true` first (free) to see the most it can cost. Say the cost before the real call ("Finding 5 lookalikes of jane.app: up to 65 credits"). Send an `idempotency_key` so a retry doesn't charge twice.

## 3. Explain each match, then drop the poor fits

Read every `summary` and `score` and keep a company only if it shares the trait from step 1. Drop:
- anything on the leave-out list (current customers, open deals, competitors);
- a company that shares the category but sells to a different buyer (an enterprise suite when the seed serves solo practitioners, or the reverse);
- a company that is much bigger or smaller than the seed when the user's price only fits one size (unfiltered results carry no headcount: judge from what you know, or run the filtered search when size decides fit);
- a summary too vague to tell ("both are software companies").

With more than one seed, a company that comes back for two seeds is a stronger match: rank it first.

## 4. Optional: confirm the trait on the company's own site

The `summary` says what a company has in common with the seed in general terms. When the deal depends on one trait, confirm it with `check_companies`: one yes/no `question` (written as a seller would ask, in words the company would use on its site) and up to 10 `company_domains`. It costs 3.5 credits per company answered yes or not_found; unclear answers and unreadable sites are free. Say the most it can cost, and pass that as `budget_credits`.

A `yes` comes with the passage that states it, quoted word for word, and its page link. Keep those first. A `not_found` means none of the pages it read mention it: move that company down the list and say so. The `account-fit-check` skill covers phrasing questions and reading answers in full.

## 5. Hand it over

1. **The seed and the trait**: "Lookalikes of jane.app, for companies whose clients book appointments online."
2. **The ranked list**, best first: company, domain, why it matches (the `summary` in your words), the `score`, and the confirmed trait with its quote and link where you checked it.
3. **What you dropped**, each with a reason in a few words.
4. **What it cost**, added up from each call's `credits_charged`.
5. **The next step**: "Want me to find the decision-makers at these companies and get emails that won't bounce?" (the `no-bounce-list` skill). Pass it the kept `company_domains`.

## Judgement calls

- **Scores close together** (0.871 and 0.8709) mean the matches are equally good. Rank by the trait check and the user's own knowledge, and don't present a third decimal as a real difference.
- **Lookalikes of a customer are often that customer's competitors.** That's usually the point: they solve the same problem for the same buyer. Say so, in case the user's contract or relationship makes it awkward.
- **Too few results.** Try a second seed rather than a bigger `limit` on the first: different seeds find different neighbours.
- **The user wants a big list (50+).** Lookalikes are the precise, pricier way. Offer a broader `find_companies` search by industry and country for the volume (2 credits a company with `mode: "firmographic"`), then run the lookalikes for the top tier only.
- **A seed that is a household name** finds other household names. Seed with a mid-size customer to find mid-size accounts.
- **A name that doesn't match the domain.** Use the name on the company's own site and keep the domain as returned.
- **`observed_at` on a match** is when the similarity was recorded. A company may have changed since; the trait check in step 4 reads its site today.

## Example

User: "Jane is our best customer. Find me more companies like it." The reason they bought: their users' clients book appointments online, and the user sells appointment reminders.

`find_companies` with `similar_to: "jane.app"`, `limit: 2`, `budget_credits: 26` (up to 26 credits; 26 charged, under a second):
- SimplePractice (simplepractice.com), score 0.871: "Both offer HIPAA-compliant EHR and practice management software tailored to the needs of health and wellness professionals."
- Practice Better (practicebetter.io, returned under the name Better Group; the site calls it Practice Better), score 0.8709: "Both provide all-in-one EHR and practice management solutions designed specifically for health and wellness practitioners to manage their business."

Both share the buyer and the category, so both stay. `check_companies` on the first, "Do they let their users' clients book appointments online inside their product?" (up to 3.5 credits; 3.5 charged, 17 seconds): **yes**, quoting "Set your availability and offer online appointment requests with our patient scheduling software" (simplepractice.com/features/scheduling-calendar/).

Hand back: 1. SimplePractice, trait confirmed with its quote. 2. Practice Better, same category and buyer, trait not yet checked. 29.5 credits in all. Offer to find the heads of partnerships or product at both with `no-bounce-list`.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Finding companies like your best customers needs live data from Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=lookalike-accounts, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
