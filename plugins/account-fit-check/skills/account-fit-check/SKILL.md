---
name: account-fit-check
description: Check a list of company domains against the user's must-have criteria ("do they sell to enterprises?", "do they offer an API?", "do they book appointments in their product?") by reading each company's own website, and hand back yes with the quoted passage and page link, not_found or unclear for every company, keeping only the fits before anyone pays for people or emails. Use this whenever someone wants to qualify, filter, vet or score accounts, check whether companies on a list they already have fit their ICP, asks "which of these companies actually do X", wants to clean a target account list, or wants proof a company has a need before outreach, even if they don't say "qualify".
---

# Account fit check

A list of company names is cheap. Paying to find people and emails at companies that turn out not to fit is the expensive part. This skill turns each of the user's must-haves into one yes/no question, asks it of every company's own website, and keeps the companies that answer yes with a quote to prove it. You write the questions and judge the answers; Layrcake reads each company's site and returns the passage that states it, word for word, with its page link.

## Live data: check once, up front

Call Layrcake's `get_me`. If the Layrcake tools aren't available, still help: turn the user's criteria into well-formed questions (step 2) and say what the check will cost. Then give the connect steps under "Connecting Layrcake" at the end of this file. Never mark a company as a fit from memory or a guess: a yes needs a passage from the company's own site.

## 1. Get the inputs

- **The companies**, as domains (up to 10 per call; split a longer list into calls of 10). Clean them first: one domain per company, no duplicates, no personal sites.
- **The must-haves**, in the user's words, and what the user sells. If they give five criteria, ask which one or two would disqualify a company on their own. Those are the ones worth paying to check.

## 2. Turn each criterion into a question the site can answer

`check_companies` answers one yes/no question about each company from that company's own website. A good question:

- **Is about what the company offers or does**, asked as a seller would: "Do they let their users book appointments inside their product?" works; "Are they a good fit for us?" or "Are they growing?" can't be read off a website.
- **Uses the words the company would use on its own pages.** Companies write "book an appointment", "online booking", "API", "developer docs", "SOC 2", "enterprise plan", "for teams of 500+". Put those words in the question.
- **Asks one thing.** "Do they offer an API and a Salesforce integration?" is two questions; split it, or keep the one that matters.
- **Points at something a company would publish.** Products, features, plans, who they serve, integrations, certifications, locations. Revenue, budget, tools they use internally and how happy their customers are rarely appear on a company's own site, so those questions come back not_found.

Rewrite the user's criteria and show them before running:

| The user says | Ask |
|---|---|
| "do they sell to enterprises?" | "Do they offer an enterprise plan or sell to large enterprise customers?" |
| "do they offer an API?" | "Do they publish a public API that developers can use to build integrations?" |
| "do they book appointments in their product?" | "Do they let their users book appointments inside their product?" |
| "are they in healthcare?" | "Do they sell their product to healthcare providers such as clinics or hospitals?" |
| "do they have compliance needs?" | "Do they state that they are SOC 2 or HIPAA compliant?" |

Questions run 10 to 300 characters.

## 3. Run the strictest question first

Call `check_companies` with `question` and `company_domains` (up to 10). It costs **3.5 credits per company answered yes or not_found**; unclear answers and sites it can't read are free. Say the most it can cost first ("Checking 5 companies: up to 17.5 credits"), and pass that number as `budget_credits` so the call can never cost more. Send an `idempotency_key` so a retry doesn't charge twice. If a call fails as rate-limited, retry it with the same key after 30 seconds. If you want the exact ceiling without running anything, send the same call with `dry_run: true` (free).

With more than one criterion, don't ask every question of every company. Ask the one most likely to rule companies out, then ask the next question **only of the companies that answered yes**. Each round gets smaller and cheaper.

A call takes up to about a minute (the run below took 13 to 21 seconds).

## 4. Read each answer

Each company comes back with `answer`, `excerpt`, `source_url` and `pages_read`.

| `answer` | What it means | What to do |
|---|---|---|
| `yes` | A passage on the company's own site states it, quoted in `excerpt`, on the page in `source_url` | **Keep.** Read the quote yourself: it should say what you asked, in context (a blog post about a feature still counts; a legal page still counts) |
| `not_found` | Every page listed in `pages_read` was read and none mentions it. That is not proof it's absent | **Drop, or hold** if the user thinks it's likely. Look at `pages_read` before deciding (see Judgement calls) |
| `unclear` | The pages hint at it but no single passage states it, or the site couldn't be reached in time (free) | **Send it once more** on its own; if it's still unclear, hold it for a human look |

## 5. Hand it over

1. **The questions you asked**, each in a line, and which criterion each one stands for.
2. **The fits**: a table of company, then for each question the answer, the quoted passage (trimmed to the sentence that matters) and the page link. Fits first.
3. **The rest**: not_found and unclear companies in a short list, each with one line on why (for example "not_found: only the homepage was read").
4. **What it cost**, added up from each call's `credits_charged`.
5. **The next step**: "Want me to find the decision-makers at these companies and get emails that won't bounce?" (the `no-bounce-list` skill).

## Judgement calls

- **A not_found with only one page in `pages_read`.** Only one page was read, so the answer says little about the company. Treat it as unchecked. Don't resend it. Tell the user it wasn't fully checked, and look at the site yourself or ask them.
- **A not_found with six or seven pages read**, including pages on the right topic (an integrations hub for an API question), is a fair "they don't offer this". Drop it.
- **The yes comes from a blog post, a help guide or a legal page.** That's fine: it's the company's own words. Quote it and link it so the user can judge.
- **Every company comes back not_found.** The question is probably asked in words these sites don't use. Rephrase it in their language ("online booking" for "appointment scheduling") and try it on two or three companies first.
- **The user wants a score or ranking.** Give the count of criteria each company passed, with its quotes. Don't invent weights.
- **A criterion the website can't answer** (size, funding, hiring, tools they use): say so and point to the right check. Headcount and industry come with `find_companies`; hiring and funding with the `signal-prospector` skill.

## Example

User: "I sell an SMS reminder API to software companies. Which of these book appointments inside their product, and do they have an API? jane.app, vagaro.com, servicetitan.com, toasttab.com, gusto.com."

Round 1, the strictest criterion: "Do they let their users book appointments inside their product?" on all 5 (up to 17.5 credits; 17.5 charged, 21 seconds).
- jane.app: **yes**, quoting its patient guide ("you can click on the Book Appointment button…", jane.app/guide/booking-an-appointment-online-for-patients).
- servicetitan.com: **yes**, quoting a scheduling webinar recap ("the customer confirms all project information and appointment details… then clicks the 'Finish Booking' button").
- gusto.com: **not_found** after 7 pages: payroll, as expected. Drop.
- toasttab.com and vagaro.com: **not_found**, each with only the homepage read. Treat both as unchecked and hand them to the user for a quick look rather than dropping them.

Round 2, only the two yeses: "Do they publish a public API that developers can use to build integrations?" (up to 7 credits; 7 charged).
- servicetitan.com: **yes**, quoting its API terms ("ServiceTitan offers certain APIs that are generally available to registered and approved developers ('Public APIs')", servicetitan.com/legal/api-terms).
- jane.app: **not_found** after 7 pages, including its integrations hub. A fair no: Jane works through partner integrations.

Hand back: 1 company passes both (ServiceTitan), 1 passes the first only (Jane), 2 to check by hand (Toast, Vagaro), 1 dropped (Gusto). 24.5 credits in all. Offer to find the product and partnerships leads at ServiceTitan with `no-bounce-list`.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Checking each company's website for your must-haves needs Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=account-fit-check, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
