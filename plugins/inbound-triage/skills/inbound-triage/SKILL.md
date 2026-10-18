---
name: inbound-triage
description: Sort new sign-ups and demo requests that arrive from a form with a name, a company and an email address. Identify the person and their company, check the company against the user's ideal customer with one yes/no question, find a dated reason to reach out for the best ones, and route every lead as call today, nurture or ignore with a one-line reason. Free-mail addresses are flagged without spending. Use this whenever someone asks who signed up, wants to triage, score, qualify or route inbound leads or demo requests, asks "which of these sign-ups should I call", "who are these new users", "is this lead worth a call", or pastes a list of new sign-up emails, even if they don't say "triage".
---

# Inbound triage

A new sign-up has already raised a hand, but most of them aren't worth a call. This skill turns a pile of sign-ups into a short list: who to call today, who to nurture, and who to leave alone, each with a reason. You read the addresses, decide what's worth paying to learn and make the call on each lead; Layrcake identifies the person behind each sign-up, checks the company's own website against the user's ideal customer, and researches the best ones for something recent to open with.

## Live data: check once, up front

Call Layrcake's `get_me` at the start. If the Layrcake tools aren't available, say so in **one line at the top of your first reply** ("I can sort these now; identifying people and checking their companies needs a free Layrcake connection, steps below"). Then still help: do step 1 (free mail, role addresses, duplicates, one company per domain), write the yes/no fit question with the user (step 2) and say what the full run would cost. Then give the connect steps under "Connecting Layrcake" at the end of this file and stop. Never guess a job title or company size from an email address: a made-up "VP at a 200-person SaaS" sends the user's best rep to the wrong call.

## 1. Sort the addresses first (free)

Before paying for anything, read every address:

- **Free mail** (gmail.com, googlemail.com, yahoo.com, outlook.com, hotmail.com, live.com, icloud.com, me.com, aol.com, proton.me, protonmail.com, gmx, yandex, qq.com, 163.com and similar): there is no company to look up and no lookup to buy. Set them aside for step 5.
- **Role addresses** (info@, sales@, hello@, team@, support@, admin@, contact@, office@): a shared inbox. You can still check the company (step 2); there is no person to identify.
- **Duplicates**: compare addresses trimmed and in lower case. Keep one.
- **Group by domain.** Two people from the same company is one account, and often a stronger signal than either alone. Check each company once.

Ask the user for anything the sign-up form captured with the address: the person's name, the company name they typed, and what they asked for (a demo, a trial, pricing). The name matters most (step 3).

## 2. Check fit against the user's ideal customer (3.5 credits per company)

Turn the user's ideal customer into **one yes/no question about what the company sells or does**, the way a seller would ask it: "Do they sell software that sales teams use to find or contact prospects?", "Do they run clinics that take patient bookings?", "Do they sell to other businesses?". One question, about the company's offering. Size and title are judged separately.

Call `check_companies` with `question` and up to 10 `company_domains` per call (every work domain from step 1, once each). Do this before identifying anyone: a company check costs less than a person lookup, and a company that doesn't fit needs no lookup at all. Each company answered `yes` or `not_found` costs 3.5 credits; `unclear` answers and unreadable sites are free. Say the cost first ("Checking 6 companies: up to 21 credits"). A call takes up to about a minute.

- `yes` comes with a word-for-word `excerpt` from the company's site and its `source_url`. Quote it in the hand-over: it is the reason the lead fits.
- `not_found` means none of the `pages_read` mention it. That is strong evidence for a clear mismatch (a beauty brand when the user sells sales software), weaker for a borderline one.
- `unclear` (free): send that company again once; if it stays unclear, judge from the company name and the person's title.

If the user also needs company size, industry or funding to route (for example, "only companies with 50 to 500 staff"), `enrich_company` with `want: ["profile"]` returns the company profile at 13 credits per company found (free when there's nothing). Use it only for companies that answered yes and only when size changes the route.

## 3. Identify the person (5 credits each, only on a match)

For sign-ups at companies that fit (and any you're unsure of), the email domain is the company's domain. To learn who the person is (title, LinkedIn, location), call `enrich` with:

- `person: { full_name, company_domain, email }` (the name and company from the sign-up form, the domain from the address, and the email when the form gave one)
- `want: ["profile"]`

It costs 5 credits when it finds the person, nothing when it doesn't. Say the most it can cost first: "Identifying 5 sign-ups: up to 25 credits, nothing for anyone not found."

**Send the full name and company domain.** enrich matches a person on their name and company, so send both, plus the email when you have it. If the form has no name, skip this step for that lead: route on the company (step 2) and let the first reply ask about their role.

Read the answer:
- `outcome: "hit"` gives `title`, `company_name`, `linkedin_url`, `location`, and the address's `email_verdict`.
- If `company_domain` in the answer differs from the address's domain, or the name differs, treat it as a different person or an old job. Flag it; don't route on it.

## 4. A dated reason to reach out, for the best one or two (9 credits each)

For the one or two leads you'd call today, call `research_company` with `company_domain`, `depth: "standard"`, an `audience` (what the user sells and that this person just signed up or asked for a demo) and an `idempotency_key`. Standard research costs up to 9 credits; pass `budget_credits: 9`. It returns an `operation_id`: poll `get_operation` with it every 10 seconds; it finishes in about half a minute.

Read the result:
- **`why_now`** lists recent dated claims, each with a source link. If there is one, it goes in the opening line of the call or email.
- **`claims`** are typed facts (what it sells, customers, funding, hiring, leadership, partnerships, expansion), each with a source URL and a date when the source had one. Use them as context.
- **`why_now` empty** means nothing recent was found. The sign-up itself is the reason to call. Don't present a year-old acquisition or launch as news; mention it as background if it helps ("since the acquisition last year").

Don't research nurture or ignore leads: 9 credits on a lead nobody will call is waste.

## 5. Free-mail and unidentified sign-ups (no spend)

For a gmail.com or similar address, say plainly what can and can't be learned:

- **Can:** the name, if the form captured one, and anything the person wrote in the form.
- **Can't:** their employer, title or company size. A person lookup has no company to match on.
- **What to do:** route as **nurture** by default, with a first reply that asks one question ("What company are you with, and what are you hoping to use it for?"). If the form gave a company name and you can find its domain, you can check the company (step 2); you still don't know the person's role.

Students, job seekers and people testing the product often use personal addresses. Treat a free-mail sign-up as a reason to ask a question, and spend nothing on it.

## 6. Route each lead

| Route | When | Reason line looks like |
|---|---|---|
| **Call today** | Company fits (`yes`) and the person can buy or strongly influence (founder, VP, head of the team that uses it), or they asked for a demo | "VP Sales at a sales-engagement platform that fits (their site, quoted: '<the fit excerpt>'). Asked for a demo." |
| **Nurture** | Company fits but the person is junior or adjacent; a second person from an account already marked call today; fit unclear; free mail; person not identified | "Fits, but RevOps manager; loop into the VP's call." |
| **Ignore** | Company clearly outside the ideal customer (`not_found` on a clear mismatch), a competitor researching, a student | "Consumer beauty brand; the user sells to sales-software companies." |

One line of reason per lead, built from what you found: the title, the quoted fit excerpt, the signal. When two people come from one company, route the account once: the senior person gets the call, the other is named in it.

## 7. Hand it over

1. **The routing table**, call today first: email (or name), who they are (title, company), fit (yes / no / unclear, with the short quote), route, one-line reason.
2. **For each call today**: the reason to reach out (a dated `why_now` with its link, or "they asked for a demo" when nothing recent came back) and a two-line opening for the call or email built only from those facts.
3. **Free-mail and unidentified sign-ups**: what was learned, what wasn't, and the one question to ask.
4. **What it cost**, added up from each answer's `credits_charged` (research is up to 9 credits a company).
5. **The next step**: "Want me to draft the replies for the nurture group, or find the decision-maker at the companies that fit but signed up through a junior person?" (the `no-bounce-list` skill finds the decision-maker).

## Judgement calls

- **Fit first, identity second.** A company check (3.5) costs less than a person lookup (5), and covers everyone from that company. Identify people only where the company fits.
- **No name on the form.** Route on the company and ask about their role in the first reply.
- **The person's title doesn't look like a buyer** (a podcast co-host, a partnerships lead, an engineer). They may still be the user, the champion or a partner lead. Nurture, and say which.
- **Several people from one company.** One account. The most senior person gets the call; mention the others ("your RevOps manager signed up too").
- **A senior person at a company that doesn't fit** (a famous founder at a consumer brand): still ignore for sales, and say why in one line. A big name is not a fit.
- **A competitor or a vendor in the user's space** signing up: flag it. It may be research, a partnership or a real buyer; let the user decide.
- **Research comes back with nothing recent.** That's normal for a company with no news in the last few months. The sign-up is the reason; don't stretch old news into a "congrats on".
- **A fit answer of `unclear` twice.** Judge from the company name and the person's title, and say it's your judgement.

## Example

User: "Five people signed up yesterday and one demo request came in. We sell contact data and email verification to sales-engagement software companies. Who should I call?" (A test list built from public search results; the sign-ups and the demo request are simulated.)

Six sign-ups: five work addresses at three companies (two at a sales-engagement platform, two at a cold-email software company, one at a consumer beauty brand) and one gmail.com address. Sorting is free: the gmail.com sign-up is set aside, and the five work addresses group into three companies. `check_companies` with "Do they sell software that sales teams use to find or contact prospects?" on the three domains: two `yes`, each with a quote from its own site, and `not_found` across seven pages for the beauty brand (10.5 credits). `enrich` on the four people at the two companies that fit, each with their name and company domain (plus the email or LinkedIn URL on file): all four identified with title and LinkedIn (20 credits). `research_company` on the best account, whose VP Sales asked for the demo (9 credits): no recent `why_now`; the newest dated claims were about a year old (an acquisition, a new product feature), useful as background.

Routed: **call today** for the VP Sales who asked for a demo (the fit quote, the demo request as the reason, the acquisition as background) and the head of sales at the cold-email company; **nurture** for the RevOps manager at the first company (named in the VP's call) and the partnerships lead at the second; **ignore** for the beauty-brand sign-up (outside the ideal customer, so nobody was looked up); **nurture** for the gmail.com sign-up, with one question about their company. Total: 39.5 credits.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Identifying sign-ups and checking their companies needs live data from Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=inbound-triage, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
