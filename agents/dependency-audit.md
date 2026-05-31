# Agent: dependency audit

A read-only agent that audits one Emacs package's dependency closure for
supply-chain and maintenance risk. It makes no code changes and opens no pull
request; its output is a report.

## Goal

Surface what a package pulls in and where the risk is, so dependence on it can
be judged without auditing every transitive dependency by hand. Complementary
to MELPA, which does not express this.

## Inputs

- A package name and its upstream repository URL.

## Checks

1. Closure: resolve the full transitive dependency set from `Package-Requires`
   (excluding what the target Emacs bundles), and the host of each (GNU ELPA,
   NonGNU ELPA, MELPA, a git forge).
2. Maintenance: for each dependency, last commit date, archived status, single
   vs multiple maintainers (bus factor).
3. Known issues: check public advisory sources (GitHub Security Advisories,
   OSV, CVE) for each dependency name; record any matches with severity.
4. Surface: flag dependencies that talk to the network, run native code, load
   or deserialize untrusted data, or evaluate downloaded code.
5. Pinning: note whether the package pins its dependencies or floats them.

## Output

A structured report: the dependency list with host, last activity, maintainer
count, any advisories, and a risk note per dependency, plus an overall summary
(for example, "all dependencies actively maintained; no known advisories").

## Notes

Read-only. Findings are informational; any remediation (pinning, replacing, or
vendoring a dependency) is a separate, deliberate decision. Cite the advisory
source for every reported issue.
