# Agent: maintenance health check

A read-only agent that produces a maintenance-health snapshot for one Emacs
package. It makes NO code changes and opens NO pull request; its output is the
data that feeds the coverage table, so coverage can be refreshed over time.

## Goal

Answer, for a single package: how well is its continued support verified, and
is it currently healthy to depend on?

## Inputs

- A package name and its upstream repository URL.
- A recent Emacs binary.

## Checks

1. Activity: date of the last commit on the default branch; number of open
   issues and open pull requests; whether the repo is archived.
2. CI: does a CI workflow exist? Which Emacs versions does it exercise? Does it
   byte-compile with warnings treated as errors, and does it run a test suite?
3. Declared vs real minimum: read `Package-Requires`; byte-compile on that
   minimum Emacs and confirm it actually loads/compiles there (flag if it uses
   features above the declared minimum).
4. Cleanliness: byte-compile every real source file at the default-branch HEAD
   with dependencies on `load-path`; count genuine warnings (ignore
   environmental "cannot open load file" for missing optional deps).
5. Release hygiene: is there a tagged release? Does the version in the header
   match it?

## Output

A structured record per package: last activity, CI presence and what it tests,
warnings-as-errors yes/no, declared vs effective minimum Emacs, genuine warning
count, and a one-line health summary. This maps directly onto the coverage
table fields (CI, tested, warnings-as-errors, last checked).

## Notes

Read-only. It never pushes or opens a pull request. Re-running it over time
keeps the coverage table current and surfaces packages whose support has
lapsed.
