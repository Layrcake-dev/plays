---
name: my-post-engagers
description: Turn the people who engaged with the user's own recent LinkedIn posts, or their own company page's posts, into a warm list. Finds the recent posts, reads who reacted and commented, keeps the people who match the user's ideal customer, leaves out the company's own staff, vendors, recruiters and job seekers, and hands back each person with what they did and a suggested opener that picks up the post naturally. Use this whenever someone asks who engaged with their posts, who liked or commented on their company page, wants warm leads from their own LinkedIn, wants to follow up with people who engaged with their own posts, or asks "who's reacting to our content", even if they don't say "engagers". For someone else's posts or audience, use warm-leads-from-post.
---

# My post engagers

Someone who reacted to the user's post this week already knows the name. That makes them a warm lead, as long as you write to the right ones and don't make it weird. This skill reads the user's recent posts, keeps the engagers who look like buyers, removes the people who aren't (colleagues, vendors, recruiters, job seekers) and drafts an opener for each that feels like a natural follow-up. You choose the posts and judge each person; Layrcake reads the posts and the people who engaged.

This skill is for the user's own posts and their company page. For a post by someone else (a competitor, a peer, an influencer), use `warm-leads-from-post`.

## Live data: check once, up front

Call Layrcake's `get_me`. If the Layrcake tools aren't available, still help: agree the buyer's title words (step 1), how to tell staff and vendors apart, and what a sample would cost. Then give the connect steps under "Connecting Layrcake" at the end of this file. Never list engagers you didn't read.

## 1. Get the inputs

- **Whose posts**: the user's LinkedIn profile URL, or their company's domain for the company page. Ask which; a founder's own posts often draw more buyers than the company page.
- **The ideal customer in title words**, up to 10, the way they appear in headlines: "Founder", "CEO", "Head of Sales", "VP Sales", "Head of Growth", "RevOps". Exact phrases keep the list clean.
- **The company's own name**, so you can spot staff (their headlines name it).

## 2. Find the recent posts

Call `find_posts` with:
- `author: { type: "profile_url", profile_url }` for the user, or `company_domain` for the company page
- `within_days: 30`, `limit: 5`
- `include: ["author_details"]`, so each post says whether it's a repost

It costs a 1-credit base even when nothing comes back, plus 0.5 credits per post returned: at most 3.5 credits. Say that before calling.

## 3. Pick one or two posts

Each post carries `engagement` (`likes`, `comments`, `shares`). Skip reposts (`is_repost: true`): those people engaged with someone else. Then pick:
- **The post with the most engagement**, and
- **The post closest to what the user sells** (a workshop on outbound for a sales tool), if that's a different post.

Culture and team posts (an office day, a race, a party) draw lots of staff and friends. Product, event and how-to posts draw buyers. Say which posts you picked and why, in a line.

## 4. Read who engaged, filtered to the ideal customer

Call `find_posts` with:
- `engagers_of`: the one or two post URLs (send no `query`, `platforms`, `intent` or `audience`)
- `author_filter`: the title words from step 1
- `per_post: 8`
- an `idempotency_key`

`per_post` applies to each source on each post (reactions and comments), so two posts at 8 read up to about 32 people. Each person read costs 0.4 credits and each person kept costs 0.5 more, so say the most it can cost: "Reading up to 32 people on two posts: about 13 credits, plus 0.5 for each match, so under 30." The read fee applies even when nobody matches.

Each match comes with `headline`, `activities` (a `reaction`, or a `comment` with its `text`) and `person` (name and LinkedIn URL, ready for `enrich`). Each post reports `read`, `engaged_total` and `more_available`, and the answer carries a `full_read` quote (`budget_credits`, `eta_seconds`) for reading everyone.

## 5. Remove the people who aren't buyers

The title filter keeps anyone whose headline has the words, so it keeps some people you don't want. Go through each match and leave out:
- **Staff**: a headline naming the company ("GTM @company", "Founder @ company"). Founders and executives of the user's own company match "Founder" and "CEO" too.
- **Vendors**: someone selling to the user rather than buying from them (a recruiter, a design agency pitching the user). An agency or freelancer who would *use* what the user sells is a buyer; keep them and say so.
- **Job seekers**: "open to work", "your next GTM engineer?", "looking for my next role".
- **Recruiters**, whatever their title.

Say how many you removed and why, in a line.

## 6. Draft an opener per person

Write one or two sentences each that follow on from the post without making the person feel watched:
- **They commented**: reply to what they said, like a continuation of the thread. "Glad the Strava route landed. Is your team doing any field events like that, or mostly online?"
- **They only reacted**: open on the post's topic, never the reaction. "We've been running small in-person workshops on outbound and they're beating webinars for us. Is that something your team is trying?" Never write "I saw you liked our post".
- Tie the topic to their role in a few words, and end with an easy question. No pitch in the first message.

## 7. Hand it over

1. **The posts read**: date, a line on each, link, engagement.
2. **The warm list**, commenters first: name, headline, what they did (a short quote for comments), LinkedIn, and the suggested opener. Flag agencies and freelancers, and anyone whose headline names no company.
3. **Who you left out**, by reason ("3 staff, 1 job seeker").
4. **The sample in numbers**: "10 of 26 people read matched; the posts have 240 people in all."
5. **What it cost**, from each answer's `credits_charged`.
6. **The next step**: "Want me to get checked work emails for these people?" (the `no-bounce-list` skill; `enrich` with the `linkedin_url` from `person`). When the posts have many more people than the sample read, offer the full read with its quote: "Reading all 240 people would cost up to 108 credits and take about a minute and a half. Want me to?"

## The full read, when the user wants everyone

Run it only on a yes. Call `find_posts` with the same `engagers_of` and `author_filter`, `read_all: true`, `budget_credits` set to the `full_read` quote, and a new `idempotency_key`. It runs in the background (about 2 minutes per 500 people) at the same prices and never charges above `budget_credits`. Poll `get_operation` with the `operation_id`; people come back 25 at a time with `next_cursor`. If the budget ran short, each post says how many were `not_read` or `withheld` with `limited_by: budget`. Then repeat steps 5 to 7.

## Judgement calls

- **Mostly staff.** Big team posts draw the team. Say so, and read a product or event post next time.
- **A headline with no company** ("GTM Manager | AI & Automation"): keep them, flag it, and let `enrich` with their `linkedin_url` find where they work before anyone writes.
- **Timing.** Date a reaction by the post's date; a comment carries its own.
- **The same person on two posts.** That's the strongest lead on the list: put them first.
- **A thin result.** Few matches means the posts reached few buyers. Suggest wider title words, the founder's own posts, or the full read.
- **Never quote someone's activity back as a list.** One natural reference to the post's topic is enough.

## Example

Treating a public sales-engagement company's page as "the user's": `find_posts` with `company_domain`, `limit: 5` (3.5 credits) returned four company posts and one repost from the last 10 days. Read: the most engaged post (a team running a route through San Francisco during a tech week, 177 reactions, 15 comments) and a GTM operators' coffee event (44 reactions, 4 comments), with `author_filter: ["Founder", "CEO", "Head of Sales", "VP Sales", "Sales Director", "Head of Growth", "Growth", "SDR Manager", "RevOps", "GTM"]` and `per_post: 8` (15.4 credits). 26 people read, 10 matched. Left out: 3 staff (one the company's own founder), 1 job seeker. Warm list of 6, including a growth operations director at a European fintech who reacted, a GTM and RevOps consultant who praised the post in a comment (flagged as an agency, a likely user), and a founder of a small startup who reacted. Each got an opener. The full read was quoted at 108 credits for the 240 people on both posts, and offered to the user. Total: 18.9 credits.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Reading who engaged with your posts needs live data from Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=my-post-engagers, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
