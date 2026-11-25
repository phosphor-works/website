<!-- SPDX-FileCopyrightText: 2026 Phosphor Works Team -->
<!-- SPDX-License-Identifier: GPL-3.0-or-later -->

# phosphor-web

phosphor-web is the umbrella site of the **Phosphor** library suite, at <https://phosphor-works.org>. The suite is a
set of Qt6 / Wayland window-management primitives that [PlasmaZones][pz] and other downstream projects use. The site
uses [Astro][astro].

## Contents

```
src/
  pages/        route files (.astro / .ts) — one file per URL
  layouts/      BaseLayout shared by every page
  components/   reusable UI (Topbar, Sidebar, Cards, Tabs, search, …)
  content/      content collections (news), schema in content.config.ts
  data/         site data (libraries, nav, palette, OG, plasmazones/*)
  styles/       global CSS (home.css, guide.css)
  scripts/      client-side TS (theme toggle, search, palette)
  themes/       Expressive Code theme built from the palette
public/         static assets served as-is (.nojekyll, palette CSS, robots)
docs/           Doxygen tooling (Doxyfile + header/footer + CSS) for the API ref
scripts/        build + data-sync tooling (build-docs.sh, sync-*.mjs)
vendor/         the pinned branding release, from `npm run fetch:branding` (gitignored)
```

## Use

Clone the [upstream checkouts](#upstream-checkouts) next to this repository. Then install the dependencies and start
the development server:

```sh
npm install
npm run dev          # astro dev — http://localhost:4321
```

`npm run dev` and `npm run build` first run `npm run fetch:branding`. This script downloads the pinned branding
release into `vendor/branding/`, one time for each version.

Other scripts:

```sh
npm run build        # production build to dist/
npm run preview      # serve the built dist/ locally
npm run check        # astro check (type + content-collection checks)
```

Site search uses [Pagefind][pagefind]. `npm run build` makes the search index. Thus search works under
`npm run preview`, but not under `npm run dev`.

## Brand assets and palette

The logos, the icons and the canonical palette are in
[`phosphor-works/branding`](https://github.com/phosphor-works/branding). The site pins a branding release by version,
URL and SHA-256, as PlasmaZones and Lambent do. The pin is in `scripts/fetch-branding.mjs`.

`npm run fetch:branding` downloads the release, checks the hash and unpacks the release into `vendor/branding/`.
`public/brand/assets` and `public/palette/phosphor.toml` are symlinks into its `svg/` and `palette/`.
`src/data/palette.ts` reads the TOML for the palette page and the Expressive Code theme.

To use new branding, follow these steps:

1. Make a branding release.
2. Get the hash from the `.sha256` asset of the release.
3. Change the four pin values together.

## Upstream checkouts

The site builds the gallery data, the API reference and the Afterglow pages from other repositories. These
repositories are checked out next to this one:

| Checkout | Used for | Override |
| --- | --- | --- |
| `../PlasmaZones` | layouts, overlays, animations, scrolling templates, algorithms, changelog, D-Bus XML | `PLASMAZONES_SRC` |
| `../phosphor-surface-packs` | the surface shader gallery | `PHOSPHOR_SURFACE_PACKS_SRC` |
| `../phosphor-libs` | core library headers, the Luau pin for the algorithm previews | `PHOSPHOR_LIBS_SRC` |
| `../phosphor-engine-libs` | placement engine headers, the `pluau` prelude for the algorithm previews | `PHOSPHOR_ENGINE_LIBS_SRC` |
| `../phosphor-shell-libs` | shell library headers | `PHOSPHOR_SHELL_LIBS_SRC` |
| `../Afterglow` | the Afterglow docs pages | `AFTERGLOW_SRC` |

To look for all of them in another directory, set `PHOSPHOR_WORKS_SRC`. To use a different location for one
checkout, set the variable of that repository.

## PlasmaZones gallery data

`src/data/plasmazones/*.json` are generated files. A script syncs them from the checkouts above. Edit them upstream,
not here. Then make them again:

```sh
npm run sync:plasmazones
PHOSPHOR_WORKS_SRC=~/src npm run sync:plasmazones
```

The algorithm previews need the `luau` binary on `PATH`. Without it, the script does not change the existing
`algorithms.json`. See `scripts/sync-plasmazones-data.mjs`.

## API documentation (Doxygen)

The build makes the Phosphor API reference under `/api/` from the `libs/phosphor-*/include/` headers. These headers
are in the sibling [phosphor-libs](https://github.com/phosphor-works/phosphor-libs) and
[phosphor-shell-libs](https://github.com/phosphor-works/phosphor-shell-libs) checkouts. The D-Bus pages come from
PlasmaZones. `docs/mainpage.md` is the start page of the reference. Install `doxygen` and `graphviz`, then run:

```sh
npm run docs                                # uses the sibling checkouts
PHOSPHOR_WORKS_SRC=~/src npm run docs       # look for them somewhere else
npm run docs:clean                          # fresh rebuild
```

The output is `api/html/index.html`. Git ignores `api/`. The deploy workflow builds it again each time and adds it to
the deployed site.

## Deploy

`.github/workflows/deploy.yml` builds the site on each push to `main` and one time each night. It does these steps:

1. It runs `npm run build`.
2. It builds the Doxygen reference.
3. It merges `api/` into the Astro `dist/`.
4. It packages the result as one tarball.

The deploy job uploads that tarball over SFTP to the project server. The server checks it and switches
<https://phosphor-works.org> to it in less than one minute. The server keeps the previous releases for rollback. The
SSH key and the pinned host key of the server are in the `vps` environment. Only `main` can deploy to it.

Each deploy first updates the data that the site copies from other repositories. It clones the upstream repositories
above at `main` into `_src/`, and runs `npm run sync:plasmazones` against them. The deploy commits none of that data
back. Thus `src/data/plasmazones/` in this repository is only the start point of a local build.

`settings.json` is the exception. It needs a built PlasmaZones tree, so the deploy uses the committed copy. You must
sync and commit it by hand.

## License

The code is licensed under `GPL-3.0-or-later`, and the license text is in `LICENSE`. The palette hex values are facts
and have no license restrictions. The brand assets from the branding repository are licensed under CC-BY-SA 4.0.

[pz]: https://github.com/phosphor-works/PlasmaZones
[astro]: https://astro.build/
[pagefind]: https://pagefind.app/
