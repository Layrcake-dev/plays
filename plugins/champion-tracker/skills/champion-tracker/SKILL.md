---
name: champion-tracker
description: Check a list of past champions, happy users or former customers for anyone who has moved to a new company, confirm the new company is a fit, and hand back each mover with their new work email (checked) and a short "congrats on the new role" opener, plus who took over at their old company. Use this whenever someone wants to follow past champions, track job changes of customers or users, "who from our customer list changed jobs", "where are my old contacts now", alumni or champion tracking, or warm leads from people who already know their product, even if they don't say "champion".
---

# Champion tracker

Someone who bought or loved the user's product, then moved to a new company, is one of the warmest leads the user has: they already know the product works, and a new job is when people bring in the tools they trust. This skill checks each past contact's current job, spots who has moved, checks the new company fits, and hands back each mover with a fresh email and an opener. It also finds who replaced them at the old company, so the account the user already has stays covered. You judge who really moved and whether the new company fits; Layrcake looks up each person's current role and email.

## Live data: check once, up front

Call Layrcake's `get_me`. If the Layrcake tools aren't available, still help: explain how the check works, what it costs for their list (step 2), and draft the opener template. Then give the connect steps under "Connecting Layrcake" at the end of this file. Never tell the user someone has moved unless a lookup says so.

## 1. Get the list

For each past contact you need:
- **Full name**
- **LinkedIn URL**: this is what makes the check reliable. Without it, a lookup by name can't tell a mover from someone else with the same name.
- **The company they were at** (name or domain) when they championed the product.

Also get **what the user sells, and who it fits** (size, industry, country), so you can judge the new companies. Lists usually come from a CRM export of closed-won contacts, a list of power users, or people who left a good review.

## 2. Look up each person's current job

For each contact call `enrich` with `person: { full_name, linkedin_url }` and `want: ["work_email", "profile"]`. **Leave out the old company's domain and name**: you want where they work now, and name plus LinkedIn finds it. Each hit costs 5 credits (a hit returns their current title, company and a checked work email); a miss costs nothing.

Say the most it can cost first ("Checking 20 past champions: up to 100 credits, nothing for anyone not found"). For long lists, start with the 10 contacts who were strongest champions.

## 3. Sort who moved

Compare each result to the list:

| What came back | Means | Do |
|---|---|---|
| Same company (same domain, or the same company under a new name or domain, e.g. a rebrand) | Still there | Not a mover. They may still be a champion where they are. |
| A different company, same person (the LinkedIn URL matches) | **Moved** | Carry on to step 4 |
| A different company and a different LinkedIn URL or name | Another person | Treat as a miss |
| `outcome: "miss"` | Unknown | Say so. Don't assume they left. |

A middle name dropped or a city that changed is fine when the LinkedIn URL matches. A new title at the same company is a promotion: worth a note to the user, and it stays in the "still there" group.

## 4. Check the new company fits

Judge first from what the user told you and what you know of the company from its domain. When fit hangs on what the company sells or does, call `check_companies` with `company_domains` (up to 10 movers' new companies) and **one plain yes/no question** about the company's offering ("Do they sell software to other businesses?", "Do they run a sales team that books demos?"). It costs 3.5 credits per company answered yes or not_found; unclear answers are free. A yes comes with a quote from the company's site. Ask about one thing only; split a two-part question into two calls, or keep the part that decides fit.

A mover at a company outside the user's market is still worth a friendly note; mark it as a relationship touch rather than a sales lead.

## 5. Write the opener

Two or three sentences, warm and short:
- **Congratulate them on the new role** at the new company, by name of the company.
- **Remind them of the shared history** in a few words ("you rolled us out at [old company]").
- **Offer one useful thing**: a quick setup for the new team, what's new since they last used it.

Don't claim to know when they moved: the lookup gives their current job and no start date. "Congrats on the move to [new company]" works whether it was last week or last spring. Don't mention how you found out.

## 6. Who took over at the old company (optional)

Offer it; don't default to it. For each mover, call `find_leads` with `company_domains: [old company]`, `titles` (the mover's old title and its neighbours, e.g. "Revenue Operations Manager", "Head of Revenue Operations", "Sales Operations Manager"), `audience` and `limit: 5`: at least 3 credits when anyone comes back, nothing when nobody does. **Drop the mover from the results** by LinkedIn URL. If nobody else comes back, say the seat looks unfilled or the new person isn't listed yet, and suggest checking again in a month.

## 7. Hand it over

1. **The summary**: "4 of 5 found; 1 has moved, 3 are still where they were, 1 not found."
2. **Movers**, one row each: old company → new company, new title, fit (yes / relationship touch, with the `check_companies` quote when you used it), new work email with its verdict (`valid` or `catch_all_valid`: send; anything else: hold), and the opener.
3. **Still in place**, briefly: these are expansion or reference conversations.
4. **Who took over**, if run, per old company.
5. **What it cost**, added up from each answer's `credits_charged`.
6. **The next step**: "Want me to find the other decision-makers at the movers' new companies?" (the `no-bounce-list` skill), and offer to run this check again every quarter.

## Judgement calls

- **Most of the list hasn't moved.** That's normal for a list checked a few times a year. Report it plainly; the ones still in place are the user's references.
- **Same company, new domain.** Companies rebrand and change domains. If the name is close and the title is unchanged, call it the same company and say so.
- **The email is at the old company's domain.** The lookup found an old job. Hold the email and flag it.
- **The mover went to a competitor of the user or to a vendor.** Mark it as a relationship touch and leave it out of the sales list.
- **No LinkedIn URL in the list.** Run `enrich` with name and old `company_domain`. A hit at the old company means they're still there. A miss can't tell you anything: ask the user for the LinkedIn URL rather than guessing.
- **Personal data.** The list is the user's own customer contacts. Keep the hand-over to the user; don't paste the list into anything shared.

## Example

User: "Here are five revenue operations managers who championed us last year. Has anyone moved?" `enrich` each with name and LinkedIn only: 4 hits (20 credits), 1 miss (free). Three were still at the same company: one listed under the company's other domain, two with a different city on file. One had moved: a revenue operations manager formerly at a health benefits software company, now in the same role at a records management software company, with a `valid` work email at the new domain. Fit: records management software sold to businesses, judged from what the company does. A `check_companies` call (3.5 credits) found no single passage stating it, so the fit is marked as judged. Opener: "Congrats on the move to [new company]! You set us up at [old company]; happy to get your new team running in a week if it's useful." Who took over at the old company: `find_leads` there for revenue operations titles (3 credits) returned nobody else, so the seat looks open or the new person isn't listed yet. Hand back "4 of 5 found; 1 moved, 3 still in place, 1 not found". Total 26.5 credits.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Checking where your past champions work now needs Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=champion-tracker, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
