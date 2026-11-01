---
name: site-building
description: Build a static website for Spaicely that is easy to find, share and use, with sitemap.xml, robots.txt, llms.txt, link previews for social media, structured data and icons. Use whenever you build or rework a site that will be hosted on Spaicely.
---

# Building a site that people and machines can find

Spaicely hosts what you upload, as it is. Whatever a good site needs beyond its pages, you make yourself. This list is what we expect of every site; after each publish, `sites.versions` shows a report whose `hints` say what is still missing, each with a `fix`.

## Ask first

- **The domain.** Sitemaps, canonical links and link previews need full addresses such as `https://bakkerijjansen.nl/over/`. Use the domain connected to the site (`account.whoami`, `domains.status`). No domain yet? Ask which it will be, and use that.
- **Languages.** One language, or several in folders (`/`, `/en/`, `/de/`).
- **The facts.** Name, what the business does, address, phone, opening hours, social media accounts. They go into the pages and into the structured data. Never make up an address, opening hours or reviews.

## Files at the root

| File | What it is for |
| --- | --- |
| `index.html` | The home page. Every other page is a folder with its own `index.html` (`/over/index.html`), linked as `/over/`. |
| `404.html` | For addresses that do not exist, in the style and language of the site, with a link home. |
| `robots.txt` | Lets everyone in and names the sitemap. |
| `sitemap.xml` | Every page that should be found, by full URL. |
| `llms.txt` | A short summary of the site in Markdown for AIs, following https://llmstxt.org. |
| `favicon.ico`, `icon.svg`, `apple-touch-icon.png` | The icon in tabs, bookmarks and on phones (apple-touch-icon is 180x180). |
| `site.webmanifest` | Name, colours and icons (192 and 512 px) when someone adds the site to their home screen. |
| `og.jpg` | The picture for link previews, 1200x630, under 300 KB. A page can have its own. |

`robots.txt`:

```
User-agent: *
Allow: /

Sitemap: https://bakkerijjansen.nl/sitemap.xml
```

`sitemap.xml`, with `lastmod` the date the page last really changed:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url><loc>https://bakkerijjansen.nl/</loc><lastmod>2026-10-26</lastmod></url>
  <url><loc>https://bakkerijjansen.nl/over/</loc><lastmod>2026-10-26</lastmod></url>
</urlset>
```

Leave out `404.html` and pages marked `noindex`. Update the sitemap whenever you add or remove a page.

`llms.txt`:

```markdown
# Bakkerij Jansen

> Bakery in Haarlem since 1952: bread, pastry and cakes to order. Open Tuesday to Saturday.

- [About us](https://bakkerijjansen.nl/over/): the family and how we bake
- [Cakes to order](https://bakkerijjansen.nl/taarten/): prices and how to order
- [Contact](https://bakkerijjansen.nl/contact/): address, opening hours, phone
```

## The head of every page

```html
<!doctype html>
<html lang="nl">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Taarten op bestelling | Bakkerij Jansen</title>
  <meta name="description" content="Verjaardag, bruiloft of zomaar: bestel uw taart twee dagen vooraf. Ophalen in Haarlem.">
  <link rel="canonical" href="https://bakkerijjansen.nl/taarten/">

  <meta property="og:type" content="website">
  <meta property="og:site_name" content="Bakkerij Jansen">
  <meta property="og:title" content="Taarten op bestelling">
  <meta property="og:description" content="Bestel uw taart twee dagen vooraf.">
  <meta property="og:url" content="https://bakkerijjansen.nl/taarten/">
  <meta property="og:image" content="https://bakkerijjansen.nl/og.jpg">
  <meta property="og:image:width" content="1200">
  <meta property="og:image:height" content="630">
  <meta property="og:locale" content="nl_NL">
  <meta name="twitter:card" content="summary_large_image">

  <link rel="icon" href="/favicon.ico" sizes="32x32">
  <link rel="icon" href="/icon.svg" type="image/svg+xml">
  <link rel="apple-touch-icon" href="/apple-touch-icon.png">
  <link rel="manifest" href="/site.webmanifest">
  <meta name="theme-color" content="#7a3e1d">
  <link rel="alternate" type="text/markdown" href="/llms.txt">
</head>
```

- Every page has its own title (about 50 to 60 characters, the subject first, the site name last) and its own description (about 120 to 160 characters, written for a person deciding whether to click).
- `canonical`, `og:url`, `og:image` and sitemap entries always use the full `https://` address on the customer's domain, with the same trailing slash as the links.
- A page that should not show up in search results (a thank-you page after a form) gets `<meta name="robots" content="noindex">` and stays out of the sitemap.

## Structured data

Describe the business in JSON-LD on the home page, so search engines can show the address, opening hours and logo. Pick the most precise type from https://schema.org (`Bakery`, `Restaurant`, `Dentist`, `LocalBusiness`, or `Organization` without a shop front), and add `WebSite`:

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Bakery",
      "@id": "https://bakkerijjansen.nl/#business",
      "name": "Bakkerij Jansen",
      "url": "https://bakkerijjansen.nl/",
      "logo": "https://bakkerijjansen.nl/icon-512.png",
      "image": "https://bakkerijjansen.nl/og.jpg",
      "telephone": "+31 23 123 4567",
      "address": { "@type": "PostalAddress", "streetAddress": "Grote Markt 1", "postalCode": "2011 RD", "addressLocality": "Haarlem", "addressCountry": "NL" },
      "openingHoursSpecification": [
        { "@type": "OpeningHoursSpecification", "dayOfWeek": ["Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"], "opens": "08:00", "closes": "17:00" }
      ],
      "sameAs": ["https://www.instagram.com/bakkerijjansen", "https://www.facebook.com/bakkerijjansen"]
    },
    { "@type": "WebSite", "@id": "https://bakkerijjansen.nl/#website", "url": "https://bakkerijjansen.nl/", "name": "Bakkerij Jansen", "inLanguage": "nl" }
  ]
}
</script>
```

Blog posts get `Article` (with `datePublished` and `author`), products `Product` with `offers`, events `Event`. Only mark up what the visitor can also read on the page.

## Redirects

When the site comes from another platform, or an address changes, send the old addresses on in a file `_redirects` at the root. One line per redirect:

```
# old              new                                status (default 301)
/about-us.php      /about-us/
/blog/*            /news/
/shop              https://shop.example.com/          302
```

- A path ending in `/*` sends everything below it to the same target.
- The target is a path on the same site or a full https address.
- 301 for permanent (search engines then take over the new address), 302 or 307 for temporary.
- At most 50 lines; the redirect wins over a file at the same path.
- The file itself is not published. A line that is wrong shows up with its line number in the hints of the version.

## More than one language

- One folder per language, each page with `lang` set and the same structure.
- Every version of a page lists all versions, itself included, plus `x-default`:

```html
<link rel="alternate" hreflang="nl" href="https://bakkerijjansen.nl/taarten/">
<link rel="alternate" hreflang="en" href="https://bakkerijjansen.nl/en/cakes/">
<link rel="alternate" hreflang="x-default" href="https://bakkerijjansen.nl/taarten/">
```

- A visible language switch with links, never an automatic redirect on the browser language.
- `og:locale` per language; the sitemap lists every version.

## Fast, readable, usable

- Semantic HTML: `header`, `nav`, `main`, `footer`, one `h1` per page, headings in order.
- Every `img` has `alt` (empty, `alt=""`, for decoration) and `width` and `height`. Images below the first screen get `loading="lazy"`. Use WebP or AVIF and size photos to how they are shown, at most twice that for sharp screens.
- Little JavaScript; the site should work without it. No frameworks for a site of a few pages.
- Fonts on the site itself (`.woff2`, `font-display: swap`), not from Google Fonts' servers: loading them from Google sends the visitor's IP address to Google, for which a German court awarded damages in 2022.
- Enough contrast, visible focus, links that say where they go, a phone number as `tel:` link, an address with a link to a map.
- Spaicely already minifies HTML, CSS and JavaScript and recompresses JPEG and PNG on publish. It does not convert images to other formats.
- Browsers keep CSS, JavaScript, images and fonts for a day. When you change such a file, give it a new name (`style.v2.css`) and update the links, so visitors see the change at once. HTML is always fetched fresh.

## Forms

A static site can still take messages and signups. Spaicely handles them on `https://scripts.spaicely.com`; do not use a form service elsewhere.

1. **Choose the kind.** `contact` mails every message to one address; `signup` keeps a list (newsletter, waiting list) with double opt-in: the visitor gets a mail and is only on the list after clicking the link in it.
2. **Switch it on** with `scripts.enable`: `site_id`, `kind`, `name` (what the person calls it), and where it fits `locale` (language of the mails and of our pages), `subject`, `recipient`, `success_url` and, for signup, `confirmed_url`. Without `recipient` a contact form mails the person who set it up. Any other address first gets a mail to confirm; until then the form has status `waiting_for_recipient` and sends nothing, so tell the person to look in that inbox.
3. **Put the `html` from the answer in a page as it is**, then style it with the site's CSS. It looks like this:

```html
<form action="https://scripts.spaicely.com/f/k3v9q2x7m1c8b4n6r0td" method="post">
  <input type="text" name="_hp" tabindex="-1" autocomplete="off" aria-hidden="true" style="position:absolute;left:-9999px">
  <label>Naam <input name="name" required></label>
  <label>E-mailadres <input type="email" name="email" required></label>
  <label>Bericht <textarea name="message" rows="6" required></textarea></label>
  <button type="submit">Versturen</button>
</form>
```

   Keep `action`, `method` and the hidden `_hp` field (it catches bots). You may add fields, such as `phone` or `subject`, with any name that does not start with `_`; they all end up in the mail. The field `email` becomes the reply address, so replying goes straight to the visitor. Keep a visible `label` for every field.
4. **Make the thank-you pages.** `success_url` (after sending) and `confirmed_url` (after confirming a signup) are pages on the site itself, full `https://` addresses on its own domain, with `noindex` and outside the sitemap. Without them the visitor sees a plain page of ours in the form's language with a link back.
5. **Or stay on the page.** Post with `fetch` and `Accept: application/json`; the answer is `{"ok": true}`, or `{"ok": false, "error": ...}` with `origin_not_allowed`, `form_inactive`, `invalid`, `rate_limited` or `failed`. Without JavaScript the plain form keeps working:

```html
<script>
  for (const form of document.querySelectorAll('form[action^="https://scripts.spaicely.com/"]')) {
    const status = document.createElement("p");
    status.className = "form-status";
    status.setAttribute("role", "status");
    form.append(status);
    form.addEventListener("submit", async (event) => {
      event.preventDefault();
      const response = await fetch(form.action, {
        method: "POST", body: new URLSearchParams(new FormData(form)), headers: { Accept: "application/json" }
      }).catch(() => null);
      const answer = response ? await response.json().catch(() => ({})) : {};
      status.textContent = answer.ok ? "Bedankt, uw bericht is verstuurd." : "Dat lukte niet. Probeer het later nog eens.";
      if (answer.ok) form.reset();
    });
  }
</script>
```

**A download in exchange for details** (a guide, white paper or price list): use `kind: "download"`.

- Put the file in the site under `_private/`, such as `_private/guide.pdf`, and publish it with the rest. Everything under `_private/` stays off the CDN, so it cannot be fetched from the site; only the form gives it.
- `scripts.enable` with `kind: "download"`, `file_path: "_private/guide.pdf"` and optionally `recipient`. What the visitor fills in goes to that recipient by mail, as with `contact`. The HTML has name, email and company; add fields if the person wants to know more.
- `delivery: "mail"` (the default) sends the download link to the address filled in, so the details are real; say on the thank-you page that the link is in the mail. `delivery: "direct"` starts the download at once; with `fetch` you get `{"ok": true, "download_url": ...}`, then set `location.href = answer.download_url`.
- A link works for 7 days (direct: an hour) and at most 5 times. While the file is not in the live version, the status is `waiting_for_file` and the form gives an error.

**A discount code in exchange for an email address**: use `kind: "coupon"`.

- One code for everyone: `coupon_code: "WELCOME10"`. A unique code per visitor: `coupon_codes: ["JANSEN-001", ...]` (create them in the shop or till first; up to 1000 per call, add more later with `scripts.configure`, `clear_coupons: true` removes the codes not given out yet).
- The recipient gets by mail what the visitor filled in and which code they got, and a notice when 10 codes are left and when they ran out. Without codes the status is `waiting_for_codes`.
- `delivery: "mail"` (the default) sends the code to the address filled in. `delivery: "direct"` works only with codes everyone gets; with `fetch` you get `{"ok": true, "code": ...}` to show on the page.
- Codes with a period: `dated_codes: [{"code": "WEEK45", "valid_from": "2026-11-03", "valid_until": "2026-11-09", "shared": true}]`. With `shared: true` everyone who asks during that period gets the same code (a code per week or per campaign); without it each code goes out once and the mail says until when it is valid. Dates without a time are whole days in Dutch time. Order: first a shared code valid now, then the unique codes, then `coupon_code`.

**Paying for a download or code**: give a download or coupon form `price_cents` (including VAT, at least 50). The visitor pays through Stripe to the owner of the site, as in the web shop: Shop plan, a connected Stripe account and 2% commission. After paying, the link or code goes to the buyer by mail and the details to the recipient.

- The form then posts to `https://app.spaicely.com/forms/<form_id>/checkout`; use the new `html` from the answer. A plain form goes on to Stripe; with `fetch` (JSON) you get `{"url": ...}` to open.
- Keep the required checkbox `start_now`: the buyer wants it right away and so gives up the right of withdrawal for digital content. Without it the checkout refuses.
- While something is missing, the status says what: `waiting_for_plan`, `waiting_for_billing_details` or `waiting_for_stripe` (then give the person `connect_url`). `price_cents: 0` makes the form free again.
- The owner is the seller. Put the company details, the price incl. VAT and a page on delivery and withdrawal on the site, as for the web shop.

**Donations**: `kind: "donation"` works on every plan. The visitor picks an amount button or types an amount and pays through Stripe to the owner's own account (connected once, 2% commission). The donor gets a thank-you mail, the recipient a mail per donation.

- `donation_amounts` in cents for the buttons (default 10, 25 and 50 euros), `donation_min_cents` and `donation_max_cents` for the range, `goal_cents` for a goal and `ends_at` for the last day (after it the form takes nothing more). There is no withdrawal checkbox: nothing is delivered.
- The form posts `amount` (a button) or `amount_other` (typed), `name`, `email`, `message` and `show_name`. Only with `show_name` do name and message appear in the progress.
- For a progress bar: `fetch(progress_url)` gives `{"raised_cents", "donors", "goal_cents", "ends_at", "open", "donations": [{"name", "message", "at"}]}`, open to any origin and fresh within a minute. Show the bar as a percentage of `goal_cents`, and say clearly on the page what the money is for and that it goes to the owner also when the goal is not reached.
- Not for investments, loans or rewards in money; that needs a licence. Gifts in return (a mention, a postcard) are fine.

Good to know:

- A form only accepts posts from the site's connected domain (and its www). On a preview or a local copy it answers `origin_not_allowed`; test it once the domain is live.
- Limits: 10 posts per visitor per 10 minutes, 100 per form per hour, and 3 signups per address per hour.
- Mail comes from `no-reply@spaicely.com` with the site name as sender.
- `scripts.list` shows the forms of a site with their status and the HTML again; `scripts.configure` changes texts, recipient or pages, or switches a form off with `enabled: false`.
- `scripts.subscribers` lists the confirmed signups, each with an `unsubscribe_url`. Every mail sent to the list must carry that link.
- Mention the form in the privacy page: what is collected, that Spaicely handles it, and how long it is kept.
- Sessions and a shopping cart need no server: keep them in the browser with `localStorage`.

## Web shop

On the Shop plan a site sells by itself. The buyer pays through Stripe Checkout, straight to the owner; Spaicely keeps 2% commission. No shop system is needed: the products are in one file.

1. **Link Stripe.** The account owner links their own Stripe account once. `shop.setup` gives the `connect_url` for that; send the person there. Until that is done the status is `waiting_for_stripe`.
2. **Products in `data/products.json`.** Prices in cents, including VAT. `id` is unique and does not change; `active: false` takes a product off sale without removing it. Without `shipping` Checkout asks no address (services, digital products).

```json
{
  "products": [
    { "id": "mok-blauw", "name": "Blue mug", "price": 1495, "image": "/img/mok-blauw.jpg", "description": "Hand thrown, 300 ml" },
    { "id": "bord", "name": "Plate", "price": 2000 }
  ],
  "shipping": [
    { "id": "nl", "name": "Shipping in the Netherlands and Belgium", "price": 495, "countries": ["NL", "BE"] }
  ]
}
```

3. **Publish and switch on** with `shop.setup` (`site_id`, and where it fits `success_url`, `cancel_url` on the site's own domain, `notify_email`). The answer gives the status, what is still missing (`problems`), the `endpoint` and a `snippet`: a buy button as a plain form.
4. **A cart** lives in the browser (`localStorage`) and is sent with `fetch`; the answer is `{"url": ...}` to go to Stripe, or `{"error": ..., "message": ...}` with a text for the visitor:

```html
<script>
  async function checkout(items, shipping) {
    const response = await fetch("https://app.spaicely.com/shop/SHOP_ID/checkout", {
      method: "POST", headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ items, shipping })
    });
    const answer = await response.json();
    if (answer.url) location.href = answer.url; else alert(answer.message);
  }
  // checkout([{ id: "mok-blauw", quantity: 2 }], "nl")
</script>
```

Good to know:

- Prices always come from the live `data/products.json`, never from the page. When you change a price, publish the pages that show it too.
- Ordering only works from the site's connected domain (and www) or its address; up to 50 lines, 1 to 99 of each product.
- The owner gets a mail for every paid order with the products and the delivery address; `shop.orders` lists the orders. Stripe sends the buyer the receipt.
- Make a thank-you page (`success_url`, `noindex`) and say on the privacy page that Stripe handles the payment.
- The owner is the seller. Make a page with the person about delivery, shipping costs, returns and the 14-day right of withdrawal for consumers (with the model form), plus the business details (name, address, registration and VAT number), and link it in the footer and at checkout. The owner refunds in their Stripe dashboard; Spaicely's commission is not returned then.

## Privacy and cookies

- No tracking without consent. Analytics or embedded videos, maps and social media posts that set cookies need a cookie banner that asks first, with refusing as easy as accepting. Better: a privacy-friendly analytics tool without cookies, a video thumbnail that loads the player on click, and a link to the map instead of an embed.
- A privacy page whenever the site collects anything: what, why, how long, who to ask. Business sites in the EU also show the company name, address, chamber of commerce and VAT number (in Germany and Austria on an `Impressum` page).

## After it is live

- Every paid site is also at an address of its own, `https://<label>.spaicely.site` (`address` in `account.whoami` and `sites.versions`; `address_live` says whether it works). It works once the hosting is paid and a version is live, also without a connected domain: useful to show the site before the domain is ready. Search engines get noindex there, so canonical links and the sitemap keep pointing at the site's own domain.
- Read the `hints` in `sites.versions` and fix what applies.
- Suggest that the person adds the domain to Google Search Console and Bing Webmaster Tools and submits the sitemap there. Both verify a domain with a TXT record, which you can set with `domains.dns.update`.
- After a big change, update `lastmod` in the sitemap for the pages that changed.

## More

- Uploading, publishing and rolling back, changing one block of a page, putting the site on its domain, DNS, mail forwarding, people and roles, the hosting plan: the skill `site-management`, https://spaicely.com/skills/en/site-management/SKILL.md.
- Every tool, parameter and HTTP endpoint: https://spaicely.com/developers/, and for AIs https://spaicely.com/llms-full.txt.
- A tool named there that is not in your list means your list is old: ask the person to reconnect the Spaicely connector, which loads the current one.
