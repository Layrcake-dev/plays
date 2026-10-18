# Layrcake plays

Single-purpose go-to-market skills for Claude. Each play does one job well, runs on live data through [Layrcake](https://layrcake.dev), and was tested on a real run before it was published. Install only the ones you need.

Looking for a complete starter kit instead? See [Claude SDR](https://github.com/Layrcake-dev/claude-sdr-skills): six skills that take you from your website to a sent, approved campaign.

## The plays

| Play | Ask Claude something like | What you get | Credits in our test run |
|---|---|---|---|
| **account-fit-check** | "Which of these 20 companies offer an API?" | Yes, not found or unclear for each company, every yes with the quoted passage from its own site | 24.5 for 5 companies, two questions |
| **lookalike-accounts** | "Find companies like our best customer, jane.app" | Similar companies with why each matches, poor fits dropped | 29.5 for 2 lookalikes and a fit check |
| **hiring-manager-finder** | "Who is the hiring manager for these SDR roles?" | The person who owns each open role, a checked work email and an opener from the job post | 30 |
| **champion-tracker** | "Have any of last year's champions changed jobs?" | Who moved, where to, their new work email and a congrats opener | 26.5 for 5 people |
| **warm-leads-from-post** | "Find the sales leaders who engaged with this CEO's latest post" | Buyers in someone else's audience, with what each did on the post | 8.6 for a sample; 240.8 to read all 548 people on a busy post |
| **my-post-engagers** | "Who engaged with our company's posts this week?" | A warm list from your own posts, staff and vendors removed, an opener each | 18.9 for two posts |
| **call-brief** | "I'm meeting Gong's sales team tomorrow. Prep me." | A one-page brief: the company, dated recent moves with sources, who you're meeting, three sharp questions | 14 |
| **who-to-contact-first** | "Here are 6 people. Who should I email first this week?" | Your list ranked by recent public activity, each with a first line; people with nothing recent cost nothing | 12 for 6 people |
| **inbound-triage** | "Sort this week's demo requests" | Call today, nurture or ignore for each sign-up, with a one-line reason | 39.5 for 6 sign-ups |
| **list-cleanup** | "Clean this list before we send" | Send, hold or remove for every row, with reasons; duplicates and role addresses removed for free | 15 for 10 rows |

A credit is $0.01. Each play states what a step will cost before it runs it, lookups that find nothing are free, and nothing is ever sent from a play: anything that sends email waits for a person to approve it.

## Install

**Claude Code:**

```
/plugin marketplace add Layrcake-dev/plays
/plugin install call-brief@layrcake-plays
```

Install any play by its name the same way. Then connect Layrcake once for all of them:

```
claude mcp add --transport http --scope user layrcake https://mcp.layrcake.dev/mcp
```

Open `/mcp`, choose layrcake and sign in.

**Claude.ai:** download a play's folder from `plugins/<play>/skills/`, zip it and upload it under Settings → Capabilities → Skills. Then add Layrcake under Settings → Connectors → Add custom connector, with the URL `https://mcp.layrcake.dev/mcp`.

**Cursor, Codex and other agents:** copy `plugins/<play>/skills/<play>/` into your agent's skills folder and add the MCP server `https://mcp.layrcake.dev/mcp`.

No Layrcake account yet? Sign up free at [layrcake.dev/connect](https://layrcake.dev/connect?utm_source=plays&utm_campaign=readme): 300 credits, no card.

## How the plays are built

Every play follows the [Agent Outbound Standard](https://layrcake.dev/guides/agent-outbound-standard): it works inside a budget, never invents people or addresses, checks addresses before anyone sends, and stops at a preview for anything that leaves. Agents can read [layrcake.dev/agents.md](https://layrcake.dev/agents.md) for the full rules and tool list.

## License

MIT. Use them, change them, share them.
