# Agent guide

- This is a single Hugo site; `config.yaml` is the root configuration and `content/` contains pages and dated posts.
- Initialize the theme before building: `git submodule update --init --recursive`. `themes/hugo-PaperMod` is a Git submodule; keep site-specific changes in root `assets/` and `layouts/` rather than editing the vendored theme.
- Use the Hugo **extended** build. The local verification command is `hugo --gc --minify`; the GitHub Pages workflow builds with Hugo 0.165.0, Dart Sass, recursive submodules, and `--baseURL` supplied by Pages.
- There is no package manager, test suite, linter, or typecheck configured; a successful Hugo build is the focused verification step.
- `public/` is generated output and ignored by Git; do not commit it. The deployment workflow uploads it and deploys pushes to `main` (or manual dispatch).
- New content must use the front matter pattern in existing posts. Local/CI production builds exclude `draft: true`, future-dated, and expired content (`buildDrafts`, `buildFuture`, and `buildExpired` are all false).
- The root `assets/` and `layouts/partials/` files override PaperMod behavior; check these before changing theme files. `config.yaml` enables unsafe HTML rendering, so preserve that requirement when posts use inline HTML.
