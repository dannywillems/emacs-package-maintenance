# emacs-package-maintenance

A small log of byte-compile-warning fixes contributed upstream to
widely-used Emacs packages. Published at
<https://dannywillems.github.io/emacs-package-maintenance/>.

## What this is

Many Emacs packages, especially on MELPA, are lightly maintained. As Emacs
evolves, byte-compile warnings accumulate (obsolete functions and macros,
signature changes, missing lexical-binding cookies). This repo tracks small,
focused patches that clear those warnings, one package at a time, and the
pull requests that carry them upstream. It is community maintenance work,
AI-assisted. It is not a product and is not affiliated with any company.

## Process

For each package:

1. Verify the warning is still present on the upstream default branch.
2. Make a minimal, behaviour-preserving fix that stays compatible with the
   package's supported Emacs versions (prefer rewrites that do not raise the
   minimum).
3. Ensure CI byte-compiles with warnings treated as errors on the minimum
   supported Emacs and the latest stable (plus a development snapshot); add a
   workflow if none exists, or extend an existing one.
4. Open one pull request per fix.

## The site

The site is a static page built by Jekyll in GitHub Actions and served from
GitHub Pages. The fixes log is data-driven: it is rendered from
[`_data/fixes.yml`](_data/fixes.yml).

### Adding a fix

Prepend a new entry to the top of `_data/fixes.yml` (newest-first) and push to
`main`. The Actions workflow rebuilds and redeploys the site. Each entry has:
`package`, `upstream_name`, `upstream_url`, `warning`, `fix`, `ci`, `pr_url`,
`pr_label`, `status` (Submitted, Merged, Closed, In progress), and an optional
`note`.

## Layout

- `_data/fixes.yml` - the fixes log (the data)
- `index.html` - Jekyll template rendering the log + the prose sections
- `assets/styles.css` - styling
- `.github/workflows/pages.yml` - build and deploy
