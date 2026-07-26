# Mountainside Design — repo guide

Website for Adam's architectural design practice (Collingwood, Ontario).
Word-of-mouth business — the site's job is to let someone who was told
"call Adam" quickly confirm this is the person their friend meant, not to
run a marketing funnel.

## Layout

- `site/` — the Hugo project. Everything needed to build the website lives here.
- `docs/` — planning docs (business context, deployment notes). Not part of the built site.
- Repo root — otherwise reserved for dev tooling / CI later; nothing there yet.

**The site is currently a design preview.** It is not live and is blocked
from indexing/crawling in every environment (`params.allowIndexing = false`)
until Adam explicitly signs off — see `docs/deployment.md` for how that gets
flipped.

## Working in `site/`

```
hugo server -D --environment development    # local dev, http://localhost:1313
hugo --gc --minify --environment staging     # what Cloudflare builds for previews
hugo --gc --minify --environment production  # what Cloudflare builds for main
```

### Structure

- `config/_default/hugo.toml` + `config/{development,staging,production}/hugo.toml` — environment-specific overrides (baseURL, `allowIndexing`, labels). Add a new environment by adding a folder here.
- `content/_index.md` — all homepage copy (hero, about, section headings/intros) lives in front matter, not in templates. Edit this file to change what the page says.
- `content/{work,plans,services}/*.md` — one file per portfolio project / pre-designed plan / service. These are headless collections (`build.render: never`) — rendered only via `range` on the homepage, no standalone URLs. **To add a new project/plan/service, copy an existing file in that folder** — no template changes needed.
- `layouts/` — `_default/baseof.html` is the shell; `partials/` has one file per page section (hero, about, work, plans, services, contact) plus `head.html`/`header.html`/`footer.html`. `layouts/index.html` just assembles the section partials in order.
- `layouts/partials/responsive-img.html` — generates a `<picture>` with webp + jpg at multiple widths from any image in `assets/images/`. Use this for any real photo that gets added (see below); don't hand-write `<img>` tags for content photography.
- `assets/css/style.css`, `assets/images/*` — processed through Hugo Pipes: minified + fingerprinted (cache-busted filename) in every environment except `development`, where they're served unprocessed for fast rebuilds.
- `static/_headers` — Cloudflare Pages reads this directly for response headers (the noindex `X-Robots-Tag`, cache headers for fingerprinted assets). Not templated — see `docs/deployment.md` for what to change here at launch.

## Content status (as of this Hugo conversion)

Everything is a placeholder pending real input from Adam:

- **Portfolio photos, plan photos, Adam's headshot** — all placeholder boxes/images with captions describing what's needed. Real photos go in `assets/images/` and get wired up via the `responsive-img.html` partial.
- **Pricing** (plan prices, fixed-service prices) — round placeholder numbers, flagged "PLACEHOLDER PRICE" in the UI. Confirm real numbers with Adam before launch.
- **Contact form** — submits via `mailto:`, no backend. Fine for a preview, not for launch — swap the form action for a real handler (Cloudflare Pages Forms, Formspree, etc.) first.
- **Phone number, Adam's last name** — blank/bracketed placeholders in `content/_index.md`.
- Portfolio project descriptions are lorem ipsum; everything else (hero copy, About narrative, section intros) is real drafted copy based on `docs/initial copy prompt.md`.

## Deployment

See `docs/deployment.md` — Cloudflare Pages, Git-integration based (no CI
config in this repo). Production = `main` branch, staging = any other
branch/PR (Cloudflare preview deployments).
