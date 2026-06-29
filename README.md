# Nirvana Biotech — Commercial Site

A separate, sister site to the research site (`nirvanabiotech.org`), for the for-profit
commercial division: products (diagnostic kits), services, and a different visual identity.
Same admin engine — login, edit-everything, save-to-GitHub — adapted for a product catalog
instead of a research portfolio.

## ⚠️ This must be a fully separate deployment

This site shares *code patterns* with the research site, but must NOT share:
- The same GitHub repo
- The same Cloudflare Worker
- The same `ADMIN_PASSWORD`
- The same `GITHUB_TOKEN`

If this site's password ever leaked, you don't want it to also grant write access to the
research site's repo (or vice versa). Set everything up fresh, even though the files look
similar.

## Visual identity — how this differs from the research site

The research site is dark ink + cream + teal/rust, serif display type, built to read as
"read me" (a journal/portfolio). This site is its light-mode sibling: clinical white +
deep blue + amber, geometric sans display type, built to read as "buy from me" (a catalog).

- **Palette**: `#FAFAF7` paper white, `#101820` ink, `#0F5C8C` clinical blue (primary),
  `#E8A33D` amber (CTAs/accents), `#6B7680` instrument gray (secondary text).
- **Type**: Space Grotesk (display, geometric — product/spec-sheet feel, not journal),
  Inter (body), IBM Plex Mono (SKUs, spec values, lot-style data).
- **Signature element**: a "spec strip" — a mono-type row of key:value pairs
  (`SENS: 98.2% · SPEC: 99.1% · READ TIME: 15 MIN`) under each product, evoking an actual
  diagnostic kit's spec sheet. This replaces the research site's chromatogram-trace motif
  with something specific to selling lab products rather than publishing research.
- **Structure**: the Home page leads with a flagship product, not a mission statement.
  Products page is a catalog grid. Services page is a simple icon+description list
  (procurement-style, scannable) rather than detailed write-ups.

## Structure

```
index.html         Home (flagship product hero)
about.html          About (company + "why choose us" highlights)
products.html       Products (catalog grid, click a card → spec/detail modal)
services.html       Services (icon + description list)
contact.html        Contact / order inquiry form
styles.css          Design system — entirely separate file from the research site's
site.js             Same admin engine as the research site, repointed at this site's
                    own Worker and own data shape (products/services, not research/team)
data.json           All editable content
worker.js           Cloudflare Worker — identical code to the research site's; only the
                    config (wrangler.toml) differs
wrangler.toml       Worker config — YOU MUST EDIT THIS before deploying (see below)
images/             Your real product photos go here
```

## Required setup before this site works at all

### 1. Pick where this site actually lives

You said this is a separate domain. Decide on:
- The GitHub repo name (e.g. `nirvanabiotech-shop.github.io` or similar)
- The actual domain (e.g. `shop.nirvanabiotech.org` or a totally separate domain)

### 2. Edit `wrangler.toml`

Open it and replace:
- `GITHUB_REPO` → your actual new repo's name
- `ALLOWED_ORIGIN` → your actual new domain (must be the exact origin, e.g.
  `https://nirvanabiotechshop.com`, no trailing slash)

### 3. Edit `site.js`

Near the top, find:

```js
const ADMIN_API_BASE = "https://nirvana-commercial-admin.rajindra04.workers.dev";
```

This is a *guess* based on the same Cloudflare account subdomain as your research site's
Worker. After you run `wrangler deploy` (see `WRANGLER_GUIDE.md` from the research site
package — the steps are identical, just run them in this folder instead), Wrangler will
print the actual URL. If it doesn't match what's in this file, update it.

### 4. Deploy the Worker with its own fresh secrets

```bash
cd this-folder
npx wrangler login          # only needed once per machine
npx wrangler secret put ADMIN_PASSWORD     # use a DIFFERENT password than the research site
npx wrangler secret put GITHUB_TOKEN       # a token scoped to THIS repo, not the research one
npx wrangler secret put SESSION_SECRET     # any random string, different from the research site's
npx wrangler deploy
```

### 5. Push the site files

Push everything except `worker.js` and `wrangler.toml` (those go in their own folder for
the Worker deployment, not your GitHub Pages repo) to your new repo's root, plus your real
product photos into `images/`.

## Editing content once it's live

Same pattern as the research site: click **Admin** in the top nav, log in, edit pencils
appear next to every field. Covered:

- Brand name + logo (every page)
- Home: tagline/title/subtitle/background, flagship product name/description/image/specs
- About: intro/description/background, "why choose us" highlights (add/edit/remove)
- Products: add new, edit name/SKU/description/image, edit full details + spec sheet
  (via "Full details"), remove
- Services: add new, edit icon label/title/description, remove
- Contact: title, description, email, label, background

Saving commits straight to this site's GitHub repo via its own Worker — completely
independent from the research site's save flow.

## Content note

The product/service content in `data.json` right now (Dengue NS1 kit, IgG/IgM combo kit,
aptamer reagent kit, etc.) is placeholder copy written to match what the research site
already describes as in-progress work, since you hadn't specified final commercial product
names, pricing, or regulatory status yet. Replace it with your actual catalog before this
goes live — none of the spec numbers (98.2% sensitivity, etc.) are real lab results, they're
illustrative placeholders.
