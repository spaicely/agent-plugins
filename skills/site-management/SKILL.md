---
name: site-management
description: Run a site on Spaicely after it is built - upload and publish versions, roll back, change part of a page, put the site on its domain, manage DNS and mail forwarding, people and roles, approvals, handovers, the hosting plan and domain renewals. Use whenever you work on an existing Spaicely site or account, not only when building one.
---

# Running a site on Spaicely

How a site gets online and stays there. For what a good site contains, see the skill `site-building`; for finding a name, `domain-search`. Every tool, parameter and HTTP endpoint is listed at https://spaicely.com/developers/ and, for AIs, in https://spaicely.com/llms-full.txt.

## Start

- Call `account.whoami` first. It says who you act for, your scopes, and per site your role and what you may do. With no site yet, `sites.create` makes one; ask the person for the name.
- Your rights are never wider than the person's role. When a tool refuses, say what is missing (role, scope or plan) instead of trying another way.
- A tool named here that is not in your list means your list is old: ask the person to reconnect the Spaicely connector (in claude.ai: Settings > Connectors; in Claude Code: `/mcp`).
- Ask the person before anything that changes what visitors see, costs money or cannot be undone. Some tools ask for approval themselves in Claude Code.
- **What you may do yourself** is the person's choice, per kind of action (`ai_permissions` in `account.whoami`): `self`, `ask` or `never`. By default buying, people, mail, the registry, deleting and billing details are `ask`. Such a call answers with `approval_required` and a `url`: give the person that link and do not repeat the call. They approve in Spaicely; we then run the call exactly as you sent it, and `approvals.status` with the `approval` id gives its answer. A `never` kind is refused; only the person can change that, at https://app.spaicely.com/ai.

## Putting a version online

A version is always the whole site, never a set of changes. Nothing is visible until it is published.

1. **Upload**, one of two ways:
   - `sites.deploy` with every file: `path` from the site root, `sha256` in hex and `size` in bytes. For an image that is already on the web, pass `source_url` instead (https, at most 20 per call). Cannot compute a sha256 (no code execution)? Pass `content` instead, with `encoding` `utf8` for text or `base64` for images, at most 2 MB of content per call; such a file is never `missing`. For unchanged files of a live site, copy `sha256` and `size` from `sites.files.list`. The answer lists `missing`; send exactly those with `sites.files.put` (at most 50 files and 2 MB per call, text as `utf8`, the rest as `base64`).
   - With the site as a folder on disk: `sites.upload_url`, then `tar -czf site.tar.gz -C <folder> . && curl -T site.tar.gz <url>`. The URL works once, for 10 minutes; `index.html` must be at the root of the archive.
2. **Publish** with `sites.publish` and the version number, after the person agreed. Visitors switch over at once, never to a mix. It takes up to half a minute; `sites.versions` shows when it is live.
3. **Read the report.** `sites.versions` gives per version the bytes saved and `hints`, each with a `fix`. Fix what applies and publish again.

Good to know:

- We keep the files exactly as uploaded; visitors get an optimised copy (minified HTML, CSS and JavaScript, recompressed images up to 2560 pixels).
- Everything under `_private/` stays off the CDN; only a download form hands it out. `_redirects` becomes redirects at the CDN and is not served itself (see `site-building`).
- Did someone else publish after your version started? Then your changes are merged onto theirs and that goes live. A conflict comes back per path: read the live file with `sites.files.get`, redo your change on it, and upload again.
- **Back to the previous version**: `sites.publish` with `rollback: true`. Any older version: `sites.publish` with its number.
- Before you change a page someone else made, read it with `sites.files.get` (the live version, or `deployment` for another one) and keep what is there. `sites.files.list` shows the files, with `prefix` for one folder.

## Changing one part of a page

A page can mark blocks that a role may change on its own:

```html
<!-- spaicely:edit news -->
<p>Open on Sunday from 1 November.</p>
<!-- /spaicely:edit -->

<!-- spaicely:append agenda -->
<li>12 November: tasting</li>
<!-- /spaicely:append -->
```

- `edit` may be changed entirely; `append` only takes additions before or after what is there. Names: lower case letters, digits, `-` and `_`. Blocks do not nest.
- `sites.files.list` lists the blocks per page. `sites.region.update` with `path`, `region` and `content` makes a new version that differs from live in that block only (`mode` replace, prepend or append; `index` when a name occurs more than once). Then publish.
- Build these markers in where the person expects others to keep things up to date (news, opening hours, agenda), so an editor never needs the whole page.

## People and roles

- **On the whole account**: `account.members`. Owners handle payment, domains and people; members work on all sites. Invite by e-mail (the link works 7 days).
- **On one site**: `access.list` shows who has which role; `access.update` gives, changes or (`role: "none"`) removes one. Only the owner. If the person has no login yet, send an invitation with `access.invite`: the email explains the steps and a newcomer only enters a name. With `kind: "payer"` you ask someone (management, say) to pay for the hosting; after their first payment their organisation becomes the payer.
- Roles belong to the account and hold for all its sites; `access.roles` shows them as a matrix. By default: `admin` (everything but owner matters), `maintainer` (all files including SEO, forms, shop and DNS), `designer` (all files except products), `editor` (texts, media, products), `viewer` (reading). The owner changes them or adds roles of their own, also with `access.roles.update` (ask first). SEO also covers the `<head>` of every page: someone with only SEO changes the head but not the text. `can_publish` lets one person publish; `rules` narrows one person's files further.
- Rules limit a designer or editor to actions on paths, optionally to marked blocks: `[{"actions": ["update"], "paths": ["blog/**", "index.html"], "regions": ["news"]}]`.
- **Approval** (plans Pro and Shop): someone who may change files but not publish calls `sites.review.request` with a `note`. Whoever may publish gets a mail; approving is `sites.publish`, sending back is `sites.review.reject` with a note. On plan Site the person asks someone who may publish.
- **Four eyes** (Pro and Shop): under the site's People tab the owner picks the roles it holds for, possibly the owners themselves too. Whoever has such a role does not put their own version live. `sites.publish` then refuses the maker; call `sites.review.request`, someone else who may publish puts it live. Bringing back an earlier version is allowed.
- **Suggestions**: when the person wants to hear what the others on the site think first, call `sites.suggest` after uploading, with the version number, a short `title` and a `note`. Everyone on the site gets a mail with their own link to a preview on the site's own domain (`/_preview/<code>/`, not indexed), with a bar at the bottom to vote (go ahead, almost, rather not) and comment. `preview_url` in the answer is the person's own link; do not share it. `sites.suggestions` shows votes and comments; `sites.suggestions.respond` votes or comments for the person, only with what they want to say. The votes advise: whoever may publish decides with `sites.suggestions.decide` (`accept` puts the version live, `decline` with a note).

## A domain for the site

- Registering or moving a domain: see `domain-search` (`domains.register`, `domains.transfer`; a partner proposes one to the customer with `domains.propose`). The customer is always the holder.
- **Connecting**: `sites.domains.connect` with the domain itself, no www. Spaicely reads the records the domain has now; call again after about half a minute for `records_to_confirm`. Show them to the person: kept records (mail, verifications) come along, records with reason `site` are replaced, reason `certificate` is a CAA record that would block the certificate. Then call with `confirm: true` (`skip` with the indexes to leave out).
- The person then sets the nameservers at their registrar to the two in the answer (`kiki.bunny.net` and `coco.bunny.net`). Domains registered at Spaicely have them already. DNSSEC at the old provider must be off first. We check every five minutes, then fetch the certificate and mail the person; `www` sends visitors on to the domain itself.
- `domains.status` shows where it stands: records to confirm, nameservers, DNSSEC, certificate, and the records we serve.
- **DNS records**: `domains.dns.update` adds or deletes A, AAAA, CNAME, TXT, MX, SRV and CAA (`name` relative to the domain, `@` for the domain itself; delete by id from `domains.status`). Where the domain and www point stays with the site. A domain of the account on no site gets Spaicely DNS this way too.
- Mail signing keys (DKIM) of an unusual service can be missed when connecting: the person sends one mail from an address on the domain to `dkim@spaicely.com` with the code from `dkim_mail` as the subject.
- `sites.domains.disconnect` stops the site answering on the domain; mail records stay.

## Mail forwarding

Mail to `info@<domain>` can go on to a mailbox the person already has, such as their Gmail. Answering as `info@` is not possible yet.

1. `mail.forward.enable` on a domain whose DNS is at Spaicely, with or without a site. Mail belongs to the domain: it keeps working when the site stops or the domain leaves the site. It sets MX, verification, DKIM, SPF and DMARC. If the domain already receives mail elsewhere, that stops: the call refuses without `replace_existing_mail: true`, so ask first.
2. `mail.forward.add` with `address` (`info`, or `*` for every address without one of its own) and up to 5 `destinations`. Each new destination gets a confirmation mail; nothing goes there before it is clicked.
3. `mail.forward.status` shows the addresses, the state of each destination and mail held back as possible spam in the last 30 days; `mail.forward.release` sends one on after all, only when the person recognises it.

Each address costs EUR 1.50 a month excl. VAT, a year in advance, on one mail subscription per company for every address on its domains, apart from the hosting plan; at most 10 addresses per domain. When that subscription is not running yet, `mail.forward.add` refuses with a link where an owner or someone with the Finance role starts it; pass that link on. A site that pays nothing (such as a test site of Spaicely's) pays nothing for its domains. `mail.forward.remove` and `mail.forward.disable` stop it.

## Mailboxes

A domain can also have its own mailboxes at Spaicely, opened with IMAP in any mail program. A domain has forwarding or mailboxes, not both. Sending works too: through `smtp.spaicely.net`, port 465, with the same password; `mail.mailboxes.status` gives the settings. Each mailbox sends at most 100 messages an hour and 500 a day, so no newsletters or bulk mail.

1. `mail.mailboxes.enable` on a domain whose DNS is at Spaicely. It sets MX, SPF, DKIM, SRV and DMARC. When the domain receives mail elsewhere the call refuses without `replace_existing_mail: true`; when it has forwarding, without `replace_forwarding: true`, and then the forwarding addresses stop. Ask first.
2. `mail.mailboxes.add` with `address` and optionally `name`. Each mailbox holds 10 GB and costs EUR 3.50 a month excl. VAT on the mail subscription, a year in advance; at most 50 per domain.
3. The person makes a password for each device on the `url` in the answer. You never see that password; do not ask for it either. Thunderbird and Outlook find the servers themselves from the address; on an iPhone, iPad or Mac the password page offers a file that sets up the mailbox.
   If the mailbox is for someone else, `mail.mailboxes.invite` (`address`, `email`, `language`) mails them a link to choose a password themselves; they need no Spaicely account. They then get the setup file for Apple and a guide as a PDF. `guide_url` in `mail.mailboxes.status` is that guide; it holds no password, so you may pass the link on. Ask the person first.
4. `mail.mailboxes.status` gives the mailboxes, the settings for a mail program and whether MX, SPF, DKIM and DMARC are right. It also compares the MX the world sees. `warnings` says what needs doing: `records` (our zone is wrong: `mail.mailboxes.repair` puts it right; say first what goes, because an MX of another party is removed), `nameservers` (the domain does not use our DNS: `domains.nameservers` for a domain registered here, otherwise the person changes them at their registrar) or `waiting` (the zone is right, the world is behind: wait a little).

`mail.spam` switches antispam for the domain. On, what looks like spam goes to the Junk Mail folder; off, everything lands in the inbox. `mail.mailboxes.remove` and `mail.mailboxes.disable` delete the mail for good: name the mailboxes and ask first. Without a paid mail subscription addresses and mailboxes go after 14 days, after warnings by mail.

## Plan, billing and renewals

- **Hosting**: `sites.subscription` shows plan, status and renewal date. With `plan` (`site`, `pro` or `shop`; EUR 9, 19 and 39 a month excl. VAT, paid a year in advance) it starts hosting, giving a `checkout_url` for the person, or switches plan at once. Without a plan a new site is removed after 14 days. Card, invoices and cancelling are at `portal_url`. Only owners of the paying account.
- **Billing details**: `account.billing` (business or not, VAT number, address). Every purchase needs them; a VAT number is checked with VIES. `account.invoices` lists our invoices with a link each.
- **Consumers** (an account that is not a business) must agree that an order starts at once, which limits their right of withdrawal. Pass `start_now: true` only when they said so themselves.
- **Domains**: renewal is automatic by default, charged 14 days before expiry to the card or mandate the domain was paid with (`domains.auto_renew`; without a saved method the answer has a `setup_url`). `domains.renew` gives a payment link to renew now.
- Also for owners of the holder account: `domains.holder` (holder details; another name or organisation goes through our staff), `domains.lock` (transfer lock), `domains.nameservers` (`ours: true`, or others for a domain on no site) and `domains.transfer_code` (the code goes by mail to the person and the holder, never to you).

## Partners

- A partner (a web designer or agency) builds on its own partner account, or works on the customer's site with a role.
- `sites.handover` mails the customer a link (14 days) to take a finished site onto their own account; `role` is what the partner keeps, `payer` who pays from then on. `sites.handover.revoke` takes it back before acceptance.
- `domains.propose` proposes a domain for a customer's site; the customer checks the holder details and agrees by mail first.

## Settings at the edge

`sites.settings` (owner or admin) shows and changes what the CDN does with the site; changes work in about half a minute.

- `maintenance: true` shows every page as a maintenance page with status 503, the site's own `maintenance.html` if it has one, else ours with `maintenance_message`. `false` brings the site back; files and versions stay.
- `protect: {path, username, password}` puts a folder (or `/` for the whole site) behind a name and password; `unprotect: "/folder/"` takes it away. Ask the person for the password, never make one up.
- `access_mode` `block` keeps out `access_countries` (two-letter codes) and `access_ips` (addresses or ranges such as 198.51.100.0/24); `allow` lets only them in; `open` is everyone.
- `hotlink_protection: true` stops other sites from embedding the site's images, video and documents.
- `prefer_www: true` makes www the main address; the bare domain then sends visitors on, and the other way round with `false`.
- `sites.cache.purge` empties the cache for up to 20 paths (`/img/*` for a folder). Publishing already empties everything, so this is rarely needed.
- Redirects, rewrites and headers are not settings but files of the site: `_redirects` and `_headers` (see the site-building skill).

## Keeping an eye on it

- `sites.stats` gives requests, bytes, `not_found` and CDN regions per day, over 7, 30 or 90 days, without cookies. Many `not_found` means broken links: find them and add a redirect or fix the link.
- `scripts.list` shows the state of every form; `shop.orders` the orders (`filter: "to_ship"` for what still has to go out), and `shop.orders.ship` marks one as sent with track-and-trace and mails the buyer.
- Deleting a site (`sites.delete`, owner only, with the name the person typed) takes every version, form and the shop offline for good. The domains stay theirs.

## Without MCP

The Spaicely CLI does the same from a terminal or CI: `spaicely login`, `spaicely deploy` (diff with live, upload, optionally publish), `spaicely publish`, `spaicely rollback`, `spaicely versions`, `spaicely domains ...`, `spaicely dns ...`, and `spaicely call <tool>` for every other tool. Install and examples: https://spaicely.com/developers/#command-line.
