# SAKSHI website

Static website for SAKSHI (Structured Accountability through Knowledge,
Scrutiny, and Health Integrity), a community-operated investigative platform
for accountability in clinical AI deployment. SAKSHI is a project of MIT
Critical Data, the Laboratory for Computational Physiology at MIT.

## Why this repository is public

The website is part of SAKSHI's evidentiary record, and so is this
repository. Every change to every page, and above all every change to a
published profile, is a visible commit with a date and an author. Nothing on
the site is edited silently. Readers, profiled companies, journalists and
regulators can check what the site said at any point in time.

Profile pages are therefore never hand-edited. Each profile is a structured
data file in `profiles-data/`, validated against `profile-schema.json` and
rendered to HTML at build time. A correction to a published finding is a
change to the data file plus an entry in its `correction_log`. The other
pages are rendered the same way from `content/pages/`, so every change to
any page is a visible commit to a source file.

No profiles have been published. `profiles-data/` is empty by design and
contains no sample or placeholder profiles.

## Repository layout

```
content/
  pages/                Source of every page except profiles (HTML body fragments)
  pages.json            Page list: file, output path, title, description, citation note
  partials/             Generated fragments pulled in with <!-- include:name --> (the world map)
tools/
  make-world-map.py     Regenerates the contributors map from Natural Earth land data
site/                   Build output plus committed static assets
  assets/               style.css, site.js, template.js, mark.svg, the MIT logo (mit-logo.svg)
  assets/vendor/        License for the DOJO design system the stylesheet is adapted from
  _headers              Security headers served by Cloudflare Pages
  _redirects            Old /team/ page redirects to /#team (Cloudflare)
  .nojekyll             Tells GitHub Pages to serve files as they are
  *.html, */index.html  Rendered at build time (not committed)
  profiles/             Rendered at build time from profiles-data/ (not committed)
profiles-data/          One JSON file per profile. Empty.
profile-schema.json     Documented JSON Schema for profile data files.
build/
  lib.js                Shared page layout, navigation, footer, citation block
  render-pages.js       Renders content/pages/ into site/
  render-profiles.js    Renders site/profiles/ from profiles-data/. Fails on invalid data.
  check-links.js        Checks that every internal link and fragment resolves.
.github/workflows/
  validate.yml          CI: build, HTML validation, link check on every push and pull request
  deploy-pages.yml      Deploys main to GitHub Pages
wrangler.toml           Cloudflare Pages output directory.
```

## Working locally

Requirements: Node 20 or later. The site itself has no runtime dependencies;
Node is used only to render profile pages and to run checks.

```
npm ci
npm run build          # render all pages and site/profiles/ into site/
npm run validate       # html-validate on every page
npm run check-links    # every internal link and #fragment must resolve
npm test               # all three
npm run test:pages     # the same, built for the GitHub Pages subpath
npm run serve          # http://localhost:8080 (needs python3)
```

Editing a page is editing its body fragment in `content/pages/` and running
`npm run build`. The header, status line, footer and citation block come
from `build/lib.js`.

The site uses root-relative links. Set `SITE_BASE` (for example
`/sakshi-project`) when building for a site served under a subpath, as
`npm run test:pages` does. Leave it unset for a domain root, which is how
the GitHub Pages workflow builds for sakshi-project.mit.edu.

## Design

The visual system (palette, type, header, section layout, cards) is adapted
from the DOJO site by MIT Critical Data, released under the MIT License
(https://github.com/criticaldata/dojo). The license is kept at
`site/assets/vendor/LICENSE-dojo.txt`. The Inter typeface loads from Google
Fonts; everything else is self-hosted.

The footer of every page links to this repository, so each page points at
its own edit history. The URL is set once as `repoUrl` in `build/lib.js`,
and the contact address shown in the footer as `email` next to it.

## MIT brand requirements

The site follows MIT's brand rules (https://brand.mit.edu):

- The MIT logo (`site/assets/mit-logo.svg`, the official black logo) sits at
  the top right of every page and links to https://web.mit.edu. The SVG's
  viewBox includes the required clear space of one bar width on every side,
  so nothing can be placed closer to the mark than the rules allow.
- The lab is always named in text as "MIT Critical Data". The logo is never
  combined with the lab name or any other text or graphic.
- The footer carries a contact email address.

## Adding or correcting a profile

1. Create `profiles-data/<slug>.json` following `profile-schema.json`. The
   `slug` field must equal the filename.
2. Run `npm test`. The build fails with a field-by-field report if the file
   does not match the schema.
3. Open a pull request. The profile is published only after the editorial
   process described on the Governance page has run.
4. For a correction, change the data and append an entry to `correction_log`
   with the date, a description, and the trigger. Do not rewrite the original
   finding without recording the change.

## Contact form

The contact form linked from the home page and from Get Involved is hosted
on Tally (form `RWopO9`, in the project owner's Tally workspace). It is a
plain link, not an embed, so the site still loads no third-party scripts.
Email notifications are turned off; replies are read and exported from the
Tally dashboard. To move the form elsewhere,
change the two links in `content/pages/home.html` and
`content/pages/get-involved.html`.

## Deploying to GitHub Pages

`.github/workflows/deploy-pages.yml` builds the site for the domain root and
deploys it to GitHub Pages on every push to `main`. The site is served at
https://sakshi-project.mit.edu/. Requirements:

1. The repository must be public (GitHub Pages on a free plan does not
   serve private repositories).
2. Under Settings, Pages, set Source to "GitHub Actions". The workflow
   attempts to enable this itself on first run; if that step fails, set it
   once by hand and re-run the workflow.
3. Under Settings, Pages, Custom domain, enter `sakshi-project.mit.edu`
   and save. Once GitHub reports the DNS check as successful, tick
   "Enforce HTTPS". `site/CNAME` records the same hostname; GitHub ignores
   the file for Actions deployments, so the setting is what counts.

### DNS records for sakshi-project.mit.edu

MIT IS&T manages the mit.edu zone, so these records are requested from them
rather than set directly.

| Type  | Name                                                | Value                      |
| ----- | --------------------------------------------------- | -------------------------- |
| CNAME | `sakshi-project.mit.edu`                            | `criticaldata.github.io`   |
| TXT   | `_github-pages-challenge-criticaldata.sakshi-project.mit.edu` | verification code from GitHub |

The CNAME record is what serves the site. The TXT record is optional but
recommended: it lets the `criticaldata` organisation verify the domain under
Organization settings, Pages, Verified domains, which stops anyone else's
Pages site from claiming the hostname. GitHub shows the code to put in the
TXT record when the verification is started.

The hostname must not have a CAA record that excludes Let's Encrypt, which
GitHub uses to issue the certificate. If `mit.edu` publishes CAA records,
ask IS&T to confirm `letsencrypt.org` is permitted for this hostname.

To serve the site under the GitHub project subpath instead
(`https://criticaldata.github.io/sakshi-project/`), remove the custom
domain and have the workflow run `npm run test:pages`, which sets
`SITE_BASE=/sakshi-project`.

## Deploying to Cloudflare Pages

Cloudflare Pages builds the site from this repository on every push to the
production branch and serves it from Cloudflare's edge.

Build settings:

| Setting                | Value            |
| ---------------------- | ---------------- |
| Framework preset       | None             |
| Build command          | `npm run build`  |
| Build output directory | `site`           |
| Root directory         | `/`              |
| Node version           | 22 (set `NODE_VERSION=22` as an environment variable if the default is older) |

### Connecting the repository

1. Push this repository to GitHub as a public repository.
2. In the Cloudflare dashboard, open Workers & Pages, choose Create, then
   Pages, then Connect to Git.
3. Authorize Cloudflare's GitHub app for the organization or account that owns
   the repository, and select the repository.
4. Set the production branch (usually `main`), the build command and the
   output directory as in the table above. Leave the framework preset as None.
5. Save and deploy. The first build gives the site a `*.pages.dev` address.
   Every later push to the production branch deploys automatically; pushes to
   other branches get preview deployments at their own addresses.

`wrangler.toml` records the output directory for anyone deploying with the
Wrangler CLI instead of the dashboard:

```
npx wrangler pages deploy site --project-name sakshi
```

## License

Content is licensed under Creative Commons Attribution 4.0 International
(CC BY 4.0); see `LICENSE`. The small build scripts in `build/` may be reused
under the same terms.
