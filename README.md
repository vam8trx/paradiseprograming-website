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

**paradiseprogramming.com → Email → Email Routing**. If Email Routing is already on (the ledger
may have turned it on), go to **Routing rules → Create address** → `hello` → action *Send to an
email* → your Gmail. If it's off, turn it on first and let it add its DNS records. Gmail sends a
confirmation link the first time.

### 5. Tell Google

[Google Search Console](https://search.google.com/search-console) → Add property → Domain →
`paradiseprogramming.com` (it gives you a `TXT` record to add in Cloudflare DNS). Then
**Sitemaps** → submit `https://paradiseprogramming.com/sitemap.xml`.

Optional: **Analytics & Logs → Web Analytics** → add the site for free, cookie-free visitor counts.
`_headers` already allows its script.

## Editing

Edit `site/index.html` and push to `main`. There's no admin area here on purpose: it's one page that
rarely changes. If it grows (a page per project, a blog), add more `.html` files to `site/` and to
`sitemap.xml`.

To preview locally:

```bash
python3 -m http.server 8000 --directory site
```

Then open http://localhost:8000.
