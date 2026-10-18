---
name: warm-leads-from-post
description: Turn the audience of someone else the user's buyers follow into warm leads. Takes a LinkedIn profile of a public business figure (or a company), finds their recent posts, picks the one with the most engagement, and returns the people who reacted or commented whose titles match the user's buyer, each with what they did on the post. Use this whenever someone asks who engages with an influencer, creator, thought leader or competitor, wants to "mine" or "scrape" the likes and comments on someone else's post, wants warm leads from someone else's audience, or asks "who is reacting to X's posts", even if they don't say "engagers". For the user's own posts or their own company page, use my-post-engagers.
---

# Warm leads from someone else's post

The people who react to a post by someone your buyers follow have just shown you two things: they care about the topic, and they're active right now. This skill finds that person's recent posts, picks the one that drew the most people, and returns the engagers whose titles match the user's buyer, each with what they did. You choose the source, the post and the titles, and judge who's a real buyer; Layrcake reads the posts and the people who engaged with them.

This skill is for someone else's posts. For the user's own posts or their company page, use `my-post-engagers`.

## Live data: check once, up front

Call Layrcake's `get_me`. If the Layrcake tools aren't available, still help: agree who the user's buyers follow, which title words describe the buyer (step 1), and what a sample would cost. Then give the connect steps under "Connecting Layrcake" at the end of this file. Never list "people who engaged" you didn't read: a made-up engager is worse than none.

## 1. Get the inputs

- **The source**: a LinkedIn profile URL of someone the user's buyers follow (an investor, an author, a well-known operator), or a company domain (a competitor, a partner, a big name in the space). Pick a source whose audience is the buyer: a founder-investor's audience is mostly founders and investors; a sales leader's audience is sales leaders.
- **The buyer's title words**, 4 to 8 of them, the way people write them in a LinkedIn headline: "VP Sales", "VP of Sales", "Head of Sales", "Chief Revenue Officer", "CRO", "Sales Director". Exact phrases keep the list clean; one broad word ("Sales", "Growth") lets in reps, students and vendors, and each one kept costs 0.5 credits.

## 2. Find the recent posts

Call `find_posts` with:
- `author: { type: "profile_url", profile_url }` for a person, or `company_domain` for a company page
- `within_days: 30`, `limit: 5` (or `limit: 10` to compare a whole month from someone who posts often)
- `include: ["author_details"]`, so each post says whether it's a repost

It costs a 1-credit base even when nothing comes back, plus 0.5 credits per post returned: at most 3.5 credits for 5 posts, 6 for 10. Say that before calling. For someone known only by name, call `enrich` with `person: { full_name, company_domain }` first and use the `linkedin_url` it returns; a name-only author is refused before any charge.

## 3. Pick the post

Each post carries `engagement` (`likes`, `comments`, `shares`). Pick the one with the most likes plus comments, with three rules:
- **Skip reposts** (`is_repost: true`): the people on them engaged with someone else's post.
- **Prefer a post on the buyer's topic** (a post about sales teams for a sales-tool seller) over a personal or event post of the same size.
- **Prefer a post a few days old.** Posts come newest first, and a post from this morning is still collecting reactions. If all five are under a day old, run step 2 again with `limit: 10` (up to 6 credits) for older ones.

Say which post you picked and why, in a line, with its link.

## 4. Sample its engagers, filtered to the buyer

Call `find_posts` with:
- `engagers_of: [<post URL>]` (send no `query`, `platforms`, `intent` or `audience`)
- `author_filter`: the buyer's title words from step 1
- `per_post: 15`
- an `idempotency_key`

`per_post` applies to each source on the post (reactions and comments), so 15 reads up to about 30 people. Each person read costs 0.4 credits, and each person kept costs 0.5 more. Say the most it can cost before calling: "Sampling up to 30 people on this post for sales-leader titles: about 12 credits to read them, plus 0.5 for each match." The read fee applies even when nobody matches.

Read the answer for each post:
- **`people`**: each match with `headline`, `activities` (a `reaction` with its type, or a `comment` with its `text`) and `person` (name and LinkedIn URL, ready for `enrich`).
- **`read`** (people read), **`filtered_out`**, **`engaged_total`** and **`more_available`**: how many people the post has and how many the sample covered.
- **`matched_by`**: `literal` when a headline contains one of your words; `judged` when it clearly means one in other words (a CCO for "chief revenue officer").
- **`full_read`**: the quote to read everyone, as `budget_credits` and `eta_seconds`.

## 5. Offer the full read only when the user wants everyone

The sample is the answer for most users. Offer `read_all` when the post has many more people than you read (`engaged_total` well above `read`) and the user wants every match. Explain it with the quote: "The sample read 30 of the 190 people on this post. Reading them all would cost up to <budget_credits> credits and take about <eta_seconds, in minutes>. Want me to?" Never run it without a yes.

When they say yes, call `find_posts` with the same `engagers_of` and `author_filter`, `read_all: true`, `budget_credits` set to at least the `full_read` quote (a little above it, within the user's cap, leaves room), and a new `idempotency_key`. It runs in the background at the same prices and never charges above `budget_credits`. Poll `get_operation` with the `operation_id`; people come back 25 at a time with `next_cursor`. If the budget ran short, each post says how many were `not_read` or `withheld` with `limited_by: budget`.

When `engaged_total` is only a few people above `read`, a full read finds little. Sample the next-best post instead.

## 6. Hand it over

1. **The post**: who posted it, a line on what it says, its date and link, and its engagement.
2. **The matching people**, strongest first: name, headline, what they did (a comment with a short quote beats a reaction), and their LinkedIn. Commenters first.
3. **The sample in numbers**: "1 of 25 people matched; the post has 36 in all."
4. **What it cost**, from each answer's `credits_charged`.
5. **The next step**: "Want me to get a checked work email for each match and write a first line that picks up the post's topic?" (the `no-bounce-list` skill; `enrich` with the `linkedin_url` from `person`). If the sample was thin, offer the full read with its quote, or a different source.

## Judgement calls

- **Few or no matches.** That tells you about the source's audience. Say so plainly, then offer two ways on: a source closer to the buyer (a sales leader's posts rather than an investor's), or wider title words. Don't keep sampling the same person's posts with the same words.
- **A match works for the poster.** A headline naming the poster's own company means staff; leave them out.
- **Timing.** Time a reaction by the post's date: describe it as "reacted to the post". A comment carries its own.
- **The same person on two posts.** That's a stronger lead: say so.
- **Posts name people.** Use the topic of the post as context for a relevant message. Never write "I saw you liked X's post": open on the topic they cared about instead.

## Example

User: "Heads of sales follow a sales-tech CEO who posts a lot. Find everyone who engaged with his most popular post from the last 30 days whose headline says head of sales, VP sales, sales director, CRO or VP revenue. Read all of them. Budget 280 credits." (A run on 3 October 2026.)

`find_posts` with his profile, `within_days: 30` and `limit: 10` (4.5 credits): seven posts. The top original post had 563 engagements, which is 548 unique people once someone who both reacted and commented is counted once. Sample it with `engagers_of` and `author_filter` set to those titles (4.1 credits): the answer's `full_read` quotes 254 credits and about three minutes to read everyone. The user asked for all of them within 280, so send `read_all: true` with `budget_credits: 260`, a little above the quote, and poll `get_operation`. When it finishes: 548 people read, 26 sales leaders kept, 232.2 credits (548 × 0.4 + 26 × 0.5). Six of the 26 had titles the filter words didn't name (four Chief Commercial Officers) and were kept by the title check, each marked so the user can see why. Hand back the 26 with what each did on the post, and offer to find their work emails. Total: 240.8 credits (4.5 + 4.1 + 232.2).

A smaller run shows the other side: on a well-known SaaS investor's post, 25 people sampled gave 1 sales leader (10.5 credits). His audience is founders and investors, so switch to a source whose followers are the buyers rather than reading more of his engagers.

## Connecting Layrcake

Give these steps once when the tools aren't available. If you know which app you're running in, show only that app's line; otherwise show both. Don't repeat the free-credits line elsewhere in your reply.

> Reading who engaged with a post needs live data from Layrcake: free to start with 300 credits (a credit is one US cent), about 30 seconds. Sign up at https://layrcake.dev/connect?utm_source=plays&utm_campaign=warm-leads-from-post, then connect it here:
> - **Claude Code:** exit, run `claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp`, then `claude --continue` to come back to this conversation, and `/mcp` to sign in. You only do this once for all Layrcake plays.
> - **Claude.ai:** Settings → Connectors → Add custom connector. Name it Layrcake and paste this as the URL: `https://mcp.layrcake.dev/mcp` (not the sign-up link). Sign in when asked, then switch Layrcake on in this chat's tools menu. On a Team or Enterprise plan, an owner adds the connector first.
>
> When it's connected, say "go" and I'll carry on from here.
