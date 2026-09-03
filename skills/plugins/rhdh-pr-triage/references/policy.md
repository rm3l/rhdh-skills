# Triage Decision Policy

Decision rules for Renovate and Dependabot PRs. Apply top-to-bottom within each
section; the first matching rule wins.

## Preconditions (apply before any section)

| Condition | Decision | Reason |
|-----------|----------|--------|
| PR is draft | skip | Draft PR |
| PR has conflicts, author is `renovate[bot]` | rebase | `@renovatebot rebase` |
| PR has conflicts, author is `dependabot[bot]` | recreate | `@dependabot recreate` |
| Same dependency from both bots | merge Renovate, close Dependabot | Renovate title includes `[security]`, preserving the reason in commit history |

## Main branch

| Condition | Decision | Reason |
|-----------|----------|--------|
| Major version bump | flag | Major bump — human review required |
| CI green | approve+merge | CI passing |
| CI pending | skip | Waiting for CI |
| CI failing | skip | CI failing |

## Release branch (release-x.y)

Look up the release state for the corresponding RHDH minor version.

| Condition | Decision | Reason |
|-----------|----------|--------|
| RHDH x.y is EOL | close | Closing — RHDH x.y is end of life |
| Operator: `go` directive bumped in go.mod | close | Go directive frozen on release branches |
| Operator: `toolchain` non-patch bump | close | Only toolchain patch bumps on release branches |
| At CF or FF | hold | /hold until x.y.z is out |
| Past GA announce but tag not yet pushed | hold | /hold if not already held — still waiting on x.y.z tag |
| Already held (`do-not-merge/hold`), no state change since last triage | skip | Still waiting on x.y.z (no action needed, already held) |
| GA'd — git tag exists AND release notes confirm z-stream | unhold+approve+merge | x.y.z released |

### Verifying GA

To confirm a z-stream has GA'd, check both:

1. `git ls-remote --tags` against the repo for the expected tag (e.g. `v1.10.5`)
2. Release notes at
   `https://docs.redhat.com/en/documentation/red_hat_developer_hub/{major.minor}/html/red_hat_developer_hub_release_notes/fixed-issues`
   for a section covering that z-stream

Both must confirm before unholding.

## Chart repo special rules

These apply in addition to the main/release branch rules above.

| Condition | Decision | Reason |
|-----------|----------|--------|
| Lint CI fails, dep is a chart dependency | bump | `/bump <chart> <minor\|patch>` per chart folder |
| Multiple chart folders modified | bump (one per chart) | One `/bump` comment per affected chart |
| GH Actions update (not a chart dep) | normal rules | No chart bump needed |

The bump level matches the semver level of the dependency version change:
minor dep bump gets `/bump <chart> minor`, patch gets `/bump <chart> patch`.

## CI status rules

All required checks must be green. Skip `tide` when evaluating CI status on
repos that use Prow — `tide` is a merge gate, not a quality signal.
