# wp-healthcheck

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Skill: Anthropic Agent Skill](https://img.shields.io/badge/Anthropic-Agent_Skill-orange)](https://code.claude.com/docs/en/skills)

An [Anthropic Agent Skill](https://code.claude.com/docs/en/skills) that diagnoses WordPress
sites from publicly accessible information only — no credentials, no plugins, no server access.
Built by [npc](https://n-pc.jp).

## About

`wp-healthcheck` is a credential-less external probe for WordPress sites. Point it at any
WordPress URL and it produces a Markdown maintenance report graded on a four-tier scale
(**OK / WARNING / CRITICAL / UNKNOWN**). It is designed to be the first pass before any
deeper, internal audit.

The skill is implemented in pure Bash with `curl` and `jq` so it runs anywhere Claude Code
runs, without language runtimes or external dependencies beyond standard Unix tools.

## What it diagnoses

| # | Item | Method |
|---|---|---|
| 1 | WordPress version | `<meta name="generator">`, `/readme.html`, `/wp-json/` |
| 2 | Active theme | `/wp-content/themes/<slug>/` paths in HTML |
| 3 | SSL / TLS certificate | `curl -vI` + expiry calculation |
| 4 | Security headers | HSTS, CSP, X-Frame-Options, X-Content-Type-Options, Referrer-Policy |
| 5 | robots.txt and sitemap.xml | HTTP GET both well-known paths |
| 6 | Top-page response time | `curl -w '%{time_total}'` (single sample) |
| 7 | Public API surface | `/xmlrpc.php`, `/wp-json/`, `/wp-cron.php` reachability |
| 8 | Known vulnerable plugin fingerprints | Match `/wp-content/plugins/<slug>/` against a curated CVE list (front-end visible plugins only — see Limitations) |

Full judgement criteria are documented in [`references/checklist.md`](references/checklist.md).

## Installation

This skill follows the standard Anthropic Agent Skills layout. Install it by placing the
directory in one of the recognized locations:

### Personal scope (available in all projects)

```bash
mkdir -p ~/.claude/skills
cp -r wp-healthcheck-skill ~/.claude/skills/wp-healthcheck
```

### Project scope (available only in the current repo)

```bash
mkdir -p .claude/skills
cp -r wp-healthcheck-skill .claude/skills/wp-healthcheck
```

The skill is detected immediately — there is no need to restart Claude Code.

### Requirements

- `bash`, `curl`, `jq`, `awk`, `sed`, `grep`, `mktemp`, `date`
  (all present on macOS and most Linux distributions by default)
- Network access to the target site

## Usage examples

Once installed, ask Claude Code in natural language:

```text
Run a healthcheck on https://example.com
```

```text
Can you audit example.com? I think they're on an old WordPress.
```

```text
Use wp-healthcheck to scan https://example.com and write the report to ./report.md
```

You can also invoke the underlying script directly:

```bash
# Print to stdout
bash scripts/check_wp.sh https://example.com

# Write to a file
bash scripts/check_wp.sh https://example.com --output ./report.md
```

## Output sample

See [`examples/sample-report.md`](examples/sample-report.md) for a real run against
`https://n-pc.jp`. A condensed excerpt:

```markdown
# WordPress Healthcheck Report

- Target: https://n-pc.jp
- Overall: **Action required**
- OK: 5 / WARNING: 1 / CRITICAL: 1 / UNKNOWN: 1

| Item | Status | Summary |
|---|---|---|
| WordPress version       | OK       | WordPress 6.9.4 appears reasonably current.    |
| Active theme            | OK       | Active theme: npc-abc-theme                    |
| SSL / TLS certificate   | OK       | Certificate valid for 69 more days.            |
| Security headers        | CRITICAL | 0 of 5 recommended security headers present.   |
| robots.txt and sitemap  | OK       | Both robots.txt and sitemap are present.       |
| Top page response time  | OK       | Top page time_total: 0.36s                     |
| Public API surface      | OK       | xmlrpc / wp-json / wp-cron probed.             |
| Plugin fingerprints     | WARNING  | Plugins with historical CVEs detected: ...     |
```

## Limitations

This skill examines **only publicly accessible information**. It cannot:

- Determine the actual installed version of detected plugins.
- See plugins that are active only in the WordPress admin (SEO, cache, backup, security
  managers). Front-end-invisible plugins leave no fingerprint in public HTML, so the
  detected plugin count is a **lower bound**, not a complete inventory.
- Inspect the database, `wp-config.php`, file integrity, or admin-only state.
- Detect compromised content not exposed on the front page.
- Replace a WAF, vulnerability scanner, or full security audit.

For deeper checks that require server access, consider a dedicated maintenance plugin
such as [`npc-wp-healthcheck`](https://github.com/npc-jp/npc-wp-healthcheck).

## Roadmap

- **Phase 1 (this release)** — Standalone external probe. Bash + curl + jq only.
- **Phase 2** — Claude.ai upload package: same Skill, distributed as a single zip for
  drag-and-drop install on Claude.ai.
- **Phase 3** — Plugin slug matching backed by the WPScan vulnerability database for
  version-aware judgements (requires API key).
- **Phase 4** — JSON output mode for piping into other tooling.
- **Phase 5** — Hybrid integration with the `npc-wp-healthcheck` WordPress plugin: when
  the plugin is installed and reachable, escalate to authenticated internal checks.

## Contributing

Issues and pull requests are welcome.

When proposing a new diagnostic item:

1. Add the item to `references/checklist.md` with explicit OK / WARNING / CRITICAL / UNKNOWN
   criteria.
2. Implement detection in `scripts/check_wp.sh`.
3. Ensure the skill still completes in under 30 seconds against a typical WordPress site.
4. Update `examples/sample-report.md` and `README.md` if user-visible behavior changes.

## License

Apache License 2.0. See [`LICENSE`](LICENSE) for the full text.

## Credits

Created by [npc](https://n-pc.jp), a freelance Japanese WordPress maintenance studio.
Inspired by the Anthropic Agent Skills specification at
[agentskills.io/specification](https://agentskills.io/specification).
