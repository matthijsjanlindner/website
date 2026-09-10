# Academic website template (Quarto)

A five-page academic homepage: landing page with photo and affiliation, plus
Research, Teaching, Outreach and Contacts. Push to `main` and GitHub Actions
renders and publishes it — no server, no build step to run by hand, free hosting.

Styling is **stock Quarto** (cosmo in light mode, darkly in dark mode). Nothing
here is themed, so it is a starting point rather than someone else's design.

---

## 1. What you need

Quarto. If you use RStudio you already have it — RStudio ships its own copy at:

```
/Applications/RStudio.app/Contents/Resources/app/quarto/bin/quarto
```

Otherwise install it from <https://quarto.org/docs/get-started/>. Check with:

```
quarto --version
```

Whatever that prints, put the same version in `.github/workflows/publish.yml`
so the site CI builds matches the one you see locally.

## 2. Working on it locally

In RStudio: open `website.Rproj` and click **Render**. It opens a live preview
that refreshes as you save.

From a terminal:

```
quarto preview
```

One rule: **run only one renderer at a time.** `quarto preview` is not a passive
server — it renders the site itself and serves its own in-memory copy. If
something else writes `_site` underneath it, the preview never notices: your
source is correct, the browser shows the old page, and the edit looks like it
vanished. If a change refuses to appear, stop everything and:

```
rm -rf _site .quarto && quarto render
```

## 3. Publishing it

1. Set `site-url:` in `_quarto.yml` to where the site will actually live.
   Do this **first** — Quarto compiles that path into the links on the 404 page,
   so if it still says `YOUR-REPO` every link on that page points at a repository
   that does not exist. For a project site it is
   `https://YOURNAME.github.io/YOUR-REPO`; for a user site, `https://YOURNAME.github.io`.
2. Create a **public** repository on GitHub and push this folder to `main`.
   (Public is required for GitHub Pages on a free account.)
3. Repo -> **Settings -> Pages -> Source: Deploy from a branch**, branch
   `gh-pages`, folder `/ (root)`.
4. Push once more. The Actions tab will show "Publish site" running; when it
   goes green the site is live at `https://YOURNAME.github.io/YOUR-REPO`.

If the run fails with a permissions error, set repo -> **Settings -> Actions ->
General -> Workflow permissions** to **Read and write**.

After that, publishing is just:

```
git add -A && git commit -m "Update" && git push
```

Because CI renders *and* deploys in one step, a broken render never reaches the
live site — the deploy simply does not run and the previous build keeps serving.
You get a red X in Actions, not a broken website. That makes it safe to edit
from GitHub's web editor or the GitHub mobile app, without Quarto installed.

### Custom domain

Buy the domain, point a CNAME record at `YOURNAME.github.io`, then create a file
called `CNAME` in this folder containing just the domain. Uncomment the
`resources:` block in `_quarto.yml` and list it there — otherwise Quarto wipes
it on every deploy and the domain breaks each time you publish.

## 4. What to edit

| File | What it is |
|---|---|
| `index.qmd` | Landing page. Name, affiliation, photo, links, opening text |
| `research.qmd` `teaching.qmd` `outreach.qmd` `contacts.qmd` | Content pages |
| `404.qmd` | Not-found page. **Links must stay absolute** (`/research.html`) |
| `_quarto.yml` | Site title, navbar, footer, theme, social cards |
| `styles.scss` | All custom styling. Empty by default |
| `images/` | Replace `profile.svg` with your own photo |

The affiliation lives in the `subtitle:` of `index.qmd`, separated by `<br>`.
There is no affiliation field — Quarto's `about:` block accepts only `id`,
`template`, `image`, `image-alt`, `image-title`, `image-width`, `image-shape`
and `links`.

## 5. Things that will bite you

**Body comments publish.** `<!-- ... -->` in the body of a `.qmd` passes straight
into the output HTML and is readable in view-source. Only `#` comments inside a
YAML header are stripped. Keep working notes in a separate file.

**Only `.qmd` files render.** `project: render: ["*.qmd"]` in `_quarto.yml` is
what stops stray `.md` files (this README included) becoming public pages.

**`description:` prints on the page.** It looks like the natural key for a search
snippet, but it also renders as a visible subtitle under the title. That is why
every page here sets the snippet as a raw `<meta>` tag via `include-in-header`
plus `open-graph`/`twitter-card` instead.

**Custom CSS must go through `styles.scss`, not `css:`.** The theme pipeline
gives the output a content-hashed filename, so an edit always changes the URL and
can never be served from a stale browser cache. A plain `css: styles.css` causes
exactly that bug — for you and for returning visitors.

**Bootswatch themes fetch fonts from Google.** cosmo `@import`s Source Sans Pro
and darkly imports Lato, used or not: two render-blocking requests per page, and
every visitor's IP sent to Google. That may matter for GDPR at a European
institution. Overriding the font stack does *not* stop it — you need
`$web-font-path: false;` in `styles.scss` and a self-hosted `@font-face`. Verify
with `grep -rl fonts.googleapis _site`.

**Quarto's selectors carry element qualifiers.** Its own rules look like
`div.quarto-about-trestles .about-entity`. A classes-only override has lower
specificity and silently loses. Match the qualifier.

**`url()` in SCSS resolves from the project root**, not from the compiled
stylesheet. Use `url("fonts/x.woff2")`; a path like `../../fonts/...` fails the
build.

**Sizing text to fill a column is fragile.** If you size a heading so it just
fits, remember it renders in a *fallback* font for the first ~100ms of a cold
load. On macOS that fallback is often several percent wider, so the text wraps
and then snaps back once the webfont arrives. Leave headroom for the widest font
in your stack, not the one you chose.

## 6. Licence

Do what you like with it.
