---
name: list-cleanup
description: Clean an existing contact list the user already has, before it gets emailed. Drop duplicates and role addresses for free, re-check every remaining address for deliverability (including catch-all domains), find people who have moved jobs, and hand back the list marked send, hold or remove with a reason on every row and the exact cost. Use this whenever someone already has a CSV, CRM export, old lead list or event list and asks to clean it, verify it, de-dupe it, check for bounces, "is this list safe to send to", "who on this list has left", or is about to import a list into a campaign, even if they don't say "clean".
---

# List cleanup

An old list is the fastest way to burn a sending domain. People change jobs, companies change mail servers, and a list that was clean a year ago now bounces. This skill checks every row before anything is sent and tells the user what to send, what to hold and what to drop, with the reason and the cost. You do the free work (duplicates, role addresses), decide where a deeper check is worth paying for and make the call on each row; Layrcake checks each address against the mail server, including domains that accept everything, and looks up where a person works now.

## Live data: check once, up front

Call Layrcake's `get_me` at the start. If the Layrcake tools aren't available, say so in **one line at the top of your first reply** ("I can de-dupe this now; checking whether the addresses still work needs a free Layrcake connection, steps below"). Then still help: do step 1 (duplicates, role addresses, free mail, malformed rows), say how many rows are left to check and what that will cost (step 2). Then give the connect steps under "Connecting Layrcake" at the end of this file and stop. Never mark an address as safe without checking it: a row that looks fine and bounces costs the user more than a row they held back.

## 1. Do the free work first

Read the whole list (columns like name, company, email) before paying for anything:

- **Duplicates.** Compare emails trimmed and in lower case (`Jason@Acme.com ` and `jason@acme.com` are one row). Keep the first, mark the rest **remove: duplicate of row N**.
- **Role addresses** (info@, sales@, hello@, team@, support@, contact@, admin@, office@, marketing@, billing@, noreply@): a shared inbox, usually filtered or read by nobody in particular, and often reported as spam by whoever reads it. Mark **remove: role address**. Don't pay to check them.
- **Malformed rows** (no @, spaces inside the address, a missing domain): **remove: not a valid address**.
- **Free mail** (gmail.com, outlook.com, yahoo.com and similar) on a B2B list: flag it. It can be checked like any address, but ask the user whether personal addresses belong in a business send.

Say what this step removed, free: "10 rows: 1 duplicate and 1 role address removed, 8 left to check."

## 2. Re-check every remaining address (1.25 credits each)

Say the cost before calling: "Checking 8 addresses: 10 credits." Then call `verify_email` with `email` once per address (add `budget_credits: 1.25` and an `idempotency_key` so a retry doesn't pay twice). Each call takes one to five seconds.

Each answer has a `status` and `is_catch_all_domain`:

| `status` | Mark | Why |
|---|---|---|
| `valid` | **Send** | The mail server confirms the mailbox. |
| `catch_all_valid` | **Send** | The domain accepts every address, so the server can't confirm this one; the second-stage check found this address looks real. |
| `catch_all_unknown` | **Hold** | A domain that accepts everything, and nothing showed this address is real. Send in small numbers at most and watch bounces. |
| `risky` or `unknown` | **Hold** | The check couldn't finish or the server gave mixed signals. Check once more in a day or two (another 1.25 credits). |
| `invalid` | **Remove** | The mailbox doesn't exist. Sending to it is a hard bounce. Step 3 finds out whether the person moved. |

**Catch-all domains.** Many companies set their mail server to accept mail for any address, so a plain server check says "yes" to everything, including addresses that don't exist. That's why `is_catch_all_domain` matters: on those domains, go by the second-stage verdict (`catch_all_valid` or `catch_all_unknown`); the server's acceptance on its own proves nothing. An address can still come back `invalid` on a catch-all domain; when it does, remove it like any other invalid row.

## 3. Find who has moved jobs (5 credits each, only on a match)

An `invalid` address at a person's old company is the most common sign they've left. For the rows that matter (people the user actually wants to reach), call `enrich` with:

- `person: { full_name, linkedin_url }` when the list or the user has the LinkedIn URL; leave the old company out, so the lookup returns where they work now
- `person: { full_name, company_domain }` otherwise (the domain from the listed address). A hit at a different company means they moved; a hit at the same company means they're still there.
- `want: ["profile"]` (add `"work_email"` if the user wants the new address)

It costs 5 credits when it finds the person, nothing when it doesn't. Say the cost first: "Looking up the 2 people whose addresses failed: up to 10 credits."

Read the answer against the row:
- **A different `company_domain`** than the address on the list: they moved. Mark the old row **remove: left [old company], now at [new company]**, and offer the new work email if one came back (it has already been checked; its `email_verdict` is in the answer).
- **The same company**: they haven't moved. If enrich returns the listed address with a deliverable `email_verdict`, mark the row **hold** and check again in a few days.
- **No match** (free): you can't tell. Keep the row as **remove** on the strength of the `invalid` result.

Don't run this on every row: at 5 credits it is four times the price of the address check, and most rows that verify as `valid` haven't moved. Run it on `invalid` rows the user cares about, or on a whole list only when the user asks "who has left?".

## 4. Re-checking costs less than a bounce

Tell the user the trade in numbers:

- **Checking costs 1.25 credits an address** (about one US cent). A 1,000-row list costs 1,250 credits (about $12.50).
- **A bounce costs the domain's reputation.** Keep hard bounces under about 2% of a send; above that, mail providers start sending the domain's mail to spam, and the fix is a new domain plus two to three weeks of warm-up. Old lists often bounce far above that: in the example below, 2 of 8 addresses came back invalid.
- **Lists decay.** People change jobs all the time, so any list not checked in the last three months gets checked again before a send.

## 5. Hand it over

1. **The cleaned list** as a table, send rows first: name, company, email, status from the check, mark (send / hold / remove), reason in a few words ("catch-all domain, address looks real", "duplicate of row 2", "left the company, now at X").
2. **The summary**: "10 rows: 6 send, 1 hold, 3 remove (1 duplicate, 1 role address, 1 invalid)."
3. **What it cost**, added up from each answer's `credits_charged`: checks plus any job-move lookups.
4. **What to do with holds**: re-check date, or send in a small batch and watch bounces.
5. **The next step**: "Want me to find who replaced the people who left, at the same companies?" (the `no-bounce-list` skill finds the right person and a checked email).

Offer a CSV with the mark and reason as extra columns, ready to import.

## Judgement calls

- **A person still at the company with an `invalid` address.** Hold the row and check again in a few days: deleting someone who is still there loses a contact.
- **A whole domain comes back catch-all.** Normal for many companies. Send the `catch_all_valid` rows, hold the `catch_all_unknown` ones, and don't send a large batch to a single catch-all domain at once.
- **A row with a name but a different person's address** (a shared mailbox, an assistant): mark hold and ask the user.
- **The same person twice with two different addresses** (old and new job): keep the one that checks `valid` at their current company; remove the other with "old address".
- **A free-mail address on a business list:** check it if the user wants it kept, and flag it in the reason.
- **A very large list.** Do step 1 on everything (free), then say the full cost before checking (1.25 per address). If the user wants to start smaller, check the rows they'll send to first.
- **A check fails or times out.** Retry that address once with the same `idempotency_key` after 30 seconds (or the `retry_after_seconds` it gives); if it fails again, mark hold with "couldn't check".

## Example

User: "Here's an old list of 10 contacts from an event. Can I send to it?" (columns name, company, email; a test list built from public search results).

Free step: one deliberate duplicate (the same address in different letter case, with the company written differently) and one info@ address removed; 8 left. `verify_email` on the 8 (10 credits): 4 `valid`, 2 `catch_all_valid` on a catch-all domain, and 2 `invalid`. One invalid row was on that same catch-all domain (a revenue operations manager), the other at a footwear brand (a co-founder). `enrich` on the co-founder by name and LinkedIn to see if he'd moved (5 credits): still at the same company, and enrich's own check rated the same address deliverable.

Handed back: **6 send** (4 valid, 2 catch-all with addresses that look real), **1 hold** (the co-founder, re-check in a few days), **3 remove** (the duplicate, the info@ address, the invalid address on the catch-all domain). Cost: 15 credits. Sending the list unchecked risked 2 hard bounces out of 8 addresses.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Checking whether the addresses on your list still work needs live data from Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=list-cleanup, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
