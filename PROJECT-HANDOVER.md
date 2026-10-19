# Project Handover — jovandhillon.com

A practical operator's guide for picking this project up cold.

*Last updated: October 2026.*

---

## 1. What this is

A single-page personal site for **jovandhillon.com**. Static, no build step. The entire site lives in one file: `index.html`.

- **Live URL:** https://jovandhillon.com
- **Pages preview URL:** https://jovandhillon17-github-io.pages.dev
- **Repo:** `Jovan-Dhillon/jovandhillon17.github.io`

---

## 2. Hosting

The site is hosted on **Cloudflare Pages** (previously GitHub Pages; migrated May 2026).

| Setting | Value |
|---|---|
| Production branch | `main` |
| Build command | *(empty)* |
| Build output directory | `/` |
| Framework preset | None |
| Custom domain | `jovandhillon.com` |

**Deployment is automatic.** Any push to `main` triggers a redeploy in ~30 seconds. There is no CI, no test suite, no build step — what's in `main` is what's served.

To deploy a change: edit `index.html`, commit to `main`, push. That's it.

---

## 3. DNS

DNS for `jovandhillon.com` is managed inside Cloudflare (nameservers: `megan.ns.cloudflare.com`, `michael.ns.cloudflare.com`). The Pages custom-domain setup adds a flattened CNAME for the apex automatically — no manual record needed.

**Do not touch:**
- **MX records** and **TXT (SPF / DKIM / DMARC)** records → these power the **iCloud Custom Email** for `@jovandhillon.com`. Deleting them breaks mail.
- **`linkedin.jovandhillon.com`** → a separate vanity redirect, unrelated to this project.

The `CNAME` file in the repo (`jovandhillon.com`) is a legacy GitHub Pages artifact. Cloudflare ignores it; safe to leave.

---

## 4. Repo layout

```
.
├── index.html              ← the entire site (~104KB, ~2400 lines)
├── images/                 ← favicons
│   ├── favicon.ico         ← multi-size (16/32/48), classic tab icon
│   ├── favicon.png         ← 512×512, modern browsers
│   └── apple-touch-icon.png← 180×180, iOS home screen
├── sitemap.xml             ← single URL, submitted to Search Console
├── robots.txt              ← crawler rules + Sitemap line (self-contained)
├── CNAME                   ← legacy GitHub Pages marker (harmless)
├── README.md               ← GitHub profile readme (not used by the site)
├── LICENSE
└── PROJECT-HANDOVER.md     ← this file
```

---

## 5. Architecture

`index.html` is a **no-build React app transpiled in the browser**:

- React 18 + ReactDOM are loaded as UMD scripts from **unpkg.com**.
- JSX is compiled at runtime by **`@babel/standalone`** (also from unpkg).
- All component code lives in `<script type="text/babel">` blocks inside `index.html`.

Not everything is React. Three plain `<script>` blocks run outside the transpile: the `CONTENT` data object, the Google Analytics tag, and the cookie banner. The last two are deliberately independent of React (see section 9) and must stay that way.

### Five design variations, only one renders

The file defines five self-contained design "variations":

| Variation | Component | Status |
|---|---|---|
| Neo-Brutalist | `Brutalist` | **Renders** |
| Dashboard | `Dashboard` | Defined but unused |
| Ambient | `Ambient` | Defined but unused |
| Editorial Serif | `EditorialSerif` | Defined but unused |
| Terminal CLI | `CLI` | Defined but unused |

The render call at the bottom of the file is:

```jsx
ReactDOM.createRoot(document.getElementById('root')).render(<Brutalist />);
```

Only `Brutalist` is shipped. The other four (and the `Tweaks` editor panel) are dead code kept for reference.

### Where the content lives

Most page text comes from a shared `CONTENT` object (search for `const CONTENT = {`). The Brutalist component then reads from `C.quickFacts`, `C.services`, `C.stack`, `C.projects`, `C.publications`, `C.now`, etc.

This document gives no line numbers on purpose. They drift every time anything is added, and stale ones are worse than none. Search for the string.

Some text (hero copy, About paragraphs) is **hardcoded directly in the Brutalist JSX**, not in `CONTENT`. If editing copy, search for the exact phrase to find it.

### Brutalist section order

| # | Section | Anchor |
|---|---|---|
| — | Hero | `#top` |
| 01 | About | `#about` |
| 02 | What I Do (services) | `#work` |
| 03 | The Stack | `#stack` |
| 04 | Projects Shipped | `#projects` |
| 05 | Publications | `#publications` |
| 06 | Right Now | *(no id)* |
| — | Contact footer | `#contact` |

---

## 6. Common editing tasks

### Add a publication (article, feature, etc.)

Find the `publications:` array inside `CONTENT` and add an entry:

```js
{
  title: "Article title",
  outlet: "Where it was published",
  type: "Article",
  year: "2026",
  desc: "Short description.",
  href: "https://link-to-article",   // omit for a "COMING SOON" placeholder
},
```

`type` is free text and is rendered verbatim next to the outlet (`outlet · type`). Existing entries use `Article` and `Feature`; anything short works.

There is **no publications counter** in the hero, so nothing else needs bumping. Entries render in array order, so newest goes last unless you want it higher up the page.

Current entries (3): ApprenticeWatch (2025), Placer Apprenticeships (2026), Association of Apprentices (2026).

### Add a project

Same pattern, in the `projects:` array. Then bump the "Projects live" counter card in the hero (`<div className="v">03</div>` etc.) if you want it accurate.

### Change the favicon

Replace the files in `images/` keeping the **same filenames** (`favicon.png`, `favicon.ico`, `apple-touch-icon.png`). The `<link>` tags in the page head reference these exact paths.

Browsers cache favicons aggressively — hard-refresh (Cmd/Ctrl+Shift+R) to see changes.

### Change the browser tab title

Edit the `<title>` element near the top of `index.html`.

### Change navigation links

Find `<nav className="br-nav">` in `index.html`.

### Sanity-check an edit before pushing

There is no build step to catch a syntax error, and a broken `CONTENT` object means a **blank page in production**. Before committing a content change, extract the object and let Node parse it:

```bash
START=$(grep -n "^const CONTENT = {" index.html | cut -d: -f1)
END=$(grep -n "^window.CONTENT = CONTENT;" index.html | cut -d: -f1)
sed -n "${START},${END}p" index.html > /tmp/content.js
node -e "global.window={}; require('/tmp/content.js'); console.log(window.CONTENT.publications.length)"
```

If it prints a number, the object is valid. A `SyntaxError` means do not push.

---

## 7. Things to know / known quirks

- **Google Analytics 4 is installed, gated behind consent.** The `gtag.js` snippet sits near the top of `<head>` in `index.html` (search for `gtag/js`), reporting to property **`G-7WM9PC1BBQ`**. It is a plain `<script>`, not a `text/babel` block, so it runs before React and is unaffected by the JSX transpile. See section 9 for the consent wiring.
- **`robots.txt` carries its own crawler rules, and they do not update themselves.** Cloudflare used to prepend a managed block supplying them; that injection stopped between September and October 2026, which left the site serving no crawler rules at all until it was caught. The rules now live in the repo file. Re-check them against Cloudflare's managed list occasionally, and see section 10 before changing anything.
- **In-browser Babel is heavy.** Every visitor downloads React's *development* builds plus the full Babel compiler (~3MB) and transpiles JSX on their device. Acceptable for a personal site, but the obvious optimisation is to introduce a build step (Vite) and serve pre-compiled, minified production React.
- **React DEV build, not production.** See above. No `react.production.min.js` is referenced.
- **Hard dependency on `unpkg.com`.** If unpkg is unavailable, the site won't render. Mitigation would be self-hosting React/Babel.
- **The unpkg script tags are pinned and SRI-protected.** React 18.3.1 and Babel 7.29.0, each with an `integrity="sha384-..."` hash. Bumping a version number **without regenerating the matching hash** makes the browser refuse the script and the site goes blank. Either regenerate the hash or drop the `integrity` attribute deliberately.
- **Page title is intentionally `jovandhillon.com`** (not the engineer's name). Set by request.
- **Em dashes have been swept from the file.** A zero-em-dash policy is in effect; if writing new copy, use colons / commas / parentheses instead of `—`.
- **The `Tweaks` panel inside `index.html`** (search for `function Tweaks`) is editor-harness code that `postMessage`s to a parent window. It only activates inside a visual editor iframe. Harmless in production.

---

## 8. Quick checks if something looks broken

- **Site shows nothing / blank page** → open DevTools → Console. Likely a JS syntax error in `index.html`, or unpkg.com is unreachable.
- **DNS_PROBE_FINISHED_NXDOMAIN locally but works elsewhere** → local DNS cache. Flush (`ipconfig /flushdns` Windows, `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder` Mac) or test on mobile data.
- **Deploy didn't update the site** → check Cloudflare → Pages → project → **Deployments** tab; confirm the latest commit shows **Success**. To confirm from the terminal instead, grep the live page for something only the new version contains:

  ```bash
  curl -s https://jovandhillon.com/ | grep -c "some new string"
  ```

  A deploy usually lands within ~30 seconds of the push, but give it a minute before assuming it failed.
- **A URL returns 200 but the wrong content** → Cloudflare Pages serves `index.html` for any unknown path, so a missing file looks like a success. Check the body and the `content-type`, never the status code alone.
- **Favicon didn't update** → browser cache. Visit `/images/favicon.png` directly to confirm the new file is being served, then hard-refresh.
- **No data in Google Analytics** → check the cookie banner. Analytics only fires after a visitor clicks **Accept**; a `Reject` or an untouched banner means no hits by design. In DevTools, `localStorage.getItem('jd-cookie-consent')` shows the stored choice.

### Periodic check: things that rot without a commit

Most of this site only changes when someone pushes. Two things do not, because they depend on Cloudflare's behaviour at the edge rather than on the repo. Both have broken silently before. Worth running every few months:

```bash
# crawler rules reaching crawlers? expect 10, not 0 and not 20 (see section 10)
curl -s https://jovandhillon.com/robots.txt | grep -cE "^User-agent:"

# sitemap still real XML, not the index.html fallback?
curl -sI https://jovandhillon.com/sitemap.xml | grep -i content-type   # want application/xml
```

---

## 9. Analytics and cookie consent

**GA4 property:** `G-7WM9PC1BBQ`. The tag is in `<head>` in `index.html` (search for `gtag/js`).

Analytics is wired through **Google Consent Mode v2** and is **denied by default**:

1. Before `gtag('config', ...)` runs, a `gtag('consent', 'default', ...)` call sets `analytics_storage` to `denied`, unless a previous choice is found in `localStorage` under the key **`jd-cookie-consent`**.
2. The banner markup, styles (`.cb*`), and logic live in `index.html`. Search for `cookie-banner`.
3. Clicking **Accept** or **Reject** stores the choice and fires `gtag('consent', 'update', ...)`. The banner does not reappear.

The banner is **deliberately vanilla JS, not a React component.** It gates a script that loads in `<head>` long before React, and it should keep working if the React tree fails to render. Do not port it into a variation component.

Its CSS cannot use the `--border` / `--accent` custom properties, because those are scoped to `.v-brutalist` inside the React tree. The brutalist dark palette is written out literally in the `.cb*` rules. It does **not** follow the site's light-mode toggle.

### To reset consent while testing

```js
localStorage.removeItem('jd-cookie-consent')
```

Then reload; the banner returns.

---

## 10. Search engines: sitemap and robots.txt

### Sitemap

`sitemap.xml` sits at the repo root and is served at **https://jovandhillon.com/sitemap.xml** for Google Search Console.

It is a single-URL sitemap because the site is one page. The section anchors (`#about`, `#projects`, ...) are deliberately **not** listed: search engines ignore fragment URLs in sitemaps. Update `<lastmod>` when you make a meaningful content change.

Note that before this file existed, `/sitemap.xml` returned `index.html` with a `200`, because Cloudflare Pages falls back to the index for unknown paths. A missing file will not 404 here, so verify content, not status codes.

**If Search Console reports "Sitemap is HTML"**, it is showing a read from before the sitemap deployed. Confirm the live file first:

```bash
curl -sI https://jovandhillon.com/sitemap.xml | grep -i content-type   # want application/xml
```

If that is `application/xml`, the error is stale. Press **Refresh** on the sitemap row in Search Console to force a re-read.

### robots.txt

**The rules live in the repo file and are a frozen snapshot. They do not update themselves.**

`robots.txt` at the repo root carries the Content Signals notice (`search=yes, ai-train=no, use=reference`), `Disallow: /` for ten AI crawlers (Amazonbot, Applebot-Extended, Bytespider, CCBot, ClaudeBot, CloudflareBrowserRenderingCrawler, Google-Extended, GPTBot, meta-externalagent) and the `Sitemap:` line.

#### Why it is arranged this way

This has changed once already, so the history matters:

1. **Originally** there was no repo `robots.txt`. Cloudflare served a managed block at the edge, and the Pages fallback appended the whole of `index.html` after it. Malformed, but the rules were live.
2. **September 2026**, a repo file was added mirroring the rules. Cloudflare still prepended its block, so every rule appeared **twice**. The file was trimmed to just the `Sitemap:` line, relying on the injection for the rules.
3. **October 2026**, that reliance broke. Cloudflare had **stopped prepending** the managed block, on both the apex and the `pages.dev` URL, uncached, for every user agent. The served file was byte-identical to the repo file, so the site was serving **no crawler rules and no Content Signals reservation at all**. Nothing in the repo had changed; the edge behaviour had. The rules were moved back into the repo file.

#### What this means for you

- **Do not assume the edge supplies anything.** It did, then it stopped, silently, with no commit and no deploy.
- **If Cloudflare's managed robots.txt is ever re-enabled**, its block is prepended again and every rule in the repo file appears twice. Either trim the repo file back to the `Sitemap:` line at that point, or leave the managed feature off.
- **Re-check the crawler list periodically** against [Cloudflare's managed robots.txt docs](https://developers.cloudflare.com/bots/additional-configurations/managed-robots-txt/) and add new entries by hand. The repo file will not track their additions.

#### Verifying

Always check what is served, never just what is in the file:

```bash
curl -s https://jovandhillon.com/robots.txt | grep -cE "^User-agent:"   # expect 10
```

A `0` means the rules are not reaching crawlers. A `20` means the managed block is being prepended again and everything is duplicated.
