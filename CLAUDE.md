# DeThenCo website — project context

This file is for Claude Code: read it first in any new session on this project. It captures how this site is built, deployed, and the decisions/gotchas behind it, so context isn't lost when work continues from a different machine.

## What this is

The public website for DeThenCo (Decoding the Non-Coding), a University of Siena biotech spin-off working on circulating RNA biomarkers for precision medicine. Built with Hugo (the "bigspring-light-hugo" theme, Tailwind CSS v4), deployed free on GitHub Pages.

- **Live site:** https://dethenco.github.io/
- **Repo:** https://github.com/DethenCo/dethenco.github.io (public repo — see "Why public" below)
- **LinkedIn:** https://www.linkedin.com/company/dethenco-decoding-the-noncoding/home/

## Repo mode: project-setup (not theme-setup)

This theme ships in two structures. This repo is in **project-setup**: site content at root (`content/`, `config/`, `data/`, `hugo.toml`), theme vendored under `themes/bigspring-light-hugo/`. See `AGENTS.md` for the full detection rules and package-manager detection (this project uses **npm**).

## Deployment

- GitHub Actions workflow `.github/workflows/pages.yml` builds and deploys to GitHub Pages on every push to `main`. Installs Go (for Hugo Modules resolution — several `gethugothemes/hugo-modules/*` components are remote modules, not vendored), Node, Hugo **0.163.3 extended** (pinned — must match `netlify.toml`'s `HUGO_VERSION`), runs `npm install && npm run project-setup && npm run build`.
- `baseURL` is overridden to `https://dethenco.github.io/` via the `HUGO_BASEURL` env var in that workflow (same mechanism Netlify uses in `netlify.toml`). `hugo.toml`'s own `baseURL = "/"` is the local-dev default — don't change it for GitHub Pages; override via env var instead.
- A `netlify.toml` also exists (this project was originally going to deploy there) but is currently unused — GitHub Pages is the live deployment. Its build command still says `pnpm` even though the project uses npm; fix that if Netlify is ever actually used.
- **Why the repo is public:** GitHub Pages requires a paid plan (GitHub Pro) to publish from a private repo on a personal account. Free tier requires public. Nothing sensitive is in the repo — see Web3Forms note below.

## Running locally

Follow `AGENTS.md`'s pre-flight checks (Hugo version must exactly match `HUGO_VERSION` in `netlify.toml`, use `mise` to pin it if not). Key commands: `npm install`, `npm run dev`, `npm run build`.

**Known dev-server gotcha:** Hugo's dev server can serve stale/deleted pages from the gitignored `public/` directory after content deletions (it writes to disk, not just memory, in this setup). If a deleted page still seems to exist after restarting, `rm -rf public` and restart (`npx kill-port 1313 -y && npm run dev`).

## Contact form: Web3Forms, not Netlify Forms

The form (`layouts/_default/contact.html`) posts via client-side `fetch()` to `https://api.web3forms.com/submit`. This was a deliberate switch from Netlify Forms once GitHub Pages was chosen as the host — GitHub Pages has no backend, so Netlify's form-catching mechanism (which works by Netlify's own infrastructure intercepting the POST) doesn't apply there.

- Destination inbox: `dethenco@gmail.com`
- The access key hardcoded in the form is **meant to be public** — that's how Web3Forms works, it's not a secret credential, don't treat it as one.
- Confirmed working end-to-end on the live site (tested by the user).

## Content structure

- `content/english/_index.md` — homepage. The 3-service preview here must stay in sync with the real list in `services/_index.md` (titles/wording), it's explicitly a preview, not the full list.
- `content/english/about.md` — team, values, mission, fun facts. Team is currently: Giuseppina Emanuela Grieco (CEO), Daniela Fignani (Head of R&D), Prof. Francesco Dotta, Prof. Guido Sebastiani, Elena Aiello, Erika Pedace, Mohamad Elrifai. **Giada Licata was intentionally removed from this list** per explicit request, though she's still credited by name in one news article as a project contributor — that's correct, not an inconsistency to fix.
- `content/english/services/_index.md` — 6 services, each with a FontAwesome icon chosen to actually represent it (not generic/mismatched): `fa-dna` (small RNA sequencing), `fa-bullseye` (targeted panels), `fa-flask` (extraction/QC), `fa-ruler-horizontal` (long non-coding RNA), `fa-laptop-code` (bioinformatics), `fa-box-archive` (retrospective trial service). Custom layout: `layouts/_default/services.html`.
- `content/english/publications/_index.md` — Prof. Guido Sebastiani's full PubMed bibliography (72 papers as of last update), custom layout `layouts/_default/publications.html`, grouped by year descending. **Important if this is ever refreshed:** PubMed has a *different* person, "Sebastiani GD" (a rheumatologist, unrelated), whose papers must be excluded — verify by checking for DeThenCo's actual collaborators (Dotta F, Grieco GE, Fignani D, Nigi L, Licata G, Aiello E, Pedace E) as co-authors.
- `content/english/news/` — company news/announcements. This replaced a generic "Blog" section. Custom layouts: `layouts/news/list.html`, `layouts/news/single.html`. **Never fabricate photos for real events** (awards, conferences) — only use real photos the user provides; it's fine to leave `image: ""` if none exist yet.
- `content/english/contact.md` — `dethenco@gmail.com` listed first as the official address, individual team `@unisi.it`/`@student.unisi.it` emails listed below as alternates.

## Images

- `static/images/` — team headshots (`firstname-lastname.jpg`), service card photos (sourced from NIAID/NCI via Unsplash — free license, no attribution required), news photos.
- **Always view an image with the Read tool before wiring it into content** — this caught real mistakes in this project (e.g. a "3-panel composite" photo that needed cropping before use, and confirming AI-generated-looking filenames actually matched the right person).
- Non-JPEG/PNG sources (e.g. `.tif` team photos) were converted with Python/Pillow. **Note:** on this Windows setup, the bare `convert` command resolves to Windows' own `system32/convert.exe`, not ImageMagick — don't use it for images.

## Git history note

The very first push to this repo (`DethenCo/dethenco.github.io`) failed repeatedly with `did not receive expected object ... unpack failed`, reproducibly, across different networks and transports (HTTPS and SSH). It turned out to be something about the *inherited* git history (this repo's local clone originally pointed at `gethugothemes/bigspring-light-hugo`, the upstream theme template, before being repointed here). The fix was squashing everything into a single fresh orphan commit and force-pushing that — history before the current `main` doesn't exist upstream. If a similar transfer error ever recurs after a lot of accumulated history, that's a reasonable thing to try again.

## Social links

Only LinkedIn is active. Facebook/Twitter/Instagram/GitHub icons were deliberately removed from `config/_default/params.toml` (`[[social]]` entries) since those accounts don't exist — don't re-add them without being asked.
