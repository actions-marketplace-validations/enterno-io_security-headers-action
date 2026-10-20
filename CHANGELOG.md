# Changelog

All notable changes to this action are listed here. Versions follow [Semantic Versioning](https://semver.org/);
the `v1` tag always points at the latest `v1.x.y` release.

## [1.0.0] — 2026-10-20

First public release.

- Grades a URL's HTTP security headers A+ to F through the keyless endpoint
  `https://enterno.io/api/securityheaders` (the response shape of the discontinued securityheaders.com API).
- Fails the job below `min-grade` (default `B`); grade `R` (redirect with `follow-redirects: false`) always fails.
- Outputs `grade`, `score`, `missing`, `passed` and `report` (path to the full JSON); writes a job summary.
- `fail-on-error: false` turns a scan that could not run (network error, rate limit) into a warning.
- Inputs are passed to the script through environment variables, never interpolated into shell.
- Response fields are validated before they become step outputs; the endpoint is called over HTTPS only.
- Requires `bash`, `curl` and `jq` (present on GitHub-hosted Ubuntu and macOS runners).

[1.0.0]: https://github.com/enterno-io/security-headers-action/releases/tag/v1.0.0
