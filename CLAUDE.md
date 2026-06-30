# Wine Coding — project guide

Personal blog ("Wine Coding") by Ivan Shafran about Android & tech. Built with
**Jekyll** + the **Minimal Mistakes** theme (loaded as a remote theme), hosted on
**GitHub Pages** at <https://shafran.dev>.

## Deploy

- Pushing to the **`master`** branch triggers the GitHub Pages build — there is no
  separate CI step. `master` is the live branch.
- The site uses `remote_theme: mmistakes/minimal-mistakes`, so the theme's files
  are **not** in this repo. Customizations are done via overrides (see below).
- The domain is set by `CNAME` (`shafran.dev`) and `url:` in `_config.yml`.

## Local build (optional)

`bundle exec jekyll serve` — the toolchain needs **Ruby ≥ 3.0**; the repo's `Gemfile`
pins `github-pages`. Most changes are verified by inspection + the GitHub Pages build
rather than a local run.

## Layout of the repo

| Path | Purpose |
|------|---------|
| `_posts/` | Blog posts (`YYYY-MM-DD-title.md`). |
| `_pages/` | Standalone pages — `about.md`, `talks.md`, archive pages. |
| `_data/navigation.yml` | Top nav menu. |
| `_data/talks.yml` | Source data for the Talks timeline. |
| `_layouts/home.html` | **Custom** home layout: magazine card grid for posts (inherits the theme's `archive` layout, so the author sidebar is untouched). |
| `_includes/head/custom.html` | Analytics (gtag), favicons, and the Fraunces web font. |
| `assets/css/main.scss` | All custom styling (brand skin overrides + custom components). |
| `assets/images/` | Images; post teasers go in `assets/images/posts/`. |

## Design system

- **Theme skin:** Minimal Mistakes `dark`, overridden with a wine/burgundy accent.
  Brand colors live at the top of `assets/css/main.scss`:
  `$primary-color: #8c2f39` (wine), `$link-color: #d06b76` (lighter wine for links).
- **Typography:** body/headings use the theme's system sans. **Fraunces** (a display
  serif, loaded in `custom.html`) is applied **only** to the "Wine Coding" masthead
  title — not to other headings.
- **Custom components** in `main.scss`:
  - `.talks-timeline` — the Talks page (vertical wine line, year markers, date/
    conference/language badges).
  - `.post-grid` / `.post-card` — the home page magazine cards (2 columns on
    desktop, 1 column ≤600px).

## Common tasks

- **Add a talk:** append an entry to `_data/talks.yml` (newest first). Fields:
  `date` (YYYY-MM-DD), `title`, `url` (optional), `conference`, `lang` (EN/RU).
  The timeline renders itself.
- **Add a post:** create `_posts/YYYY-MM-DD-title.md` with front matter
  (`title`, `date`, optional `tags`). Posts use the permalink `/:title/`
  (categories are intentionally not used).
- **Add a post image (card thumbnail):** put a ~1200×630 landscape JPG in
  `assets/images/posts/` and reference it in the post's front matter:
  ```yaml
  header:
    teaser: /assets/images/posts/my-post.jpg
  ```
  Cards without a teaser render text-only (no broken image).
- **Social link previews:** `og_image` in `_config.yml` is the default share image;
  a post can override with `header.image`.

## Conventions

- 10 posts per page (`paginate: 10`).
- Keep commits scoped; design changes are CSS/Sass + Liquid, no theme forking.
