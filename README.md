# Paradise Programming — main website

The front door for **paradiseprogramming.com** (and `www.`): who you are, what you've built, and how
to get in touch. Plain HTML/CSS, no build step, no Worker code. It's one Cloudflare Worker that only
serves the `site/` folder, set up like the church and fire department sites.

```
site/
  index.html      The whole site: hero, Work, How it works, About, Get in touch
  css/style.css   One stylesheet; brand colors are the variables at the top
  favicon.svg     Logo / browser-tab icon (sunset + code brackets)
  404.html        "Page not found"
  _headers        Security headers (Cloudflare reads it; it isn't served)
  robots.txt, sitemap.xml
wrangler.toml     Worker name and the two custom domains
```

## How the domain is laid out

| Address | What's there | Repo |
|---|---|---|
| `paradiseprogramming.com` / `www.` | This site | this one |
| `ledger.paradiseprogramming.com` | Paradise Ledger (private, not linked from this site) | `digital-check-register` |
| `simulator.paradiseprogramming.com` | Fire & Rescue Simulator | `Caroline-Fire-Rescue-Sim` |

Each app keeps its own subdomain and its own Worker. This site only links to them.

## Before launch: fill in the yellow boxes

Anything highlighted yellow on the page is a placeholder. Search `index.html` for `TODO(Jason)`:

1. **Simulator description**: a sentence or two on what it does and who uses it.
2. **About**: a few sentences in your own words.
3. **Contact email**: `hello@paradiseprogramming.com` only works once step 4 below is done.
   Use a different address if you prefer; it's in one place.

When those are filled in, the `.todo` style at the bottom of `css/style.css` can stay (nothing uses it).

It's also courteous to let the chief and the pastor know their sites are listed under Work.

## Going live (about 15 minutes, free tier)

### 1. Check that nothing else holds the bare domain or `www`

Cloudflare dashboard → **paradiseprogramming.com → DNS → Records**. If there's an `A`, `AAAA` or
`CNAME` record named `paradiseprogramming.com` (shown as `@`) or `www`, note what it points to.

- The simulator lives at `simulator.paradiseprogramming.com`, so it doesn't use either name.
  If the simulator's Worker *also* has the bare domain or `www` under **Settings → Domains &
  Routes**, remove those there first.
- Delete any leftover parking or registrar records for `@` and `www`. Leave the `ledger` and
  `simulator` records and any `MX`/`TXT` email records alone.

### 2. Connect this repo to a Worker

**Workers & Pages → Create → Import a repository** → pick `vam8trx/paradiseprograming-website`.
- Project name: `paradiseprogramming-website` (must match `name` in `wrangler.toml`)
- Build command: *(leave empty)*
- Deploy command: `npx wrangler deploy` (the default)
- Production branch: `main`

Deploy. Wrangler reads `wrangler.toml` and adds both custom domains, with DNS records and
certificates, by itself. After that, every push to `main` updates the live site.

### 3. Send `www` to the bare domain (optional, recommended)

Both addresses already show the site, and the page tells search engines the main one is
`paradiseprogramming.com`. For a real redirect: **paradiseprogramming.com → Rules → Redirect Rules
→ Create rule → template "Redirect from WWW to root"** → Deploy.

### 4. Make the contact email work

See **Email** below. The contact button uses `hello@paradiseprogramming.com`.

### 5. Tell Google

[Google Search Console](https://search.google.com/search-console) → Add property → Domain →
`paradiseprogramming.com` (it gives you a `TXT` record to add in Cloudflare DNS). Then
**Sitemaps** → submit `https://paradiseprogramming.com/sitemap.xml`.

Optional: **Analytics & Logs → Web Analytics** → add the site for free, cookie-free visitor counts.
`_headers` already allows its script.

## Email

`hello@paradiseprogramming.com` is free: **Cloudflare Email Routing** receives mail and forwards
it to Jason's Gmail, and Gmail sends *as* `hello@` through **Resend's** mail server. There's no
separate mailbox; everything lives in Gmail. (To move to Google Workspace later, remove Email
Routing and switch the `MX` records; nothing on the site changes.)

### Receive: Cloudflare Email Routing

1. **paradiseprogramming.com → DNS → Records**: check that there are no `MX` records. If there
   are, something else already handles mail; sort that out first.
2. **Compute → Email Service → Email Routing → Onboard Domain** → `paradiseprogramming.com` →
   **Done**. Cloudflare adds its `MX`, SPF (`TXT`) and DKIM records itself.
3. **Destination Addresses** → add the Gmail address → click the verify link Cloudflare emails.
4. **Routing Rules → Create routing rule**: custom address `hello` → *Send to an email* → the Gmail.
5. Optional: **Catch-all** → *Send to an email* → the Gmail, so any other `@paradiseprogramming.com`
   address (or typo) still arrives.
6. Test: from a different account (not the Gmail itself, which Gmail hides as a duplicate),
   send to `hello@paradiseprogramming.com`.

### Send: Gmail "Send mail as" through Resend

1. **Resend → Domains → Add domain** → `paradiseprogramming.com`. Add the records it lists in
   Cloudflare DNS (DKIM `resend._domainkey`, plus `MX` and SPF `TXT` on the `send` subdomain), or use
   Resend's **Auto configure** with Cloudflare. These live on `send.` and `resend._domainkey`,
   so they don't clash with Email Routing's records on the bare domain. Wait for **Verified**.
2. **Resend → API Keys → Create API key**: name `Gmail send-as`, permission *Sending access*,
   domain `paradiseprogramming.com`. Copy it; Resend shows it only once.
3. **Gmail → Settings (gear) → See all settings → Accounts and Import → Send mail as → Add another
   email address**:
   - Name `Paradise Programming` (or your name), address `hello@paradiseprogramming.com`;
     leave *Treat as an alias* ticked → Next.
   - SMTP server `smtp.resend.com`, port `587`, username `resend`, password = the API key,
     *Secured connection using TLS* → Add account.
   - Gmail emails a confirmation to `hello@`; Email Routing forwards it back to the Gmail. Click
     the link (or enter the code).
4. Same screen → **When replying to a message**: *Reply from the same address the message was
   sent to*. Optionally **make default** if every email should go out as `hello@`.
5. Test: write a new email, pick `hello@` in **From**, send it to a different account, and check
   it doesn't land in spam.

Resend's free plan covers 3,000 emails a month (100 a day), plenty for one person's mail.

## Editing

Edit `site/index.html` and push to `main`. There's no admin area here on purpose: it's one page that
rarely changes. If it grows (a page per project, a blog), add more `.html` files to `site/` and to
`sitemap.xml`.

To preview locally:

```bash
python3 -m http.server 8000 --directory site
```

Then open http://localhost:8000.
