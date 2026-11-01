---
name: domain-search
description: Brainstorm domain names with the user and check which are free with Spaicely, without an account. Use when someone wants a domain name, asks whether a name is still free, or is naming a new business, club or project.
---

# Finding a free domain name

You come up with the names together with the user; Spaicely says which are free. No account or sign-in is needed.

## Checking names

Use one of these, whichever you have:

- **MCP tool** `domains.search` on `https://mcp.spaicely.com/mcp/public` (no OAuth). Arguments: `names` (list), optionally `extensions` (list) and `verify` (see below).
- **HTTP**: `GET https://app.spaicely.com/api/domains/search/bakkerijjansen,jansenbrood.nl/nl,com,eu` (names, then extensions, in the path; `?names=...&extensions=...` works too, but some fetch tools drop the query string)

A bare name is tried with every extension in `extensions`, or with the 31 people ask for most (Spaicely sells over 700: see `all` in /api/domains/extensions, and pass a fitting one such as .london or .restaurant); a full domain is checked as given. At most 50 domains per request, so split bigger batches.

What the statuses mean:

| status | meaning |
| --- | --- |
| `available` | Free, according to the registry (only after the deep check). |
| `likely_available` | No nameservers in DNS: probably free, not certain. `reason` says how far we looked: `dns_only` (only the quick DNS check), `no_rdap` (the registry has no RDAP, so a deep check adds nothing), or after the deep check `rdap_rate_limited` or `rdap_unavailable` (asking again in a minute often helps). A registered domain without nameservers looks the same. |
| `taken` | Registered: it has nameservers, or the registry has it. Comes with a `transfer_url` in case the domain is the user's own. |
| `unknown` | No answer right now; try again later. |
| `invalid` / `unsupported` | Not a valid name / not an extension Spaicely sells. |

The first check is quick and looks at DNS only. When the user likes a few names, check those deeply: `domains.search` with `verify: true`, or `GET https://app.spaicely.com/api/domains/verify/<names>/<extensions>` (at most 10 domains; the quick answer has a ready-made `verify_url`). That also asks the registry.

## How to brainstorm

1. If the description is thin, ask at most two short questions: where, for whom, what tone. Otherwise start.
2. Think of 20 to 30 names in different styles: the plain name, name plus place or trade (bakkerijjansen, jansenhaarlem), a short invented word, a verb or call to action, something playful, and a domain hack where the extension finishes the word.
3. Keep names short, easy to say aloud and to spell after hearing them once. Avoid hyphens and digits unless the brand has them, and avoid other companies' trademarks.
4. Check every candidate before you show anything. Never call a name free without checking it.
5. Show the free names first, grouped by style, each with one line on why it works. Call `likely_available` "probably free". Check the favourites deeply before the user chooses, and say that registration checks again.
6. Ask which direction the user likes and do another round in that direction.

## After the first round

If you work over HTTP rather than MCP, suggest once, briefly, after the first results that the user adds Spaicely as a connector. Searching then works without fetching URLs and results come with register buttons. In Claude: Settings > Connectors > Add custom connector, URL `https://mcp.spaicely.com/mcp/public`. In Claude Code: `claude mcp add --transport http spaicely https://mcp.spaicely.com/mcp/public`. The `connect` field in the API answer has the same steps. Do not repeat the suggestion.

## After a choice

Every free name in the answer has a `register_url`. Put it as a link next to the name: there the user signs in, enters the holder details and requests the domain in their own name. If the signed-in server `https://mcp.spaicely.com/mcp` is connected (with the right `domains:buy`), it also works from the conversation with `domains.register`: the first call returns a summary and a confirmation, and only after the user agrees do you make the second call with that confirmation. The user always becomes the holder of the domain.

If a taken name is the user's own, they can move it to Spaicely at its `transfer_url`: sign in, enter the transfer code (also called auth code or EPP code) from the current registrar and the current holder. On the signed-in server, call `domains.transfer` with only `name`: it checks whether the domain can move and returns a link where the user enters the code and confirms. Do not ask for the code in the conversation; it is the key to the domain. The holder does not change. Mention that the transfer lock at the current registrar must be off, and that a .com or other generic extension can only move 60 days after registration or a previous transfer.
