# Security Headers Grade — GitHub Action

Grade a URL's HTTP security headers (A+ to F) and fail the job when the grade drops below a minimum.
No API key, no signup.

It calls a free endpoint that answers in the shape of the discontinued securityheaders.com API
(`https://enterno.io/api/securityheaders`), so the JSON keys and the grading scale are the ones you are used to.
Grades follow the published securityheaders.com point table but will not match the old service for every site;
the documentation page explains where and why.
Documentation, field mapping and the grading method: <https://enterno.io/en/securityheaders-api>.

## Usage

```yaml
name: Security headers

on:
  push:
    branches: [main]
  schedule:
    - cron: '17 6 * * 1'

permissions:
  contents: read

jobs:
  grade:
    runs-on: ubuntu-latest
    steps:
      - uses: enterno-io/security-headers-action@v1
        with:
          url: https://example.com
          min-grade: B
```

The job fails when the grade is below `min-grade`. The grade, the minimum and the missing headers are written to the
job summary.

## Inputs

| Input | Default | Meaning |
|---|---|---|
| `url` | — (required) | URL or host name to scan. A bare host is scanned over https. |
| `min-grade` | `B` | Lowest passing grade: `A+`, `A`, `B`, `C`, `D`, `E` or `F`. |
| `follow-redirects` | `true` | Follow up to 5 redirects and grade the final response. With `false` a redirect is grade `R`, which fails any minimum. |
| `fail-on-error` | `true` | Fail the job when the scan itself cannot run (site unreachable, rate limit). With `false` it only warns. |
| `api-url` | `https://enterno.io/api/securityheaders` | Endpoint to call. HTTPS only. |

## Outputs

| Output | Example |
|---|---|
| `grade` | `A` |
| `score` | `87` — percent of the 160 points behind the grade |
| `missing` | `Permissions-Policy` — comma-separated |
| `passed` | `true` / `false` |
| `report` | path to the full JSON response on the runner |

With `fail-on-error: false`, a scan that could not run leaves `grade`, `score` and `missing` empty and sets
`passed` to `true`, so check `grade` as well if you need to tell "passed" from "not scanned".

```yaml
      - uses: enterno-io/security-headers-action@v1
        id: headers
        with:
          url: https://example.com
          min-grade: A
          fail-on-error: false
      - env:
          GRADE: ${{ steps.headers.outputs.grade }}
          MISSING: ${{ steps.headers.outputs.missing }}
        run: echo "Grade $GRADE; missing $MISSING"
```

Pass outputs to `run:` through `env:` as above rather than writing `${{ … }}` inside the script; that is
GitHub's [recommended pattern](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#using-an-intermediate-environment-variable)
for any value that comes from outside the workflow.

## How the grade is calculated

The point table and bands securityheaders.com published in 2023: HTTPS 30, Strict-Transport-Security 25,
Content-Security-Policy 25, X-Frame-Options, X-Content-Type-Options, Referrer-Policy and Permissions-Policy 20 each
(160 in total). A+ from 95 %, A from 75 %, B from 60 %, C from 50 %, D from 29 %, E from 14 %, F below. A header a
browser would ignore earns nothing; a weak one (for example `'unsafe-inline'` scripts in CSP) caps the grade at A.
The full method is on the documentation page.

## Limits

The endpoint is free and keyless: 10 requests a minute and 300 a day per IP, each target host scanned at most
6 times a minute, results cached for 5 minutes. GitHub-hosted runners share IP ranges, so a burst of jobs can meet
the per-minute limit — schedule large matrices or set `fail-on-error: false`.

## Privacy

Each run sends the URL you configure to enterno.io, from the runner's IP address, with a User-Agent that names
this action. Scans are never listed publicly (`hide=on` is sent only for compatibility with the old API).
Results are cached for 5 minutes. The application keeps only the scheme and host of each scanned URL (30 days),
but like any web request the call also appears in web-server access logs with the full request URL.
Do not pass URLs that carry secrets (preview links with tokens, signed URLs) — scan the plain host instead.

## Requirements

`bash`, `curl` and `jq` — present on GitHub-hosted Ubuntu and macOS runners. Self-hosted runners need them installed.

## Versioning

`v1` is a moving tag that always points at the latest `v1.x.y` release; pin a full tag (`@v1.0.0`) or a commit
SHA if you want the code to never change under you. Changes are listed in [CHANGELOG.md](CHANGELOG.md).

## Issues

Wrong grade, false positive or a feature request: open an issue in this repository, or write to us at
<https://enterno.io/en/contact>.

## License

MIT — see [LICENSE](LICENSE).
