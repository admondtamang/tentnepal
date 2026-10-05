# Nepal Tent & Tarpaulin — static site

Static recreation of [nepaltent.com.np](https://nepaltent.com.np/) for
Nepal Tent & Tarpaulin P. Ltd., Kathmandu.

**TL;DR** — 7 static HTML pages, no build step, no framework, no runtime
dependency. Every page carries a title, description, canonical and JSON-LD.
Content is present in the HTML source, so it renders with JavaScript disabled.
Verified against commit `c16de00`, 2026-10-05.

## Pages

| File | Title | H1 | Schema |
| --- | --- | --- | --- |
| `index.html` | Tent & Tarpaulin Manufacturer in Nepal \| Nepal Tent | Nepal Tent & Tarpaulin | Organization, WebSite |
| `company.html` | The Company - 40 Years of Tent Making \| Nepal Tent | Catering to the tourism trade… | Organization, BreadcrumbList |
| `products.html` | Tents, Tarpaulin & Camping Products \| Nepal Tent | Products | + ItemList (22 Products) |
| `awards.html` | Awards & ISO 9001:2015 Certification \| Nepal Tent | Awards | Organization, BreadcrumbList |
| `clients.html` | Our Clients - UN Agencies, NGOs & Government \| NTT | Our clients | Organization, BreadcrumbList |
| `press-release.html` | Press Release - UNMIN Service Medal \| Nepal Tent | Press release | Organization, BreadcrumbList |
| `contact.html` | Contact Nepal Tent & Tarpaulin, Kathmandu \| NTT | Contact | + ContactPage |

Also: `robots.txt`, `sitemap.xml` (7 URLs).

## Running locally

No build, no dependencies. Serve the directory with any static server:

```sh
python3 -m http.server 8765
# → http://localhost:8765/
```

Opening `index.html` over `file://` also works, though relative links behave
better over HTTP.

## Deploying

Flat `.html` files at the repo root — drop-in for GitHub Pages, Netlify,
Cloudflare Pages, or any web root.

Two things to settle **before** going live:

1. **Canonicals are hardcoded to `https://nepaltent.com.np/<file>.html`**
   (see `<link rel="canonical">` in each page, and `sitemap.xml`). The current
   WordPress site uses different URLs. Replacing it means adding 301 redirects
   from the old paths, or the existing ranking signal is lost. On a preview
   host the canonicals will still point at the production domain.
2. **All images hotlink `nepaltent.com.np/wp-content/uploads/`.** This repo
   contains no image assets, so the site depends on the WordPress install
   staying online. Download them locally and rewrite the `src` attributes
   before cutting over.

## How this was built

The pages originally used `dc-runtime`, a client-side renderer: each
`*.dc.html` file wrapped its body in `<x-dc>` and used `<helmet>`, `<sc-if>`,
`<sc-for>` and `{{ }}` interpolation, rendered into React at runtime by
`support.js` (58KB) plus a React 18 UMD fetch from unpkg. Nothing was in the
HTML source — with JS off, or unpkg unreachable, every page was blank, and the
entire SEO `<helmet>` block only existed after JS ran.

Conversion was mechanical rather than hand-ported, so the output matches the
original exactly:

1. Each page rendered in headless Chromium, letting dc-runtime expand its own
   directives, then the DOM dumped.
2. dc-runtime injects its hover and focus styles through the CSSOM
   (`.scp0:hover`, `.scp2:focus`, …), which a DOM dump does **not** capture — a
   serializer was injected into temp copies to recover them into a real
   stylesheet.
3. `data-dc-tpl`, `sc-interp` wrappers and `sc-*` classes stripped; links
   rewritten from `Home.dc.html` to `index.html` and friends.
4. A fresh `<head>` authored per page, and the header scroll behaviour
   re-emitted as ~1.5KB of plain inline JS.

`index.html` renders byte-identically to the original — before and after
screenshots matched exactly.

The pre-conversion sources are preserved under `_dc-original/` so the
conversion stays reversible. Nothing in the live site loads them.

## Other changes made during conversion

- **Alt text** — 38 client and partner logos all shared `alt="Client logo"` /
  `alt="Partner agency"`. Now derived per logo ("UNICEF logo", "World Food
  Programme logo", …).
- **Contact form** — the dc version was inert: `submit` flipped a state flag to
  show a thank-you and delivered nothing. It is now a real `<form>` with named
  fields, input types, `autocomplete`, `required` and `aria-label`s, composing
  a `mailto:info@nepaltent.com.np`. There is still no backend; a form service
  or mail handler is the next step if `mailto:` isn't acceptable.
- **Director photos** (`contact.html`) — enlarged 96px → 180px and shown in
  colour; the original applied `filter: grayscale(1)`.
- **Accessibility** — skip link, `:focus-visible` rings, `prefers-reduced-motion`
  handling, mobile grid collapse.

## Known debt

- **`products.html` is 181KB**, mostly repeated inline `style` attributes left
  by dc-runtime's expansion. Deduplicating identical style strings into
  generated classes would likely halve it.
- **Company facts** — "established 1980" and "40+ years" appear throughout and
  feed `foundingDate` in the JSON-LD. The live homepage does not state a
  founding year (it does confirm ISO 9001:2015 and UN-enlisted status); the
  claim comes from the company page. Worth confirming with the client.
- No analytics, no favicon, no 404 page.
