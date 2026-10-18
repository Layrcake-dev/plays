---
name: hiring-manager-finder
description: Find the person who owns a specific open role. Finds companies with a live job post for the role the user serves, then the manager that hire will report to at each one, with a checked work email and an opening line built from the post. Use this whenever someone sells to teams that are growing (recruiters, outsourced SDR or support agencies, training, tools for a team) and asks "who is the hiring manager for this role", "find the hiring manager", "who owns this job opening", "reach the person behind this job post", or wants to write to the manager behind a job post, even if they don't say "hiring manager".
---

# Hiring manager finder

A job post is a company saying out loud, with a date on it, that a team is growing. The person to write to is the manager the new hire will report to: they feel the gap every day and they decide what help to buy. This skill finds companies with a live post for the role the user serves, works out who that manager is likely to be, finds them, checks their email, and writes an opener from the post itself. You choose the role, the manager titles and the best person; Layrcake finds the posts, the people and the emails.

## Live data: check once, up front

Call Layrcake's `get_me`. If the Layrcake tools aren't available, still help: map the user's role to its manager titles (step 2) and say what the run would cost. Then give the connect steps under "Connecting Layrcake" at the end of this file. Never present a job post you couldn't check, and never write a guessed email.

## 1. Get the inputs

- **The role the user serves**: the job title a company would post ("Sales Development Representative", "Customer Support Specialist", "Data Engineer").
- **What the user sells, and to whom**, in a sentence. It goes in `audience`.
- **Companies to check** (domains), if they have a list. If not, you'll search by role (step 3, option B).
- **Where and how big**: country, and a headcount range if it matters.

## 2. Map the role to its manager

Search for the person the role reports to, plus one level up as a fallback. Name two to six titles, the most specific first.

| Role being hired | Likely hiring manager | One level up (fallback) |
|---|---|---|
| SDR / BDR | Sales Development Manager, SDR Manager, Head of Sales Development | VP Sales, Head of Sales, CRO |
| Account Executive | Sales Manager, Director of Sales | VP Sales, CRO |
| Customer Success Manager | Head of Customer Success, Director of Customer Success | VP Customer Success, COO |
| Support agent | Support Manager, Head of Support | VP Customer Experience |
| Marketing manager, demand gen | Head of Marketing, Director of Demand Generation | VP Marketing, CMO |
| Software or data engineer | Engineering Manager, Head of Data | VP Engineering, CTO |
| Recruiter | Head of Talent, Talent Acquisition Manager | VP People |
| Bookkeeper, accountant | Controller, Finance Manager | CFO |

At companies under about 50 staff, the founder or CEO usually hires the first person in a role: add "Founder" and "CEO". A post titled "first SDR" or "founding AE" means the same.

Say the titles you picked and why, in a line.

## 3. Find companies hiring for the role

**Option A, the user has a list.** Call `find_intent` with up to 25 `company_domains`, `roles` set to the role's titles (it matches the job title literally, so add the common spellings: "SDR", "BDR", "Sales Development Representative") and `audience`. Leave `signals` out: only hiring is checked. A company hiring now costs 11 credits and brings its latest funding round with it; a company that isn't hiring costs nothing. Say the most it can cost first ("Checking 8 companies: up to 88 credits, nothing for those not hiring") and set `budget_credits` to that.

**Option B, no list.** Call `find_companies` with `hiring: { titles, posted_within_days: 30 }`, `country_codes`, `company_headcount`, `industries` if the user sells to one, and `audience`. Each company returned costs 11 credits; rows that don't fit `audience` are left out and not charged. Start small: `limit: 3` to `5`, with `budget_credits` at 11 per company. Say the cost first ("Finding 3 companies hiring SDRs: up to 33 credits").

Each company comes back with the job title, the post's link (`evidence_url`) and the date it was seen (`observed_at`), plus headcount, `jobs_last_30_days` and its last funding round.

## 4. Find the manager at each company

Call `find_leads` once for all the companies with `company_domains` (the hiring companies), `titles` (the manager titles from step 2, fallback titles included), `audience` and `limit: 12`. It costs 0.25 credits per person returned, at least 3 credits when anyone comes back; nobody found is free. Set `budget_credits: 3` for a first pass.

Pick the **best one, or two at most, per company**: the most specific title first (a Head of Sales Development beats a VP Sales for an SDR post). Read the job title for clues about scope: "APAC" or "EMEA" in the post title means the hire sits in that region: prefer the manager who owns that region, then a global lead.

If a company returns nobody, say so. Offer one more search for that company with wider titles (Director of Sales, SVP Sales, Founder, CEO), up to 3 credits, and don't guess a name.

## 5. Get and check the email

For each person you picked, call `enrich` with `person: { full_name, company_domain, linkedin_url }` and `want: ["work_email"]`. Each email found costs 5 credits; nothing found costs nothing. Say it first ("Looking up 2 managers: up to 10 credits").

Every email `enrich` returns has already been checked, and its `email_verdict` is that check's result. `valid` and `catch_all_valid` are ready to send. For `catch_all_unknown`, `risky`, `unknown` or no verdict, call `verify_email` on that address (1.25 credits) before you put it in the hand-over; hold it if the result is still unsure.

If `enrich` returns a different person, title or company than you searched for, treat it as a miss and say so.

## 6. Write the opener from the post

One or two sentences, built only from what the post and the company data say:
- **Name the post**: the role, the team, and anything specific in the title (region, segment, "inbound and outbound", "first").
- **Connect it to the user's offer** in the manager's terms: ramp time, coverage while they hire, quality of pipeline.
- **No invented facts.** The post's title and date are what you have; don't claim to know the salary, the team size or why they are hiring.

Example shape: "Saw you're hiring an SDR for APAC. While that seat is open, we can cover APAC outbound for you from week one, then hand the pipeline to whoever you hire."

## 7. Hand it over

1. **The role and manager titles** you searched for, in a line.
2. **One row per company**: company, the post (title, date seen, link), the manager (title only in shared reports; name in the user's own working list), the email with its verdict (send or hold), and the opener.
3. **Companies with no manager found**, and the next search you'd run for them, with its cost.
4. **What it cost**, added up from each answer's `credits_charged`.
5. **The next step**: "Want me to research these companies and write the full first email for each?" (the `receipts-only-emails` skill).

## Judgement calls

- **The manager search comes back thin.** Mid-size companies often have no public "sales development" title. Search that company again with wider titles (Director of Sales, SVP Sales, Founder, CEO) before giving up on it.
- **The post is for a region.** The hiring manager may sit there, or a global lead may own it. Prefer whoever the title says owns that region.
- **An old or stale post.** Anything older than about 60 days may be filled. Say the date and let the user judge.
- **A recruiter or talent partner shows up.** They help, but they don't buy. Put them second, as a route in, after the manager.
- **Several open roles.** One posting per company comes back, so don't claim a count of openings. `jobs_last_30_days` is total hiring at the company, across all roles.
- **Agency or staffing firm posts.** A staffing company hiring SDRs for its clients is not a buyer. Skip it.

## Example

User: "We run an outsourced SDR team for US software companies. Find two companies hiring SDRs and the person I should email." Role: SDR. Manager titles: Sales Development Manager, SDR Manager, BDR Manager, Head of Sales Development, Director of Sales Development, with VP Sales, Head of Sales and CRO as fallback. `find_companies` with `hiring.titles: ["Sales Development Representative", "Business Development Representative"]`, US, 50–500 staff, Software Development, `limit: 2` (22 credits): a 210-person data platform company hiring an SDR for APAC (post seen that day) and a 193-person fleet software company hiring an "Inbound & SMB Outbound" BDR (seen the day before). `find_leads` across both with the eight titles (3 credits): one manager at the data platform company; nobody with those titles at the fleet software company. `enrich` on that manager (5 credits): a work email, verdict `valid`. Opener: "Saw [company] is hiring an SDR for APAC. We can run APAC outbound from week one while you hire, then hand the pipeline over." The fleet software company goes back to the user with the next step: a search with wider titles (Director of Sales, SVP Sales, Founder, CEO), up to 3 credits. Total 30 credits.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Finding live job posts and the managers behind them needs Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=hiring-manager-finder, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
