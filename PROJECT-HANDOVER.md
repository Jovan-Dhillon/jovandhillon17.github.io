# Project Handover — jovandhillon.com

A practical operator's guide for picking this project up cold.

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
├── index.html              ← the entire site (~97KB)
├── images/                 ← favicons
│   ├── favicon.ico         ← multi-size (16/32/48), classic tab icon
│   ├── favicon.png         ← 512×512, modern browsers
│   └── apple-touch-icon.png← 180×180, iOS home screen
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

Most page text comes from a shared `CONTENT` object (search for `window.CONTENT = CONTENT`). The Brutalist component then reads from `C.quickFacts`, `C.services`, `C.stack`, `C.projects`, `C.publications`, `C.now`, etc.

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

### Add a project

Same pattern, in the `projects:` array. Then bump the "Projects live" counter card in the hero (`<div className="v">03</div>` etc.) if you want it accurate.

### Change the favicon

Replace the files in `images/` keeping the **same filenames** (`favicon.png`, `favicon.ico`, `apple-touch-icon.png`). The `<link>` tags in the page head reference these exact paths.

Browsers cache favicons aggressively — hard-refresh (Cmd/Ctrl+Shift+R) to see changes.

### Change the browser tab title

Edit the `<title>` element near the top of `index.html`.

### Change navigation links

Find `<nav className="br-nav">` (around line ~1375 in `index.html`).

---

## 7. Things to know / known quirks

- **In-browser Babel is heavy.** Every visitor downloads React's *development* builds plus the full Babel compiler (~3MB) and transpiles JSX on their device. Acceptable for a personal site, but the obvious optimisation is to introduce a build step (Vite) and serve pre-compiled, minified production React.
- **React DEV build, not production.** See above. No `react.production.min.js` is referenced.
- **Hard dependency on `unpkg.com`.** If unpkg is unavailable, the site won't render. Mitigation would be self-hosting React/Babel.
- **Page title is intentionally `jovandhillon.com`** (not the engineer's name). Set by request.
- **Em dashes have been swept from the file.** A zero-em-dash policy is in effect; if writing new copy, use colons / commas / parentheses instead of `—`.
- **The `Tweaks` panel inside `index.html`** (search for `function Tweaks`) is editor-harness code that `postMessage`s to a parent window. It only activates inside a visual editor iframe. Harmless in production.

---

## 8. Quick checks if something looks broken

- **Site shows nothing / blank page** → open DevTools → Console. Likely a JS syntax error in `index.html`, or unpkg.com is unreachable.
- **DNS_PROBE_FINISHED_NXDOMAIN locally but works elsewhere** → local DNS cache. Flush (`ipconfig /flushdns` Windows, `sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder` Mac) or test on mobile data.
- **Deploy didn't update the site** → check Cloudflare → Pages → project → **Deployments** tab; confirm the latest commit shows **Success**.
- **Favicon didn't update** → browser cache. Visit `/images/favicon.png` directly to confirm the new file is being served, then hard-refresh.
