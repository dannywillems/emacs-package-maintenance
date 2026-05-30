# Agent: byte-compile warning fix

A model-agnostic specification of the agent used in this repo to clear
byte-compile warnings in Emacs packages and open upstream pull requests. It is
written so a person, or another model, can run it. One run handles one package
and produces at most one pull request.

## Goal

For a single Emacs package, find a byte-compile warning, make a minimal,
behaviour-preserving, version-compatible fix, ensure CI compiles the package
with warnings treated as errors on the minimum and latest stable Emacs, and
open one focused pull request upstream.

## Scope

- In scope: packages hosted on GitHub that accept pull requests without FSF
  copyright assignment.
- Out of scope: non-GitHub hosts (the fork step assumes GitHub), and GNU ELPA
  packages that require FSF copyright assignment.

## Inputs

- A package name and its upstream repository URL.
- An Emacs binary (a recent build is used to surface warnings; CI exercises the
  declared minimum and the latest stable).

## Process

1. Resolve the upstream URL. If it is not on GitHub, stop and report skipped.
2. Find the real warnings: byte-compile the package with its dependencies on
   `load-path` so cross-package references resolve. Ignore "Cannot open load
   file" errors caused by missing optional dependencies; they are
   environmental, not real warnings.
3. Fork the repo, clone the default branch, add the fork as a remote, and
   create a working branch.
4. Confirm the warning is still present on the default branch. If it is already
   fixed upstream, stop and report skipped.
5. Make the minimal version-compatible fix. Examples:
   - `if-let` / `when-let` obsolete (Emacs 31.1): rewrite with `let` + `if` /
     `when`. Do not switch to `if-let*` / `when-let*` if that would raise the
     minimum Emacs version.
   - `point-at-bol` / `point-at-eol` obsolete (29.1): use
     `line-beginning-position` / `line-end-position`.
   - unused lexical variable: drop it, or prefix with an underscore.
   - docstring with unescaped single quotes: rephrase or escape.
   - missing `lexical-binding` cookie: add it only if the file clearly does not
     rely on dynamic binding; otherwise skip.
   If the package already uses a feature above its declared minimum (so the true
   minimum is higher and it does not byte-compile on the declared one), correct
   `Package-Requires` to the true minimum and say so. Do not raise the minimum
   merely to silence a warning that has a version-compatible rewrite.
6. Verify: byte-compile the changed files with warnings as errors and confirm
   the target warning is gone with no new warnings; check balanced parens.
7. CI. If the repo has no workflow, add one that byte-compiles (warnings as
   errors) and loads/tests the package on the declared minimum Emacs, the latest
   stable, and the development snapshot, plus dependabot for the actions. If a
   workflow exists, make sure it tests the latest stable and uses
   warnings-as-errors; add the missing piece minimally rather than rewriting it.
8. Commit the fix and the CI change as separate commits, push to the fork, and
   wait for the fork CI to pass.
9. Only if CI is green and the fix is clean, open one pull request to the
   upstream default branch, then add a short comment noting this is an automated
   community-maintenance effort.

## Hard rule

Only open a pull request when the fix is minimal, clearly correct,
behaviour-preserving, version-compatible, and CI is green. If anything is
uncertain, nuanced, behavioural, or CI cannot be made green, do not open a
pull request: report skipped with a reason. A skip is a good outcome; a wrong
pull request is not.

## Status

Active. This agent produced the entries in the fixes log.
