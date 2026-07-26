# Deployment

Three Hugo environments map to three purposes. Which one gets built is driven
entirely by which branch triggered the build — nobody hand-edits a config
file per deployment.

| Environment | Where it runs | Trigger |
|---|---|---|
| `development` | Your machine (`hugo server`) | n/a — never deployed |
| `staging` | Cloudflare Pages preview deployment | any push that isn't `main` (a `staging` branch, or a PR) |
| `production` | Cloudflare Pages production deployment | push to `main` |

Indexing (`params.allowIndexing`) defaults to `false` in every environment,
including production — see `site/config/production/hugo.toml`. Flipping the
site live for search engines and AI crawlers is a single-line config change
in that file, made deliberately when Adam signs off, not a side effect of
deploying.

## Cloudflare Pages setup (one-time, in the Cloudflare dashboard)

This repo isn't connected to a Cloudflare Pages project yet — that has to be
done once, by hand, in the dashboard (needs your Cloudflare account):

1. **Pages → Create a project → Connect to Git**, select this repo.
2. **Framework preset:** Hugo.
3. **Root directory:** `site` (the Hugo project lives in a subfolder, not the repo root).
4. **Build command:**
   ```
   if [ "$CF_PAGES_BRANCH" = "main" ]; then hugo --gc --minify --environment production; else hugo --gc --minify --environment staging; fi
   ```
5. **Build output directory:** `public`
6. **Environment variables:**
   - `HUGO_VERSION` — pin to the version used locally (check with `hugo version`; currently `0.163.0`). Cloudflare's build image needs this set explicitly or it'll fall back to an old default.
7. **Production branch:** `main`.

That's it — Cloudflare's Git integration handles preview URLs for every other
branch/PR automatically (free tier), which is what serves as staging. No
GitHub Actions or API tokens needed for this flow.

### Custom staging URL (optional)

By default preview deployments get an auto-generated `*.pages.dev` URL that
changes per-commit. If you want a stable `staging.<project>.pages.dev`
alias, set it up under **Pages project → Settings → Branch deployments**,
then update `baseURL` in `site/config/staging/hugo.toml` to match.

### Going live

When Adam signs off on launch:

1. Point the real domain (`mountainsidedesign.ca`) at the Cloudflare Pages project (**Custom domains** tab).
2. In `site/config/production/hugo.toml`, set `allowIndexing = true`.
3. In `site/static/_headers`, remove the `X-Robots-Tag` line (it currently forces noindex via HTTP header regardless of the meta tag).
