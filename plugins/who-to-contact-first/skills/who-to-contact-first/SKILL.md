---
name: who-to-contact-first
description: Rank a list of people the user already has (name, company, LinkedIn URL) by who to contact this week, using each person's recent public LinkedIn activity (their own posts, comments, reactions, or posts they were named in), each with the signal, its date, the link and a first line that refers to it naturally. People with nothing recent go last and cost nothing. Use this whenever someone pastes or shares a list of people they already have and asks who to start with, who's warm, who's active, who to prioritise, how to order their outreach this week, or wants a reason to reach out to each person, even if they don't say "signals" or "LinkedIn".
---

# Who to contact first

The user has more people than time. This skill tells them where to start: everyone on their list, ordered by how good a reason they have to write this week, each with the dated evidence and an opening line that uses it the way a person would. You judge which signals are worth mentioning and write the lines; Layrcake reads each person's recent LinkedIn activity and returns their best recent reason to reach out, charging only for people where it finds one.

## Live data: check once, up front

Call Layrcake's `get_me`. If the Layrcake tools aren't available, still help: explain how you'll rank the list (step 3) and what a good opener looks like, and offer to rank by what the user already knows about each person. Then give the connect steps under "Connecting Layrcake" at the end of this file. Never invent activity: "loved your post last week" about a post that doesn't exist ends the conversation before it starts.

## 1. Get the list

- **People**: name, company and LinkedIn URL for each. The LinkedIn URL is what makes the match reliable; ask for it if it's missing. Rows from `find_leads`, `enrich`, a CRM export or a spreadsheet all work.
- **What the user sells, and to whom**, in a sentence. It decides which signals are worth referencing.

If they have no list yet, build one with `find_leads` (the `no-bounce-list` skill) and come back.

## 2. Read everyone's recent activity

Say the cost first: 3 credits per person with a signal, nothing for a person with none ("Checking 6 people: up to 18 credits, nothing for anyone who's been quiet").

Call `find_posts` with:
- `leads`: the people, each as `{ full_name, company_name, linkedin_url }` (add `title`, `company_domain` or `fingerprint` if you have them from `find_leads`)
- `budget_credits`: 3 × the number of people, or less if the user wants a cap (people beyond it come back `not_processed`; run them later)
- a unique `idempotency_key`

Send no `query`, `platforms` or `intent`: leads mode reads LinkedIn only. Up to 10 people answer inline (6 people take about 15 seconds). For 11 to 1,000 people the call returns an operation: poll `get_operation` with its `operation_id`, which returns the people 25 at a time; pass `next_cursor` as `cursor` for the next page.

Each person comes back under `result.leads` with:
- `status`: `signal` or `none`
- `signal_kind`: `own_post`, `comment`, `reaction`, or a company-post kind (their company's post)
- `evidence`: the post `text`, its `url`, its `date`, `post_author`, and for a reaction or comment, what they did (`activity`, `reaction`)
- `person`: ready for `enrich` if the user wants an email afterwards

Their own posts also appear in `result.posts` with the author's current LinkedIn `headline`. Read it.

## 3. Rank them

The signal type matters more than the date. Within a tier, newer first.

1. **Their own post about their work or company** (a launch, a hire, growth, a view on their market), last 30 days. The strongest opener.
2. **A post that names them** (thanked after an event, tagged in an announcement), or **a comment they wrote** with substance. Check `evidence.text` for their name: a post that names them ranks here whatever its `signal_kind`.
3. **A sign they've changed jobs**: their `headline` names a different company from the list. It's a strong reason to write, to the new company. Flag it and let the user decide whether they still want this person.
4. **A reaction to something in their industry or role.** Usable, lightly ("the tour-operator crowd seems to be all about AI this year").
5. **A reaction to generic content** (motivational quotes, viral posts) or a post about something unrelated to work. Too thin to mention. Rank them above the quiet people only if the user is short of names, and write a plain opener.
6. **`status: "none"`**: nothing recent. Last, free, and still worth contacting with a good company-level reason (the `signal-prospector` skill finds those).

## 4. Write the first lines

One line per person in tiers 1 to 4. Rules:
- Refer to the thing they did, never to the fact you watched them do it. "Congrats on the growth this quarter" beats "I saw you posted on 13 October that…".
- No quoting their post back at them, no list of their activity, no "I noticed you liked…". A reaction is a private-feeling signal: use the topic and never mention the click.
- Tie it to the reason for the email in the same line or the next one. A compliment that leads nowhere reads as a template.
- Keep it under 25 words.

## 5. Hand it over

1. **The ranked list** as a table: rank, name, company, signal in a line, date, link, first line. Signals first, quiet people last.
2. **Flags**: job changes, and anyone whose match looked doubtful (a headline or company that doesn't fit the list).
3. **What it cost**, from `credits_charged` (3 × people with a signal).
4. **The next step**: "Want me to find work emails for the top ones and draft the full first email from these lines?" (`no-bounce-list`, then `receipts-only-emails`).

## Judgement calls

- **Everyone came back quiet.** Say so: many buyers don't post. Offer company-level signals (hiring, funding, news) through `signal-prospector`.
- **The signal is about another company.** The person has probably moved on; their LinkedIn headline is the tell. Don't write to them about the old company.
- **An old signal.** Anything over 30 days is weak for "this week"; over 90, treat as none.
- **The `reaction` field is a short summary.** Read `evidence.text` and `post_author` for the full context of what they engaged with.
- **People beyond the budget** come back `not_processed`: they weren't checked, so don't rank them as quiet.
- **Private people.** Use public activity only to choose the opening topic. Never show the user a dossier of someone's likes.

## Example

User: "Here are 6 heads of sales at US software companies. Who should I email first this week?" (a test list built from public search results with `find_leads`: 6 people, 3 credits). `find_posts` with the 6 as `leads`, `budget_credits: 18`: 4 signals, 2 quiet, 12 credits, 14 seconds. Ranked: (1) a head of sales at a training-software company who posted on 13 Oct about the company's growth and a colleague's part in it: "Sounds like a big quarter over there. Is the sales team growing with it?" (2) a head of sales at a travel-tech company, thanked by name on 17 Oct in a post about a tour-operator conference: "Hope the conference was worth the trip. Was AI the topic there too?" (3) a VP whose list company was a social-listening firm but whose own posts (9 Oct) and headline are now about a space-tech startup: flagged as a likely job change. (4) a head of sales whose only signal was liking a motivational quote: plain opener, no reference. (5, 6) two quiet people, free. Total: 15 credits with the list.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Reading each person's recent LinkedIn activity to rank your list needs live data from Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=who-to-contact-first, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
