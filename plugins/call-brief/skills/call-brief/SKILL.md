---
name: call-brief
description: Before a sales call, build a one-page brief on the company and the person the user is meeting: what the company does and how it is doing, its recent moves with dated sources, who the user is likely meeting and what they have posted lately, and three sharp questions that only make sense after reading it. Use this whenever someone says they have a call, a meeting, a demo or a discovery call coming up, asks to "prep me for", "brief me on", "research before my call with", "who am I meeting at", wants talking points or questions for a meeting, or pastes a calendar invite with a company in it, even if they don't say "brief".
---

# Call brief

Ten minutes before a call, the user needs one page they can read in two: what this company is up to right now, who is on the other side of the table, and three questions that show they did the homework. This skill builds that page from dated, linked sources and nothing else. You decide what matters for the user's offer and write the questions; Layrcake researches the company, finds the person if the user doesn't know who it is, and reads their recent public posts.

## Live data: check once, up front

Call Layrcake's `get_me`. If the Layrcake tools aren't available, still help: ask who the meeting is with, give the user the brief's outline and the kinds of questions worth asking for their offer, and say you can fill it with dated sources once Layrcake is connected. Then give the connect steps under "Connecting Layrcake" at the end of this file. Never fill the brief from memory: a stale funding round or a wrong job title read aloud on a call does more harm than a blank line.

## 1. Get the inputs

- **The company**, as a domain (`gong.io`). If they give a name, confirm the domain before paying for anything.
- **The person**, if they know them: name and LinkedIn URL from the invite or the CRM. If they only know a role ("their head of sales"), you will find likely people in step 3.
- **What the user sells, and the call's purpose**, in a sentence ("we sell contact data to sales teams; this is a first discovery call"). It drives which facts go on the page and which questions you write.

## 2. Pick the route: the cheaper one that still answers

There are two ways to get this data. Pick by cost, and say which you picked in a line.

| Route | What it covers | Most it can cost |
|---|---|---|
| `research_company` (standard) + `find_posts` by author | Dated, sourced company facts and news; the person's recent posts | 9 + 3.5 = **12.5 credits** (plus 3 to find the person with `find_leads`) |
| `prospect_picture` `depth: "deep"` (the only depth that reads the person's posts) | Same research and posts, plus a verified email, company profile, funding and hiring signals | **65.5 credits** (`standard` 54 without the posts, `quick` 19 without any research) |

**Default to `research_company` + `find_posts`.** A call brief needs what the company is doing and what the person cares about; it doesn't need the company profile, hiring signals or a verified email, which are most of `prospect_picture`'s price. Use `prospect_picture` only when the user also wants a verified email and current buying signals in the same step (for example, they will follow up by email straight after), and run it with `dry_run: true` and an `idempotency_key` (required, even for a dry run) first to quote the exact maximum, then send the real call with a new key.

## 3. Find who they are meeting (only if they don't know)

Call `find_leads` with `company_domains: ["<domain>"]`, the `titles` the meeting is likely with (for a sales call: "Head of Sales", "VP Sales", "Chief Revenue Officer", "Director of Sales"), an `audience` sentence, `limit: 5` and `budget_credits: 3`. Cost: 0.25 credits per person, 3 credits minimum when anyone comes back; nobody found is free.

Before you use a result, **check each row's `company_name` matches the company**, and drop any row that names a different one. If nobody matching is left, ask the user who's on the invite.

Pick the most likely person by title and region (a regional VP for the user's region beats a global one they'll never meet). If two are plausible, show both in the brief and say which one you'd expect.

## 4. Research the company

Say the cost first ("Researching gong.io: 9 credits"). Call `research_company` with:
- `company_domain`
- `audience`: what the user sells and the call's purpose
- `depth: "standard"`
- `budget_credits: 9`
- a unique `idempotency_key`

It returns an `operation_id`. Poll `get_operation` with it every 10 seconds; standard research usually finishes in about 20 seconds. A recent cached result comes back already completed.

Read three parts of the result:
- **`claims`**: facts, each with a `kind` (identity, product, customers, news, expansion, funding, hiring, leadership, partnerships), a verbatim `excerpt` and a `source_url`.
- **`why_now`**: the recent dated claims.
- **`sources`**: every page it found, with `title` and `published_at`, including ones marked `used: false`. A dated news headline here (a product launch, a conference) is a recent move even if no claim was extracted from it; cite it by its title, date and link.

If the result is thin (fewer than three useful claims), offer `depth: "deep"` (up to 17 credits; it also reads the homepage and up to three pages on the company's site) before running it.

## 5. Read what the person has posted

Say the cost first ("Reading their last 90 days of LinkedIn posts: up to 3.5 credits"). Call `find_posts` with:
- `author: { "type": "profile_url", "profile_url": "<their LinkedIn URL>" }`
- `within_days: 90`, `limit: 5`
- `audience`: the same sentence as above
- a unique `idempotency_key`

It costs a 1-credit base even when nothing comes back, plus 0.5 credits per post returned. If you have only a name, call `enrich` with `person: { full_name, company_domain }` first for the LinkedIn URL (the tool refuses a name-only author, at no charge). Each post comes with its date, link, engagement and the author's current `headline`; check the headline still names this company.

## 6. Write the brief

One page, in this order. Every fact carries its date (or "undated") and a link.

1. **The company in two lines**: what it sells, to whom, and its size or traction as stated by a source ("over 5,000 customers; said in May 2026 it passed $500M ARR").
2. **Recent moves**, newest first, three to five bullets: date, what happened, source link. Leave out anything older than about 12 months unless it explains the present.
3. **Who you're meeting**: name, title, region, how long you can tell they've been there if the sources say, and **what they've posted lately**: one line per post with the date and the point they made, newest first. Note which post drew the most engagement.
4. **Three questions** (see below).
5. **What it cost**, from each answer's `credits_charged` (research shows its own in `cost_breakdown.total_credits`).

**Three questions that only make sense after reading it.** Each one ties a specific fact from the brief to the user's offer and asks the buyer to say something you couldn't look up. Test each: if it would make sense at any company, rewrite it. Good: "You split sales by industry now. When a rep opens a new vertical like healthcare, where does their account list come from?" Weak: "What are your priorities this quarter?"

## 7. Hand it over

1. **The brief** (above), ready to read on a phone.
2. **The route you chose**, and why, in one line.
3. **What to confirm on the call**: anything undated or from a third-party page, and any doubt about who's attending.
4. **What it cost**.
5. **The next step**: "Want me to draft a follow-up email for after the call, using what they tell you and these sources?"

## Judgement calls

- **A claim with no date, or a vague one** ("opened offices this August" with no year): put it under "undated" or leave it out. Never present it as recent.
- **Dates that disagree.** If a claim's `published_at` and its source's `published_at` differ, use the source's date.
- **Third-party pages** (review sites, competitors' blogs, directories) can be right but aren't the company's word. Prefer a press release or the company's own page for numbers you'll say out loud.
- **`ranking.by: "recency"`** means the facts weren't ordered for the user's goal: do that ordering yourself.
- **The person has posted nothing in 90 days.** Say so; it's useful (they don't live on LinkedIn, so don't open with "saw your post"). Write the questions from the company facts.
- **Their headline names a different company.** They may have moved. Tell the user before the call.
- **Posts are personal.** Use them to understand what the person cares about. On the call, "I saw your piece on…" is fine; reciting their activity back to them is not.

## Example

User: "I've got a discovery call with Gong's sales leadership tomorrow. We sell contact data to B2B sales teams. Prep me." Route: `research_company` + `find_posts` (most it could cost about 16 credits with the person lookup, against 65.5 for a deep `prospect_picture`). `find_leads` on `gong.io` with sales-leader titles, `limit: 5`: 4 people, 3 credits; picked a sales VP who leads one of its segments. `research_company` standard: 7 sourced claims in about 20 seconds, 9 credits: over 5,000 customers and $500M+ ARR said in May 2026, named a Leader in the 2026 Gartner Magic Quadrant for Revenue Action Orchestration (16 Oct 2026), new Revenue AI launches at its Celebrate '26 event (30 Sep 2026), new offices in New York and Austin (undated). `find_posts` by author, 90 days: 2 posts, 2 credits, the latest on getting conversation data into the CRM. Questions included: "Your team now sells by industry. When reps open a vertical like healthcare, where do their account lists come from today?" Total: 14 credits.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Building a sourced brief on a company and the person you're meeting needs live data from Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=call-brief, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
